# mp8x25-sse2.c — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `aten/src/ATen/native/quantized/cpu/qnnpack/src/q8dwconv/mp8x25-sse2.c`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements ATen native operators, checks, helper kernels, and backend dispatch behavior. This specific file centers on `mp8x25-sse2.c`. The file header highlights: "All rights reserved. This source code is licensed under the BSD-style license found in the LICENSE file in the root directory of this source tree.."
- **Purpose (CN)**: 实现 ATen 原生算子、运行时检查、辅助内核以及后端分发行为。 该文件具体围绕 `mp8x25-sse2.c` 展开。 文件头部注释也概括了其核心职责。

## Line-by-Line Analysis / 逐行分析
### Lines 1-44 / 第 1-44 行

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
0013: void pytorch_q8dwconv_ukernel_mp8x25__sse2(
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
0026:   const __m128i vkernel_zero_point = _mm_set1_epi16(
0027:       quantization_params->sse2.kernel_zero_points[0]);
0028:   const __m128i vzero = _mm_setzero_si128();
0029: 
0030:   do {
0031:     int32_t* outacc = outacc32;
0032:     const void* w = weights;
0033:     {
0034:       const uint8_t* i00 = input[0];
0035:       const uint8_t* i01 = input[1];
0036:       const uint8_t* i02 = input[2];
0037:       const uint8_t* i10 = input[3];
0038:       const uint8_t* i11 = input[4];
0039:       const uint8_t* i12 = input[5];
0040:       const uint8_t* i20 = input[6];
0041:       const uint8_t* i21 = input[7];
0042:       const uint8_t* i22 = input[8];
0043:       const uint8_t* i23 = input[9];
0044: 
```

- **EN:** This block implements local helper logic for `mp8x25-sse2`. Key symbols: `pytorch_q8dwconv_ukernel_mp8x25__sse2`.
- **CN:** 该代码块实现与 `mp8x25-sse2` 相关的局部辅助逻辑。关键符号：`pytorch_q8dwconv_ukernel_mp8x25__sse2`。

### Lines 45-79 / 第 45-79 行

```c
0045:       size_t c = channels;
0046:       for (; c >= 8; c -= 8) {
0047:         __m128i vacc_lo = _mm_loadu_si128((const __m128i*)w);
0048:         __m128i vacc_hi = _mm_loadu_si128((const __m128i*)((uintptr_t)w + 16));
0049: 
0050:         const __m128i vi00 = _mm_loadl_epi64((const __m128i*)i00);
0051:         i00 += 8;
0052:         const __m128i vxi00 =
0053:             _mm_sub_epi16(_mm_unpacklo_epi8(vi00, vzero), vinput_zero_point);
0054:         const __m128i vk00 =
0055:             _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 32));
0056:         const __m128i vxk00 =
0057:             _mm_sub_epi16(_mm_unpacklo_epi8(vk00, vzero), vkernel_zero_point);
0058:         const __m128i vprod00_odd = _mm_mullo_epi16(vxi00, vxk00);
0059:         const __m128i vprod00_even = _mm_mulhi_epi16(vxi00, vxk00);
0060:         vacc_lo = _mm_add_epi32(
0061:             vacc_lo, _mm_unpacklo_epi16(vprod00_odd, vprod00_even));
0062:         vacc_hi = _mm_add_epi32(
0063:             vacc_hi, _mm_unpackhi_epi16(vprod00_odd, vprod00_even));
0064: 
0065:         const __m128i vi01 = _mm_loadl_epi64((const __m128i*)i01);
0066:         i01 += 8;
0067:         const __m128i vxi01 =
0068:             _mm_sub_epi16(_mm_unpacklo_epi8(vi01, vzero), vinput_zero_point);
0069:         const __m128i vk01 =
0070:             _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 40));
0071:         const __m128i vxk01 =
0072:             _mm_sub_epi16(_mm_unpacklo_epi8(vk01, vzero), vkernel_zero_point);
0073:         const __m128i vprod01_odd = _mm_mullo_epi16(vxi01, vxk01);
0074:         const __m128i vprod01_even = _mm_mulhi_epi16(vxi01, vxk01);
0075:         vacc_lo = _mm_add_epi32(
0076:             vacc_lo, _mm_unpacklo_epi16(vprod01_odd, vprod01_even));
0077:         vacc_hi = _mm_add_epi32(
0078:             vacc_hi, _mm_unpackhi_epi16(vprod01_odd, vprod01_even));
0079: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`。

### Lines 80-109 / 第 80-109 行

```c
0080:         const __m128i vi02 = _mm_loadl_epi64((const __m128i*)i02);
0081:         i02 += 8;
0082:         const __m128i vxi02 =
0083:             _mm_sub_epi16(_mm_unpacklo_epi8(vi02, vzero), vinput_zero_point);
0084:         const __m128i vk02 =
0085:             _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 48));
0086:         const __m128i vxk02 =
0087:             _mm_sub_epi16(_mm_unpacklo_epi8(vk02, vzero), vkernel_zero_point);
0088:         const __m128i vprod02_odd = _mm_mullo_epi16(vxi02, vxk02);
0089:         const __m128i vprod02_even = _mm_mulhi_epi16(vxi02, vxk02);
0090:         vacc_lo = _mm_add_epi32(
0091:             vacc_lo, _mm_unpacklo_epi16(vprod02_odd, vprod02_even));
0092:         vacc_hi = _mm_add_epi32(
0093:             vacc_hi, _mm_unpackhi_epi16(vprod02_odd, vprod02_even));
0094: 
0095:         const __m128i vi10 = _mm_loadl_epi64((const __m128i*)i10);
0096:         i10 += 8;
0097:         const __m128i vxi10 =
0098:             _mm_sub_epi16(_mm_unpacklo_epi8(vi10, vzero), vinput_zero_point);
0099:         const __m128i vk10 =
0100:             _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 56));
0101:         const __m128i vxk10 =
0102:             _mm_sub_epi16(_mm_unpacklo_epi8(vk10, vzero), vkernel_zero_point);
0103:         const __m128i vprod10_odd = _mm_mullo_epi16(vxi10, vxk10);
0104:         const __m128i vprod10_even = _mm_mulhi_epi16(vxi10, vxk10);
0105:         vacc_lo = _mm_add_epi32(
0106:             vacc_lo, _mm_unpacklo_epi16(vprod10_odd, vprod10_even));
0107:         vacc_hi = _mm_add_epi32(
0108:             vacc_hi, _mm_unpackhi_epi16(vprod10_odd, vprod10_even));
0109: 
```

- **EN:** This block implements local helper logic for `mp8x25-sse2`. Key symbols: `_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`.
- **CN:** 该代码块实现与 `mp8x25-sse2` 相关的局部辅助逻辑。关键符号：`_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`。

### Lines 110-139 / 第 110-139 行

```c
0110:         const __m128i vi11 = _mm_loadl_epi64((const __m128i*)i11);
0111:         i11 += 8;
0112:         const __m128i vxi11 =
0113:             _mm_sub_epi16(_mm_unpacklo_epi8(vi11, vzero), vinput_zero_point);
0114:         const __m128i vk11 =
0115:             _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 64));
0116:         const __m128i vxk11 =
0117:             _mm_sub_epi16(_mm_unpacklo_epi8(vk11, vzero), vkernel_zero_point);
0118:         const __m128i vprod11_odd = _mm_mullo_epi16(vxi11, vxk11);
0119:         const __m128i vprod11_even = _mm_mulhi_epi16(vxi11, vxk11);
0120:         vacc_lo = _mm_add_epi32(
0121:             vacc_lo, _mm_unpacklo_epi16(vprod11_odd, vprod11_even));
0122:         vacc_hi = _mm_add_epi32(
0123:             vacc_hi, _mm_unpackhi_epi16(vprod11_odd, vprod11_even));
0124: 
0125:         const __m128i vi12 = _mm_loadl_epi64((const __m128i*)i12);
0126:         i12 += 8;
0127:         const __m128i vxi12 =
0128:             _mm_sub_epi16(_mm_unpacklo_epi8(vi12, vzero), vinput_zero_point);
0129:         const __m128i vk12 =
0130:             _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 72));
0131:         const __m128i vxk12 =
0132:             _mm_sub_epi16(_mm_unpacklo_epi8(vk12, vzero), vkernel_zero_point);
0133:         const __m128i vprod12_odd = _mm_mullo_epi16(vxi12, vxk12);
0134:         const __m128i vprod12_even = _mm_mulhi_epi16(vxi12, vxk12);
0135:         vacc_lo = _mm_add_epi32(
0136:             vacc_lo, _mm_unpacklo_epi16(vprod12_odd, vprod12_even));
0137:         vacc_hi = _mm_add_epi32(
0138:             vacc_hi, _mm_unpackhi_epi16(vprod12_odd, vprod12_even));
0139: 
```

- **EN:** This block implements local helper logic for `mp8x25-sse2`. Key symbols: `_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`.
- **CN:** 该代码块实现与 `mp8x25-sse2` 相关的局部辅助逻辑。关键符号：`_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`。

### Lines 140-169 / 第 140-169 行

```c
0140:         const __m128i vi20 = _mm_loadl_epi64((const __m128i*)i20);
0141:         i20 += 8;
0142:         const __m128i vxi20 =
0143:             _mm_sub_epi16(_mm_unpacklo_epi8(vi20, vzero), vinput_zero_point);
0144:         const __m128i vk20 =
0145:             _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 80));
0146:         const __m128i vxk20 =
0147:             _mm_sub_epi16(_mm_unpacklo_epi8(vk20, vzero), vkernel_zero_point);
0148:         const __m128i vprod20_odd = _mm_mullo_epi16(vxi20, vxk20);
0149:         const __m128i vprod20_even = _mm_mulhi_epi16(vxi20, vxk20);
0150:         vacc_lo = _mm_add_epi32(
0151:             vacc_lo, _mm_unpacklo_epi16(vprod20_odd, vprod20_even));
0152:         vacc_hi = _mm_add_epi32(
0153:             vacc_hi, _mm_unpackhi_epi16(vprod20_odd, vprod20_even));
0154: 
0155:         const __m128i vi21 = _mm_loadl_epi64((const __m128i*)i21);
0156:         i21 += 8;
0157:         const __m128i vxi21 =
0158:             _mm_sub_epi16(_mm_unpacklo_epi8(vi21, vzero), vinput_zero_point);
0159:         const __m128i vk21 =
0160:             _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 88));
0161:         const __m128i vxk21 =
0162:             _mm_sub_epi16(_mm_unpacklo_epi8(vk21, vzero), vkernel_zero_point);
0163:         const __m128i vprod21_odd = _mm_mullo_epi16(vxi21, vxk21);
0164:         const __m128i vprod21_even = _mm_mulhi_epi16(vxi21, vxk21);
0165:         vacc_lo = _mm_add_epi32(
0166:             vacc_lo, _mm_unpacklo_epi16(vprod21_odd, vprod21_even));
0167:         vacc_hi = _mm_add_epi32(
0168:             vacc_hi, _mm_unpackhi_epi16(vprod21_odd, vprod21_even));
0169: 
```

- **EN:** This block implements local helper logic for `mp8x25-sse2`. Key symbols: `_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`.
- **CN:** 该代码块实现与 `mp8x25-sse2` 相关的局部辅助逻辑。关键符号：`_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`。

### Lines 170-199 / 第 170-199 行

```c
0170:         const __m128i vi22 = _mm_loadl_epi64((const __m128i*)i22);
0171:         i22 += 8;
0172:         const __m128i vxi22 =
0173:             _mm_sub_epi16(_mm_unpacklo_epi8(vi22, vzero), vinput_zero_point);
0174:         const __m128i vk22 =
0175:             _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 96));
0176:         const __m128i vxk22 =
0177:             _mm_sub_epi16(_mm_unpacklo_epi8(vk22, vzero), vkernel_zero_point);
0178:         const __m128i vprod22_odd = _mm_mullo_epi16(vxi22, vxk22);
0179:         const __m128i vprod22_even = _mm_mulhi_epi16(vxi22, vxk22);
0180:         vacc_lo = _mm_add_epi32(
0181:             vacc_lo, _mm_unpacklo_epi16(vprod22_odd, vprod22_even));
0182:         vacc_hi = _mm_add_epi32(
0183:             vacc_hi, _mm_unpackhi_epi16(vprod22_odd, vprod22_even));
0184: 
0185:         const __m128i vi23 = _mm_loadl_epi64((const __m128i*)i23);
0186:         i23 += 8;
0187:         const __m128i vxi23 =
0188:             _mm_sub_epi16(_mm_unpacklo_epi8(vi23, vzero), vinput_zero_point);
0189:         const __m128i vk23 =
0190:             _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 104));
0191:         const __m128i vxk23 =
0192:             _mm_sub_epi16(_mm_unpacklo_epi8(vk23, vzero), vkernel_zero_point);
0193:         const __m128i vprod23_odd = _mm_mullo_epi16(vxi23, vxk23);
0194:         const __m128i vprod23_even = _mm_mulhi_epi16(vxi23, vxk23);
0195:         vacc_lo = _mm_add_epi32(
0196:             vacc_lo, _mm_unpacklo_epi16(vprod23_odd, vprod23_even));
0197:         vacc_hi = _mm_add_epi32(
0198:             vacc_hi, _mm_unpackhi_epi16(vprod23_odd, vprod23_even));
0199: 
```

- **EN:** This block implements local helper logic for `mp8x25-sse2`. Key symbols: `_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`.
- **CN:** 该代码块实现与 `mp8x25-sse2` 相关的局部辅助逻辑。关键符号：`_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`。

### Lines 200-237 / 第 200-237 行

```c
0200:         w = (const void*)((uintptr_t)w + 112);
0201:         _mm_storeu_si128((__m128i*)outacc, vacc_lo);
0202:         outacc += 4;
0203:         _mm_storeu_si128((__m128i*)outacc, vacc_hi);
0204:         outacc += 4;
0205:       }
0206:       if (c != 0) {
0207:         const size_t i_predecrement = 8 - c;
0208:         const __m128i vi_shift = _mm_cvtsi32_si128(8 * i_predecrement);
0209:         i00 -= i_predecrement;
0210:         i01 -= i_predecrement;
0211:         i02 -= i_predecrement;
0212:         i10 -= i_predecrement;
0213:         i11 -= i_predecrement;
0214:         i12 -= i_predecrement;
0215:         i20 -= i_predecrement;
0216:         i21 -= i_predecrement;
0217:         i22 -= i_predecrement;
0218:         i23 -= i_predecrement;
0219: 
0220:         __m128i vacc_lo = _mm_loadu_si128((const __m128i*)w);
0221:         __m128i vacc_hi = _mm_loadu_si128((const __m128i*)((uintptr_t)w + 16));
0222: 
0223:         const __m128i vi00 =
0224:             _mm_srl_epi64(_mm_loadl_epi64((const __m128i*)i00), vi_shift);
0225:         const __m128i vxi00 =
0226:             _mm_sub_epi16(_mm_unpacklo_epi8(vi00, vzero), vinput_zero_point);
0227:         const __m128i vk00 =
0228:             _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 32));
0229:         const __m128i vxk00 =
0230:             _mm_sub_epi16(_mm_unpacklo_epi8(vk00, vzero), vkernel_zero_point);
0231:         const __m128i vprod00_odd = _mm_mullo_epi16(vxi00, vxk00);
0232:         const __m128i vprod00_even = _mm_mulhi_epi16(vxi00, vxk00);
0233:         vacc_lo = _mm_add_epi32(
0234:             vacc_lo, _mm_unpacklo_epi16(vprod00_odd, vprod00_even));
0235:         vacc_hi = _mm_add_epi32(
0236:             vacc_hi, _mm_unpackhi_epi16(vprod00_odd, vprod00_even));
0237: 
```

- **EN:** This block handles conditional branches and special cases. Key symbols: `_mm_storeu_si128`, `_mm_srl_epi64`, `_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`.
- **CN:** 该代码块处理条件分支与特殊情况。关键符号：`_mm_storeu_si128`, `_mm_srl_epi64`, `_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`。

### Lines 238-267 / 第 238-267 行

```c
0238:         const __m128i vi01 =
0239:             _mm_srl_epi64(_mm_loadl_epi64((const __m128i*)i01), vi_shift);
0240:         const __m128i vxi01 =
0241:             _mm_sub_epi16(_mm_unpacklo_epi8(vi01, vzero), vinput_zero_point);
0242:         const __m128i vk01 =
0243:             _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 40));
0244:         const __m128i vxk01 =
0245:             _mm_sub_epi16(_mm_unpacklo_epi8(vk01, vzero), vkernel_zero_point);
0246:         const __m128i vprod01_odd = _mm_mullo_epi16(vxi01, vxk01);
0247:         const __m128i vprod01_even = _mm_mulhi_epi16(vxi01, vxk01);
0248:         vacc_lo = _mm_add_epi32(
0249:             vacc_lo, _mm_unpacklo_epi16(vprod01_odd, vprod01_even));
0250:         vacc_hi = _mm_add_epi32(
0251:             vacc_hi, _mm_unpackhi_epi16(vprod01_odd, vprod01_even));
0252: 
0253:         const __m128i vi02 =
0254:             _mm_srl_epi64(_mm_loadl_epi64((const __m128i*)i02), vi_shift);
0255:         const __m128i vxi02 =
0256:             _mm_sub_epi16(_mm_unpacklo_epi8(vi02, vzero), vinput_zero_point);
0257:         const __m128i vk02 =
0258:             _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 48));
0259:         const __m128i vxk02 =
0260:             _mm_sub_epi16(_mm_unpacklo_epi8(vk02, vzero), vkernel_zero_point);
0261:         const __m128i vprod02_odd = _mm_mullo_epi16(vxi02, vxk02);
0262:         const __m128i vprod02_even = _mm_mulhi_epi16(vxi02, vxk02);
0263:         vacc_lo = _mm_add_epi32(
0264:             vacc_lo, _mm_unpacklo_epi16(vprod02_odd, vprod02_even));
0265:         vacc_hi = _mm_add_epi32(
0266:             vacc_hi, _mm_unpackhi_epi16(vprod02_odd, vprod02_even));
0267: 
```

- **EN:** This block implements local helper logic for `mp8x25-sse2`. Key symbols: `_mm_srl_epi64`, `_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`.
- **CN:** 该代码块实现与 `mp8x25-sse2` 相关的局部辅助逻辑。关键符号：`_mm_srl_epi64`, `_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`。

### Lines 268-297 / 第 268-297 行

```c
0268:         const __m128i vi10 =
0269:             _mm_srl_epi64(_mm_loadl_epi64((const __m128i*)i10), vi_shift);
0270:         const __m128i vxi10 =
0271:             _mm_sub_epi16(_mm_unpacklo_epi8(vi10, vzero), vinput_zero_point);
0272:         const __m128i vk10 =
0273:             _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 56));
0274:         const __m128i vxk10 =
0275:             _mm_sub_epi16(_mm_unpacklo_epi8(vk10, vzero), vkernel_zero_point);
0276:         const __m128i vprod10_odd = _mm_mullo_epi16(vxi10, vxk10);
0277:         const __m128i vprod10_even = _mm_mulhi_epi16(vxi10, vxk10);
0278:         vacc_lo = _mm_add_epi32(
0279:             vacc_lo, _mm_unpacklo_epi16(vprod10_odd, vprod10_even));
0280:         vacc_hi = _mm_add_epi32(
0281:             vacc_hi, _mm_unpackhi_epi16(vprod10_odd, vprod10_even));
0282: 
0283:         const __m128i vi11 =
0284:             _mm_srl_epi64(_mm_loadl_epi64((const __m128i*)i11), vi_shift);
0285:         const __m128i vxi11 =
0286:             _mm_sub_epi16(_mm_unpacklo_epi8(vi11, vzero), vinput_zero_point);
0287:         const __m128i vk11 =
0288:             _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 64));
0289:         const __m128i vxk11 =
0290:             _mm_sub_epi16(_mm_unpacklo_epi8(vk11, vzero), vkernel_zero_point);
0291:         const __m128i vprod11_odd = _mm_mullo_epi16(vxi11, vxk11);
0292:         const __m128i vprod11_even = _mm_mulhi_epi16(vxi11, vxk11);
0293:         vacc_lo = _mm_add_epi32(
0294:             vacc_lo, _mm_unpacklo_epi16(vprod11_odd, vprod11_even));
0295:         vacc_hi = _mm_add_epi32(
0296:             vacc_hi, _mm_unpackhi_epi16(vprod11_odd, vprod11_even));
0297: 
```

- **EN:** This block implements local helper logic for `mp8x25-sse2`. Key symbols: `_mm_srl_epi64`, `_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`.
- **CN:** 该代码块实现与 `mp8x25-sse2` 相关的局部辅助逻辑。关键符号：`_mm_srl_epi64`, `_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`。

### Lines 298-327 / 第 298-327 行

```c
0298:         const __m128i vi12 =
0299:             _mm_srl_epi64(_mm_loadl_epi64((const __m128i*)i12), vi_shift);
0300:         const __m128i vxi12 =
0301:             _mm_sub_epi16(_mm_unpacklo_epi8(vi12, vzero), vinput_zero_point);
0302:         const __m128i vk12 =
0303:             _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 72));
0304:         const __m128i vxk12 =
0305:             _mm_sub_epi16(_mm_unpacklo_epi8(vk12, vzero), vkernel_zero_point);
0306:         const __m128i vprod12_odd = _mm_mullo_epi16(vxi12, vxk12);
0307:         const __m128i vprod12_even = _mm_mulhi_epi16(vxi12, vxk12);
0308:         vacc_lo = _mm_add_epi32(
0309:             vacc_lo, _mm_unpacklo_epi16(vprod12_odd, vprod12_even));
0310:         vacc_hi = _mm_add_epi32(
0311:             vacc_hi, _mm_unpackhi_epi16(vprod12_odd, vprod12_even));
0312: 
0313:         const __m128i vi20 =
0314:             _mm_srl_epi64(_mm_loadl_epi64((const __m128i*)i20), vi_shift);
0315:         const __m128i vxi20 =
0316:             _mm_sub_epi16(_mm_unpacklo_epi8(vi20, vzero), vinput_zero_point);
0317:         const __m128i vk20 =
0318:             _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 80));
0319:         const __m128i vxk20 =
0320:             _mm_sub_epi16(_mm_unpacklo_epi8(vk20, vzero), vkernel_zero_point);
0321:         const __m128i vprod20_odd = _mm_mullo_epi16(vxi20, vxk20);
0322:         const __m128i vprod20_even = _mm_mulhi_epi16(vxi20, vxk20);
0323:         vacc_lo = _mm_add_epi32(
0324:             vacc_lo, _mm_unpacklo_epi16(vprod20_odd, vprod20_even));
0325:         vacc_hi = _mm_add_epi32(
0326:             vacc_hi, _mm_unpackhi_epi16(vprod20_odd, vprod20_even));
0327: 
```

- **EN:** This block implements local helper logic for `mp8x25-sse2`. Key symbols: `_mm_srl_epi64`, `_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`.
- **CN:** 该代码块实现与 `mp8x25-sse2` 相关的局部辅助逻辑。关键符号：`_mm_srl_epi64`, `_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`。

### Lines 328-357 / 第 328-357 行

```c
0328:         const __m128i vi21 =
0329:             _mm_srl_epi64(_mm_loadl_epi64((const __m128i*)i21), vi_shift);
0330:         const __m128i vxi21 =
0331:             _mm_sub_epi16(_mm_unpacklo_epi8(vi21, vzero), vinput_zero_point);
0332:         const __m128i vk21 =
0333:             _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 88));
0334:         const __m128i vxk21 =
0335:             _mm_sub_epi16(_mm_unpacklo_epi8(vk21, vzero), vkernel_zero_point);
0336:         const __m128i vprod21_odd = _mm_mullo_epi16(vxi21, vxk21);
0337:         const __m128i vprod21_even = _mm_mulhi_epi16(vxi21, vxk21);
0338:         vacc_lo = _mm_add_epi32(
0339:             vacc_lo, _mm_unpacklo_epi16(vprod21_odd, vprod21_even));
0340:         vacc_hi = _mm_add_epi32(
0341:             vacc_hi, _mm_unpackhi_epi16(vprod21_odd, vprod21_even));
0342: 
0343:         const __m128i vi22 =
0344:             _mm_srl_epi64(_mm_loadl_epi64((const __m128i*)i22), vi_shift);
0345:         const __m128i vxi22 =
0346:             _mm_sub_epi16(_mm_unpacklo_epi8(vi22, vzero), vinput_zero_point);
0347:         const __m128i vk22 =
0348:             _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 96));
0349:         const __m128i vxk22 =
0350:             _mm_sub_epi16(_mm_unpacklo_epi8(vk22, vzero), vkernel_zero_point);
0351:         const __m128i vprod22_odd = _mm_mullo_epi16(vxi22, vxk22);
0352:         const __m128i vprod22_even = _mm_mulhi_epi16(vxi22, vxk22);
0353:         vacc_lo = _mm_add_epi32(
0354:             vacc_lo, _mm_unpacklo_epi16(vprod22_odd, vprod22_even));
0355:         vacc_hi = _mm_add_epi32(
0356:             vacc_hi, _mm_unpackhi_epi16(vprod22_odd, vprod22_even));
0357: 
```

- **EN:** This block implements local helper logic for `mp8x25-sse2`. Key symbols: `_mm_srl_epi64`, `_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`.
- **CN:** 该代码块实现与 `mp8x25-sse2` 相关的局部辅助逻辑。关键符号：`_mm_srl_epi64`, `_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`。

### Lines 358-392 / 第 358-392 行

```c
0358:         const __m128i vi23 =
0359:             _mm_srl_epi64(_mm_loadl_epi64((const __m128i*)i23), vi_shift);
0360:         const __m128i vxi23 =
0361:             _mm_sub_epi16(_mm_unpacklo_epi8(vi23, vzero), vinput_zero_point);
0362:         const __m128i vk23 =
0363:             _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 104));
0364:         const __m128i vxk23 =
0365:             _mm_sub_epi16(_mm_unpacklo_epi8(vk23, vzero), vkernel_zero_point);
0366:         const __m128i vprod23_odd = _mm_mullo_epi16(vxi23, vxk23);
0367:         const __m128i vprod23_even = _mm_mulhi_epi16(vxi23, vxk23);
0368:         vacc_lo = _mm_add_epi32(
0369:             vacc_lo, _mm_unpacklo_epi16(vprod23_odd, vprod23_even));
0370:         vacc_hi = _mm_add_epi32(
0371:             vacc_hi, _mm_unpackhi_epi16(vprod23_odd, vprod23_even));
0372: 
0373:         w = (const void*)((uintptr_t)w + 112);
0374:         _mm_storeu_si128((__m128i*)outacc, vacc_lo);
0375:         outacc += 4;
0376:         _mm_storeu_si128((__m128i*)outacc, vacc_hi);
0377:         outacc += 4;
0378:       }
0379:     }
0380:     {
0381:       const uint8_t* i00 = input[10];
0382:       const uint8_t* i01 = input[11];
0383:       const uint8_t* i02 = input[12];
0384:       const uint8_t* i10 = input[13];
0385:       const uint8_t* i11 = input[14];
0386:       const uint8_t* i12 = input[15];
0387:       const uint8_t* i20 = input[16];
0388:       const uint8_t* i21 = input[17];
0389:       const uint8_t* i22 = input[18];
0390:       const uint8_t* i23 = input[19];
0391:       outacc = outacc32;
0392: 
```

- **EN:** This block implements local helper logic for `mp8x25-sse2`. Key symbols: `_mm_srl_epi64`, `_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`, `_mm_storeu_si128`.
- **CN:** 该代码块实现与 `mp8x25-sse2` 相关的局部辅助逻辑。关键符号：`_mm_srl_epi64`, `_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`, `_mm_storeu_si128`。

### Lines 393-436 / 第 393-436 行

```c
0393:       size_t c = channels;
0394:       for (; c >= 8; c -= 8) {
0395:         const __m128i vi00 = _mm_loadl_epi64((const __m128i*)i00);
0396:         i00 += 8;
0397:         const __m128i vxi00 =
0398:             _mm_sub_epi16(_mm_unpacklo_epi8(vi00, vzero), vinput_zero_point);
0399:         const __m128i vk00 = _mm_loadl_epi64((const __m128i*)((uintptr_t)w));
0400:         const __m128i vxk00 =
0401:             _mm_sub_epi16(_mm_unpacklo_epi8(vk00, vzero), vkernel_zero_point);
0402:         const __m128i vprod00_odd = _mm_mullo_epi16(vxi00, vxk00);
0403:         const __m128i vprod00_even = _mm_mulhi_epi16(vxi00, vxk00);
0404:         __m128i vacc_lo = _mm_unpacklo_epi16(vprod00_odd, vprod00_even);
0405:         __m128i vacc_hi = _mm_unpackhi_epi16(vprod00_odd, vprod00_even);
0406: 
0407:         const __m128i vi01 = _mm_loadl_epi64((const __m128i*)i01);
0408:         i01 += 8;
0409:         const __m128i vxi01 =
0410:             _mm_sub_epi16(_mm_unpacklo_epi8(vi01, vzero), vinput_zero_point);
0411:         const __m128i vk01 =
0412:             _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 8));
0413:         const __m128i vxk01 =
0414:             _mm_sub_epi16(_mm_unpacklo_epi8(vk01, vzero), vkernel_zero_point);
0415:         const __m128i vprod01_odd = _mm_mullo_epi16(vxi01, vxk01);
0416:         const __m128i vprod01_even = _mm_mulhi_epi16(vxi01, vxk01);
0417:         vacc_lo = _mm_add_epi32(
0418:             vacc_lo, _mm_unpacklo_epi16(vprod01_odd, vprod01_even));
0419:         vacc_hi = _mm_add_epi32(
0420:             vacc_hi, _mm_unpackhi_epi16(vprod01_odd, vprod01_even));
0421: 
0422:         const __m128i vi02 = _mm_loadl_epi64((const __m128i*)i02);
0423:         i02 += 8;
0424:         const __m128i vxi02 =
0425:             _mm_sub_epi16(_mm_unpacklo_epi8(vi02, vzero), vinput_zero_point);
0426:         const __m128i vk02 =
0427:             _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 16));
0428:         const __m128i vxk02 =
0429:             _mm_sub_epi16(_mm_unpacklo_epi8(vk02, vzero), vkernel_zero_point);
0430:         const __m128i vprod02_odd = _mm_mullo_epi16(vxi02, vxk02);
0431:         const __m128i vprod02_even = _mm_mulhi_epi16(vxi02, vxk02);
0432:         vacc_lo = _mm_add_epi32(
0433:             vacc_lo, _mm_unpacklo_epi16(vprod02_odd, vprod02_even));
0434:         vacc_hi = _mm_add_epi32(
0435:             vacc_hi, _mm_unpackhi_epi16(vprod02_odd, vprod02_even));
0436: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`。

### Lines 437-466 / 第 437-466 行

```c
0437:         const __m128i vi10 = _mm_loadl_epi64((const __m128i*)i10);
0438:         i10 += 8;
0439:         const __m128i vxi10 =
0440:             _mm_sub_epi16(_mm_unpacklo_epi8(vi10, vzero), vinput_zero_point);
0441:         const __m128i vk10 =
0442:             _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 24));
0443:         const __m128i vxk10 =
0444:             _mm_sub_epi16(_mm_unpacklo_epi8(vk10, vzero), vkernel_zero_point);
0445:         const __m128i vprod10_odd = _mm_mullo_epi16(vxi10, vxk10);
0446:         const __m128i vprod10_even = _mm_mulhi_epi16(vxi10, vxk10);
0447:         vacc_lo = _mm_add_epi32(
0448:             vacc_lo, _mm_unpacklo_epi16(vprod10_odd, vprod10_even));
0449:         vacc_hi = _mm_add_epi32(
0450:             vacc_hi, _mm_unpackhi_epi16(vprod10_odd, vprod10_even));
0451: 
0452:         const __m128i vi11 = _mm_loadl_epi64((const __m128i*)i11);
0453:         i11 += 8;
0454:         const __m128i vxi11 =
0455:             _mm_sub_epi16(_mm_unpacklo_epi8(vi11, vzero), vinput_zero_point);
0456:         const __m128i vk11 =
0457:             _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 32));
0458:         const __m128i vxk11 =
0459:             _mm_sub_epi16(_mm_unpacklo_epi8(vk11, vzero), vkernel_zero_point);
0460:         const __m128i vprod11_odd = _mm_mullo_epi16(vxi11, vxk11);
0461:         const __m128i vprod11_even = _mm_mulhi_epi16(vxi11, vxk11);
0462:         vacc_lo = _mm_add_epi32(
0463:             vacc_lo, _mm_unpacklo_epi16(vprod11_odd, vprod11_even));
0464:         vacc_hi = _mm_add_epi32(
0465:             vacc_hi, _mm_unpackhi_epi16(vprod11_odd, vprod11_even));
0466: 
```

- **EN:** This block implements local helper logic for `mp8x25-sse2`. Key symbols: `_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`.
- **CN:** 该代码块实现与 `mp8x25-sse2` 相关的局部辅助逻辑。关键符号：`_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`。

### Lines 467-496 / 第 467-496 行

```c
0467:         const __m128i vi12 = _mm_loadl_epi64((const __m128i*)i12);
0468:         i12 += 8;
0469:         const __m128i vxi12 =
0470:             _mm_sub_epi16(_mm_unpacklo_epi8(vi12, vzero), vinput_zero_point);
0471:         const __m128i vk12 =
0472:             _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 40));
0473:         const __m128i vxk12 =
0474:             _mm_sub_epi16(_mm_unpacklo_epi8(vk12, vzero), vkernel_zero_point);
0475:         const __m128i vprod12_odd = _mm_mullo_epi16(vxi12, vxk12);
0476:         const __m128i vprod12_even = _mm_mulhi_epi16(vxi12, vxk12);
0477:         vacc_lo = _mm_add_epi32(
0478:             vacc_lo, _mm_unpacklo_epi16(vprod12_odd, vprod12_even));
0479:         vacc_hi = _mm_add_epi32(
0480:             vacc_hi, _mm_unpackhi_epi16(vprod12_odd, vprod12_even));
0481: 
0482:         const __m128i vi20 = _mm_loadl_epi64((const __m128i*)i20);
0483:         i20 += 8;
0484:         const __m128i vxi20 =
0485:             _mm_sub_epi16(_mm_unpacklo_epi8(vi20, vzero), vinput_zero_point);
0486:         const __m128i vk20 =
0487:             _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 48));
0488:         const __m128i vxk20 =
0489:             _mm_sub_epi16(_mm_unpacklo_epi8(vk20, vzero), vkernel_zero_point);
0490:         const __m128i vprod20_odd = _mm_mullo_epi16(vxi20, vxk20);
0491:         const __m128i vprod20_even = _mm_mulhi_epi16(vxi20, vxk20);
0492:         vacc_lo = _mm_add_epi32(
0493:             vacc_lo, _mm_unpacklo_epi16(vprod20_odd, vprod20_even));
0494:         vacc_hi = _mm_add_epi32(
0495:             vacc_hi, _mm_unpackhi_epi16(vprod20_odd, vprod20_even));
0496: 
```

- **EN:** This block implements local helper logic for `mp8x25-sse2`. Key symbols: `_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`.
- **CN:** 该代码块实现与 `mp8x25-sse2` 相关的局部辅助逻辑。关键符号：`_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`。

### Lines 497-526 / 第 497-526 行

```c
0497:         const __m128i vi21 = _mm_loadl_epi64((const __m128i*)i21);
0498:         i21 += 8;
0499:         const __m128i vxi21 =
0500:             _mm_sub_epi16(_mm_unpacklo_epi8(vi21, vzero), vinput_zero_point);
0501:         const __m128i vk21 =
0502:             _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 56));
0503:         const __m128i vxk21 =
0504:             _mm_sub_epi16(_mm_unpacklo_epi8(vk21, vzero), vkernel_zero_point);
0505:         const __m128i vprod21_odd = _mm_mullo_epi16(vxi21, vxk21);
0506:         const __m128i vprod21_even = _mm_mulhi_epi16(vxi21, vxk21);
0507:         vacc_lo = _mm_add_epi32(
0508:             vacc_lo, _mm_unpacklo_epi16(vprod21_odd, vprod21_even));
0509:         vacc_hi = _mm_add_epi32(
0510:             vacc_hi, _mm_unpackhi_epi16(vprod21_odd, vprod21_even));
0511: 
0512:         const __m128i vi22 = _mm_loadl_epi64((const __m128i*)i22);
0513:         i22 += 8;
0514:         const __m128i vxi22 =
0515:             _mm_sub_epi16(_mm_unpacklo_epi8(vi22, vzero), vinput_zero_point);
0516:         const __m128i vk22 =
0517:             _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 64));
0518:         const __m128i vxk22 =
0519:             _mm_sub_epi16(_mm_unpacklo_epi8(vk22, vzero), vkernel_zero_point);
0520:         const __m128i vprod22_odd = _mm_mullo_epi16(vxi22, vxk22);
0521:         const __m128i vprod22_even = _mm_mulhi_epi16(vxi22, vxk22);
0522:         vacc_lo = _mm_add_epi32(
0523:             vacc_lo, _mm_unpacklo_epi16(vprod22_odd, vprod22_even));
0524:         vacc_hi = _mm_add_epi32(
0525:             vacc_hi, _mm_unpackhi_epi16(vprod22_odd, vprod22_even));
0526: 
```

- **EN:** This block implements local helper logic for `mp8x25-sse2`. Key symbols: `_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`.
- **CN:** 该代码块实现与 `mp8x25-sse2` 相关的局部辅助逻辑。关键符号：`_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`。

### Lines 527-564 / 第 527-564 行

```c
0527:         const __m128i vi23 = _mm_loadl_epi64((const __m128i*)i23);
0528:         i23 += 8;
0529:         const __m128i vxi23 =
0530:             _mm_sub_epi16(_mm_unpacklo_epi8(vi23, vzero), vinput_zero_point);
0531:         const __m128i vk23 =
0532:             _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 72));
0533:         const __m128i vxk23 =
0534:             _mm_sub_epi16(_mm_unpacklo_epi8(vk23, vzero), vkernel_zero_point);
0535:         const __m128i vprod23_odd = _mm_mullo_epi16(vxi23, vxk23);
0536:         const __m128i vprod23_even = _mm_mulhi_epi16(vxi23, vxk23);
0537:         vacc_lo = _mm_add_epi32(
0538:             vacc_lo, _mm_unpacklo_epi16(vprod23_odd, vprod23_even));
0539:         vacc_hi = _mm_add_epi32(
0540:             vacc_hi, _mm_unpackhi_epi16(vprod23_odd, vprod23_even));
0541: 
0542:         w = (const void*)((uintptr_t)w + 80);
0543:         vacc_lo = _mm_add_epi32(vacc_lo, _mm_loadu_si128((__m128i*)outacc));
0544:         vacc_hi =
0545:             _mm_add_epi32(vacc_hi, _mm_loadu_si128((__m128i*)(outacc + 4)));
0546:         _mm_storeu_si128((__m128i*)outacc, vacc_lo);
0547:         outacc += 4;
0548:         _mm_storeu_si128((__m128i*)outacc, vacc_hi);
0549:         outacc += 4;
0550:       }
0551:       if (c != 0) {
0552:         const size_t i_predecrement = 8 - c;
0553:         const __m128i vi_shift = _mm_cvtsi32_si128(8 * i_predecrement);
0554:         i00 -= i_predecrement;
0555:         i01 -= i_predecrement;
0556:         i02 -= i_predecrement;
0557:         i10 -= i_predecrement;
0558:         i11 -= i_predecrement;
0559:         i12 -= i_predecrement;
0560:         i20 -= i_predecrement;
0561:         i21 -= i_predecrement;
0562:         i22 -= i_predecrement;
0563:         i23 -= i_predecrement;
0564: 
```

- **EN:** This block handles conditional branches and special cases. Key symbols: `_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`, `_mm_add_epi32`, `_mm_storeu_si128`.
- **CN:** 该代码块处理条件分支与特殊情况。关键符号：`_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`, `_mm_add_epi32`, `_mm_storeu_si128`。

### Lines 565-606 / 第 565-606 行

```c
0565:         const __m128i vi00 =
0566:             _mm_srl_epi64(_mm_loadl_epi64((const __m128i*)i00), vi_shift);
0567:         const __m128i vxi00 =
0568:             _mm_sub_epi16(_mm_unpacklo_epi8(vi00, vzero), vinput_zero_point);
0569:         const __m128i vk00 = _mm_loadl_epi64((const __m128i*)((uintptr_t)w));
0570:         const __m128i vxk00 =
0571:             _mm_sub_epi16(_mm_unpacklo_epi8(vk00, vzero), vkernel_zero_point);
0572:         const __m128i vprod00_odd = _mm_mullo_epi16(vxi00, vxk00);
0573:         const __m128i vprod00_even = _mm_mulhi_epi16(vxi00, vxk00);
0574:         __m128i vacc_lo = _mm_unpacklo_epi16(vprod00_odd, vprod00_even);
0575:         __m128i vacc_hi = _mm_unpackhi_epi16(vprod00_odd, vprod00_even);
0576: 
0577:         const __m128i vi01 =
0578:             _mm_srl_epi64(_mm_loadl_epi64((const __m128i*)i01), vi_shift);
0579:         const __m128i vxi01 =
0580:             _mm_sub_epi16(_mm_unpacklo_epi8(vi01, vzero), vinput_zero_point);
0581:         const __m128i vk01 =
0582:             _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 8));
0583:         const __m128i vxk01 =
0584:             _mm_sub_epi16(_mm_unpacklo_epi8(vk01, vzero), vkernel_zero_point);
0585:         const __m128i vprod01_odd = _mm_mullo_epi16(vxi01, vxk01);
0586:         const __m128i vprod01_even = _mm_mulhi_epi16(vxi01, vxk01);
0587:         vacc_lo = _mm_add_epi32(
0588:             vacc_lo, _mm_unpacklo_epi16(vprod01_odd, vprod01_even));
0589:         vacc_hi = _mm_add_epi32(
0590:             vacc_hi, _mm_unpackhi_epi16(vprod01_odd, vprod01_even));
0591: 
0592:         const __m128i vi02 =
0593:             _mm_srl_epi64(_mm_loadl_epi64((const __m128i*)i02), vi_shift);
0594:         const __m128i vxi02 =
0595:             _mm_sub_epi16(_mm_unpacklo_epi8(vi02, vzero), vinput_zero_point);
0596:         const __m128i vk02 =
0597:             _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 16));
0598:         const __m128i vxk02 =
0599:             _mm_sub_epi16(_mm_unpacklo_epi8(vk02, vzero), vkernel_zero_point);
0600:         const __m128i vprod02_odd = _mm_mullo_epi16(vxi02, vxk02);
0601:         const __m128i vprod02_even = _mm_mulhi_epi16(vxi02, vxk02);
0602:         vacc_lo = _mm_add_epi32(
0603:             vacc_lo, _mm_unpacklo_epi16(vprod02_odd, vprod02_even));
0604:         vacc_hi = _mm_add_epi32(
0605:             vacc_hi, _mm_unpackhi_epi16(vprod02_odd, vprod02_even));
0606: 
```

- **EN:** This block implements local helper logic for `mp8x25-sse2`. Key symbols: `_mm_srl_epi64`, `_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`.
- **CN:** 该代码块实现与 `mp8x25-sse2` 相关的局部辅助逻辑。关键符号：`_mm_srl_epi64`, `_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`。

### Lines 607-636 / 第 607-636 行

```c
0607:         const __m128i vi10 =
0608:             _mm_srl_epi64(_mm_loadl_epi64((const __m128i*)i10), vi_shift);
0609:         const __m128i vxi10 =
0610:             _mm_sub_epi16(_mm_unpacklo_epi8(vi10, vzero), vinput_zero_point);
0611:         const __m128i vk10 =
0612:             _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 24));
0613:         const __m128i vxk10 =
0614:             _mm_sub_epi16(_mm_unpacklo_epi8(vk10, vzero), vkernel_zero_point);
0615:         const __m128i vprod10_odd = _mm_mullo_epi16(vxi10, vxk10);
0616:         const __m128i vprod10_even = _mm_mulhi_epi16(vxi10, vxk10);
0617:         vacc_lo = _mm_add_epi32(
0618:             vacc_lo, _mm_unpacklo_epi16(vprod10_odd, vprod10_even));
0619:         vacc_hi = _mm_add_epi32(
0620:             vacc_hi, _mm_unpackhi_epi16(vprod10_odd, vprod10_even));
0621: 
0622:         const __m128i vi11 =
0623:             _mm_srl_epi64(_mm_loadl_epi64((const __m128i*)i11), vi_shift);
0624:         const __m128i vxi11 =
0625:             _mm_sub_epi16(_mm_unpacklo_epi8(vi11, vzero), vinput_zero_point);
0626:         const __m128i vk11 =
0627:             _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 32));
0628:         const __m128i vxk11 =
0629:             _mm_sub_epi16(_mm_unpacklo_epi8(vk11, vzero), vkernel_zero_point);
0630:         const __m128i vprod11_odd = _mm_mullo_epi16(vxi11, vxk11);
0631:         const __m128i vprod11_even = _mm_mulhi_epi16(vxi11, vxk11);
0632:         vacc_lo = _mm_add_epi32(
0633:             vacc_lo, _mm_unpacklo_epi16(vprod11_odd, vprod11_even));
0634:         vacc_hi = _mm_add_epi32(
0635:             vacc_hi, _mm_unpackhi_epi16(vprod11_odd, vprod11_even));
0636: 
```

- **EN:** This block implements local helper logic for `mp8x25-sse2`. Key symbols: `_mm_srl_epi64`, `_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`.
- **CN:** 该代码块实现与 `mp8x25-sse2` 相关的局部辅助逻辑。关键符号：`_mm_srl_epi64`, `_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`。

### Lines 637-666 / 第 637-666 行

```c
0637:         const __m128i vi12 =
0638:             _mm_srl_epi64(_mm_loadl_epi64((const __m128i*)i12), vi_shift);
0639:         const __m128i vxi12 =
0640:             _mm_sub_epi16(_mm_unpacklo_epi8(vi12, vzero), vinput_zero_point);
0641:         const __m128i vk12 =
0642:             _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 40));
0643:         const __m128i vxk12 =
0644:             _mm_sub_epi16(_mm_unpacklo_epi8(vk12, vzero), vkernel_zero_point);
0645:         const __m128i vprod12_odd = _mm_mullo_epi16(vxi12, vxk12);
0646:         const __m128i vprod12_even = _mm_mulhi_epi16(vxi12, vxk12);
0647:         vacc_lo = _mm_add_epi32(
0648:             vacc_lo, _mm_unpacklo_epi16(vprod12_odd, vprod12_even));
0649:         vacc_hi = _mm_add_epi32(
0650:             vacc_hi, _mm_unpackhi_epi16(vprod12_odd, vprod12_even));
0651: 
0652:         const __m128i vi20 =
0653:             _mm_srl_epi64(_mm_loadl_epi64((const __m128i*)i20), vi_shift);
0654:         const __m128i vxi20 =
0655:             _mm_sub_epi16(_mm_unpacklo_epi8(vi20, vzero), vinput_zero_point);
0656:         const __m128i vk20 =
0657:             _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 48));
0658:         const __m128i vxk20 =
0659:             _mm_sub_epi16(_mm_unpacklo_epi8(vk20, vzero), vkernel_zero_point);
0660:         const __m128i vprod20_odd = _mm_mullo_epi16(vxi20, vxk20);
0661:         const __m128i vprod20_even = _mm_mulhi_epi16(vxi20, vxk20);
0662:         vacc_lo = _mm_add_epi32(
0663:             vacc_lo, _mm_unpacklo_epi16(vprod20_odd, vprod20_even));
0664:         vacc_hi = _mm_add_epi32(
0665:             vacc_hi, _mm_unpackhi_epi16(vprod20_odd, vprod20_even));
0666: 
```

- **EN:** This block implements local helper logic for `mp8x25-sse2`. Key symbols: `_mm_srl_epi64`, `_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`.
- **CN:** 该代码块实现与 `mp8x25-sse2` 相关的局部辅助逻辑。关键符号：`_mm_srl_epi64`, `_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`。

### Lines 667-696 / 第 667-696 行

```c
0667:         const __m128i vi21 =
0668:             _mm_srl_epi64(_mm_loadl_epi64((const __m128i*)i21), vi_shift);
0669:         const __m128i vxi21 =
0670:             _mm_sub_epi16(_mm_unpacklo_epi8(vi21, vzero), vinput_zero_point);
0671:         const __m128i vk21 =
0672:             _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 56));
0673:         const __m128i vxk21 =
0674:             _mm_sub_epi16(_mm_unpacklo_epi8(vk21, vzero), vkernel_zero_point);
0675:         const __m128i vprod21_odd = _mm_mullo_epi16(vxi21, vxk21);
0676:         const __m128i vprod21_even = _mm_mulhi_epi16(vxi21, vxk21);
0677:         vacc_lo = _mm_add_epi32(
0678:             vacc_lo, _mm_unpacklo_epi16(vprod21_odd, vprod21_even));
0679:         vacc_hi = _mm_add_epi32(
0680:             vacc_hi, _mm_unpackhi_epi16(vprod21_odd, vprod21_even));
0681: 
0682:         const __m128i vi22 =
0683:             _mm_srl_epi64(_mm_loadl_epi64((const __m128i*)i22), vi_shift);
0684:         const __m128i vxi22 =
0685:             _mm_sub_epi16(_mm_unpacklo_epi8(vi22, vzero), vinput_zero_point);
0686:         const __m128i vk22 =
0687:             _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 64));
0688:         const __m128i vxk22 =
0689:             _mm_sub_epi16(_mm_unpacklo_epi8(vk22, vzero), vkernel_zero_point);
0690:         const __m128i vprod22_odd = _mm_mullo_epi16(vxi22, vxk22);
0691:         const __m128i vprod22_even = _mm_mulhi_epi16(vxi22, vxk22);
0692:         vacc_lo = _mm_add_epi32(
0693:             vacc_lo, _mm_unpacklo_epi16(vprod22_odd, vprod22_even));
0694:         vacc_hi = _mm_add_epi32(
0695:             vacc_hi, _mm_unpackhi_epi16(vprod22_odd, vprod22_even));
0696: 
```

- **EN:** This block implements local helper logic for `mp8x25-sse2`. Key symbols: `_mm_srl_epi64`, `_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`.
- **CN:** 该代码块实现与 `mp8x25-sse2` 相关的局部辅助逻辑。关键符号：`_mm_srl_epi64`, `_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`。

### Lines 697-731 / 第 697-731 行

```c
0697:         const __m128i vi23 =
0698:             _mm_srl_epi64(_mm_loadl_epi64((const __m128i*)i23), vi_shift);
0699:         const __m128i vxi23 =
0700:             _mm_sub_epi16(_mm_unpacklo_epi8(vi23, vzero), vinput_zero_point);
0701:         const __m128i vk23 =
0702:             _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 72));
0703:         const __m128i vxk23 =
0704:             _mm_sub_epi16(_mm_unpacklo_epi8(vk23, vzero), vkernel_zero_point);
0705:         const __m128i vprod23_odd = _mm_mullo_epi16(vxi23, vxk23);
0706:         const __m128i vprod23_even = _mm_mulhi_epi16(vxi23, vxk23);
0707:         vacc_lo = _mm_add_epi32(
0708:             vacc_lo, _mm_unpacklo_epi16(vprod23_odd, vprod23_even));
0709:         vacc_hi = _mm_add_epi32(
0710:             vacc_hi, _mm_unpackhi_epi16(vprod23_odd, vprod23_even));
0711: 
0712:         w = (const void*)((uintptr_t)w + 80);
0713:         vacc_lo = _mm_add_epi32(vacc_lo, _mm_loadu_si128((__m128i*)outacc));
0714:         vacc_hi =
0715:             _mm_add_epi32(vacc_hi, _mm_loadu_si128((__m128i*)(outacc + 4)));
0716:         _mm_storeu_si128((__m128i*)outacc, vacc_lo);
0717:         outacc += 4;
0718:         _mm_storeu_si128((__m128i*)outacc, vacc_hi);
0719:         outacc += 4;
0720:       }
0721:     }
0722:     {
0723:       const uint8_t* i00 = input[20];
0724:       const uint8_t* i01 = input[21];
0725:       const uint8_t* i02 = input[22];
0726:       const uint8_t* i10 = input[23];
0727:       const uint8_t* i11 = input[24];
0728:       input = (const uint8_t**)((uintptr_t)input + input_stride);
0729:       outacc = outacc32;
0730:       size_t c = channels;
0731:       for (; c >= 8; c -= 8) {
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `_mm_srl_epi64`, `_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`, `_mm_add_epi32`, `_mm_storeu_si128`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`_mm_srl_epi64`, `_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`, `_mm_add_epi32`, `_mm_storeu_si128`。

### Lines 732-773 / 第 732-773 行

```c
0732:         const __m128i vi00 = _mm_loadl_epi64((const __m128i*)i00);
0733:         i00 += 8;
0734:         const __m128i vxi00 =
0735:             _mm_sub_epi16(_mm_unpacklo_epi8(vi00, vzero), vinput_zero_point);
0736:         const __m128i vk00 = _mm_loadl_epi64((const __m128i*)((uintptr_t)w));
0737:         const __m128i vxk00 =
0738:             _mm_sub_epi16(_mm_unpacklo_epi8(vk00, vzero), vkernel_zero_point);
0739:         const __m128i vprod00_odd = _mm_mullo_epi16(vxi00, vxk00);
0740:         const __m128i vprod00_even = _mm_mulhi_epi16(vxi00, vxk00);
0741:         __m128i vacc_lo = _mm_unpacklo_epi16(vprod00_odd, vprod00_even);
0742:         __m128i vacc_hi = _mm_unpackhi_epi16(vprod00_odd, vprod00_even);
0743: 
0744:         const __m128i vi01 = _mm_loadl_epi64((const __m128i*)i01);
0745:         i01 += 8;
0746:         const __m128i vxi01 =
0747:             _mm_sub_epi16(_mm_unpacklo_epi8(vi01, vzero), vinput_zero_point);
0748:         const __m128i vk01 =
0749:             _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 8));
0750:         const __m128i vxk01 =
0751:             _mm_sub_epi16(_mm_unpacklo_epi8(vk01, vzero), vkernel_zero_point);
0752:         const __m128i vprod01_odd = _mm_mullo_epi16(vxi01, vxk01);
0753:         const __m128i vprod01_even = _mm_mulhi_epi16(vxi01, vxk01);
0754:         vacc_lo = _mm_add_epi32(
0755:             vacc_lo, _mm_unpacklo_epi16(vprod01_odd, vprod01_even));
0756:         vacc_hi = _mm_add_epi32(
0757:             vacc_hi, _mm_unpackhi_epi16(vprod01_odd, vprod01_even));
0758: 
0759:         const __m128i vi02 = _mm_loadl_epi64((const __m128i*)i02);
0760:         i02 += 8;
0761:         const __m128i vxi02 =
0762:             _mm_sub_epi16(_mm_unpacklo_epi8(vi02, vzero), vinput_zero_point);
0763:         const __m128i vk02 =
0764:             _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 16));
0765:         const __m128i vxk02 =
0766:             _mm_sub_epi16(_mm_unpacklo_epi8(vk02, vzero), vkernel_zero_point);
0767:         const __m128i vprod02_odd = _mm_mullo_epi16(vxi02, vxk02);
0768:         const __m128i vprod02_even = _mm_mulhi_epi16(vxi02, vxk02);
0769:         vacc_lo = _mm_add_epi32(
0770:             vacc_lo, _mm_unpacklo_epi16(vprod02_odd, vprod02_even));
0771:         vacc_hi = _mm_add_epi32(
0772:             vacc_hi, _mm_unpackhi_epi16(vprod02_odd, vprod02_even));
0773: 
```

- **EN:** This block implements local helper logic for `mp8x25-sse2`. Key symbols: `_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`.
- **CN:** 该代码块实现与 `mp8x25-sse2` 相关的局部辅助逻辑。关键符号：`_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`。

### Lines 774-803 / 第 774-803 行

```c
0774:         const __m128i vi10 = _mm_loadl_epi64((const __m128i*)i10);
0775:         i10 += 8;
0776:         const __m128i vxi10 =
0777:             _mm_sub_epi16(_mm_unpacklo_epi8(vi10, vzero), vinput_zero_point);
0778:         const __m128i vk10 =
0779:             _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 24));
0780:         const __m128i vxk10 =
0781:             _mm_sub_epi16(_mm_unpacklo_epi8(vk10, vzero), vkernel_zero_point);
0782:         const __m128i vprod10_odd = _mm_mullo_epi16(vxi10, vxk10);
0783:         const __m128i vprod10_even = _mm_mulhi_epi16(vxi10, vxk10);
0784:         vacc_lo = _mm_add_epi32(
0785:             vacc_lo, _mm_unpacklo_epi16(vprod10_odd, vprod10_even));
0786:         vacc_hi = _mm_add_epi32(
0787:             vacc_hi, _mm_unpackhi_epi16(vprod10_odd, vprod10_even));
0788: 
0789:         const __m128i vi11 = _mm_loadl_epi64((const __m128i*)i11);
0790:         i11 += 8;
0791:         const __m128i vxi11 =
0792:             _mm_sub_epi16(_mm_unpacklo_epi8(vi11, vzero), vinput_zero_point);
0793:         const __m128i vk11 =
0794:             _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 32));
0795:         const __m128i vxk11 =
0796:             _mm_sub_epi16(_mm_unpacklo_epi8(vk11, vzero), vkernel_zero_point);
0797:         const __m128i vprod11_odd = _mm_mullo_epi16(vxi11, vxk11);
0798:         const __m128i vprod11_even = _mm_mulhi_epi16(vxi11, vxk11);
0799:         vacc_lo = _mm_add_epi32(
0800:             vacc_lo, _mm_unpacklo_epi16(vprod11_odd, vprod11_even));
0801:         vacc_hi = _mm_add_epi32(
0802:             vacc_hi, _mm_unpackhi_epi16(vprod11_odd, vprod11_even));
0803: 
```

- **EN:** This block implements local helper logic for `mp8x25-sse2`. Key symbols: `_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`.
- **CN:** 该代码块实现与 `mp8x25-sse2` 相关的局部辅助逻辑。关键符号：`_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`。

### Lines 804-840 / 第 804-840 行

```c
0804:         w = (const void*)((uintptr_t)w + 40);
0805: 
0806:         vacc_lo = _mm_add_epi32(vacc_lo, _mm_loadu_si128((__m128i*)outacc));
0807:         vacc_hi =
0808:             _mm_add_epi32(vacc_hi, _mm_loadu_si128((__m128i*)(outacc + 4)));
0809:         outacc += 8;
0810: 
0811:         const __m128 vmultiplier =
0812:             _mm_set1_ps(quantization_params->sse2.requantization_scales[0]);
0813: 
0814:         vacc_lo = _mm_cvtps_epi32(
0815:                       _mm_mul_ps(
0816:                         _mm_cvtepi32_ps(vacc_lo),
0817:                         vmultiplier
0818:                         )
0819:                       );
0820:         vacc_hi = _mm_cvtps_epi32(
0821:                       _mm_mul_ps(
0822:                         _mm_cvtepi32_ps(vacc_hi),
0823:                         vmultiplier
0824:                         )
0825:                       );
0826: 
0827:         const __m128i voutput_zero_point = _mm_load_si128(
0828:             (const __m128i*)quantization_params->sse2.output_zero_point);
0829:         __m128i vout = _mm_adds_epi16(
0830:             _mm_packs_epi32(vacc_lo, vacc_hi), voutput_zero_point);
0831:         vout = _mm_packus_epi16(vout, vout);
0832:         vout = _mm_max_epu8(
0833:             vout,
0834:             _mm_load_si128(
0835:                 (const __m128i*)quantization_params->sse2.output_min));
0836:         vout = _mm_min_epu8(
0837:             vout,
0838:             _mm_load_si128(
0839:                 (const __m128i*)quantization_params->sse2.output_max));
0840: 
```

- **EN:** This block implements local helper logic for `mp8x25-sse2`. Key symbols: `_mm_add_epi32`, `_mm_set1_ps`, `_mm_mul_ps`, `_mm_packs_epi32`, `_mm_load_si128`.
- **CN:** 该代码块实现与 `mp8x25-sse2` 相关的局部辅助逻辑。关键符号：`_mm_add_epi32`, `_mm_set1_ps`, `_mm_mul_ps`, `_mm_packs_epi32`, `_mm_load_si128`。

### Lines 841-879 / 第 841-879 行

```c
0841:         _mm_storel_epi64((__m128i*)output, vout);
0842:         output += 8;
0843:       }
0844:       if (c != 0) {
0845:         const size_t i_predecrement = 8 - c;
0846:         const __m128i vi_shift = _mm_cvtsi32_si128(8 * i_predecrement);
0847:         i00 -= i_predecrement;
0848:         i01 -= i_predecrement;
0849:         i02 -= i_predecrement;
0850:         i10 -= i_predecrement;
0851:         i11 -= i_predecrement;
0852: 
0853:         const __m128i vi00 =
0854:             _mm_srl_epi64(_mm_loadl_epi64((const __m128i*)i00), vi_shift);
0855:         const __m128i vxi00 =
0856:             _mm_sub_epi16(_mm_unpacklo_epi8(vi00, vzero), vinput_zero_point);
0857:         const __m128i vk00 = _mm_loadl_epi64((const __m128i*)((uintptr_t)w));
0858:         const __m128i vxk00 =
0859:             _mm_sub_epi16(_mm_unpacklo_epi8(vk00, vzero), vkernel_zero_point);
0860:         const __m128i vprod00_odd = _mm_mullo_epi16(vxi00, vxk00);
0861:         const __m128i vprod00_even = _mm_mulhi_epi16(vxi00, vxk00);
0862:         __m128i vacc_lo = _mm_unpacklo_epi16(vprod00_odd, vprod00_even);
0863:         __m128i vacc_hi = _mm_unpackhi_epi16(vprod00_odd, vprod00_even);
0864: 
0865:         const __m128i vi01 =
0866:             _mm_srl_epi64(_mm_loadl_epi64((const __m128i*)i01), vi_shift);
0867:         const __m128i vxi01 =
0868:             _mm_sub_epi16(_mm_unpacklo_epi8(vi01, vzero), vinput_zero_point);
0869:         const __m128i vk01 =
0870:             _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 8));
0871:         const __m128i vxk01 =
0872:             _mm_sub_epi16(_mm_unpacklo_epi8(vk01, vzero), vkernel_zero_point);
0873:         const __m128i vprod01_odd = _mm_mullo_epi16(vxi01, vxk01);
0874:         const __m128i vprod01_even = _mm_mulhi_epi16(vxi01, vxk01);
0875:         vacc_lo = _mm_add_epi32(
0876:             vacc_lo, _mm_unpacklo_epi16(vprod01_odd, vprod01_even));
0877:         vacc_hi = _mm_add_epi32(
0878:             vacc_hi, _mm_unpackhi_epi16(vprod01_odd, vprod01_even));
0879: 
```

- **EN:** This block handles conditional branches and special cases. Key symbols: `_mm_storel_epi64`, `_mm_srl_epi64`, `_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`.
- **CN:** 该代码块处理条件分支与特殊情况。关键符号：`_mm_storel_epi64`, `_mm_srl_epi64`, `_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`。

### Lines 880-909 / 第 880-909 行

```c
0880:         const __m128i vi02 =
0881:             _mm_srl_epi64(_mm_loadl_epi64((const __m128i*)i02), vi_shift);
0882:         const __m128i vxi02 =
0883:             _mm_sub_epi16(_mm_unpacklo_epi8(vi02, vzero), vinput_zero_point);
0884:         const __m128i vk02 =
0885:             _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 16));
0886:         const __m128i vxk02 =
0887:             _mm_sub_epi16(_mm_unpacklo_epi8(vk02, vzero), vkernel_zero_point);
0888:         const __m128i vprod02_odd = _mm_mullo_epi16(vxi02, vxk02);
0889:         const __m128i vprod02_even = _mm_mulhi_epi16(vxi02, vxk02);
0890:         vacc_lo = _mm_add_epi32(
0891:             vacc_lo, _mm_unpacklo_epi16(vprod02_odd, vprod02_even));
0892:         vacc_hi = _mm_add_epi32(
0893:             vacc_hi, _mm_unpackhi_epi16(vprod02_odd, vprod02_even));
0894: 
0895:         const __m128i vi10 =
0896:             _mm_srl_epi64(_mm_loadl_epi64((const __m128i*)i10), vi_shift);
0897:         const __m128i vxi10 =
0898:             _mm_sub_epi16(_mm_unpacklo_epi8(vi10, vzero), vinput_zero_point);
0899:         const __m128i vk10 =
0900:             _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 24));
0901:         const __m128i vxk10 =
0902:             _mm_sub_epi16(_mm_unpacklo_epi8(vk10, vzero), vkernel_zero_point);
0903:         const __m128i vprod10_odd = _mm_mullo_epi16(vxi10, vxk10);
0904:         const __m128i vprod10_even = _mm_mulhi_epi16(vxi10, vxk10);
0905:         vacc_lo = _mm_add_epi32(
0906:             vacc_lo, _mm_unpacklo_epi16(vprod10_odd, vprod10_even));
0907:         vacc_hi = _mm_add_epi32(
0908:             vacc_hi, _mm_unpackhi_epi16(vprod10_odd, vprod10_even));
0909: 
```

- **EN:** This block implements local helper logic for `mp8x25-sse2`. Key symbols: `_mm_srl_epi64`, `_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`.
- **CN:** 该代码块实现与 `mp8x25-sse2` 相关的局部辅助逻辑。关键符号：`_mm_srl_epi64`, `_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`。

### Lines 910-945 / 第 910-945 行

```c
0910:         const __m128i vi11 =
0911:             _mm_srl_epi64(_mm_loadl_epi64((const __m128i*)i11), vi_shift);
0912:         const __m128i vxi11 =
0913:             _mm_sub_epi16(_mm_unpacklo_epi8(vi11, vzero), vinput_zero_point);
0914:         const __m128i vk11 =
0915:             _mm_loadl_epi64((const __m128i*)((uintptr_t)w + 32));
0916:         const __m128i vxk11 =
0917:             _mm_sub_epi16(_mm_unpacklo_epi8(vk11, vzero), vkernel_zero_point);
0918:         const __m128i vprod11_odd = _mm_mullo_epi16(vxi11, vxk11);
0919:         const __m128i vprod11_even = _mm_mulhi_epi16(vxi11, vxk11);
0920:         vacc_lo = _mm_add_epi32(
0921:             vacc_lo, _mm_unpacklo_epi16(vprod11_odd, vprod11_even));
0922:         vacc_hi = _mm_add_epi32(
0923:             vacc_hi, _mm_unpackhi_epi16(vprod11_odd, vprod11_even));
0924: 
0925:         vacc_lo = _mm_add_epi32(vacc_lo, _mm_loadu_si128((__m128i*)outacc));
0926:         vacc_hi =
0927:             _mm_add_epi32(vacc_hi, _mm_loadu_si128((__m128i*)(outacc + 4)));
0928:         outacc += 8;
0929: 
0930:         const __m128 vmultiplier =
0931:             _mm_set1_ps(quantization_params->sse2.requantization_scales[0]);
0932: 
0933:         vacc_lo = _mm_cvtps_epi32(
0934:                       _mm_mul_ps(
0935:                         _mm_cvtepi32_ps(vacc_lo),
0936:                         vmultiplier
0937:                         )
0938:                       );
0939:         vacc_hi = _mm_cvtps_epi32(
0940:                       _mm_mul_ps(
0941:                         _mm_cvtepi32_ps(vacc_hi),
0942:                         vmultiplier
0943:                         )
0944:                       );
0945: 
```

- **EN:** This block implements local helper logic for `mp8x25-sse2`. Key symbols: `_mm_srl_epi64`, `_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`, `_mm_add_epi32`, `_mm_set1_ps`, `_mm_mul_ps`.
- **CN:** 该代码块实现与 `mp8x25-sse2` 相关的局部辅助逻辑。关键符号：`_mm_srl_epi64`, `_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`, `_mm_add_epi32`, `_mm_set1_ps`, `_mm_mul_ps`。

### Lines 946-978 / 第 946-978 行

```c
0946:         const __m128i voutput_zero_point = _mm_load_si128(
0947:             (const __m128i*)quantization_params->sse2.output_zero_point);
0948:         __m128i vout = _mm_adds_epi16(
0949:             _mm_packs_epi32(vacc_lo, vacc_hi), voutput_zero_point);
0950:         vout = _mm_packus_epi16(vout, vout);
0951:         vout = _mm_max_epu8(
0952:             vout,
0953:             _mm_load_si128(
0954:                 (const __m128i*)quantization_params->sse2.output_min));
0955:         vout = _mm_min_epu8(
0956:             vout,
0957:             _mm_load_si128(
0958:                 (const __m128i*)quantization_params->sse2.output_max));
0959: 
0960:         if (c & 4) {
0961:           *((uint32_t*)output) = (uint32_t)_mm_cvtsi128_si32(vout);
0962:           output += 4;
0963:           vout = _mm_srli_epi64(vout, 32);
0964:         }
0965:         if (c & 2) {
0966:           *((uint16_t*)output) = (uint16_t)_mm_extract_epi16(vout, 0);
0967:           output += 2;
0968:           vout = _mm_srli_epi32(vout, 16);
0969:         }
0970:         if (c & 1) {
0971:           *((uint8_t*)output) = (uint8_t)_mm_cvtsi128_si32(vout);
0972:           output += 1;
0973:         }
0974:       }
0975:     }
0976:     output = (uint8_t*)((uintptr_t)output + output_increment);
0977:   } while (--output_width != 0);
0978: }
```

- **EN:** This block handles conditional branches and special cases; iterates over tensors, descriptors, options, or runtime state. Key symbols: `_mm_packs_epi32`, `_mm_load_si128`.
- **CN:** 该代码块处理条件分支与特殊情况；遍历张量、描述符、配置项或运行时状态。关键符号：`_mm_packs_epi32`, `_mm_load_si128`。


## Key Concepts / 关键概念
- **ATen native operator logic** — ATen 原生算子逻辑
- **GPU kernel/runtime interaction** — GPU 内核/运行时交互
- **Core symbols: pytorch_q8dwconv_ukernel_mp8x25__sse2, _mm_sub_epi16, _mm_loadl_epi64, _mm_unpacklo_epi16, _mm_unpackhi_epi16, _mm_storeu_si128, _mm_srl_epi64, _mm_add_epi32** — 核心符号：pytorch_q8dwconv_ukernel_mp8x25__sse2、_mm_sub_epi16、_mm_loadl_epi64、_mm_unpacklo_epi16、_mm_unpackhi_epi16、_mm_storeu_si128、_mm_srl_epi64、_mm_add_epi32

## Dependencies / 依赖关系
- **Internal includes / 内部头文件**: 无
- **External includes / 外部头文件**: `immintrin.h`, `qnnpack/q8dwconv.h`
- **Namespaces / 命名空间**: 无
- **Representative symbols / 代表性符号**: `pytorch_q8dwconv_ukernel_mp8x25__sse2`, `_mm_sub_epi16`, `_mm_loadl_epi64`, `_mm_unpacklo_epi16`, `_mm_unpackhi_epi16`, `_mm_storeu_si128`, `_mm_srl_epi64`, `_mm_add_epi32`, `_mm_set1_ps`, `_mm_mul_ps`, `_mm_packs_epi32`, `_mm_load_si128`, `...`
