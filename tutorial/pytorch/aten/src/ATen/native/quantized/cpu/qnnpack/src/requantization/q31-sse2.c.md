# q31-sse2.c — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `aten/src/ATen/native/quantized/cpu/qnnpack/src/requantization/q31-sse2.c`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements ATen native operators, checks, helper kernels, and backend dispatch behavior. This specific file centers on `q31-sse2.c`. The file header highlights: "All rights reserved. This source code is licensed under the BSD-style license found in the LICENSE file in the root directory of this source tree.."
- **Purpose (CN)**: 实现 ATen 原生算子、运行时检查、辅助内核以及后端分发行为。 该文件具体围绕 `q31-sse2.c` 展开。 文件头部注释也概括了其核心职责。

## Line-by-Line Analysis / 逐行分析
### Lines 1-16 / 第 1-16 行

```c
0001: /*
0002:  * Copyright (c) Facebook, Inc. and its affiliates.
0003:  * All rights reserved.
0004:  *
0005:  * This source code is licensed under the BSD-style license found in the
0006:  * LICENSE file in the root directory of this source tree.
0007:  */
0008: 
0009: #include <assert.h>
0010: #include <stdint.h>
0011: 
0012: #include <emmintrin.h>
0013: 
0014: #include <fp16/bitcasts.h>
0015: #include <qnnpack/requantization-stubs.h>
0016: 
```

- **EN:** This block implements local helper logic for `q31-sse2`. Key symbols: no prominent local symbols.
- **CN:** 该代码块实现与 `q31-sse2` 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 17-31 / 第 17-31 行

```c
0017: void pytorch_qnnp_requantize_q31__sse2(
0018:     size_t n,
0019:     const int32_t* input,
0020:     float scale,
0021:     uint8_t zero_point,
0022:     uint8_t qmin,
0023:     uint8_t qmax,
0024:     uint8_t* output) {
0025:   assert(n % 16 == 0);
0026:   assert(scale < 1.0f);
0027:   assert(scale >= 0x1.0p-32f);
0028: 
0029:   /* Compute requantization parameters */
0030:   const uint32_t scale_bits = fp32_to_bits(scale);
0031: 
```

- **EN:** This block validates runtime invariants before continuing. Key symbols: `pytorch_qnnp_requantize_q31__sse2`, `assert`.
- **CN:** 该代码块在继续执行前校验运行时不变量。关键符号：`pytorch_qnnp_requantize_q31__sse2`, `assert`。

### Lines 32-52 / 第 32-52 行

```c
0032:   /* Multiplier is in [0x40000000, 0x7FFFFF80] range */
0033:   const int32_t multiplier = (int32_t)(
0034:       ((scale_bits & UINT32_C(0x007FFFFF)) | UINT32_C(0x00800000)) << 7);
0035:   assert(multiplier >= INT32_C(0x40000000));
0036:   assert(multiplier <= INT32_C(0x7FFFFF80));
0037: 
0038:   /* Shift is in [0, 31] range */
0039:   const int32_t shift = 127 + 31 - 32 - (fp32_to_bits(scale) >> 23);
0040:   assert(shift >= 0);
0041:   assert(shift < 32);
0042: 
0043:   const __m128i vmultiplier = _mm_set1_epi32(multiplier);
0044:   const __m128i vzero_point = _mm_set1_epi16((short)(uint16_t)zero_point);
0045:   const __m128i vqmin = _mm_set1_epi8((char)qmin);
0046:   const __m128i vqmax = _mm_set1_epi8((char)qmax);
0047:   const __m128i vshift = _mm_cvtsi32_si128((int)shift);
0048:   const uint32_t remainder_mask = (UINT32_C(1) << shift) - UINT32_C(1);
0049:   const __m128i vremainder_mask = _mm_set1_epi32((int)remainder_mask);
0050:   const __m128i vthreshold = _mm_set1_epi32((int)(remainder_mask >> 1));
0051:   const __m128i vq31rounding = _mm_set1_epi64x(UINT64_C(0x40000000));
0052:   for (; n != 0; n -= 16) {
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state; validates runtime invariants before continuing. Key symbols: `assert`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态；在继续执行前校验运行时不变量。关键符号：`assert`。

### Lines 53-72 / 第 53-72 行

```c
0053:     const __m128i x = _mm_loadu_si128((const __m128i*)input);
0054:     const __m128i y = _mm_loadu_si128((const __m128i*)(input + 4));
0055:     const __m128i z = _mm_loadu_si128((const __m128i*)(input + 8));
0056:     const __m128i w = _mm_loadu_si128((const __m128i*)(input + 12));
0057:     input += 16;
0058: 
0059:     const __m128i x_neg_mask = _mm_cmpgt_epi32(_mm_setzero_si128(), x);
0060:     const __m128i y_neg_mask = _mm_cmpgt_epi32(_mm_setzero_si128(), y);
0061:     const __m128i z_neg_mask = _mm_cmpgt_epi32(_mm_setzero_si128(), z);
0062:     const __m128i w_neg_mask = _mm_cmpgt_epi32(_mm_setzero_si128(), w);
0063: 
0064:     const __m128i x_abs =
0065:         _mm_sub_epi32(_mm_xor_si128(x, x_neg_mask), x_neg_mask);
0066:     const __m128i y_abs =
0067:         _mm_sub_epi32(_mm_xor_si128(y, y_neg_mask), y_neg_mask);
0068:     const __m128i z_abs =
0069:         _mm_sub_epi32(_mm_xor_si128(z, z_neg_mask), z_neg_mask);
0070:     const __m128i w_abs =
0071:         _mm_sub_epi32(_mm_xor_si128(w, w_neg_mask), w_neg_mask);
0072: 
```

- **EN:** This block implements local helper logic for `q31-sse2`. Key symbols: `_mm_sub_epi32`.
- **CN:** 该代码块实现与 `q31-sse2` 相关的局部辅助逻辑。关键符号：`_mm_sub_epi32`。

### Lines 73-91 / 第 73-91 行

```c
0073:     const __m128i x_abs_rev = _mm_shuffle_epi32(x_abs, _MM_SHUFFLE(2, 3, 0, 1));
0074:     const __m128i y_abs_rev = _mm_shuffle_epi32(y_abs, _MM_SHUFFLE(2, 3, 0, 1));
0075:     const __m128i z_abs_rev = _mm_shuffle_epi32(z_abs, _MM_SHUFFLE(2, 3, 0, 1));
0076:     const __m128i w_abs_rev = _mm_shuffle_epi32(w_abs, _MM_SHUFFLE(2, 3, 0, 1));
0077: 
0078:     const __m128i x_abs_product_even = _mm_mul_epu32(x_abs, vmultiplier);
0079:     const __m128i y_abs_product_even = _mm_mul_epu32(y_abs, vmultiplier);
0080:     const __m128i z_abs_product_even = _mm_mul_epu32(z_abs, vmultiplier);
0081:     const __m128i w_abs_product_even = _mm_mul_epu32(w_abs, vmultiplier);
0082: 
0083:     const __m128i x_neg_mask_even =
0084:         _mm_shuffle_epi32(x_neg_mask, _MM_SHUFFLE(2, 2, 0, 0));
0085:     const __m128i y_neg_mask_even =
0086:         _mm_shuffle_epi32(y_neg_mask, _MM_SHUFFLE(2, 2, 0, 0));
0087:     const __m128i z_neg_mask_even =
0088:         _mm_shuffle_epi32(z_neg_mask, _MM_SHUFFLE(2, 2, 0, 0));
0089:     const __m128i w_neg_mask_even =
0090:         _mm_shuffle_epi32(w_neg_mask, _MM_SHUFFLE(2, 2, 0, 0));
0091: 
```

- **EN:** This block implements local helper logic for `q31-sse2`. Key symbols: `_mm_shuffle_epi32`.
- **CN:** 该代码块实现与 `q31-sse2` 相关的局部辅助逻辑。关键符号：`_mm_shuffle_epi32`。

### Lines 92-109 / 第 92-109 行

```c
0092:     const __m128i x_product_even = _mm_sub_epi64(
0093:         _mm_xor_si128(x_abs_product_even, x_neg_mask_even), x_neg_mask_even);
0094:     const __m128i y_product_even = _mm_sub_epi64(
0095:         _mm_xor_si128(y_abs_product_even, y_neg_mask_even), y_neg_mask_even);
0096:     const __m128i z_product_even = _mm_sub_epi64(
0097:         _mm_xor_si128(z_abs_product_even, z_neg_mask_even), z_neg_mask_even);
0098:     const __m128i w_product_even = _mm_sub_epi64(
0099:         _mm_xor_si128(w_abs_product_even, w_neg_mask_even), w_neg_mask_even);
0100: 
0101:     const __m128i x_rounded_product_even =
0102:         _mm_add_epi64(x_product_even, vq31rounding);
0103:     const __m128i y_rounded_product_even =
0104:         _mm_add_epi64(y_product_even, vq31rounding);
0105:     const __m128i z_rounded_product_even =
0106:         _mm_add_epi64(z_product_even, vq31rounding);
0107:     const __m128i w_rounded_product_even =
0108:         _mm_add_epi64(w_product_even, vq31rounding);
0109: 
```

- **EN:** This block implements local helper logic for `q31-sse2`. Key symbols: `_mm_xor_si128`, `_mm_add_epi64`.
- **CN:** 该代码块实现与 `q31-sse2` 相关的局部辅助逻辑。关键符号：`_mm_xor_si128`, `_mm_add_epi64`。

### Lines 110-123 / 第 110-123 行

```c
0110:     const __m128i x_abs_product_odd = _mm_mul_epu32(x_abs_rev, vmultiplier);
0111:     const __m128i y_abs_product_odd = _mm_mul_epu32(y_abs_rev, vmultiplier);
0112:     const __m128i z_abs_product_odd = _mm_mul_epu32(z_abs_rev, vmultiplier);
0113:     const __m128i w_abs_product_odd = _mm_mul_epu32(w_abs_rev, vmultiplier);
0114: 
0115:     const __m128i x_neg_mask_odd =
0116:         _mm_shuffle_epi32(x_neg_mask, _MM_SHUFFLE(3, 3, 1, 1));
0117:     const __m128i y_neg_mask_odd =
0118:         _mm_shuffle_epi32(y_neg_mask, _MM_SHUFFLE(3, 3, 1, 1));
0119:     const __m128i z_neg_mask_odd =
0120:         _mm_shuffle_epi32(z_neg_mask, _MM_SHUFFLE(3, 3, 1, 1));
0121:     const __m128i w_neg_mask_odd =
0122:         _mm_shuffle_epi32(w_neg_mask, _MM_SHUFFLE(3, 3, 1, 1));
0123: 
```

- **EN:** This block implements local helper logic for `q31-sse2`. Key symbols: `_mm_shuffle_epi32`.
- **CN:** 该代码块实现与 `q31-sse2` 相关的局部辅助逻辑。关键符号：`_mm_shuffle_epi32`。

### Lines 124-141 / 第 124-141 行

```c
0124:     const __m128i x_product_odd = _mm_sub_epi64(
0125:         _mm_xor_si128(x_abs_product_odd, x_neg_mask_odd), x_neg_mask_odd);
0126:     const __m128i y_product_odd = _mm_sub_epi64(
0127:         _mm_xor_si128(y_abs_product_odd, y_neg_mask_odd), y_neg_mask_odd);
0128:     const __m128i z_product_odd = _mm_sub_epi64(
0129:         _mm_xor_si128(z_abs_product_odd, z_neg_mask_odd), z_neg_mask_odd);
0130:     const __m128i w_product_odd = _mm_sub_epi64(
0131:         _mm_xor_si128(w_abs_product_odd, w_neg_mask_odd), w_neg_mask_odd);
0132: 
0133:     const __m128i x_rounded_product_odd =
0134:         _mm_add_epi64(x_product_odd, vq31rounding);
0135:     const __m128i y_rounded_product_odd =
0136:         _mm_add_epi64(y_product_odd, vq31rounding);
0137:     const __m128i z_rounded_product_odd =
0138:         _mm_add_epi64(z_product_odd, vq31rounding);
0139:     const __m128i w_rounded_product_odd =
0140:         _mm_add_epi64(w_product_odd, vq31rounding);
0141: 
```

- **EN:** This block implements local helper logic for `q31-sse2`. Key symbols: `_mm_xor_si128`, `_mm_add_epi64`.
- **CN:** 该代码块实现与 `q31-sse2` 相关的局部辅助逻辑。关键符号：`_mm_xor_si128`, `_mm_add_epi64`。

### Lines 142-169 / 第 142-169 行

```c
0142:     const __m128i x_q31product_even =
0143:         _mm_srli_epi64(x_rounded_product_even, 31);
0144:     const __m128i x_q31product_odd = _mm_srli_epi64(x_rounded_product_odd, 31);
0145:     const __m128i y_q31product_even =
0146:         _mm_srli_epi64(y_rounded_product_even, 31);
0147:     const __m128i y_q31product_odd = _mm_srli_epi64(y_rounded_product_odd, 31);
0148:     const __m128i z_q31product_even =
0149:         _mm_srli_epi64(z_rounded_product_even, 31);
0150:     const __m128i z_q31product_odd = _mm_srli_epi64(z_rounded_product_odd, 31);
0151:     const __m128i w_q31product_even =
0152:         _mm_srli_epi64(w_rounded_product_even, 31);
0153:     const __m128i w_q31product_odd = _mm_srli_epi64(w_rounded_product_odd, 31);
0154: 
0155:     const __m128i x_q31product_0213 = _mm_castps_si128(_mm_shuffle_ps(
0156:         _mm_castsi128_ps(x_q31product_even),
0157:         _mm_castsi128_ps(x_q31product_odd),
0158:         _MM_SHUFFLE(2, 0, 2, 0)));
0159:     const __m128i y_q31product_0213 = _mm_castps_si128(_mm_shuffle_ps(
0160:         _mm_castsi128_ps(y_q31product_even),
0161:         _mm_castsi128_ps(y_q31product_odd),
0162:         _MM_SHUFFLE(2, 0, 2, 0)));
0163:     const __m128i z_q31product_0213 = _mm_castps_si128(_mm_shuffle_ps(
0164:         _mm_castsi128_ps(z_q31product_even),
0165:         _mm_castsi128_ps(z_q31product_odd),
0166:         _MM_SHUFFLE(2, 0, 2, 0)));
0167:     const __m128i w_q31product_0213 = _mm_castps_si128(_mm_shuffle_ps(
0168:         _mm_castsi128_ps(w_q31product_even),
0169:         _mm_castsi128_ps(w_q31product_odd),
```

- **EN:** This block implements local helper logic for `q31-sse2`. Key symbols: `_mm_srli_epi64`, `_mm_castsi128_ps`.
- **CN:** 该代码块实现与 `q31-sse2` 相关的局部辅助逻辑。关键符号：`_mm_srli_epi64`, `_mm_castsi128_ps`。

### Lines 170-193 / 第 170-193 行

```c
0170:         _MM_SHUFFLE(2, 0, 2, 0)));
0171: 
0172:     const __m128i x_q31product =
0173:         _mm_shuffle_epi32(x_q31product_0213, _MM_SHUFFLE(3, 1, 2, 0));
0174:     const __m128i y_q31product =
0175:         _mm_shuffle_epi32(y_q31product_0213, _MM_SHUFFLE(3, 1, 2, 0));
0176:     const __m128i z_q31product =
0177:         _mm_shuffle_epi32(z_q31product_0213, _MM_SHUFFLE(3, 1, 2, 0));
0178:     const __m128i w_q31product =
0179:         _mm_shuffle_epi32(w_q31product_0213, _MM_SHUFFLE(3, 1, 2, 0));
0180: 
0181:     const __m128i x_remainder = _mm_add_epi32(
0182:         _mm_and_si128(x_q31product, vremainder_mask),
0183:         _mm_cmpgt_epi32(_mm_setzero_si128(), x_q31product));
0184:     const __m128i y_remainder = _mm_add_epi32(
0185:         _mm_and_si128(y_q31product, vremainder_mask),
0186:         _mm_cmpgt_epi32(_mm_setzero_si128(), y_q31product));
0187:     const __m128i z_remainder = _mm_add_epi32(
0188:         _mm_and_si128(z_q31product, vremainder_mask),
0189:         _mm_cmpgt_epi32(_mm_setzero_si128(), z_q31product));
0190:     const __m128i w_remainder = _mm_add_epi32(
0191:         _mm_and_si128(w_q31product, vremainder_mask),
0192:         _mm_cmpgt_epi32(_mm_setzero_si128(), w_q31product));
0193: 
```

- **EN:** This block implements local helper logic for `q31-sse2`. Key symbols: `_mm_shuffle_epi32`, `_mm_and_si128`.
- **CN:** 该代码块实现与 `q31-sse2` 相关的局部辅助逻辑。关键符号：`_mm_shuffle_epi32`, `_mm_and_si128`。

### Lines 194-214 / 第 194-214 行

```c
0194:     const __m128i x_scaled = _mm_sub_epi32(
0195:         _mm_sra_epi32(x_q31product, vshift),
0196:         _mm_cmpgt_epi32(x_remainder, vthreshold));
0197:     const __m128i y_scaled = _mm_sub_epi32(
0198:         _mm_sra_epi32(y_q31product, vshift),
0199:         _mm_cmpgt_epi32(y_remainder, vthreshold));
0200:     const __m128i z_scaled = _mm_sub_epi32(
0201:         _mm_sra_epi32(z_q31product, vshift),
0202:         _mm_cmpgt_epi32(z_remainder, vthreshold));
0203:     const __m128i w_scaled = _mm_sub_epi32(
0204:         _mm_sra_epi32(w_q31product, vshift),
0205:         _mm_cmpgt_epi32(w_remainder, vthreshold));
0206: 
0207:     const __m128i xy_packed =
0208:         _mm_adds_epi16(_mm_packs_epi32(x_scaled, y_scaled), vzero_point);
0209:     const __m128i zw_packed =
0210:         _mm_adds_epi16(_mm_packs_epi32(z_scaled, w_scaled), vzero_point);
0211:     const __m128i xyzw_packed = _mm_packus_epi16(xy_packed, zw_packed);
0212:     const __m128i xyzw_clamped =
0213:         _mm_max_epu8(_mm_min_epu8(xyzw_packed, vqmax), vqmin);
0214: 
```

- **EN:** This block implements local helper logic for `q31-sse2`. Key symbols: `_mm_sra_epi32`, `_mm_adds_epi16`, `_mm_max_epu8`.
- **CN:** 该代码块实现与 `q31-sse2` 相关的局部辅助逻辑。关键符号：`_mm_sra_epi32`, `_mm_adds_epi16`, `_mm_max_epu8`。

### Lines 215-237 / 第 215-237 行

```c
0215:     /*
0216:      * 16x PSHUFD
0217:      * 4x SHUFPS
0218:      * 8x PMULUDQ
0219:      * 8x PXOR (setzero)
0220:      * 12x PXOR
0221:      * 4x PAND
0222:      * 8x PADDQ
0223:      * 4x PADDD
0224:      * 2x PADDW
0225:      * 8x PSUBQ
0226:      * 8x PSUBD
0227:      * 8x PSRLQ (immediate)
0228:      * 4x PSRAD (register)
0229:      * 12x PCMPGTD
0230:      * 2x PACKSSDW
0231:      * 1x PACKUSWB
0232:      * 1x PMAXUB
0233:      * 1x PMINUB
0234:      * ---------------------
0235:      * 111 instructions total
0236:      */
0237: 
```

- **EN:** Documents design intent, constraints, compatibility notes, or usage expectations for the surrounding implementation.
- **CN:** 说明周围实现的设计目标、约束条件、兼容性注意事项或使用预期。

### Lines 238-241 / 第 238-241 行

```c
0238:     _mm_storeu_si128((__m128i*)output, xyzw_clamped);
0239:     output += 16;
0240:   }
0241: }
```

- **EN:** This block implements local helper logic for `q31-sse2`. Key symbols: `_mm_storeu_si128`.
- **CN:** 该代码块实现与 `q31-sse2` 相关的局部辅助逻辑。关键符号：`_mm_storeu_si128`。


## Key Concepts / 关键概念
- **ATen native operator logic** — ATen 原生算子逻辑
- **Dispatch and backend routing** — 调度与后端路由
- **Mixed-precision/autocast logic** — 混合精度/自动混精逻辑
- **Runtime validation and invariants** — 运行时校验与不变量
- **Core symbols: pytorch_qnnp_requantize_q31__sse2, assert, _mm_sub_epi32, _mm_shuffle_epi32, _mm_xor_si128, _mm_add_epi64, _mm_srli_epi64, _mm_castsi128_ps** — 核心符号：pytorch_qnnp_requantize_q31__sse2、assert、_mm_sub_epi32、_mm_shuffle_epi32、_mm_xor_si128、_mm_add_epi64、_mm_srli_epi64、_mm_castsi128_ps

## Dependencies / 依赖关系
- **Internal includes / 内部头文件**: 无
- **External includes / 外部头文件**: `assert.h`, `stdint.h`, `emmintrin.h`, `fp16/bitcasts.h`, `qnnpack/requantization-stubs.h`
- **Namespaces / 命名空间**: 无
- **Representative symbols / 代表性符号**: `pytorch_qnnp_requantize_q31__sse2`, `assert`, `_mm_sub_epi32`, `_mm_shuffle_epi32`, `_mm_xor_si128`, `_mm_add_epi64`, `_mm_srli_epi64`, `_mm_castsi128_ps`, `_mm_and_si128`, `_mm_sra_epi32`, `_mm_adds_epi16`, `_mm_max_epu8`, `...`
