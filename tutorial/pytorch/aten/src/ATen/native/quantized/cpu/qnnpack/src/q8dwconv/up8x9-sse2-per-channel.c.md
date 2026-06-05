# up8x9-sse2-per-channel.c — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `aten/src/ATen/native/quantized/cpu/qnnpack/src/q8dwconv/up8x9-sse2-per-channel.c`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements ATen native operators, checks, helper kernels, and backend dispatch behavior. This specific file centers on `up8x9-sse2-per-channel.c`. The file header highlights: "All rights reserved. This source code is licensed under the BSD-style license found in the LICENSE file in the root directory of this source tree.."
- **Purpose (CN)**: 实现 ATen 原生算子、运行时检查、辅助内核以及后端分发行为。 该文件具体围绕 `up8x9-sse2-per-channel.c` 展开。 文件头部注释也概括了其核心职责。

## Line-by-Line Analysis / 逐行分析
### Lines 1-27 / 第 1-27 行

```c
0001: /*
0002:  * Copyright (c) Facebook, Inc. and its affiliates.
0003:  * All rights reserved.
0004:  *
0005:  * This source code is licensed under the BSD-style license found in the
0006:  * LICENSE file in the root directory of this source tree.
0007:  */
0008: 
0009: #include <immintrin.h>
0010: 
0011: #include <qnnpack/q8dwconv.h>
0012: #include <requantization/runtime-sse2.h>
0013: 
0014: void pytorch_q8dwconv_ukernel_up8x9_per_channel__sse2(
0015:     size_t channels,
0016:     size_t output_width,
0017:     const uint8_t** input,
0018:     const void* weights,
0019:     uint8_t* output,
0020:     size_t input_stride,
0021:     size_t output_increment,
0022:     const union pytorch_qnnp_conv_quantization_params
0023:         quantization_params[RESTRICT_STATIC 1]) {
0024:   const __m128i va_zero_point = _mm_load_si128(
0025:       (const __m128i*)quantization_params->sse2.input_zero_point);
0026:   const __m128i vzero = _mm_setzero_si128();
0027: 
```

- **EN:** This block implements local helper logic for `up8x9-sse2-per-channel`. Key symbols: `pytorch_q8dwconv_ukernel_up8x9_per_channel__sse2`.
- **CN:** 该代码块实现与 `up8x9-sse2-per-channel` 相关的局部辅助逻辑。关键符号：`pytorch_q8dwconv_ukernel_up8x9_per_channel__sse2`。

### Lines 28-49 / 第 28-49 行

```c
0028:   do {
0029:     const uint8_t* i0 = input[0];
0030:     const uint8_t* i1 = input[1];
0031:     const uint8_t* i2 = input[2];
0032:     const uint8_t* i3 = input[3];
0033:     const uint8_t* i4 = input[4];
0034:     const uint8_t* i5 = input[5];
0035:     const uint8_t* i6 = input[6];
0036:     const uint8_t* i7 = input[7];
0037:     const uint8_t* i8 = input[8];
0038: 
0039:     input = (const uint8_t**)((uintptr_t)input + input_stride);
0040: 
0041:     size_t c = channels;
0042:     const void* w = weights;
0043:     for (; c >= 8; c -= 8) {
0044:       __m128i vacc_lo = _mm_loadu_si128((const __m128i*)w);
0045:       __m128i vacc_hi = _mm_loadu_si128((const __m128i*)((uintptr_t)w + 16));
0046:       const __m128i vkernel_zero_point = _mm_loadl_epi64(
0047:           (const __m128i*)
0048:           &quantization_params->sse2.kernel_zero_points[channels - c]);
0049: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: no prominent local symbols.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：无明显局部符号。

### Lines 50-81 / 第 50-81 行

```c
0050:       const __m128i vi0 = _mm_loadl_epi64((const __m128i*)i0);
0051:       i0 += 8;
0052:       const __m128i vxi0 =
0053:           sub_zero_point(_mm_unpacklo_epi8(vi0, vzero), va_zero_point);
0054:       const __m128i vk0 = _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 32));
0055:       const __m128i vxk0 =
0056:           _mm_sub_epi16(
0057:               _mm_unpacklo_epi8(vk0, vzero),
0058:               _mm_unpacklo_epi8(vkernel_zero_point, vzero));
0059:       const __m128i vprod0_odd = _mm_mullo_epi16(vxi0, vxk0);
0060:       const __m128i vprod0_even = _mm_mulhi_epi16(vxi0, vxk0);
0061:       vacc_lo =
0062:           _mm_add_epi32(vacc_lo, _mm_unpacklo_epi16(vprod0_odd, vprod0_even));
0063:       vacc_hi =
0064:           _mm_add_epi32(vacc_hi, _mm_unpackhi_epi16(vprod0_odd, vprod0_even));
0065: 
0066:       const __m128i vi1 = _mm_loadl_epi64((const __m128i*)i1);
0067:       i1 += 8;
0068:       const __m128i vxi1 =
0069:           sub_zero_point(_mm_unpacklo_epi8(vi1, vzero), va_zero_point);
0070:       const __m128i vk1 = _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 40));
0071:       const __m128i vxk1 =
0072:           _mm_sub_epi16(
0073:               _mm_unpacklo_epi8(vk1, vzero),
0074:               _mm_unpacklo_epi8(vkernel_zero_point, vzero));
0075:       const __m128i vprod1_odd = _mm_mullo_epi16(vxi1, vxk1);
0076:       const __m128i vprod1_even = _mm_mulhi_epi16(vxi1, vxk1);
0077:       vacc_lo =
0078:           _mm_add_epi32(vacc_lo, _mm_unpacklo_epi16(vprod1_odd, vprod1_even));
0079:       vacc_hi =
0080:           _mm_add_epi32(vacc_hi, _mm_unpackhi_epi16(vprod1_odd, vprod1_even));
0081: 
```

- **EN:** This block implements local helper logic for `up8x9-sse2-per-channel`. Key symbols: `sub_zero_point`, `_mm_sub_epi16`, `_mm_add_epi32`.
- **CN:** 该代码块实现与 `up8x9-sse2-per-channel` 相关的局部辅助逻辑。关键符号：`sub_zero_point`, `_mm_sub_epi16`, `_mm_add_epi32`。

### Lines 82-113 / 第 82-113 行

```c
0082:       const __m128i vi2 = _mm_loadl_epi64((const __m128i*)i2);
0083:       i2 += 8;
0084:       const __m128i vxi2 =
0085:           sub_zero_point(_mm_unpacklo_epi8(vi2, vzero), va_zero_point);
0086:       const __m128i vk2 = _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 48));
0087:       const __m128i vxk2 =
0088:           _mm_sub_epi16(
0089:               _mm_unpacklo_epi8(vk2, vzero),
0090:               _mm_unpacklo_epi8(vkernel_zero_point, vzero));
0091:       const __m128i vprod2_odd = _mm_mullo_epi16(vxi2, vxk2);
0092:       const __m128i vprod2_even = _mm_mulhi_epi16(vxi2, vxk2);
0093:       vacc_lo =
0094:           _mm_add_epi32(vacc_lo, _mm_unpacklo_epi16(vprod2_odd, vprod2_even));
0095:       vacc_hi =
0096:           _mm_add_epi32(vacc_hi, _mm_unpackhi_epi16(vprod2_odd, vprod2_even));
0097: 
0098:       const __m128i vi3 = _mm_loadl_epi64((const __m128i*)i3);
0099:       i3 += 8;
0100:       const __m128i vxi3 =
0101:           sub_zero_point(_mm_unpacklo_epi8(vi3, vzero), va_zero_point);
0102:       const __m128i vk3 = _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 56));
0103:       const __m128i vxk3 =
0104:           _mm_sub_epi16(
0105:               _mm_unpacklo_epi8(vk3, vzero),
0106:               _mm_unpacklo_epi8(vkernel_zero_point, vzero));
0107:       const __m128i vprod3_odd = _mm_mullo_epi16(vxi3, vxk3);
0108:       const __m128i vprod3_even = _mm_mulhi_epi16(vxi3, vxk3);
0109:       vacc_lo =
0110:           _mm_add_epi32(vacc_lo, _mm_unpacklo_epi16(vprod3_odd, vprod3_even));
0111:       vacc_hi =
0112:           _mm_add_epi32(vacc_hi, _mm_unpackhi_epi16(vprod3_odd, vprod3_even));
0113: 
```

- **EN:** This block implements local helper logic for `up8x9-sse2-per-channel`. Key symbols: `sub_zero_point`, `_mm_sub_epi16`, `_mm_add_epi32`.
- **CN:** 该代码块实现与 `up8x9-sse2-per-channel` 相关的局部辅助逻辑。关键符号：`sub_zero_point`, `_mm_sub_epi16`, `_mm_add_epi32`。

### Lines 114-145 / 第 114-145 行

```c
0114:       const __m128i vi4 = _mm_loadl_epi64((const __m128i*)i4);
0115:       i4 += 8;
0116:       const __m128i vxi4 =
0117:           sub_zero_point(_mm_unpacklo_epi8(vi4, vzero), va_zero_point);
0118:       const __m128i vk4 = _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 64));
0119:       const __m128i vxk4 =
0120:           _mm_sub_epi16(
0121:               _mm_unpacklo_epi8(vk4, vzero),
0122:               _mm_unpacklo_epi8(vkernel_zero_point, vzero));
0123:       const __m128i vprod4_odd = _mm_mullo_epi16(vxi4, vxk4);
0124:       const __m128i vprod4_even = _mm_mulhi_epi16(vxi4, vxk4);
0125:       vacc_lo =
0126:           _mm_add_epi32(vacc_lo, _mm_unpacklo_epi16(vprod4_odd, vprod4_even));
0127:       vacc_hi =
0128:           _mm_add_epi32(vacc_hi, _mm_unpackhi_epi16(vprod4_odd, vprod4_even));
0129: 
0130:       const __m128i vi5 = _mm_loadl_epi64((const __m128i*)i5);
0131:       i5 += 8;
0132:       const __m128i vxi5 =
0133:           sub_zero_point(_mm_unpacklo_epi8(vi5, vzero), va_zero_point);
0134:       const __m128i vk5 = _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 72));
0135:       const __m128i vxk5 =
0136:           _mm_sub_epi16(
0137:               _mm_unpacklo_epi8(vk5, vzero),
0138:               _mm_unpacklo_epi8(vkernel_zero_point, vzero));
0139:       const __m128i vprod5_odd = _mm_mullo_epi16(vxi5, vxk5);
0140:       const __m128i vprod5_even = _mm_mulhi_epi16(vxi5, vxk5);
0141:       vacc_lo =
0142:           _mm_add_epi32(vacc_lo, _mm_unpacklo_epi16(vprod5_odd, vprod5_even));
0143:       vacc_hi =
0144:           _mm_add_epi32(vacc_hi, _mm_unpackhi_epi16(vprod5_odd, vprod5_even));
0145: 
```

- **EN:** This block implements local helper logic for `up8x9-sse2-per-channel`. Key symbols: `sub_zero_point`, `_mm_sub_epi16`, `_mm_add_epi32`.
- **CN:** 该代码块实现与 `up8x9-sse2-per-channel` 相关的局部辅助逻辑。关键符号：`sub_zero_point`, `_mm_sub_epi16`, `_mm_add_epi32`。

### Lines 146-177 / 第 146-177 行

```c
0146:       const __m128i vi6 = _mm_loadl_epi64((const __m128i*)i6);
0147:       i6 += 8;
0148:       const __m128i vxi6 =
0149:           sub_zero_point(_mm_unpacklo_epi8(vi6, vzero), va_zero_point);
0150:       const __m128i vk6 = _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 80));
0151:       const __m128i vxk6 =
0152:           _mm_sub_epi16(
0153:               _mm_unpacklo_epi8(vk6, vzero),
0154:               _mm_unpacklo_epi8(vkernel_zero_point, vzero));
0155:       const __m128i vprod6_odd = _mm_mullo_epi16(vxi6, vxk6);
0156:       const __m128i vprod6_even = _mm_mulhi_epi16(vxi6, vxk6);
0157:       vacc_lo =
0158:           _mm_add_epi32(vacc_lo, _mm_unpacklo_epi16(vprod6_odd, vprod6_even));
0159:       vacc_hi =
0160:           _mm_add_epi32(vacc_hi, _mm_unpackhi_epi16(vprod6_odd, vprod6_even));
0161: 
0162:       const __m128i vi7 = _mm_loadl_epi64((const __m128i*)i7);
0163:       i7 += 8;
0164:       const __m128i vxi7 =
0165:           sub_zero_point(_mm_unpacklo_epi8(vi7, vzero), va_zero_point);
0166:       const __m128i vk7 = _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 88));
0167:       const __m128i vxk7 =
0168:           _mm_sub_epi16(
0169:               _mm_unpacklo_epi8(vk7, vzero),
0170:               _mm_unpacklo_epi8(vkernel_zero_point, vzero));
0171:       const __m128i vprod7_odd = _mm_mullo_epi16(vxi7, vxk7);
0172:       const __m128i vprod7_even = _mm_mulhi_epi16(vxi7, vxk7);
0173:       vacc_lo =
0174:           _mm_add_epi32(vacc_lo, _mm_unpacklo_epi16(vprod7_odd, vprod7_even));
0175:       vacc_hi =
0176:           _mm_add_epi32(vacc_hi, _mm_unpackhi_epi16(vprod7_odd, vprod7_even));
0177: 
```

- **EN:** This block implements local helper logic for `up8x9-sse2-per-channel`. Key symbols: `sub_zero_point`, `_mm_sub_epi16`, `_mm_add_epi32`.
- **CN:** 该代码块实现与 `up8x9-sse2-per-channel` 相关的局部辅助逻辑。关键符号：`sub_zero_point`, `_mm_sub_epi16`, `_mm_add_epi32`。

### Lines 178-200 / 第 178-200 行

```c
0178:       const __m128i vi8 = _mm_loadl_epi64((const __m128i*)i8);
0179:       i8 += 8;
0180:       const __m128i vxi8 =
0181:           sub_zero_point(_mm_unpacklo_epi8(vi8, vzero), va_zero_point);
0182:       const __m128i vk8 = _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 96));
0183:       const __m128i vxk8 =
0184:           _mm_sub_epi16(
0185:               _mm_unpacklo_epi8(vk8, vzero),
0186:               _mm_unpacklo_epi8(vkernel_zero_point, vzero));
0187:       const __m128i vprod8_odd = _mm_mullo_epi16(vxi8, vxk8);
0188:       const __m128i vprod8_even = _mm_mulhi_epi16(vxi8, vxk8);
0189:       vacc_lo =
0190:           _mm_add_epi32(vacc_lo, _mm_unpacklo_epi16(vprod8_odd, vprod8_even));
0191:       vacc_hi =
0192:           _mm_add_epi32(vacc_hi, _mm_unpackhi_epi16(vprod8_odd, vprod8_even));
0193: 
0194:       w = (void*)((uintptr_t)w + 104);
0195: 
0196:       const __m128 vmultiplier_lo =
0197:           _mm_loadu_ps(&quantization_params->sse2.requantization_scales[channels - c]);
0198:       const __m128 vmultiplier_hi =
0199:           _mm_loadu_ps(&quantization_params->sse2.requantization_scales[channels - c + 4]);
0200: 
```

- **EN:** This block implements local helper logic for `up8x9-sse2-per-channel`. Key symbols: `sub_zero_point`, `_mm_sub_epi16`, `_mm_add_epi32`, `_mm_loadu_ps`.
- **CN:** 该代码块实现与 `up8x9-sse2-per-channel` 相关的局部辅助逻辑。关键符号：`sub_zero_point`, `_mm_sub_epi16`, `_mm_add_epi32`, `_mm_loadu_ps`。

### Lines 201-225 / 第 201-225 行

```c
0201:       vacc_lo = _mm_cvtps_epi32(
0202:                     _mm_mul_ps(
0203:                       _mm_cvtepi32_ps(vacc_lo),
0204:                       vmultiplier_lo
0205:                       )
0206:                     );
0207:       vacc_hi = _mm_cvtps_epi32(
0208:                     _mm_mul_ps(
0209:                       _mm_cvtepi32_ps(vacc_hi),
0210:                       vmultiplier_hi
0211:                       )
0212:                     );
0213: 
0214:       const __m128i voutput_zero_point = _mm_load_si128(
0215:           (const __m128i*)quantization_params->sse2.output_zero_point);
0216:       __m128i vout =
0217:           _mm_adds_epi16(_mm_packs_epi32(vacc_lo, vacc_hi), voutput_zero_point);
0218:       vout = _mm_packus_epi16(vout, vout);
0219:       vout = _mm_min_epu8(
0220:           vout,
0221:           _mm_load_si128((const __m128i*)quantization_params->sse2.output_max));
0222:       vout = _mm_max_epu8(
0223:           vout,
0224:           _mm_load_si128((const __m128i*)quantization_params->sse2.output_min));
0225: 
```

- **EN:** This block implements local helper logic for `up8x9-sse2-per-channel`. Key symbols: `_mm_mul_ps`, `_mm_adds_epi16`, `_mm_load_si128`.
- **CN:** 该代码块实现与 `up8x9-sse2-per-channel` 相关的局部辅助逻辑。关键符号：`_mm_mul_ps`, `_mm_adds_epi16`, `_mm_load_si128`。

### Lines 226-247 / 第 226-247 行

```c
0226:       _mm_storel_epi64((__m128i*)output, vout);
0227:       output += 8;
0228:     }
0229:     if (c != 0) {
0230:       const size_t i_predecrement = 8 - c;
0231:       const __m128i vi_shift = _mm_cvtsi32_si128(8 * i_predecrement);
0232:       const __m128i vkernel_zero_point = _mm_loadl_epi64(
0233:           (const __m128i*)
0234:           &quantization_params->sse2.kernel_zero_points[channels - c]);
0235:       i0 -= i_predecrement;
0236:       i1 -= i_predecrement;
0237:       i2 -= i_predecrement;
0238:       i3 -= i_predecrement;
0239:       i4 -= i_predecrement;
0240:       i5 -= i_predecrement;
0241:       i6 -= i_predecrement;
0242:       i7 -= i_predecrement;
0243:       i8 -= i_predecrement;
0244: 
0245:       __m128i vacc_lo = _mm_loadu_si128((const __m128i*)w);
0246:       __m128i vacc_hi = _mm_loadu_si128((const __m128i*)((uintptr_t)w + 16));
0247: 
```

- **EN:** This block handles conditional branches and special cases. Key symbols: `_mm_storel_epi64`.
- **CN:** 该代码块处理条件分支与特殊情况。关键符号：`_mm_storel_epi64`。

### Lines 248-279 / 第 248-279 行

```c
0248:       const __m128i vi0 =
0249:           _mm_srl_epi64(_mm_loadl_epi64((const __m128i*)i0), vi_shift);
0250:       const __m128i vxi0 =
0251:           sub_zero_point(_mm_unpacklo_epi8(vi0, vzero), va_zero_point);
0252:       const __m128i vk0 = _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 32));
0253:       const __m128i vxk0 =
0254:           _mm_sub_epi16(
0255:               _mm_unpacklo_epi8(vk0, vzero),
0256:               _mm_unpacklo_epi8(vkernel_zero_point, vzero));
0257:       const __m128i vprod0_odd = _mm_mullo_epi16(vxi0, vxk0);
0258:       const __m128i vprod0_even = _mm_mulhi_epi16(vxi0, vxk0);
0259:       vacc_lo =
0260:           _mm_add_epi32(vacc_lo, _mm_unpacklo_epi16(vprod0_odd, vprod0_even));
0261:       vacc_hi =
0262:           _mm_add_epi32(vacc_hi, _mm_unpackhi_epi16(vprod0_odd, vprod0_even));
0263: 
0264:       const __m128i vi1 =
0265:           _mm_srl_epi64(_mm_loadl_epi64((const __m128i*)i1), vi_shift);
0266:       const __m128i vxi1 =
0267:           sub_zero_point(_mm_unpacklo_epi8(vi1, vzero), va_zero_point);
0268:       const __m128i vk1 = _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 40));
0269:       const __m128i vxk1 =
0270:           _mm_sub_epi16(
0271:               _mm_unpacklo_epi8(vk1, vzero),
0272:               _mm_unpacklo_epi8(vkernel_zero_point, vzero));
0273:       const __m128i vprod1_odd = _mm_mullo_epi16(vxi1, vxk1);
0274:       const __m128i vprod1_even = _mm_mulhi_epi16(vxi1, vxk1);
0275:       vacc_lo =
0276:           _mm_add_epi32(vacc_lo, _mm_unpacklo_epi16(vprod1_odd, vprod1_even));
0277:       vacc_hi =
0278:           _mm_add_epi32(vacc_hi, _mm_unpackhi_epi16(vprod1_odd, vprod1_even));
0279: 
```

- **EN:** This block implements local helper logic for `up8x9-sse2-per-channel`. Key symbols: `_mm_srl_epi64`, `sub_zero_point`, `_mm_sub_epi16`, `_mm_add_epi32`.
- **CN:** 该代码块实现与 `up8x9-sse2-per-channel` 相关的局部辅助逻辑。关键符号：`_mm_srl_epi64`, `sub_zero_point`, `_mm_sub_epi16`, `_mm_add_epi32`。

### Lines 280-311 / 第 280-311 行

```c
0280:       const __m128i vi2 =
0281:           _mm_srl_epi64(_mm_loadl_epi64((const __m128i*)i2), vi_shift);
0282:       const __m128i vxi2 =
0283:           sub_zero_point(_mm_unpacklo_epi8(vi2, vzero), va_zero_point);
0284:       const __m128i vk2 = _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 48));
0285:       const __m128i vxk2 =
0286:           _mm_sub_epi16(
0287:               _mm_unpacklo_epi8(vk2, vzero),
0288:               _mm_unpacklo_epi8(vkernel_zero_point, vzero));
0289:       const __m128i vprod2_odd = _mm_mullo_epi16(vxi2, vxk2);
0290:       const __m128i vprod2_even = _mm_mulhi_epi16(vxi2, vxk2);
0291:       vacc_lo =
0292:           _mm_add_epi32(vacc_lo, _mm_unpacklo_epi16(vprod2_odd, vprod2_even));
0293:       vacc_hi =
0294:           _mm_add_epi32(vacc_hi, _mm_unpackhi_epi16(vprod2_odd, vprod2_even));
0295: 
0296:       const __m128i vi3 =
0297:           _mm_srl_epi64(_mm_loadl_epi64((const __m128i*)i3), vi_shift);
0298:       const __m128i vxi3 =
0299:           sub_zero_point(_mm_unpacklo_epi8(vi3, vzero), va_zero_point);
0300:       const __m128i vk3 = _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 56));
0301:       const __m128i vxk3 =
0302:           _mm_sub_epi16(
0303:               _mm_unpacklo_epi8(vk3, vzero),
0304:               _mm_unpacklo_epi8(vkernel_zero_point, vzero));
0305:       const __m128i vprod3_odd = _mm_mullo_epi16(vxi3, vxk3);
0306:       const __m128i vprod3_even = _mm_mulhi_epi16(vxi3, vxk3);
0307:       vacc_lo =
0308:           _mm_add_epi32(vacc_lo, _mm_unpacklo_epi16(vprod3_odd, vprod3_even));
0309:       vacc_hi =
0310:           _mm_add_epi32(vacc_hi, _mm_unpackhi_epi16(vprod3_odd, vprod3_even));
0311: 
```

- **EN:** This block implements local helper logic for `up8x9-sse2-per-channel`. Key symbols: `_mm_srl_epi64`, `sub_zero_point`, `_mm_sub_epi16`, `_mm_add_epi32`.
- **CN:** 该代码块实现与 `up8x9-sse2-per-channel` 相关的局部辅助逻辑。关键符号：`_mm_srl_epi64`, `sub_zero_point`, `_mm_sub_epi16`, `_mm_add_epi32`。

### Lines 312-343 / 第 312-343 行

```c
0312:       const __m128i vi4 =
0313:           _mm_srl_epi64(_mm_loadl_epi64((const __m128i*)i4), vi_shift);
0314:       const __m128i vxi4 =
0315:           sub_zero_point(_mm_unpacklo_epi8(vi4, vzero), va_zero_point);
0316:       const __m128i vk4 = _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 64));
0317:       const __m128i vxk4 =
0318:           _mm_sub_epi16(
0319:               _mm_unpacklo_epi8(vk4, vzero),
0320:               _mm_unpacklo_epi8(vkernel_zero_point, vzero));
0321:       const __m128i vprod4_odd = _mm_mullo_epi16(vxi4, vxk4);
0322:       const __m128i vprod4_even = _mm_mulhi_epi16(vxi4, vxk4);
0323:       vacc_lo =
0324:           _mm_add_epi32(vacc_lo, _mm_unpacklo_epi16(vprod4_odd, vprod4_even));
0325:       vacc_hi =
0326:           _mm_add_epi32(vacc_hi, _mm_unpackhi_epi16(vprod4_odd, vprod4_even));
0327: 
0328:       const __m128i vi5 =
0329:           _mm_srl_epi64(_mm_loadl_epi64((const __m128i*)i5), vi_shift);
0330:       const __m128i vxi5 =
0331:           sub_zero_point(_mm_unpacklo_epi8(vi5, vzero), va_zero_point);
0332:       const __m128i vk5 = _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 72));
0333:       const __m128i vxk5 =
0334:           _mm_sub_epi16(
0335:               _mm_unpacklo_epi8(vk5, vzero),
0336:               _mm_unpacklo_epi8(vkernel_zero_point, vzero));
0337:       const __m128i vprod5_odd = _mm_mullo_epi16(vxi5, vxk5);
0338:       const __m128i vprod5_even = _mm_mulhi_epi16(vxi5, vxk5);
0339:       vacc_lo =
0340:           _mm_add_epi32(vacc_lo, _mm_unpacklo_epi16(vprod5_odd, vprod5_even));
0341:       vacc_hi =
0342:           _mm_add_epi32(vacc_hi, _mm_unpackhi_epi16(vprod5_odd, vprod5_even));
0343: 
```

- **EN:** This block implements local helper logic for `up8x9-sse2-per-channel`. Key symbols: `_mm_srl_epi64`, `sub_zero_point`, `_mm_sub_epi16`, `_mm_add_epi32`.
- **CN:** 该代码块实现与 `up8x9-sse2-per-channel` 相关的局部辅助逻辑。关键符号：`_mm_srl_epi64`, `sub_zero_point`, `_mm_sub_epi16`, `_mm_add_epi32`。

### Lines 344-375 / 第 344-375 行

```c
0344:       const __m128i vi6 =
0345:           _mm_srl_epi64(_mm_loadl_epi64((const __m128i*)i6), vi_shift);
0346:       const __m128i vxi6 =
0347:           sub_zero_point(_mm_unpacklo_epi8(vi6, vzero), va_zero_point);
0348:       const __m128i vk6 = _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 80));
0349:       const __m128i vxk6 =
0350:           _mm_sub_epi16(
0351:               _mm_unpacklo_epi8(vk6, vzero),
0352:               _mm_unpacklo_epi8(vkernel_zero_point, vzero));
0353:       const __m128i vprod6_odd = _mm_mullo_epi16(vxi6, vxk6);
0354:       const __m128i vprod6_even = _mm_mulhi_epi16(vxi6, vxk6);
0355:       vacc_lo =
0356:           _mm_add_epi32(vacc_lo, _mm_unpacklo_epi16(vprod6_odd, vprod6_even));
0357:       vacc_hi =
0358:           _mm_add_epi32(vacc_hi, _mm_unpackhi_epi16(vprod6_odd, vprod6_even));
0359: 
0360:       const __m128i vi7 =
0361:           _mm_srl_epi64(_mm_loadl_epi64((const __m128i*)i7), vi_shift);
0362:       const __m128i vxi7 =
0363:           sub_zero_point(_mm_unpacklo_epi8(vi7, vzero), va_zero_point);
0364:       const __m128i vk7 = _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 88));
0365:       const __m128i vxk7 =
0366:           _mm_sub_epi16(
0367:               _mm_unpacklo_epi8(vk7, vzero),
0368:               _mm_unpacklo_epi8(vkernel_zero_point, vzero));
0369:       const __m128i vprod7_odd = _mm_mullo_epi16(vxi7, vxk7);
0370:       const __m128i vprod7_even = _mm_mulhi_epi16(vxi7, vxk7);
0371:       vacc_lo =
0372:           _mm_add_epi32(vacc_lo, _mm_unpacklo_epi16(vprod7_odd, vprod7_even));
0373:       vacc_hi =
0374:           _mm_add_epi32(vacc_hi, _mm_unpackhi_epi16(vprod7_odd, vprod7_even));
0375: 
```

- **EN:** This block implements local helper logic for `up8x9-sse2-per-channel`. Key symbols: `_mm_srl_epi64`, `sub_zero_point`, `_mm_sub_epi16`, `_mm_add_epi32`.
- **CN:** 该代码块实现与 `up8x9-sse2-per-channel` 相关的局部辅助逻辑。关键符号：`_mm_srl_epi64`, `sub_zero_point`, `_mm_sub_epi16`, `_mm_add_epi32`。

### Lines 376-396 / 第 376-396 行

```c
0376:       const __m128i vi8 =
0377:           _mm_srl_epi64(_mm_loadl_epi64((const __m128i*)i8), vi_shift);
0378:       const __m128i vxi8 =
0379:           sub_zero_point(_mm_unpacklo_epi8(vi8, vzero), va_zero_point);
0380:       const __m128i vk8 = _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 96));
0381:       const __m128i vxk8 =
0382:           _mm_sub_epi16(
0383:               _mm_unpacklo_epi8(vk8, vzero),
0384:               _mm_unpacklo_epi8(vkernel_zero_point, vzero));
0385:       const __m128i vprod8_odd = _mm_mullo_epi16(vxi8, vxk8);
0386:       const __m128i vprod8_even = _mm_mulhi_epi16(vxi8, vxk8);
0387:       vacc_lo =
0388:           _mm_add_epi32(vacc_lo, _mm_unpacklo_epi16(vprod8_odd, vprod8_even));
0389:       vacc_hi =
0390:           _mm_add_epi32(vacc_hi, _mm_unpackhi_epi16(vprod8_odd, vprod8_even));
0391: 
0392:       const __m128 vmultiplier_lo =
0393:           _mm_loadu_ps(&quantization_params->sse2.requantization_scales[channels - c]);
0394:       const __m128 vmultiplier_hi =
0395:           _mm_loadu_ps(&quantization_params->sse2.requantization_scales[channels - c + 4]);
0396: 
```

- **EN:** This block implements local helper logic for `up8x9-sse2-per-channel`. Key symbols: `_mm_srl_epi64`, `sub_zero_point`, `_mm_sub_epi16`, `_mm_add_epi32`, `_mm_loadu_ps`.
- **CN:** 该代码块实现与 `up8x9-sse2-per-channel` 相关的局部辅助逻辑。关键符号：`_mm_srl_epi64`, `sub_zero_point`, `_mm_sub_epi16`, `_mm_add_epi32`, `_mm_loadu_ps`。

### Lines 397-421 / 第 397-421 行

```c
0397:       vacc_lo = _mm_cvtps_epi32(
0398:                     _mm_mul_ps(
0399:                       _mm_cvtepi32_ps(vacc_lo),
0400:                       vmultiplier_lo
0401:                       )
0402:                     );
0403:       vacc_hi = _mm_cvtps_epi32(
0404:                     _mm_mul_ps(
0405:                       _mm_cvtepi32_ps(vacc_hi),
0406:                       vmultiplier_hi
0407:                       )
0408:                     );
0409: 
0410:       const __m128i voutput_zero_point = _mm_load_si128(
0411:           (const __m128i*)quantization_params->sse2.output_zero_point);
0412:       __m128i vout =
0413:           _mm_adds_epi16(_mm_packs_epi32(vacc_lo, vacc_hi), voutput_zero_point);
0414:       vout = _mm_packus_epi16(vout, vout);
0415:       vout = _mm_min_epu8(
0416:           vout,
0417:           _mm_load_si128((const __m128i*)quantization_params->sse2.output_max));
0418:       vout = _mm_max_epu8(
0419:           vout,
0420:           _mm_load_si128((const __m128i*)quantization_params->sse2.output_min));
0421: 
```

- **EN:** This block implements local helper logic for `up8x9-sse2-per-channel`. Key symbols: `_mm_mul_ps`, `_mm_adds_epi16`, `_mm_load_si128`.
- **CN:** 该代码块实现与 `up8x9-sse2-per-channel` 相关的局部辅助逻辑。关键符号：`_mm_mul_ps`, `_mm_adds_epi16`, `_mm_load_si128`。

### Lines 422-440 / 第 422-440 行

```c
0422:       if (c & 4) {
0423:         *((uint32_t*)output) = (uint32_t)_mm_cvtsi128_si32(vout);
0424:         output += 4;
0425:         vout = _mm_srli_epi64(vout, 32);
0426:       }
0427:       if (c & 2) {
0428:         *((uint16_t*)output) = (uint16_t)_mm_extract_epi16(vout, 0);
0429:         output += 2;
0430:         vout = _mm_srli_epi32(vout, 16);
0431:       }
0432:       if (c & 1) {
0433:         *((uint8_t*)output) = (uint8_t)_mm_cvtsi128_si32(vout);
0434:         output += 1;
0435:       }
0436:     }
0437: 
0438:     output = (uint8_t*)((uintptr_t)output + output_increment);
0439:   } while (--output_width != 0);
0440: }
```

- **EN:** This block handles conditional branches and special cases; iterates over tensors, descriptors, options, or runtime state. Key symbols: no prominent local symbols.
- **CN:** 该代码块处理条件分支与特殊情况；遍历张量、描述符、配置项或运行时状态。关键符号：无明显局部符号。


## Key Concepts / 关键概念
- **ATen native operator logic** — ATen 原生算子逻辑
- **GPU kernel/runtime interaction** — GPU 内核/运行时交互
- **Core symbols: pytorch_q8dwconv_ukernel_up8x9_per_channel__sse2, sub_zero_point, _mm_sub_epi16, _mm_add_epi32, _mm_loadu_ps, _mm_mul_ps, _mm_adds_epi16, _mm_load_si128** — 核心符号：pytorch_q8dwconv_ukernel_up8x9_per_channel__sse2、sub_zero_point、_mm_sub_epi16、_mm_add_epi32、_mm_loadu_ps、_mm_mul_ps、_mm_adds_epi16、_mm_load_si128

## Dependencies / 依赖关系
- **Internal includes / 内部头文件**: 无
- **External includes / 外部头文件**: `immintrin.h`, `qnnpack/q8dwconv.h`, `requantization/runtime-sse2.h`
- **Namespaces / 命名空间**: 无
- **Representative symbols / 代表性符号**: `pytorch_q8dwconv_ukernel_up8x9_per_channel__sse2`, `sub_zero_point`, `_mm_sub_epi16`, `_mm_add_epi32`, `_mm_loadu_ps`, `_mm_mul_ps`, `_mm_adds_epi16`, `_mm_load_si128`, `_mm_storel_epi64`, `_mm_srl_epi64`
