# mp8x25-sse2-per-channel.c — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `aten/src/ATen/native/quantized/cpu/qnnpack/src/q8dwconv/mp8x25-sse2-per-channel.c`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements ATen native operators, checks, helper kernels, and backend dispatch behavior. This specific file centers on `mp8x25-sse2-per-channel.c`. The file header highlights: "All rights reserved. This source code is licensed under the BSD-style license found in the LICENSE file in the root directory of this source tree.."
- **Purpose (CN)**: 实现 ATen 原生算子、运行时检查、辅助内核以及后端分发行为。 该文件具体围绕 `mp8x25-sse2-per-channel.c` 展开。 文件头部注释也概括了其核心职责。

## Line-by-Line Analysis / 逐行分析
### Lines 1-42 / 第 1-42 行

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
0012: 
0013: void pytorch_q8dwconv_ukernel_mp8x25_per_channel__sse2(
0014:     size_t channels,
0015:     size_t output_width,
0016:     const uint8_t** input,
0017:     const void* weights,
0018:     int32_t* outacc32,
0019:     uint8_t* output,
0020:     size_t input_stride,
0021:     size_t output_increment,
0022:     const union pytorch_qnnp_conv_quantization_params
0023:         quantization_params[RESTRICT_STATIC 1]) {
0024:   const __m128i vinput_zero_point = _mm_load_si128(
0025:       (const __m128i*)quantization_params->sse2.input_zero_point);
0026:   const __m128i vzero = _mm_setzero_si128();
0027: 
0028:   do {
0029:     int32_t* outacc = outacc32;
0030:     const void* w = weights;
0031:     {
0032:       const uint8_t* i00 = input[0];
0033:       const uint8_t* i01 = input[1];
0034:       const uint8_t* i02 = input[2];
0035:       const uint8_t* i10 = input[3];
0036:       const uint8_t* i11 = input[4];
0037:       const uint8_t* i12 = input[5];
0038:       const uint8_t* i20 = input[6];
0039:       const uint8_t* i21 = input[7];
0040:       const uint8_t* i22 = input[8];
0041:       const uint8_t* i23 = input[9];
0042: 
```

- **EN:** This block implements local helper logic for `mp8x25-sse2-per-channel`. Key symbols: `pytorch_q8dwconv_ukernel_mp8x25_per_channel__sse2`.
- **CN:** 该代码块实现与 `mp8x25-sse2-per-channel` 相关的局部辅助逻辑。关键符号：`pytorch_q8dwconv_ukernel_mp8x25_per_channel__sse2`。

### Lines 43-84 / 第 43-84 行

```c
0043:       size_t c = channels;
0044:       for (; c >= 8; c -= 8) {
0045:         __m128i vacc_lo = _mm_loadu_si128((const __m128i*)w);
0046:         __m128i vacc_hi = _mm_loadu_si128((const __m128i*)((uintptr_t)w + 16));
0047:         const __m128i vkernel_zero_point = _mm_loadl_epi64(
0048:             (const __m128i*)
0049:             &quantization_params->sse2.kernel_zero_points[channels - c]);
0050: 
0051:         const __m128i vi00 = _mm_loadl_epi64((const __m128i*)i00);
0052:         i00 += 8;
0053:         const __m128i vxi00 =
0054:             _mm_sub_epi16(_mm_unpacklo_epi8(vi00, vzero), vinput_zero_point);
0055:         const __m128i vk00 =
0056:             _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 32));
0057:         const __m128i vxk00 =
0058:             _mm_sub_epi16(
0059:                 _mm_unpacklo_epi8(vk00, vzero),
0060:                 _mm_unpacklo_epi8(vkernel_zero_point, vzero));
0061:         const __m128i vprod00_odd = _mm_mullo_epi16(vxi00, vxk00);
0062:         const __m128i vprod00_even = _mm_mulhi_epi16(vxi00, vxk00);
0063:         vacc_lo = _mm_add_epi32(
0064:             vacc_lo, _mm_unpacklo_epi16(vprod00_odd, vprod00_even));
0065:         vacc_hi = _mm_add_epi32(
0066:             vacc_hi, _mm_unpackhi_epi16(vprod00_odd, vprod00_even));
0067: 
0068:         const __m128i vi01 = _mm_loadl_epi64((const __m128i*)i01);
0069:         i01 += 8;
0070:         const __m128i vxi01 =
0071:             _mm_sub_epi16(_mm_unpacklo_epi8(vi01, vzero), vinput_zero_point);
0072:         const __m128i vk01 =
0073:             _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 40));
0074:         const __m128i vxk01 =
0075:             _mm_sub_epi16(
0076:                 _mm_unpacklo_epi8(vk01, vzero),
0077:                 _mm_unpacklo_epi8(vkernel_zero_point, vzero));
0078:         const __m128i vprod01_odd = _mm_mullo_epi16(vxi01, vxk01);
0079:         const __m128i vprod01_even = _mm_mulhi_epi16(vxi01, vxk01);
0080:         vacc_lo = _mm_add_epi32(
0081:             vacc_lo, _mm_unpacklo_epi16(vprod01_odd, vprod01_even));
0082:         vacc_hi = _mm_add_epi32(
0083:             vacc_hi, _mm_unpackhi_epi16(vprod01_odd, vprod01_even));
0084: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`。

### Lines 85-118 / 第 85-118 行

```c
0085:         const __m128i vi02 = _mm_loadl_epi64((const __m128i*)i02);
0086:         i02 += 8;
0087:         const __m128i vxi02 =
0088:             _mm_sub_epi16(_mm_unpacklo_epi8(vi02, vzero), vinput_zero_point);
0089:         const __m128i vk02 =
0090:             _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 48));
0091:         const __m128i vxk02 =
0092:             _mm_sub_epi16(
0093:                 _mm_unpacklo_epi8(vk02, vzero),
0094:                 _mm_unpacklo_epi8(vkernel_zero_point, vzero));
0095:         const __m128i vprod02_odd = _mm_mullo_epi16(vxi02, vxk02);
0096:         const __m128i vprod02_even = _mm_mulhi_epi16(vxi02, vxk02);
0097:         vacc_lo = _mm_add_epi32(
0098:             vacc_lo, _mm_unpacklo_epi16(vprod02_odd, vprod02_even));
0099:         vacc_hi = _mm_add_epi32(
0100:             vacc_hi, _mm_unpackhi_epi16(vprod02_odd, vprod02_even));
0101: 
0102:         const __m128i vi10 = _mm_loadl_epi64((const __m128i*)i10);
0103:         i10 += 8;
0104:         const __m128i vxi10 =
0105:             _mm_sub_epi16(_mm_unpacklo_epi8(vi10, vzero), vinput_zero_point);
0106:         const __m128i vk10 =
0107:             _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 56));
0108:         const __m128i vxk10 =
0109:             _mm_sub_epi16(
0110:                 _mm_unpacklo_epi8(vk10, vzero),
0111:                 _mm_unpacklo_epi8(vkernel_zero_point, vzero));
0112:         const __m128i vprod10_odd = _mm_mullo_epi16(vxi10, vxk10);
0113:         const __m128i vprod10_even = _mm_mulhi_epi16(vxi10, vxk10);
0114:         vacc_lo = _mm_add_epi32(
0115:             vacc_lo, _mm_unpacklo_epi16(vprod10_odd, vprod10_even));
0116:         vacc_hi = _mm_add_epi32(
0117:             vacc_hi, _mm_unpackhi_epi16(vprod10_odd, vprod10_even));
0118: 
```

- **EN:** This block implements local helper logic for `mp8x25-sse2-per-channel`. Key symbols: `_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`.
- **CN:** 该代码块实现与 `mp8x25-sse2-per-channel` 相关的局部辅助逻辑。关键符号：`_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`。

### Lines 119-152 / 第 119-152 行

```c
0119:         const __m128i vi11 = _mm_loadl_epi64((const __m128i*)i11);
0120:         i11 += 8;
0121:         const __m128i vxi11 =
0122:             _mm_sub_epi16(_mm_unpacklo_epi8(vi11, vzero), vinput_zero_point);
0123:         const __m128i vk11 =
0124:             _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 64));
0125:         const __m128i vxk11 =
0126:             _mm_sub_epi16(
0127:                 _mm_unpacklo_epi8(vk11, vzero),
0128:                 _mm_unpacklo_epi8(vkernel_zero_point, vzero));
0129:         const __m128i vprod11_odd = _mm_mullo_epi16(vxi11, vxk11);
0130:         const __m128i vprod11_even = _mm_mulhi_epi16(vxi11, vxk11);
0131:         vacc_lo = _mm_add_epi32(
0132:             vacc_lo, _mm_unpacklo_epi16(vprod11_odd, vprod11_even));
0133:         vacc_hi = _mm_add_epi32(
0134:             vacc_hi, _mm_unpackhi_epi16(vprod11_odd, vprod11_even));
0135: 
0136:         const __m128i vi12 = _mm_loadl_epi64((const __m128i*)i12);
0137:         i12 += 8;
0138:         const __m128i vxi12 =
0139:             _mm_sub_epi16(_mm_unpacklo_epi8(vi12, vzero), vinput_zero_point);
0140:         const __m128i vk12 =
0141:             _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 72));
0142:         const __m128i vxk12 =
0143:             _mm_sub_epi16(
0144:                 _mm_unpacklo_epi8(vk12, vzero),
0145:                 _mm_unpacklo_epi8(vkernel_zero_point, vzero));
0146:         const __m128i vprod12_odd = _mm_mullo_epi16(vxi12, vxk12);
0147:         const __m128i vprod12_even = _mm_mulhi_epi16(vxi12, vxk12);
0148:         vacc_lo = _mm_add_epi32(
0149:             vacc_lo, _mm_unpacklo_epi16(vprod12_odd, vprod12_even));
0150:         vacc_hi = _mm_add_epi32(
0151:             vacc_hi, _mm_unpackhi_epi16(vprod12_odd, vprod12_even));
0152: 
```

- **EN:** This block implements local helper logic for `mp8x25-sse2-per-channel`. Key symbols: `_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`.
- **CN:** 该代码块实现与 `mp8x25-sse2-per-channel` 相关的局部辅助逻辑。关键符号：`_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`。

### Lines 153-186 / 第 153-186 行

```c
0153:         const __m128i vi20 = _mm_loadl_epi64((const __m128i*)i20);
0154:         i20 += 8;
0155:         const __m128i vxi20 =
0156:             _mm_sub_epi16(_mm_unpacklo_epi8(vi20, vzero), vinput_zero_point);
0157:         const __m128i vk20 =
0158:             _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 80));
0159:         const __m128i vxk20 =
0160:             _mm_sub_epi16(
0161:                 _mm_unpacklo_epi8(vk20, vzero),
0162:                 _mm_unpacklo_epi8(vkernel_zero_point, vzero));
0163:         const __m128i vprod20_odd = _mm_mullo_epi16(vxi20, vxk20);
0164:         const __m128i vprod20_even = _mm_mulhi_epi16(vxi20, vxk20);
0165:         vacc_lo = _mm_add_epi32(
0166:             vacc_lo, _mm_unpacklo_epi16(vprod20_odd, vprod20_even));
0167:         vacc_hi = _mm_add_epi32(
0168:             vacc_hi, _mm_unpackhi_epi16(vprod20_odd, vprod20_even));
0169: 
0170:         const __m128i vi21 = _mm_loadl_epi64((const __m128i*)i21);
0171:         i21 += 8;
0172:         const __m128i vxi21 =
0173:             _mm_sub_epi16(_mm_unpacklo_epi8(vi21, vzero), vinput_zero_point);
0174:         const __m128i vk21 =
0175:             _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 88));
0176:         const __m128i vxk21 =
0177:             _mm_sub_epi16(
0178:                 _mm_unpacklo_epi8(vk21, vzero),
0179:                 _mm_unpacklo_epi8(vkernel_zero_point, vzero));
0180:         const __m128i vprod21_odd = _mm_mullo_epi16(vxi21, vxk21);
0181:         const __m128i vprod21_even = _mm_mulhi_epi16(vxi21, vxk21);
0182:         vacc_lo = _mm_add_epi32(
0183:             vacc_lo, _mm_unpacklo_epi16(vprod21_odd, vprod21_even));
0184:         vacc_hi = _mm_add_epi32(
0185:             vacc_hi, _mm_unpackhi_epi16(vprod21_odd, vprod21_even));
0186: 
```

- **EN:** This block implements local helper logic for `mp8x25-sse2-per-channel`. Key symbols: `_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`.
- **CN:** 该代码块实现与 `mp8x25-sse2-per-channel` 相关的局部辅助逻辑。关键符号：`_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`。

### Lines 187-220 / 第 187-220 行

```c
0187:         const __m128i vi22 = _mm_loadl_epi64((const __m128i*)i22);
0188:         i22 += 8;
0189:         const __m128i vxi22 =
0190:             _mm_sub_epi16(_mm_unpacklo_epi8(vi22, vzero), vinput_zero_point);
0191:         const __m128i vk22 =
0192:             _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 96));
0193:         const __m128i vxk22 =
0194:             _mm_sub_epi16(
0195:                 _mm_unpacklo_epi8(vk22, vzero),
0196:                 _mm_unpacklo_epi8(vkernel_zero_point, vzero));
0197:         const __m128i vprod22_odd = _mm_mullo_epi16(vxi22, vxk22);
0198:         const __m128i vprod22_even = _mm_mulhi_epi16(vxi22, vxk22);
0199:         vacc_lo = _mm_add_epi32(
0200:             vacc_lo, _mm_unpacklo_epi16(vprod22_odd, vprod22_even));
0201:         vacc_hi = _mm_add_epi32(
0202:             vacc_hi, _mm_unpackhi_epi16(vprod22_odd, vprod22_even));
0203: 
0204:         const __m128i vi23 = _mm_loadl_epi64((const __m128i*)i23);
0205:         i23 += 8;
0206:         const __m128i vxi23 =
0207:             _mm_sub_epi16(_mm_unpacklo_epi8(vi23, vzero), vinput_zero_point);
0208:         const __m128i vk23 =
0209:             _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 104));
0210:         const __m128i vxk23 =
0211:             _mm_sub_epi16(
0212:                 _mm_unpacklo_epi8(vk23, vzero),
0213:                 _mm_unpacklo_epi8(vkernel_zero_point, vzero));
0214:         const __m128i vprod23_odd = _mm_mullo_epi16(vxi23, vxk23);
0215:         const __m128i vprod23_even = _mm_mulhi_epi16(vxi23, vxk23);
0216:         vacc_lo = _mm_add_epi32(
0217:             vacc_lo, _mm_unpacklo_epi16(vprod23_odd, vprod23_even));
0218:         vacc_hi = _mm_add_epi32(
0219:             vacc_hi, _mm_unpackhi_epi16(vprod23_odd, vprod23_even));
0220: 
```

- **EN:** This block implements local helper logic for `mp8x25-sse2-per-channel`. Key symbols: `_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`.
- **CN:** 该代码块实现与 `mp8x25-sse2-per-channel` 相关的局部辅助逻辑。关键符号：`_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`。

### Lines 221-263 / 第 221-263 行

```c
0221:         w = (const void*)((uintptr_t)w + 112);
0222:         _mm_storeu_si128((__m128i*)outacc, vacc_lo);
0223:         outacc += 4;
0224:         _mm_storeu_si128((__m128i*)outacc, vacc_hi);
0225:         outacc += 4;
0226:       }
0227:       if (c != 0) {
0228:         const size_t i_predecrement = 8 - c;
0229:         const __m128i vi_shift = _mm_cvtsi32_si128(8 * i_predecrement);
0230:         const __m128i vkernel_zero_point = _mm_loadl_epi64(
0231:             (const __m128i*)
0232:             &quantization_params->sse2.kernel_zero_points[channels - c]);
0233:         i00 -= i_predecrement;
0234:         i01 -= i_predecrement;
0235:         i02 -= i_predecrement;
0236:         i10 -= i_predecrement;
0237:         i11 -= i_predecrement;
0238:         i12 -= i_predecrement;
0239:         i20 -= i_predecrement;
0240:         i21 -= i_predecrement;
0241:         i22 -= i_predecrement;
0242:         i23 -= i_predecrement;
0243: 
0244:         __m128i vacc_lo = _mm_loadu_si128((const __m128i*)w);
0245:         __m128i vacc_hi = _mm_loadu_si128((const __m128i*)((uintptr_t)w + 16));
0246: 
0247:         const __m128i vi00 =
0248:             _mm_srl_epi64(_mm_loadl_epi64((const __m128i*)i00), vi_shift);
0249:         const __m128i vxi00 =
0250:             _mm_sub_epi16(_mm_unpacklo_epi8(vi00, vzero), vinput_zero_point);
0251:         const __m128i vk00 =
0252:             _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 32));
0253:         const __m128i vxk00 =
0254:             _mm_sub_epi16(
0255:                 _mm_unpacklo_epi8(vk00, vzero),
0256:                 _mm_unpacklo_epi8(vkernel_zero_point, vzero));
0257:         const __m128i vprod00_odd = _mm_mullo_epi16(vxi00, vxk00);
0258:         const __m128i vprod00_even = _mm_mulhi_epi16(vxi00, vxk00);
0259:         vacc_lo = _mm_add_epi32(
0260:             vacc_lo, _mm_unpacklo_epi16(vprod00_odd, vprod00_even));
0261:         vacc_hi = _mm_add_epi32(
0262:             vacc_hi, _mm_unpackhi_epi16(vprod00_odd, vprod00_even));
0263: 
```

- **EN:** This block handles conditional branches and special cases. Key symbols: `_mm_storeu_si128`, `_mm_srl_epi64`, `_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`.
- **CN:** 该代码块处理条件分支与特殊情况。关键符号：`_mm_storeu_si128`, `_mm_srl_epi64`, `_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`。

### Lines 264-297 / 第 264-297 行

```c
0264:         const __m128i vi01 =
0265:             _mm_srl_epi64(_mm_loadl_epi64((const __m128i*)i01), vi_shift);
0266:         const __m128i vxi01 =
0267:             _mm_sub_epi16(_mm_unpacklo_epi8(vi01, vzero), vinput_zero_point);
0268:         const __m128i vk01 =
0269:             _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 40));
0270:         const __m128i vxk01 =
0271:             _mm_sub_epi16(
0272:                 _mm_unpacklo_epi8(vk01, vzero),
0273:                 _mm_unpacklo_epi8(vkernel_zero_point, vzero));
0274:         const __m128i vprod01_odd = _mm_mullo_epi16(vxi01, vxk01);
0275:         const __m128i vprod01_even = _mm_mulhi_epi16(vxi01, vxk01);
0276:         vacc_lo = _mm_add_epi32(
0277:             vacc_lo, _mm_unpacklo_epi16(vprod01_odd, vprod01_even));
0278:         vacc_hi = _mm_add_epi32(
0279:             vacc_hi, _mm_unpackhi_epi16(vprod01_odd, vprod01_even));
0280: 
0281:         const __m128i vi02 =
0282:             _mm_srl_epi64(_mm_loadl_epi64((const __m128i*)i02), vi_shift);
0283:         const __m128i vxi02 =
0284:             _mm_sub_epi16(_mm_unpacklo_epi8(vi02, vzero), vinput_zero_point);
0285:         const __m128i vk02 =
0286:             _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 48));
0287:         const __m128i vxk02 =
0288:             _mm_sub_epi16(
0289:                 _mm_unpacklo_epi8(vk02, vzero),
0290:                 _mm_unpacklo_epi8(vkernel_zero_point, vzero));
0291:         const __m128i vprod02_odd = _mm_mullo_epi16(vxi02, vxk02);
0292:         const __m128i vprod02_even = _mm_mulhi_epi16(vxi02, vxk02);
0293:         vacc_lo = _mm_add_epi32(
0294:             vacc_lo, _mm_unpacklo_epi16(vprod02_odd, vprod02_even));
0295:         vacc_hi = _mm_add_epi32(
0296:             vacc_hi, _mm_unpackhi_epi16(vprod02_odd, vprod02_even));
0297: 
```

- **EN:** This block implements local helper logic for `mp8x25-sse2-per-channel`. Key symbols: `_mm_srl_epi64`, `_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`.
- **CN:** 该代码块实现与 `mp8x25-sse2-per-channel` 相关的局部辅助逻辑。关键符号：`_mm_srl_epi64`, `_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`。

### Lines 298-331 / 第 298-331 行

```c
0298:         const __m128i vi10 =
0299:             _mm_srl_epi64(_mm_loadl_epi64((const __m128i*)i10), vi_shift);
0300:         const __m128i vxi10 =
0301:             _mm_sub_epi16(_mm_unpacklo_epi8(vi10, vzero), vinput_zero_point);
0302:         const __m128i vk10 =
0303:             _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 56));
0304:         const __m128i vxk10 =
0305:             _mm_sub_epi16(
0306:                 _mm_unpacklo_epi8(vk10, vzero),
0307:                 _mm_unpacklo_epi8(vkernel_zero_point, vzero));
0308:         const __m128i vprod10_odd = _mm_mullo_epi16(vxi10, vxk10);
0309:         const __m128i vprod10_even = _mm_mulhi_epi16(vxi10, vxk10);
0310:         vacc_lo = _mm_add_epi32(
0311:             vacc_lo, _mm_unpacklo_epi16(vprod10_odd, vprod10_even));
0312:         vacc_hi = _mm_add_epi32(
0313:             vacc_hi, _mm_unpackhi_epi16(vprod10_odd, vprod10_even));
0314: 
0315:         const __m128i vi11 =
0316:             _mm_srl_epi64(_mm_loadl_epi64((const __m128i*)i11), vi_shift);
0317:         const __m128i vxi11 =
0318:             _mm_sub_epi16(_mm_unpacklo_epi8(vi11, vzero), vinput_zero_point);
0319:         const __m128i vk11 =
0320:             _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 64));
0321:         const __m128i vxk11 =
0322:             _mm_sub_epi16(
0323:                 _mm_unpacklo_epi8(vk11, vzero),
0324:                 _mm_unpacklo_epi8(vkernel_zero_point, vzero));
0325:         const __m128i vprod11_odd = _mm_mullo_epi16(vxi11, vxk11);
0326:         const __m128i vprod11_even = _mm_mulhi_epi16(vxi11, vxk11);
0327:         vacc_lo = _mm_add_epi32(
0328:             vacc_lo, _mm_unpacklo_epi16(vprod11_odd, vprod11_even));
0329:         vacc_hi = _mm_add_epi32(
0330:             vacc_hi, _mm_unpackhi_epi16(vprod11_odd, vprod11_even));
0331: 
```

- **EN:** This block implements local helper logic for `mp8x25-sse2-per-channel`. Key symbols: `_mm_srl_epi64`, `_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`.
- **CN:** 该代码块实现与 `mp8x25-sse2-per-channel` 相关的局部辅助逻辑。关键符号：`_mm_srl_epi64`, `_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`。

### Lines 332-365 / 第 332-365 行

```c
0332:         const __m128i vi12 =
0333:             _mm_srl_epi64(_mm_loadl_epi64((const __m128i*)i12), vi_shift);
0334:         const __m128i vxi12 =
0335:             _mm_sub_epi16(_mm_unpacklo_epi8(vi12, vzero), vinput_zero_point);
0336:         const __m128i vk12 =
0337:             _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 72));
0338:         const __m128i vxk12 =
0339:             _mm_sub_epi16(
0340:                 _mm_unpacklo_epi8(vk12, vzero),
0341:                 _mm_unpacklo_epi8(vkernel_zero_point, vzero));
0342:         const __m128i vprod12_odd = _mm_mullo_epi16(vxi12, vxk12);
0343:         const __m128i vprod12_even = _mm_mulhi_epi16(vxi12, vxk12);
0344:         vacc_lo = _mm_add_epi32(
0345:             vacc_lo, _mm_unpacklo_epi16(vprod12_odd, vprod12_even));
0346:         vacc_hi = _mm_add_epi32(
0347:             vacc_hi, _mm_unpackhi_epi16(vprod12_odd, vprod12_even));
0348: 
0349:         const __m128i vi20 =
0350:             _mm_srl_epi64(_mm_loadl_epi64((const __m128i*)i20), vi_shift);
0351:         const __m128i vxi20 =
0352:             _mm_sub_epi16(_mm_unpacklo_epi8(vi20, vzero), vinput_zero_point);
0353:         const __m128i vk20 =
0354:             _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 80));
0355:         const __m128i vxk20 =
0356:             _mm_sub_epi16(
0357:                 _mm_unpacklo_epi8(vk20, vzero),
0358:                 _mm_unpacklo_epi8(vkernel_zero_point, vzero));
0359:         const __m128i vprod20_odd = _mm_mullo_epi16(vxi20, vxk20);
0360:         const __m128i vprod20_even = _mm_mulhi_epi16(vxi20, vxk20);
0361:         vacc_lo = _mm_add_epi32(
0362:             vacc_lo, _mm_unpacklo_epi16(vprod20_odd, vprod20_even));
0363:         vacc_hi = _mm_add_epi32(
0364:             vacc_hi, _mm_unpackhi_epi16(vprod20_odd, vprod20_even));
0365: 
```

- **EN:** This block implements local helper logic for `mp8x25-sse2-per-channel`. Key symbols: `_mm_srl_epi64`, `_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`.
- **CN:** 该代码块实现与 `mp8x25-sse2-per-channel` 相关的局部辅助逻辑。关键符号：`_mm_srl_epi64`, `_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`。

### Lines 366-399 / 第 366-399 行

```c
0366:         const __m128i vi21 =
0367:             _mm_srl_epi64(_mm_loadl_epi64((const __m128i*)i21), vi_shift);
0368:         const __m128i vxi21 =
0369:             _mm_sub_epi16(_mm_unpacklo_epi8(vi21, vzero), vinput_zero_point);
0370:         const __m128i vk21 =
0371:             _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 88));
0372:         const __m128i vxk21 =
0373:             _mm_sub_epi16(
0374:                 _mm_unpacklo_epi8(vk21, vzero),
0375:                 _mm_unpacklo_epi8(vkernel_zero_point, vzero));
0376:         const __m128i vprod21_odd = _mm_mullo_epi16(vxi21, vxk21);
0377:         const __m128i vprod21_even = _mm_mulhi_epi16(vxi21, vxk21);
0378:         vacc_lo = _mm_add_epi32(
0379:             vacc_lo, _mm_unpacklo_epi16(vprod21_odd, vprod21_even));
0380:         vacc_hi = _mm_add_epi32(
0381:             vacc_hi, _mm_unpackhi_epi16(vprod21_odd, vprod21_even));
0382: 
0383:         const __m128i vi22 =
0384:             _mm_srl_epi64(_mm_loadl_epi64((const __m128i*)i22), vi_shift);
0385:         const __m128i vxi22 =
0386:             _mm_sub_epi16(_mm_unpacklo_epi8(vi22, vzero), vinput_zero_point);
0387:         const __m128i vk22 =
0388:             _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 96));
0389:         const __m128i vxk22 =
0390:             _mm_sub_epi16(
0391:                 _mm_unpacklo_epi8(vk22, vzero),
0392:                 _mm_unpacklo_epi8(vkernel_zero_point, vzero));
0393:         const __m128i vprod22_odd = _mm_mullo_epi16(vxi22, vxk22);
0394:         const __m128i vprod22_even = _mm_mulhi_epi16(vxi22, vxk22);
0395:         vacc_lo = _mm_add_epi32(
0396:             vacc_lo, _mm_unpacklo_epi16(vprod22_odd, vprod22_even));
0397:         vacc_hi = _mm_add_epi32(
0398:             vacc_hi, _mm_unpackhi_epi16(vprod22_odd, vprod22_even));
0399: 
```

- **EN:** This block implements local helper logic for `mp8x25-sse2-per-channel`. Key symbols: `_mm_srl_epi64`, `_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`.
- **CN:** 该代码块实现与 `mp8x25-sse2-per-channel` 相关的局部辅助逻辑。关键符号：`_mm_srl_epi64`, `_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`。

### Lines 400-435 / 第 400-435 行

```c
0400:         const __m128i vi23 =
0401:             _mm_srl_epi64(_mm_loadl_epi64((const __m128i*)i23), vi_shift);
0402:         const __m128i vxi23 =
0403:             _mm_sub_epi16(_mm_unpacklo_epi8(vi23, vzero), vinput_zero_point);
0404:         const __m128i vk23 =
0405:             _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 104));
0406:         const __m128i vxk23 =
0407:             _mm_sub_epi16(
0408:                 _mm_unpacklo_epi8(vk23, vzero),
0409:                 _mm_unpacklo_epi8(vkernel_zero_point, vzero));
0410:         const __m128i vprod23_odd = _mm_mullo_epi16(vxi23, vxk23);
0411:         const __m128i vprod23_even = _mm_mulhi_epi16(vxi23, vxk23);
0412:         vacc_lo = _mm_add_epi32(
0413:             vacc_lo, _mm_unpacklo_epi16(vprod23_odd, vprod23_even));
0414:         vacc_hi = _mm_add_epi32(
0415:             vacc_hi, _mm_unpackhi_epi16(vprod23_odd, vprod23_even));
0416: 
0417:         w = (const void*)((uintptr_t)w + 112);
0418:         _mm_storeu_si128((__m128i*)outacc, vacc_lo);
0419:         outacc += 4;
0420:         _mm_storeu_si128((__m128i*)outacc, vacc_hi);
0421:       }
0422:     }
0423:     {
0424:       const uint8_t* i00 = input[10];
0425:       const uint8_t* i01 = input[11];
0426:       const uint8_t* i02 = input[12];
0427:       const uint8_t* i10 = input[13];
0428:       const uint8_t* i11 = input[14];
0429:       const uint8_t* i12 = input[15];
0430:       const uint8_t* i20 = input[16];
0431:       const uint8_t* i21 = input[17];
0432:       const uint8_t* i22 = input[18];
0433:       const uint8_t* i23 = input[19];
0434:       outacc = outacc32;
0435: 
```

- **EN:** This block implements local helper logic for `mp8x25-sse2-per-channel`. Key symbols: `_mm_srl_epi64`, `_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`, `_mm_storeu_si128`.
- **CN:** 该代码块实现与 `mp8x25-sse2-per-channel` 相关的局部辅助逻辑。关键符号：`_mm_srl_epi64`, `_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`, `_mm_storeu_si128`。

### Lines 436-471 / 第 436-471 行

```c
0436:       size_t c = channels;
0437:       for (; c >= 8; c -= 8) {
0438:         const __m128i vi00 = _mm_loadl_epi64((const __m128i*)i00);
0439:         const __m128i vkernel_zero_point = _mm_loadl_epi64(
0440:             (const __m128i*)
0441:             &quantization_params->sse2.kernel_zero_points[channels - c]);
0442:         i00 += 8;
0443:         const __m128i vxi00 =
0444:             _mm_sub_epi16(_mm_unpacklo_epi8(vi00, vzero), vinput_zero_point);
0445:         const __m128i vk00 = _mm_loadl_epi64((const __m128i*)((uintptr_t)w));
0446:         const __m128i vxk00 =
0447:             _mm_sub_epi16(
0448:                 _mm_unpacklo_epi8(vk00, vzero),
0449:                 _mm_unpacklo_epi8(vkernel_zero_point, vzero));
0450:         const __m128i vprod00_odd = _mm_mullo_epi16(vxi00, vxk00);
0451:         const __m128i vprod00_even = _mm_mulhi_epi16(vxi00, vxk00);
0452:         __m128i vacc_lo = _mm_unpacklo_epi16(vprod00_odd, vprod00_even);
0453:         __m128i vacc_hi = _mm_unpackhi_epi16(vprod00_odd, vprod00_even);
0454: 
0455:         const __m128i vi01 = _mm_loadl_epi64((const __m128i*)i01);
0456:         i01 += 8;
0457:         const __m128i vxi01 =
0458:             _mm_sub_epi16(_mm_unpacklo_epi8(vi01, vzero), vinput_zero_point);
0459:         const __m128i vk01 =
0460:             _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 8));
0461:         const __m128i vxk01 =
0462:             _mm_sub_epi16(
0463:                 _mm_unpacklo_epi8(vk01, vzero),
0464:                 _mm_unpacklo_epi8(vkernel_zero_point, vzero));
0465:         const __m128i vprod01_odd = _mm_mullo_epi16(vxi01, vxk01);
0466:         const __m128i vprod01_even = _mm_mulhi_epi16(vxi01, vxk01);
0467:         vacc_lo = _mm_add_epi32(
0468:             vacc_lo, _mm_unpacklo_epi16(vprod01_odd, vprod01_even));
0469:         vacc_hi = _mm_add_epi32(
0470:             vacc_hi, _mm_unpackhi_epi16(vprod01_odd, vprod01_even));
0471: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`。

### Lines 472-505 / 第 472-505 行

```c
0472:         const __m128i vi02 = _mm_loadl_epi64((const __m128i*)i02);
0473:         i02 += 8;
0474:         const __m128i vxi02 =
0475:             _mm_sub_epi16(_mm_unpacklo_epi8(vi02, vzero), vinput_zero_point);
0476:         const __m128i vk02 =
0477:             _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 16));
0478:         const __m128i vxk02 =
0479:             _mm_sub_epi16(
0480:                 _mm_unpacklo_epi8(vk02, vzero),
0481:                 _mm_unpacklo_epi8(vkernel_zero_point, vzero));
0482:         const __m128i vprod02_odd = _mm_mullo_epi16(vxi02, vxk02);
0483:         const __m128i vprod02_even = _mm_mulhi_epi16(vxi02, vxk02);
0484:         vacc_lo = _mm_add_epi32(
0485:             vacc_lo, _mm_unpacklo_epi16(vprod02_odd, vprod02_even));
0486:         vacc_hi = _mm_add_epi32(
0487:             vacc_hi, _mm_unpackhi_epi16(vprod02_odd, vprod02_even));
0488: 
0489:         const __m128i vi10 = _mm_loadl_epi64((const __m128i*)i10);
0490:         i10 += 8;
0491:         const __m128i vxi10 =
0492:             _mm_sub_epi16(_mm_unpacklo_epi8(vi10, vzero), vinput_zero_point);
0493:         const __m128i vk10 =
0494:             _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 24));
0495:         const __m128i vxk10 =
0496:             _mm_sub_epi16(
0497:                 _mm_unpacklo_epi8(vk10, vzero),
0498:                 _mm_unpacklo_epi8(vkernel_zero_point, vzero));
0499:         const __m128i vprod10_odd = _mm_mullo_epi16(vxi10, vxk10);
0500:         const __m128i vprod10_even = _mm_mulhi_epi16(vxi10, vxk10);
0501:         vacc_lo = _mm_add_epi32(
0502:             vacc_lo, _mm_unpacklo_epi16(vprod10_odd, vprod10_even));
0503:         vacc_hi = _mm_add_epi32(
0504:             vacc_hi, _mm_unpackhi_epi16(vprod10_odd, vprod10_even));
0505: 
```

- **EN:** This block implements local helper logic for `mp8x25-sse2-per-channel`. Key symbols: `_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`.
- **CN:** 该代码块实现与 `mp8x25-sse2-per-channel` 相关的局部辅助逻辑。关键符号：`_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`。

### Lines 506-539 / 第 506-539 行

```c
0506:         const __m128i vi11 = _mm_loadl_epi64((const __m128i*)i11);
0507:         i11 += 8;
0508:         const __m128i vxi11 =
0509:             _mm_sub_epi16(_mm_unpacklo_epi8(vi11, vzero), vinput_zero_point);
0510:         const __m128i vk11 =
0511:             _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 32));
0512:         const __m128i vxk11 =
0513:             _mm_sub_epi16(
0514:                 _mm_unpacklo_epi8(vk11, vzero),
0515:                 _mm_unpacklo_epi8(vkernel_zero_point, vzero));
0516:         const __m128i vprod11_odd = _mm_mullo_epi16(vxi11, vxk11);
0517:         const __m128i vprod11_even = _mm_mulhi_epi16(vxi11, vxk11);
0518:         vacc_lo = _mm_add_epi32(
0519:             vacc_lo, _mm_unpacklo_epi16(vprod11_odd, vprod11_even));
0520:         vacc_hi = _mm_add_epi32(
0521:             vacc_hi, _mm_unpackhi_epi16(vprod11_odd, vprod11_even));
0522: 
0523:         const __m128i vi12 = _mm_loadl_epi64((const __m128i*)i12);
0524:         i12 += 8;
0525:         const __m128i vxi12 =
0526:             _mm_sub_epi16(_mm_unpacklo_epi8(vi12, vzero), vinput_zero_point);
0527:         const __m128i vk12 =
0528:             _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 40));
0529:         const __m128i vxk12 =
0530:             _mm_sub_epi16(
0531:                 _mm_unpacklo_epi8(vk12, vzero),
0532:                 _mm_unpacklo_epi8(vkernel_zero_point, vzero));
0533:         const __m128i vprod12_odd = _mm_mullo_epi16(vxi12, vxk12);
0534:         const __m128i vprod12_even = _mm_mulhi_epi16(vxi12, vxk12);
0535:         vacc_lo = _mm_add_epi32(
0536:             vacc_lo, _mm_unpacklo_epi16(vprod12_odd, vprod12_even));
0537:         vacc_hi = _mm_add_epi32(
0538:             vacc_hi, _mm_unpackhi_epi16(vprod12_odd, vprod12_even));
0539: 
```

- **EN:** This block implements local helper logic for `mp8x25-sse2-per-channel`. Key symbols: `_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`.
- **CN:** 该代码块实现与 `mp8x25-sse2-per-channel` 相关的局部辅助逻辑。关键符号：`_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`。

### Lines 540-573 / 第 540-573 行

```c
0540:         const __m128i vi20 = _mm_loadl_epi64((const __m128i*)i20);
0541:         i20 += 8;
0542:         const __m128i vxi20 =
0543:             _mm_sub_epi16(_mm_unpacklo_epi8(vi20, vzero), vinput_zero_point);
0544:         const __m128i vk20 =
0545:             _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 48));
0546:         const __m128i vxk20 =
0547:             _mm_sub_epi16(
0548:                 _mm_unpacklo_epi8(vk20, vzero),
0549:                 _mm_unpacklo_epi8(vkernel_zero_point, vzero));
0550:         const __m128i vprod20_odd = _mm_mullo_epi16(vxi20, vxk20);
0551:         const __m128i vprod20_even = _mm_mulhi_epi16(vxi20, vxk20);
0552:         vacc_lo = _mm_add_epi32(
0553:             vacc_lo, _mm_unpacklo_epi16(vprod20_odd, vprod20_even));
0554:         vacc_hi = _mm_add_epi32(
0555:             vacc_hi, _mm_unpackhi_epi16(vprod20_odd, vprod20_even));
0556: 
0557:         const __m128i vi21 = _mm_loadl_epi64((const __m128i*)i21);
0558:         i21 += 8;
0559:         const __m128i vxi21 =
0560:             _mm_sub_epi16(_mm_unpacklo_epi8(vi21, vzero), vinput_zero_point);
0561:         const __m128i vk21 =
0562:             _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 56));
0563:         const __m128i vxk21 =
0564:             _mm_sub_epi16(
0565:                 _mm_unpacklo_epi8(vk21, vzero),
0566:                 _mm_unpacklo_epi8(vkernel_zero_point, vzero));
0567:         const __m128i vprod21_odd = _mm_mullo_epi16(vxi21, vxk21);
0568:         const __m128i vprod21_even = _mm_mulhi_epi16(vxi21, vxk21);
0569:         vacc_lo = _mm_add_epi32(
0570:             vacc_lo, _mm_unpacklo_epi16(vprod21_odd, vprod21_even));
0571:         vacc_hi = _mm_add_epi32(
0572:             vacc_hi, _mm_unpackhi_epi16(vprod21_odd, vprod21_even));
0573: 
```

- **EN:** This block implements local helper logic for `mp8x25-sse2-per-channel`. Key symbols: `_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`.
- **CN:** 该代码块实现与 `mp8x25-sse2-per-channel` 相关的局部辅助逻辑。关键符号：`_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`。

### Lines 574-607 / 第 574-607 行

```c
0574:         const __m128i vi22 = _mm_loadl_epi64((const __m128i*)i22);
0575:         i22 += 8;
0576:         const __m128i vxi22 =
0577:             _mm_sub_epi16(_mm_unpacklo_epi8(vi22, vzero), vinput_zero_point);
0578:         const __m128i vk22 =
0579:             _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 64));
0580:         const __m128i vxk22 =
0581:             _mm_sub_epi16(
0582:                 _mm_unpacklo_epi8(vk22, vzero),
0583:                 _mm_unpacklo_epi8(vkernel_zero_point, vzero));
0584:         const __m128i vprod22_odd = _mm_mullo_epi16(vxi22, vxk22);
0585:         const __m128i vprod22_even = _mm_mulhi_epi16(vxi22, vxk22);
0586:         vacc_lo = _mm_add_epi32(
0587:             vacc_lo, _mm_unpacklo_epi16(vprod22_odd, vprod22_even));
0588:         vacc_hi = _mm_add_epi32(
0589:             vacc_hi, _mm_unpackhi_epi16(vprod22_odd, vprod22_even));
0590: 
0591:         const __m128i vi23 = _mm_loadl_epi64((const __m128i*)i23);
0592:         i23 += 8;
0593:         const __m128i vxi23 =
0594:             _mm_sub_epi16(_mm_unpacklo_epi8(vi23, vzero), vinput_zero_point);
0595:         const __m128i vk23 =
0596:             _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 72));
0597:         const __m128i vxk23 =
0598:             _mm_sub_epi16(
0599:                 _mm_unpacklo_epi8(vk23, vzero),
0600:                 _mm_unpacklo_epi8(vkernel_zero_point, vzero));
0601:         const __m128i vprod23_odd = _mm_mullo_epi16(vxi23, vxk23);
0602:         const __m128i vprod23_even = _mm_mulhi_epi16(vxi23, vxk23);
0603:         vacc_lo = _mm_add_epi32(
0604:             vacc_lo, _mm_unpacklo_epi16(vprod23_odd, vprod23_even));
0605:         vacc_hi = _mm_add_epi32(
0606:             vacc_hi, _mm_unpackhi_epi16(vprod23_odd, vprod23_even));
0607: 
```

- **EN:** This block implements local helper logic for `mp8x25-sse2-per-channel`. Key symbols: `_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`.
- **CN:** 该代码块实现与 `mp8x25-sse2-per-channel` 相关的局部辅助逻辑。关键符号：`_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`。

### Lines 608-647 / 第 608-647 行

```c
0608:         w = (const void*)((uintptr_t)w + 80);
0609:         vacc_lo = _mm_add_epi32(vacc_lo, _mm_loadu_si128((__m128i*)outacc));
0610:         vacc_hi =
0611:             _mm_add_epi32(vacc_hi, _mm_loadu_si128((__m128i*)(outacc + 4)));
0612:         _mm_storeu_si128((__m128i*)outacc, vacc_lo);
0613:         outacc += 4;
0614:         _mm_storeu_si128((__m128i*)outacc, vacc_hi);
0615:         outacc += 4;
0616:       }
0617:       if (c != 0) {
0618:         const size_t i_predecrement = 8 - c;
0619:         const __m128i vi_shift = _mm_cvtsi32_si128(8 * i_predecrement);
0620:         const __m128i vkernel_zero_point = _mm_loadl_epi64(
0621:             (const __m128i*)
0622:             &quantization_params->sse2.kernel_zero_points[channels - c]);
0623:         i00 -= i_predecrement;
0624:         i01 -= i_predecrement;
0625:         i02 -= i_predecrement;
0626:         i10 -= i_predecrement;
0627:         i11 -= i_predecrement;
0628:         i12 -= i_predecrement;
0629:         i20 -= i_predecrement;
0630:         i21 -= i_predecrement;
0631:         i22 -= i_predecrement;
0632:         i23 -= i_predecrement;
0633: 
0634:         const __m128i vi00 =
0635:             _mm_srl_epi64(_mm_loadl_epi64((const __m128i*)i00), vi_shift);
0636:         const __m128i vxi00 =
0637:             _mm_sub_epi16(_mm_unpacklo_epi8(vi00, vzero), vinput_zero_point);
0638:         const __m128i vk00 = _mm_loadl_epi64((const __m128i*)((uintptr_t)w));
0639:         const __m128i vxk00 =
0640:             _mm_sub_epi16(
0641:                 _mm_unpacklo_epi8(vk00, vzero),
0642:                 _mm_unpacklo_epi8(vkernel_zero_point, vzero));
0643:         const __m128i vprod00_odd = _mm_mullo_epi16(vxi00, vxk00);
0644:         const __m128i vprod00_even = _mm_mulhi_epi16(vxi00, vxk00);
0645:         __m128i vacc_lo = _mm_unpacklo_epi16(vprod00_odd, vprod00_even);
0646:         __m128i vacc_hi = _mm_unpackhi_epi16(vprod00_odd, vprod00_even);
0647: 
```

- **EN:** This block handles conditional branches and special cases. Key symbols: `_mm_add_epi32`, `_mm_storeu_si128`, `_mm_srl_epi64`, `_mm_sub_epi16`.
- **CN:** 该代码块处理条件分支与特殊情况。关键符号：`_mm_add_epi32`, `_mm_storeu_si128`, `_mm_srl_epi64`, `_mm_sub_epi16`。

### Lines 648-681 / 第 648-681 行

```c
0648:         const __m128i vi01 =
0649:             _mm_srl_epi64(_mm_loadl_epi64((const __m128i*)i01), vi_shift);
0650:         const __m128i vxi01 =
0651:             _mm_sub_epi16(_mm_unpacklo_epi8(vi01, vzero), vinput_zero_point);
0652:         const __m128i vk01 =
0653:             _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 8));
0654:         const __m128i vxk01 =
0655:             _mm_sub_epi16(
0656:                 _mm_unpacklo_epi8(vk01, vzero),
0657:                 _mm_unpacklo_epi8(vkernel_zero_point, vzero));
0658:         const __m128i vprod01_odd = _mm_mullo_epi16(vxi01, vxk01);
0659:         const __m128i vprod01_even = _mm_mulhi_epi16(vxi01, vxk01);
0660:         vacc_lo = _mm_add_epi32(
0661:             vacc_lo, _mm_unpacklo_epi16(vprod01_odd, vprod01_even));
0662:         vacc_hi = _mm_add_epi32(
0663:             vacc_hi, _mm_unpackhi_epi16(vprod01_odd, vprod01_even));
0664: 
0665:         const __m128i vi02 =
0666:             _mm_srl_epi64(_mm_loadl_epi64((const __m128i*)i02), vi_shift);
0667:         const __m128i vxi02 =
0668:             _mm_sub_epi16(_mm_unpacklo_epi8(vi02, vzero), vinput_zero_point);
0669:         const __m128i vk02 =
0670:             _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 16));
0671:         const __m128i vxk02 =
0672:             _mm_sub_epi16(
0673:                 _mm_unpacklo_epi8(vk02, vzero),
0674:                 _mm_unpacklo_epi8(vkernel_zero_point, vzero));
0675:         const __m128i vprod02_odd = _mm_mullo_epi16(vxi02, vxk02);
0676:         const __m128i vprod02_even = _mm_mulhi_epi16(vxi02, vxk02);
0677:         vacc_lo = _mm_add_epi32(
0678:             vacc_lo, _mm_unpacklo_epi16(vprod02_odd, vprod02_even));
0679:         vacc_hi = _mm_add_epi32(
0680:             vacc_hi, _mm_unpackhi_epi16(vprod02_odd, vprod02_even));
0681: 
```

- **EN:** This block implements local helper logic for `mp8x25-sse2-per-channel`. Key symbols: `_mm_srl_epi64`, `_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`.
- **CN:** 该代码块实现与 `mp8x25-sse2-per-channel` 相关的局部辅助逻辑。关键符号：`_mm_srl_epi64`, `_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`。

### Lines 682-715 / 第 682-715 行

```c
0682:         const __m128i vi10 =
0683:             _mm_srl_epi64(_mm_loadl_epi64((const __m128i*)i10), vi_shift);
0684:         const __m128i vxi10 =
0685:             _mm_sub_epi16(_mm_unpacklo_epi8(vi10, vzero), vinput_zero_point);
0686:         const __m128i vk10 =
0687:             _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 24));
0688:         const __m128i vxk10 =
0689:             _mm_sub_epi16(
0690:                 _mm_unpacklo_epi8(vk10, vzero),
0691:                 _mm_unpacklo_epi8(vkernel_zero_point, vzero));
0692:         const __m128i vprod10_odd = _mm_mullo_epi16(vxi10, vxk10);
0693:         const __m128i vprod10_even = _mm_mulhi_epi16(vxi10, vxk10);
0694:         vacc_lo = _mm_add_epi32(
0695:             vacc_lo, _mm_unpacklo_epi16(vprod10_odd, vprod10_even));
0696:         vacc_hi = _mm_add_epi32(
0697:             vacc_hi, _mm_unpackhi_epi16(vprod10_odd, vprod10_even));
0698: 
0699:         const __m128i vi11 =
0700:             _mm_srl_epi64(_mm_loadl_epi64((const __m128i*)i11), vi_shift);
0701:         const __m128i vxi11 =
0702:             _mm_sub_epi16(_mm_unpacklo_epi8(vi11, vzero), vinput_zero_point);
0703:         const __m128i vk11 =
0704:             _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 32));
0705:         const __m128i vxk11 =
0706:             _mm_sub_epi16(
0707:                 _mm_unpacklo_epi8(vk11, vzero),
0708:                 _mm_unpacklo_epi8(vkernel_zero_point, vzero));
0709:         const __m128i vprod11_odd = _mm_mullo_epi16(vxi11, vxk11);
0710:         const __m128i vprod11_even = _mm_mulhi_epi16(vxi11, vxk11);
0711:         vacc_lo = _mm_add_epi32(
0712:             vacc_lo, _mm_unpacklo_epi16(vprod11_odd, vprod11_even));
0713:         vacc_hi = _mm_add_epi32(
0714:             vacc_hi, _mm_unpackhi_epi16(vprod11_odd, vprod11_even));
0715: 
```

- **EN:** This block implements local helper logic for `mp8x25-sse2-per-channel`. Key symbols: `_mm_srl_epi64`, `_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`.
- **CN:** 该代码块实现与 `mp8x25-sse2-per-channel` 相关的局部辅助逻辑。关键符号：`_mm_srl_epi64`, `_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`。

### Lines 716-749 / 第 716-749 行

```c
0716:         const __m128i vi12 =
0717:             _mm_srl_epi64(_mm_loadl_epi64((const __m128i*)i12), vi_shift);
0718:         const __m128i vxi12 =
0719:             _mm_sub_epi16(_mm_unpacklo_epi8(vi12, vzero), vinput_zero_point);
0720:         const __m128i vk12 =
0721:             _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 40));
0722:         const __m128i vxk12 =
0723:             _mm_sub_epi16(
0724:                 _mm_unpacklo_epi8(vk12, vzero),
0725:                 _mm_unpacklo_epi8(vkernel_zero_point, vzero));
0726:         const __m128i vprod12_odd = _mm_mullo_epi16(vxi12, vxk12);
0727:         const __m128i vprod12_even = _mm_mulhi_epi16(vxi12, vxk12);
0728:         vacc_lo = _mm_add_epi32(
0729:             vacc_lo, _mm_unpacklo_epi16(vprod12_odd, vprod12_even));
0730:         vacc_hi = _mm_add_epi32(
0731:             vacc_hi, _mm_unpackhi_epi16(vprod12_odd, vprod12_even));
0732: 
0733:         const __m128i vi20 =
0734:             _mm_srl_epi64(_mm_loadl_epi64((const __m128i*)i20), vi_shift);
0735:         const __m128i vxi20 =
0736:             _mm_sub_epi16(_mm_unpacklo_epi8(vi20, vzero), vinput_zero_point);
0737:         const __m128i vk20 =
0738:             _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 48));
0739:         const __m128i vxk20 =
0740:             _mm_sub_epi16(
0741:                 _mm_unpacklo_epi8(vk20, vzero),
0742:                 _mm_unpacklo_epi8(vkernel_zero_point, vzero));
0743:         const __m128i vprod20_odd = _mm_mullo_epi16(vxi20, vxk20);
0744:         const __m128i vprod20_even = _mm_mulhi_epi16(vxi20, vxk20);
0745:         vacc_lo = _mm_add_epi32(
0746:             vacc_lo, _mm_unpacklo_epi16(vprod20_odd, vprod20_even));
0747:         vacc_hi = _mm_add_epi32(
0748:             vacc_hi, _mm_unpackhi_epi16(vprod20_odd, vprod20_even));
0749: 
```

- **EN:** This block implements local helper logic for `mp8x25-sse2-per-channel`. Key symbols: `_mm_srl_epi64`, `_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`.
- **CN:** 该代码块实现与 `mp8x25-sse2-per-channel` 相关的局部辅助逻辑。关键符号：`_mm_srl_epi64`, `_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`。

### Lines 750-783 / 第 750-783 行

```c
0750:         const __m128i vi21 =
0751:             _mm_srl_epi64(_mm_loadl_epi64((const __m128i*)i21), vi_shift);
0752:         const __m128i vxi21 =
0753:             _mm_sub_epi16(_mm_unpacklo_epi8(vi21, vzero), vinput_zero_point);
0754:         const __m128i vk21 =
0755:             _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 56));
0756:         const __m128i vxk21 =
0757:             _mm_sub_epi16(
0758:                 _mm_unpacklo_epi8(vk21, vzero),
0759:                 _mm_unpacklo_epi8(vkernel_zero_point, vzero));
0760:         const __m128i vprod21_odd = _mm_mullo_epi16(vxi21, vxk21);
0761:         const __m128i vprod21_even = _mm_mulhi_epi16(vxi21, vxk21);
0762:         vacc_lo = _mm_add_epi32(
0763:             vacc_lo, _mm_unpacklo_epi16(vprod21_odd, vprod21_even));
0764:         vacc_hi = _mm_add_epi32(
0765:             vacc_hi, _mm_unpackhi_epi16(vprod21_odd, vprod21_even));
0766: 
0767:         const __m128i vi22 =
0768:             _mm_srl_epi64(_mm_loadl_epi64((const __m128i*)i22), vi_shift);
0769:         const __m128i vxi22 =
0770:             _mm_sub_epi16(_mm_unpacklo_epi8(vi22, vzero), vinput_zero_point);
0771:         const __m128i vk22 =
0772:             _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 64));
0773:         const __m128i vxk22 =
0774:             _mm_sub_epi16(
0775:                 _mm_unpacklo_epi8(vk22, vzero),
0776:                 _mm_unpacklo_epi8(vkernel_zero_point, vzero));
0777:         const __m128i vprod22_odd = _mm_mullo_epi16(vxi22, vxk22);
0778:         const __m128i vprod22_even = _mm_mulhi_epi16(vxi22, vxk22);
0779:         vacc_lo = _mm_add_epi32(
0780:             vacc_lo, _mm_unpacklo_epi16(vprod22_odd, vprod22_even));
0781:         vacc_hi = _mm_add_epi32(
0782:             vacc_hi, _mm_unpackhi_epi16(vprod22_odd, vprod22_even));
0783: 
```

- **EN:** This block implements local helper logic for `mp8x25-sse2-per-channel`. Key symbols: `_mm_srl_epi64`, `_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`.
- **CN:** 该代码块实现与 `mp8x25-sse2-per-channel` 相关的局部辅助逻辑。关键符号：`_mm_srl_epi64`, `_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`。

### Lines 784-819 / 第 784-819 行

```c
0784:         const __m128i vi23 =
0785:             _mm_srl_epi64(_mm_loadl_epi64((const __m128i*)i23), vi_shift);
0786:         const __m128i vxi23 =
0787:             _mm_sub_epi16(_mm_unpacklo_epi8(vi23, vzero), vinput_zero_point);
0788:         const __m128i vk23 =
0789:             _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 72));
0790:         const __m128i vxk23 =
0791:             _mm_sub_epi16(
0792:                 _mm_unpacklo_epi8(vk23, vzero),
0793:                 _mm_unpacklo_epi8(vkernel_zero_point, vzero));
0794:         const __m128i vprod23_odd = _mm_mullo_epi16(vxi23, vxk23);
0795:         const __m128i vprod23_even = _mm_mulhi_epi16(vxi23, vxk23);
0796:         vacc_lo = _mm_add_epi32(
0797:             vacc_lo, _mm_unpacklo_epi16(vprod23_odd, vprod23_even));
0798:         vacc_hi = _mm_add_epi32(
0799:             vacc_hi, _mm_unpackhi_epi16(vprod23_odd, vprod23_even));
0800: 
0801:         w = (const void*)((uintptr_t)w + 80);
0802:         vacc_lo = _mm_add_epi32(vacc_lo, _mm_loadu_si128((__m128i*)outacc));
0803:         vacc_hi =
0804:             _mm_add_epi32(vacc_hi, _mm_loadu_si128((__m128i*)(outacc + 4)));
0805:         _mm_storeu_si128((__m128i*)outacc, vacc_lo);
0806:         outacc += 4;
0807:         _mm_storeu_si128((__m128i*)outacc, vacc_hi);
0808:       }
0809:     }
0810:     {
0811:       const uint8_t* i00 = input[20];
0812:       const uint8_t* i01 = input[21];
0813:       const uint8_t* i02 = input[22];
0814:       const uint8_t* i10 = input[23];
0815:       const uint8_t* i11 = input[24];
0816:       input = (const uint8_t**)((uintptr_t)input + input_stride);
0817:       outacc = outacc32;
0818:       size_t c = channels;
0819:       for (; c >= 8; c -= 8) {
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `_mm_srl_epi64`, `_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`, `_mm_add_epi32`, `_mm_storeu_si128`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`_mm_srl_epi64`, `_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`, `_mm_add_epi32`, `_mm_storeu_si128`。

### Lines 820-853 / 第 820-853 行

```c
0820:         const __m128i vi00 = _mm_loadl_epi64((const __m128i*)i00);
0821:         const __m128i vkernel_zero_point = _mm_loadl_epi64(
0822:             (const __m128i*)
0823:             &quantization_params->sse2.kernel_zero_points[channels - c]);
0824:         i00 += 8;
0825:         const __m128i vxi00 =
0826:             _mm_sub_epi16(_mm_unpacklo_epi8(vi00, vzero), vinput_zero_point);
0827:         const __m128i vk00 = _mm_loadl_epi64((const __m128i*)((uintptr_t)w));
0828:         const __m128i vxk00 =
0829:             _mm_sub_epi16(
0830:                 _mm_unpacklo_epi8(vk00, vzero),
0831:                 _mm_unpacklo_epi8(vkernel_zero_point, vzero));
0832:         const __m128i vprod00_odd = _mm_mullo_epi16(vxi00, vxk00);
0833:         const __m128i vprod00_even = _mm_mulhi_epi16(vxi00, vxk00);
0834:         __m128i vacc_lo = _mm_unpacklo_epi16(vprod00_odd, vprod00_even);
0835:         __m128i vacc_hi = _mm_unpackhi_epi16(vprod00_odd, vprod00_even);
0836: 
0837:         const __m128i vi01 = _mm_loadl_epi64((const __m128i*)i01);
0838:         i01 += 8;
0839:         const __m128i vxi01 =
0840:             _mm_sub_epi16(_mm_unpacklo_epi8(vi01, vzero), vinput_zero_point);
0841:         const __m128i vk01 =
0842:             _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 8));
0843:         const __m128i vxk01 =
0844:             _mm_sub_epi16(
0845:                 _mm_unpacklo_epi8(vk01, vzero),
0846:                 _mm_unpacklo_epi8(vkernel_zero_point, vzero));
0847:         const __m128i vprod01_odd = _mm_mullo_epi16(vxi01, vxk01);
0848:         const __m128i vprod01_even = _mm_mulhi_epi16(vxi01, vxk01);
0849:         vacc_lo = _mm_add_epi32(
0850:             vacc_lo, _mm_unpacklo_epi16(vprod01_odd, vprod01_even));
0851:         vacc_hi = _mm_add_epi32(
0852:             vacc_hi, _mm_unpackhi_epi16(vprod01_odd, vprod01_even));
0853: 
```

- **EN:** This block implements local helper logic for `mp8x25-sse2-per-channel`. Key symbols: `_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`.
- **CN:** 该代码块实现与 `mp8x25-sse2-per-channel` 相关的局部辅助逻辑。关键符号：`_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`。

### Lines 854-887 / 第 854-887 行

```c
0854:         const __m128i vi02 = _mm_loadl_epi64((const __m128i*)i02);
0855:         i02 += 8;
0856:         const __m128i vxi02 =
0857:             _mm_sub_epi16(_mm_unpacklo_epi8(vi02, vzero), vinput_zero_point);
0858:         const __m128i vk02 =
0859:             _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 16));
0860:         const __m128i vxk02 =
0861:             _mm_sub_epi16(
0862:                 _mm_unpacklo_epi8(vk02, vzero),
0863:                 _mm_unpacklo_epi8(vkernel_zero_point, vzero));
0864:         const __m128i vprod02_odd = _mm_mullo_epi16(vxi02, vxk02);
0865:         const __m128i vprod02_even = _mm_mulhi_epi16(vxi02, vxk02);
0866:         vacc_lo = _mm_add_epi32(
0867:             vacc_lo, _mm_unpacklo_epi16(vprod02_odd, vprod02_even));
0868:         vacc_hi = _mm_add_epi32(
0869:             vacc_hi, _mm_unpackhi_epi16(vprod02_odd, vprod02_even));
0870: 
0871:         const __m128i vi10 = _mm_loadl_epi64((const __m128i*)i10);
0872:         i10 += 8;
0873:         const __m128i vxi10 =
0874:             _mm_sub_epi16(_mm_unpacklo_epi8(vi10, vzero), vinput_zero_point);
0875:         const __m128i vk10 =
0876:             _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 24));
0877:         const __m128i vxk10 =
0878:             _mm_sub_epi16(
0879:                 _mm_unpacklo_epi8(vk10, vzero),
0880:                 _mm_unpacklo_epi8(vkernel_zero_point, vzero));
0881:         const __m128i vprod10_odd = _mm_mullo_epi16(vxi10, vxk10);
0882:         const __m128i vprod10_even = _mm_mulhi_epi16(vxi10, vxk10);
0883:         vacc_lo = _mm_add_epi32(
0884:             vacc_lo, _mm_unpacklo_epi16(vprod10_odd, vprod10_even));
0885:         vacc_hi = _mm_add_epi32(
0886:             vacc_hi, _mm_unpackhi_epi16(vprod10_odd, vprod10_even));
0887: 
```

- **EN:** This block implements local helper logic for `mp8x25-sse2-per-channel`. Key symbols: `_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`.
- **CN:** 该代码块实现与 `mp8x25-sse2-per-channel` 相关的局部辅助逻辑。关键符号：`_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`。

### Lines 888-929 / 第 888-929 行

```c
0888:         const __m128i vi11 = _mm_loadl_epi64((const __m128i*)i11);
0889:         i11 += 8;
0890:         const __m128i vxi11 =
0891:             _mm_sub_epi16(_mm_unpacklo_epi8(vi11, vzero), vinput_zero_point);
0892:         const __m128i vk11 =
0893:             _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 32));
0894:         const __m128i vxk11 =
0895:             _mm_sub_epi16(
0896:                 _mm_unpacklo_epi8(vk11, vzero),
0897:                 _mm_unpacklo_epi8(vkernel_zero_point, vzero));
0898:         const __m128i vprod11_odd = _mm_mullo_epi16(vxi11, vxk11);
0899:         const __m128i vprod11_even = _mm_mulhi_epi16(vxi11, vxk11);
0900:         vacc_lo = _mm_add_epi32(
0901:             vacc_lo, _mm_unpacklo_epi16(vprod11_odd, vprod11_even));
0902:         vacc_hi = _mm_add_epi32(
0903:             vacc_hi, _mm_unpackhi_epi16(vprod11_odd, vprod11_even));
0904: 
0905:         w = (const void*)((uintptr_t)w + 40);
0906: 
0907:         vacc_lo = _mm_add_epi32(vacc_lo, _mm_loadu_si128((__m128i*)outacc));
0908:         vacc_hi =
0909:             _mm_add_epi32(vacc_hi, _mm_loadu_si128((__m128i*)(outacc + 4)));
0910:         outacc += 8;
0911: 
0912:         const __m128 vmultiplier_lo =
0913:             _mm_loadu_ps(&quantization_params->sse2.requantization_scales[channels - c]);
0914:         const __m128 vmultiplier_hi =
0915:             _mm_loadu_ps(&quantization_params->sse2.requantization_scales[channels - c + 4]);
0916: 
0917:         vacc_lo = _mm_cvtps_epi32(
0918:                       _mm_mul_ps(
0919:                         _mm_cvtepi32_ps(vacc_lo),
0920:                         vmultiplier_lo
0921:                         )
0922:                       );
0923:         vacc_hi = _mm_cvtps_epi32(
0924:                       _mm_mul_ps(
0925:                         _mm_cvtepi32_ps(vacc_hi),
0926:                         vmultiplier_hi
0927:                         )
0928:                       );
0929: 
```

- **EN:** This block implements local helper logic for `mp8x25-sse2-per-channel`. Key symbols: `_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`, `_mm_add_epi32`, `_mm_loadu_ps`, `_mm_mul_ps`.
- **CN:** 该代码块实现与 `mp8x25-sse2-per-channel` 相关的局部辅助逻辑。关键符号：`_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`, `_mm_add_epi32`, `_mm_loadu_ps`, `_mm_mul_ps`。

### Lines 930-972 / 第 930-972 行

```c
0930:         const __m128i voutput_zero_point = _mm_load_si128(
0931:             (const __m128i*)quantization_params->sse2.output_zero_point);
0932:         __m128i vout = _mm_adds_epi16(
0933:             _mm_packs_epi32(vacc_lo, vacc_hi), voutput_zero_point);
0934:         vout = _mm_packus_epi16(vout, vout);
0935:         vout = _mm_max_epu8(
0936:             vout,
0937:             _mm_load_si128(
0938:                 (const __m128i*)quantization_params->sse2.output_min));
0939:         vout = _mm_min_epu8(
0940:             vout,
0941:             _mm_load_si128(
0942:                 (const __m128i*)quantization_params->sse2.output_max));
0943: 
0944:         _mm_storel_epi64((__m128i*)output, vout);
0945:         output += 8;
0946:       }
0947:       if (c != 0) {
0948:         const size_t i_predecrement = 8 - c;
0949:         const __m128i vi_shift = _mm_cvtsi32_si128(8 * i_predecrement);
0950:         const __m128i vkernel_zero_point = _mm_loadl_epi64(
0951:             (const __m128i*)
0952:             &quantization_params->sse2.kernel_zero_points[channels - c]);
0953:         i00 -= i_predecrement;
0954:         i01 -= i_predecrement;
0955:         i02 -= i_predecrement;
0956:         i10 -= i_predecrement;
0957:         i11 -= i_predecrement;
0958: 
0959:         const __m128i vi00 =
0960:             _mm_srl_epi64(_mm_loadl_epi64((const __m128i*)i00), vi_shift);
0961:         const __m128i vxi00 =
0962:             _mm_sub_epi16(_mm_unpacklo_epi8(vi00, vzero), vinput_zero_point);
0963:         const __m128i vk00 = _mm_loadl_epi64((const __m128i*)((uintptr_t)w));
0964:         const __m128i vxk00 =
0965:             _mm_sub_epi16(
0966:                 _mm_unpacklo_epi8(vk00, vzero),
0967:                 _mm_unpacklo_epi8(vkernel_zero_point, vzero));
0968:         const __m128i vprod00_odd = _mm_mullo_epi16(vxi00, vxk00);
0969:         const __m128i vprod00_even = _mm_mulhi_epi16(vxi00, vxk00);
0970:         __m128i vacc_lo = _mm_unpacklo_epi16(vprod00_odd, vprod00_even);
0971:         __m128i vacc_hi = _mm_unpackhi_epi16(vprod00_odd, vprod00_even);
0972: 
```

- **EN:** This block handles conditional branches and special cases. Key symbols: `_mm_packs_epi32`, `_mm_load_si128`, `_mm_storel_epi64`, `_mm_srl_epi64`, `_mm_sub_epi16`.
- **CN:** 该代码块处理条件分支与特殊情况。关键符号：`_mm_packs_epi32`, `_mm_load_si128`, `_mm_storel_epi64`, `_mm_srl_epi64`, `_mm_sub_epi16`。

### Lines 973-1006 / 第 973-1006 行

```c
0973:         const __m128i vi01 =
0974:             _mm_srl_epi64(_mm_loadl_epi64((const __m128i*)i01), vi_shift);
0975:         const __m128i vxi01 =
0976:             _mm_sub_epi16(_mm_unpacklo_epi8(vi01, vzero), vinput_zero_point);
0977:         const __m128i vk01 =
0978:             _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 8));
0979:         const __m128i vxk01 =
0980:             _mm_sub_epi16(
0981:                 _mm_unpacklo_epi8(vk01, vzero),
0982:                 _mm_unpacklo_epi8(vkernel_zero_point, vzero));
0983:         const __m128i vprod01_odd = _mm_mullo_epi16(vxi01, vxk01);
0984:         const __m128i vprod01_even = _mm_mulhi_epi16(vxi01, vxk01);
0985:         vacc_lo = _mm_add_epi32(
0986:             vacc_lo, _mm_unpacklo_epi16(vprod01_odd, vprod01_even));
0987:         vacc_hi = _mm_add_epi32(
0988:             vacc_hi, _mm_unpackhi_epi16(vprod01_odd, vprod01_even));
0989: 
0990:         const __m128i vi02 =
0991:             _mm_srl_epi64(_mm_loadl_epi64((const __m128i*)i02), vi_shift);
0992:         const __m128i vxi02 =
0993:             _mm_sub_epi16(_mm_unpacklo_epi8(vi02, vzero), vinput_zero_point);
0994:         const __m128i vk02 =
0995:             _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 16));
0996:         const __m128i vxk02 =
0997:             _mm_sub_epi16(
0998:                 _mm_unpacklo_epi8(vk02, vzero),
0999:                 _mm_unpacklo_epi8(vkernel_zero_point, vzero));
1000:         const __m128i vprod02_odd = _mm_mullo_epi16(vxi02, vxk02);
1001:         const __m128i vprod02_even = _mm_mulhi_epi16(vxi02, vxk02);
1002:         vacc_lo = _mm_add_epi32(
1003:             vacc_lo, _mm_unpacklo_epi16(vprod02_odd, vprod02_even));
1004:         vacc_hi = _mm_add_epi32(
1005:             vacc_hi, _mm_unpackhi_epi16(vprod02_odd, vprod02_even));
1006: 
```

- **EN:** This block implements local helper logic for `mp8x25-sse2-per-channel`. Key symbols: `_mm_srl_epi64`, `_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`.
- **CN:** 该代码块实现与 `mp8x25-sse2-per-channel` 相关的局部辅助逻辑。关键符号：`_mm_srl_epi64`, `_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`。

### Lines 1007-1040 / 第 1007-1040 行

```c
1007:         const __m128i vi10 =
1008:             _mm_srl_epi64(_mm_loadl_epi64((const __m128i*)i10), vi_shift);
1009:         const __m128i vxi10 =
1010:             _mm_sub_epi16(_mm_unpacklo_epi8(vi10, vzero), vinput_zero_point);
1011:         const __m128i vk10 =
1012:             _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 24));
1013:         const __m128i vxk10 =
1014:             _mm_sub_epi16(
1015:                 _mm_unpacklo_epi8(vk10, vzero),
1016:                 _mm_unpacklo_epi8(vkernel_zero_point, vzero));
1017:         const __m128i vprod10_odd = _mm_mullo_epi16(vxi10, vxk10);
1018:         const __m128i vprod10_even = _mm_mulhi_epi16(vxi10, vxk10);
1019:         vacc_lo = _mm_add_epi32(
1020:             vacc_lo, _mm_unpacklo_epi16(vprod10_odd, vprod10_even));
1021:         vacc_hi = _mm_add_epi32(
1022:             vacc_hi, _mm_unpackhi_epi16(vprod10_odd, vprod10_even));
1023: 
1024:         const __m128i vi11 =
1025:             _mm_srl_epi64(_mm_loadl_epi64((const __m128i*)i11), vi_shift);
1026:         const __m128i vxi11 =
1027:             _mm_sub_epi16(_mm_unpacklo_epi8(vi11, vzero), vinput_zero_point);
1028:         const __m128i vk11 =
1029:             _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 32));
1030:         const __m128i vxk11 =
1031:             _mm_sub_epi16(
1032:                 _mm_unpacklo_epi8(vk11, vzero),
1033:                 _mm_unpacklo_epi8(vkernel_zero_point, vzero));
1034:         const __m128i vprod11_odd = _mm_mullo_epi16(vxi11, vxk11);
1035:         const __m128i vprod11_even = _mm_mulhi_epi16(vxi11, vxk11);
1036:         vacc_lo = _mm_add_epi32(
1037:             vacc_lo, _mm_unpacklo_epi16(vprod11_odd, vprod11_even));
1038:         vacc_hi = _mm_add_epi32(
1039:             vacc_hi, _mm_unpackhi_epi16(vprod11_odd, vprod11_even));
1040: 
```

- **EN:** This block implements local helper logic for `mp8x25-sse2-per-channel`. Key symbols: `_mm_srl_epi64`, `_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`.
- **CN:** 该代码块实现与 `mp8x25-sse2-per-channel` 相关的局部辅助逻辑。关键符号：`_mm_srl_epi64`, `_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`。

### Lines 1041-1076 / 第 1041-1076 行

```c
1041:         vacc_lo = _mm_add_epi32(vacc_lo, _mm_loadu_si128((__m128i*)outacc));
1042:         vacc_hi =
1043:             _mm_add_epi32(vacc_hi, _mm_loadu_si128((__m128i*)(outacc + 4)));
1044: 
1045:         const __m128 vmultiplier_lo =
1046:             _mm_loadu_ps(&quantization_params->sse2.requantization_scales[channels - c]);
1047:         const __m128 vmultiplier_hi =
1048:             _mm_loadu_ps(&quantization_params->sse2.requantization_scales[channels - c + 4]);
1049: 
1050:         vacc_lo = _mm_cvtps_epi32(
1051:                       _mm_mul_ps(
1052:                         _mm_cvtepi32_ps(vacc_lo),
1053:                         vmultiplier_lo
1054:                         )
1055:                       );
1056:         vacc_hi = _mm_cvtps_epi32(
1057:                       _mm_mul_ps(
1058:                         _mm_cvtepi32_ps(vacc_hi),
1059:                         vmultiplier_hi
1060:                         )
1061:                       );
1062: 
1063:         const __m128i voutput_zero_point = _mm_load_si128(
1064:             (const __m128i*)quantization_params->sse2.output_zero_point);
1065:         __m128i vout = _mm_adds_epi16(
1066:             _mm_packs_epi32(vacc_lo, vacc_hi), voutput_zero_point);
1067:         vout = _mm_packus_epi16(vout, vout);
1068:         vout = _mm_max_epu8(
1069:             vout,
1070:             _mm_load_si128(
1071:                 (const __m128i*)quantization_params->sse2.output_min));
1072:         vout = _mm_min_epu8(
1073:             vout,
1074:             _mm_load_si128(
1075:                 (const __m128i*)quantization_params->sse2.output_max));
1076: 
```

- **EN:** This block implements local helper logic for `mp8x25-sse2-per-channel`. Key symbols: `_mm_add_epi32`, `_mm_loadu_ps`, `_mm_mul_ps`, `_mm_packs_epi32`, `_mm_load_si128`.
- **CN:** 该代码块实现与 `mp8x25-sse2-per-channel` 相关的局部辅助逻辑。关键符号：`_mm_add_epi32`, `_mm_loadu_ps`, `_mm_mul_ps`, `_mm_packs_epi32`, `_mm_load_si128`。

### Lines 1077-1095 / 第 1077-1095 行

```c
1077:         if (c & 4) {
1078:           *((uint32_t*)output) = (uint32_t)_mm_cvtsi128_si32(vout);
1079:           output += 4;
1080:           vout = _mm_srli_epi64(vout, 32);
1081:         }
1082:         if (c & 2) {
1083:           *((uint16_t*)output) = (uint16_t)_mm_extract_epi16(vout, 0);
1084:           output += 2;
1085:           vout = _mm_srli_epi32(vout, 16);
1086:         }
1087:         if (c & 1) {
1088:           *((uint8_t*)output) = (uint8_t)_mm_cvtsi128_si32(vout);
1089:           output += 1;
1090:         }
1091:       }
1092:     }
1093:     output = (uint8_t*)((uintptr_t)output + output_increment);
1094:   } while (--output_width != 0);
1095: }
```

- **EN:** This block handles conditional branches and special cases; iterates over tensors, descriptors, options, or runtime state. Key symbols: no prominent local symbols.
- **CN:** 该代码块处理条件分支与特殊情况；遍历张量、描述符、配置项或运行时状态。关键符号：无明显局部符号。


## Key Concepts / 关键概念
- **ATen native operator logic** — ATen 原生算子逻辑
- **GPU kernel/runtime interaction** — GPU 内核/运行时交互
- **Core symbols: pytorch_q8dwconv_ukernel_mp8x25_per_channel__sse2, _mm_sub_epi16, _mm_loadl_epi64, _mm_unpacklo_epi16, _mm_unpackhi_epi16, _mm_storeu_si128, _mm_srl_epi64, _mm_add_epi32** — 核心符号：pytorch_q8dwconv_ukernel_mp8x25_per_channel__sse2、_mm_sub_epi16、_mm_loadl_epi64、_mm_unpacklo_epi16、_mm_unpackhi_epi16、_mm_storeu_si128、_mm_srl_epi64、_mm_add_epi32

## Dependencies / 依赖关系
- **Internal includes / 内部头文件**: 无
- **External includes / 外部头文件**: `immintrin.h`, `qnnpack/q8dwconv.h`
- **Namespaces / 命名空间**: 无
- **Representative symbols / 代表性符号**: `pytorch_q8dwconv_ukernel_mp8x25_per_channel__sse2`, `_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`, `_mm_storeu_si128`, `_mm_srl_epi64`, `_mm_add_epi32`, `_mm_loadu_ps`, `_mm_mul_ps`, `_mm_packs_epi32`, `_mm_load_si128`, `...`
