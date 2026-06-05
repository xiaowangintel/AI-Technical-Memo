# 2x4c8-sse2.c — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `aten/src/ATen/native/quantized/cpu/qnnpack/src/q8gemm/2x4c8-sse2.c`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements ATen native operators, checks, helper kernels, and backend dispatch behavior. This specific file centers on `2x4c8-sse2.c`. The file header highlights: "All rights reserved. This source code is licensed under the BSD-style license found in the LICENSE file in the root directory of this source tree.." Matrix multiplication and GEMM-style kernels are a central concern.
- **Purpose (CN)**: 实现 ATen 原生算子、运行时检查、辅助内核以及后端分发行为。 该文件具体围绕 `2x4c8-sse2.c` 展开。 文件头部注释也概括了其核心职责。 矩阵乘法与 GEMM 风格内核是该文件的核心关注点。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14 / 第 1-14 行

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
0011: #include <qnnpack/q8gemm.h>
0012: #include <requantization/runtime-sse2.h>
0013: 
0014: static inline __m128i pytorch_sse_reduce4_i32(
```

- **EN:** This block implements scan/reduction-style dataflow. Key symbols: no prominent local symbols.
- **CN:** 该代码块实现扫描/归约式数据流。关键符号：无明显局部符号。

### Lines 15-36 / 第 15-36 行

```c
0015:     __m128i x,
0016:     __m128i y,
0017:     __m128i z,
0018:     __m128i w) {
0019: #if defined(__SSSE3__) && !defined(__ANDROID__)
0020:   /* xxyy = ( y2 + y3, y0 + y1, x2 + x3, x0 + x1 ) */
0021:   const __m128i xxyy = _mm_hadd_epi32(x, y);
0022:   /* zzww = ( w2 + w3, w0 + w1, z2 + z3, z0 + z1 ) */
0023:   const __m128i zzww = _mm_hadd_epi32(z, w);
0024:   /* xyzw = ( w0 + w1 + w2 + w3, y0 + y1 + y2 + y3, z0 + z1 + z2 + z3, x0 + x1 +
0025:    * x2 + x3 ) */
0026:   return _mm_hadd_epi32(xxyy, zzww);
0027: #else
0028:   /* xzxz = ( z1 + z3, x1 + x3, z0 + z2, x0 + x2 ) */
0029:   const __m128i xzxz =
0030:       _mm_add_epi32(_mm_unpacklo_epi32(x, z), _mm_unpackhi_epi32(x, z));
0031:   /* ywyw = ( w1 + w3, y1 + y3, w0 + w2, y0 + y2 ) */
0032:   const __m128i ywyw =
0033:       _mm_add_epi32(_mm_unpacklo_epi32(y, w), _mm_unpackhi_epi32(y, w));
0034:   /* xyzw = ( w0 + w2 + w1 + w3, y0 + y2 + y1 + y3, z0 + z2 + z1 + z3, x0 + x2 +
0035:    * x1 + x3 ) */
0036:   return _mm_add_epi32(
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends; produces a result or forwards a computed value. Key symbols: `_mm_hadd_epi32`, `_mm_add_epi32`.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端；返回结果或转发已计算的值。关键符号：`_mm_hadd_epi32`, `_mm_add_epi32`。

### Lines 37-62 / 第 37-62 行

```c
0037:       _mm_unpacklo_epi32(xzxz, ywyw), _mm_unpackhi_epi32(xzxz, ywyw));
0038: #endif
0039: }
0040: 
0041: void pytorch_q8gemm_ukernel_2x4c8__sse2(
0042:     size_t mr,
0043:     size_t nr,
0044:     size_t k,
0045:     const uint8_t* restrict a,
0046:     size_t a_stride,
0047:     const void* restrict w,
0048:     uint8_t* restrict c,
0049:     size_t c_stride,
0050:     size_t output_channel_index,
0051:     const union pytorch_qnnp_conv_quantization_params
0052:         quantization_params[RESTRICT_STATIC 1]) {
0053:   __m128i vacc00 = _mm_cvtsi32_si128((int)((const int32_t*)w)[0]);
0054:   __m128i vacc01 = _mm_cvtsi32_si128((int)((const int32_t*)w)[1]);
0055:   __m128i vacc02 = _mm_cvtsi32_si128((int)((const int32_t*)w)[2]);
0056:   __m128i vacc03 = _mm_cvtsi32_si128((int)((const int32_t*)w)[3]);
0057:   __m128i vacc10 = vacc00;
0058:   __m128i vacc11 = vacc01;
0059:   __m128i vacc12 = vacc02;
0060:   __m128i vacc13 = vacc03;
0061:   w = (const void*)((uintptr_t)w + 16);
0062: 
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends. Key symbols: `_mm_unpacklo_epi32`, `pytorch_q8gemm_ukernel_2x4c8__sse2`.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端。关键符号：`_mm_unpacklo_epi32`, `pytorch_q8gemm_ukernel_2x4c8__sse2`。

### Lines 63-79 / 第 63-79 行

```c
0063:   const uint8_t* a0 = a;
0064:   const uint8_t* a1 = (const uint8_t*)((uintptr_t)a0 + a_stride);
0065:   if (mr != 2) {
0066:     a1 = a0;
0067:   }
0068: 
0069:   const uint8_t* b0 = w;
0070:   const uint8_t* b1 = b0 + 8;
0071:   if (nr < 2) {
0072:     b1 = b0;
0073:   }
0074:   const uint8_t* b2 = b1 + 8;
0075:   if (nr <= 2) {
0076:     b2 = b1;
0077:   }
0078:   const uint8_t* b3 = b2 + 8;
0079:   if (nr != 4) {
```

- **EN:** This block handles conditional branches and special cases. Key symbols: no prominent local symbols.
- **CN:** 该代码块处理条件分支与特殊情况。关键符号：无明显局部符号。

### Lines 80-100 / 第 80-100 行

```c
0080:     b3 = b2;
0081:   }
0082:   const size_t b_stride = nr * 8;
0083: 
0084:   const __m128i va_zero_point = _mm_load_si128(
0085:       (const __m128i*)quantization_params->sse2.input_zero_point);
0086:   const __m128i vb_zero_point_0 = _mm_set1_epi16(
0087:       (int16_t)(uint16_t)quantization_params->sse2.kernel_zero_points[
0088:         output_channel_index]);
0089:   // Assumes kernel_zero_point allocated memory is always multiple of nr=4.
0090:   const __m128i vb_zero_point_1 = _mm_set1_epi16(
0091:       (int16_t)(uint16_t)quantization_params->sse2.kernel_zero_points[
0092:         output_channel_index + 1]);
0093:   const __m128i vb_zero_point_2 = _mm_set1_epi16(
0094:       (int16_t)(uint16_t)quantization_params->sse2.kernel_zero_points[
0095:         output_channel_index + 2]);
0096:   const __m128i vb_zero_point_3 = _mm_set1_epi16(
0097:       (int16_t)(uint16_t)quantization_params->sse2.kernel_zero_points[
0098:         output_channel_index + 3]);
0099:   const __m128i vzero = _mm_setzero_si128();
0100:   for (; k >= 8; k -= 8) {
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: no prominent local symbols.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：无明显局部符号。

### Lines 101-126 / 第 101-126 行

```c
0101:     const __m128i va0 = _mm_loadl_epi64((const __m128i*)a0);
0102:     const __m128i vxa0 =
0103:         sub_zero_point(_mm_unpacklo_epi8(va0, vzero), va_zero_point);
0104:     a0 += 8;
0105:     const __m128i va1 = _mm_loadl_epi64((const __m128i*)a1);
0106:     const __m128i vxa1 =
0107:         sub_zero_point(_mm_unpacklo_epi8(va1, vzero), va_zero_point);
0108:     a1 += 8;
0109: 
0110:     const __m128i vb0 = _mm_loadl_epi64((const __m128i*)b0);
0111:     const __m128i vxb0 =
0112:         _mm_sub_epi16(_mm_unpacklo_epi8(vb0, vzero), vb_zero_point_0);
0113:     b0 += b_stride;
0114:     const __m128i vb1 = _mm_loadl_epi64((const __m128i*)b1);
0115:     const __m128i vxb1 =
0116:         _mm_sub_epi16(_mm_unpacklo_epi8(vb1, vzero), vb_zero_point_1);
0117:     b1 += b_stride;
0118:     const __m128i vb2 = _mm_loadl_epi64((const __m128i*)b2);
0119:     const __m128i vxb2 =
0120:         _mm_sub_epi16(_mm_unpacklo_epi8(vb2, vzero), vb_zero_point_2);
0121:     b2 += b_stride;
0122:     const __m128i vb3 = _mm_loadl_epi64((const __m128i*)b3);
0123:     const __m128i vxb3 =
0124:         _mm_sub_epi16(_mm_unpacklo_epi8(vb3, vzero), vb_zero_point_3);
0125:     b3 += b_stride;
0126: 
```

- **EN:** This block implements local helper logic for `2x4c8-sse2`. Key symbols: `sub_zero_point`, `_mm_sub_epi16`.
- **CN:** 该代码块实现与 `2x4c8-sse2` 相关的局部辅助逻辑。关键符号：`sub_zero_point`, `_mm_sub_epi16`。

### Lines 127-143 / 第 127-143 行

```c
0127:     vacc00 = _mm_add_epi32(vacc00, _mm_madd_epi16(vxa0, vxb0));
0128:     vacc01 = _mm_add_epi32(vacc01, _mm_madd_epi16(vxa0, vxb1));
0129:     vacc02 = _mm_add_epi32(vacc02, _mm_madd_epi16(vxa0, vxb2));
0130:     vacc03 = _mm_add_epi32(vacc03, _mm_madd_epi16(vxa0, vxb3));
0131:     vacc10 = _mm_add_epi32(vacc10, _mm_madd_epi16(vxa1, vxb0));
0132:     vacc11 = _mm_add_epi32(vacc11, _mm_madd_epi16(vxa1, vxb1));
0133:     vacc12 = _mm_add_epi32(vacc12, _mm_madd_epi16(vxa1, vxb2));
0134:     vacc13 = _mm_add_epi32(vacc13, _mm_madd_epi16(vxa1, vxb3));
0135:   }
0136:   if (k != 0) {
0137:     const size_t a_predecrement = 8 - k;
0138:     const __m128i va_shift = _mm_cvtsi32_si128(8 * a_predecrement);
0139: 
0140:     const __m128i va_zero_point_partial = _mm_unpacklo_epi8(
0141:         _mm_srl_epi64(_mm_packus_epi16(va_zero_point, va_zero_point), va_shift),
0142:         vzero);
0143: 
```

- **EN:** This block handles conditional branches and special cases. Key symbols: `_mm_srl_epi64`.
- **CN:** 该代码块处理条件分支与特殊情况。关键符号：`_mm_srl_epi64`。

### Lines 144-165 / 第 144-165 行

```c
0144:     const __m128i va0 = _mm_srl_epi64(
0145:         _mm_loadl_epi64((const __m128i*)(a0 - a_predecrement)), va_shift);
0146:     const __m128i vxa0 =
0147:         sub_zero_point(_mm_unpacklo_epi8(va0, vzero), va_zero_point_partial);
0148:     const __m128i va1 = _mm_srl_epi64(
0149:         _mm_loadl_epi64((const __m128i*)(a1 - a_predecrement)), va_shift);
0150:     const __m128i vxa1 =
0151:         sub_zero_point(_mm_unpacklo_epi8(va1, vzero), va_zero_point_partial);
0152: 
0153:     const __m128i vb0 = _mm_loadl_epi64((const __m128i*)b0);
0154:     const __m128i vxb0 =
0155:         _mm_sub_epi16(_mm_unpacklo_epi8(vb0, vzero), vb_zero_point_0);
0156:     const __m128i vb1 = _mm_loadl_epi64((const __m128i*)b1);
0157:     const __m128i vxb1 =
0158:         _mm_sub_epi16(_mm_unpacklo_epi8(vb1, vzero), vb_zero_point_1);
0159:     const __m128i vb2 = _mm_loadl_epi64((const __m128i*)b2);
0160:     const __m128i vxb2 =
0161:         _mm_sub_epi16(_mm_unpacklo_epi8(vb2, vzero), vb_zero_point_2);
0162:     const __m128i vb3 = _mm_loadl_epi64((const __m128i*)b3);
0163:     const __m128i vxb3 =
0164:         _mm_sub_epi16(_mm_unpacklo_epi8(vb3, vzero), vb_zero_point_3);
0165: 
```

- **EN:** This block implements local helper logic for `2x4c8-sse2`. Key symbols: `_mm_loadl_epi64`, `sub_zero_point`, `_mm_sub_epi16`.
- **CN:** 该代码块实现与 `2x4c8-sse2` 相关的局部辅助逻辑。关键符号：`_mm_loadl_epi64`, `sub_zero_point`, `_mm_sub_epi16`。

### Lines 166-182 / 第 166-182 行

```c
0166:     vacc00 = _mm_add_epi32(vacc00, _mm_madd_epi16(vxa0, vxb0));
0167:     vacc01 = _mm_add_epi32(vacc01, _mm_madd_epi16(vxa0, vxb1));
0168:     vacc02 = _mm_add_epi32(vacc02, _mm_madd_epi16(vxa0, vxb2));
0169:     vacc03 = _mm_add_epi32(vacc03, _mm_madd_epi16(vxa0, vxb3));
0170:     vacc10 = _mm_add_epi32(vacc10, _mm_madd_epi16(vxa1, vxb0));
0171:     vacc11 = _mm_add_epi32(vacc11, _mm_madd_epi16(vxa1, vxb1));
0172:     vacc12 = _mm_add_epi32(vacc12, _mm_madd_epi16(vxa1, vxb2));
0173:     vacc13 = _mm_add_epi32(vacc13, _mm_madd_epi16(vxa1, vxb3));
0174:   }
0175: 
0176:   __m128i vacc0x0123 = pytorch_sse_reduce4_i32(vacc00, vacc01, vacc02, vacc03);
0177:   __m128i vacc1x0123 = pytorch_sse_reduce4_i32(vacc10, vacc11, vacc12, vacc13);
0178: 
0179:   const __m128 vmultiplier =
0180:       _mm_loadu_ps(&quantization_params->sse2.requantization_scales
0181:           [output_channel_index]);
0182: 
```

- **EN:** This block implements scan/reduction-style dataflow. Key symbols: `_mm_loadu_ps`.
- **CN:** 该代码块实现扫描/归约式数据流。关键符号：`_mm_loadu_ps`。

### Lines 183-207 / 第 183-207 行

```c
0183:   vacc0x0123 = _mm_cvtps_epi32(
0184:                 _mm_mul_ps(
0185:                   _mm_cvtepi32_ps(vacc0x0123),
0186:                   vmultiplier
0187:                   )
0188:                 );
0189:   vacc1x0123 = _mm_cvtps_epi32(
0190:                 _mm_mul_ps(
0191:                   _mm_cvtepi32_ps(vacc1x0123),
0192:                   vmultiplier
0193:                   )
0194:                 );
0195: 
0196:   const __m128i voutput_zero_point = _mm_load_si128(
0197:       (const __m128i*)quantization_params->sse2.output_zero_point);
0198:   const __m128i vacc01x0123 = _mm_adds_epi16(
0199:       _mm_packs_epi32(vacc0x0123, vacc1x0123), voutput_zero_point);
0200:   __m128i vout = _mm_packus_epi16(vacc01x0123, vacc01x0123);
0201:   vout = _mm_min_epu8(
0202:       vout,
0203:       _mm_load_si128((const __m128i*)quantization_params->sse2.output_max));
0204:   vout = _mm_max_epu8(
0205:       vout,
0206:       _mm_load_si128((const __m128i*)quantization_params->sse2.output_min));
0207: 
```

- **EN:** This block implements local helper logic for `2x4c8-sse2`. Key symbols: `_mm_mul_ps`, `_mm_packs_epi32`, `_mm_load_si128`.
- **CN:** 该代码块实现与 `2x4c8-sse2` 相关的局部辅助逻辑。关键符号：`_mm_mul_ps`, `_mm_packs_epi32`, `_mm_load_si128`。

### Lines 208-225 / 第 208-225 行

```c
0208:   uint8_t* c0 = c;
0209:   uint8_t* c1 = (uint8_t*)((uintptr_t)c0 + c_stride);
0210:   if (mr != 2) {
0211:     c1 = c0;
0212:   }
0213:   if (nr == 4) {
0214:     *((uint32_t*)c0) = (uint32_t)_mm_cvtsi128_si32(vout);
0215:     *((uint32_t*)c1) = (uint32_t)_mm_cvtsi128_si32(_mm_srli_epi64(vout, 32));
0216:   } else {
0217:     if (nr >= 2) {
0218:       *((uint16_t*)c0) = (uint16_t)_mm_extract_epi16(vout, 0);
0219:       c0 += 2;
0220:       *((uint16_t*)c1) = (uint16_t)_mm_extract_epi16(vout, 2);
0221:       c1 += 2;
0222:       vout = _mm_srli_epi32(vout, 16);
0223:       nr -= 2;
0224:     }
0225:     if (nr != 0) {
```

- **EN:** This block handles conditional branches and special cases. Key symbols: no prominent local symbols.
- **CN:** 该代码块处理条件分支与特殊情况。关键符号：无明显局部符号。

### Lines 226-230 / 第 226-230 行

```c
0226:       *((uint8_t*)c0) = (uint8_t)_mm_cvtsi128_si32(vout);
0227:       *((uint8_t*)c1) = (uint8_t)_mm_extract_epi16(vout, 2);
0228:     }
0229:   }
0230: }
```

- **EN:** This block implements local helper logic for `2x4c8-sse2`. Key symbols: no prominent local symbols.
- **CN:** 该代码块实现与 `2x4c8-sse2` 相关的局部辅助逻辑。关键符号：无明显局部符号。


## Key Concepts / 关键概念
- **ATen native operator logic** — ATen 原生算子逻辑
- **GPU kernel/runtime interaction** — GPU 内核/运行时交互
- **Linear algebra backend integration** — 线性代数后端集成
- **Memory allocation strategy** — 内存分配策略
- **Parallel scan/reduction behavior** — 并行扫描/归约行为
- **Core symbols: pytorch_sse_reduce4_i32, _mm_hadd_epi32, _mm_add_epi32, pytorch_q8gemm_ukernel_2x4c8__sse2, sub_zero_point, _mm_sub_epi16, _mm_srl_epi64, _mm_loadl_epi64** — 核心符号：pytorch_sse_reduce4_i32、_mm_hadd_epi32、_mm_add_epi32、pytorch_q8gemm_ukernel_2x4c8__sse2、sub_zero_point、_mm_sub_epi16、_mm_srl_epi64、_mm_loadl_epi64

## Dependencies / 依赖关系
- **Internal includes / 内部头文件**: 无
- **External includes / 外部头文件**: `immintrin.h`, `qnnpack/q8gemm.h`, `requantization/runtime-sse2.h`
- **Namespaces / 命名空间**: 无
- **Representative symbols / 代表性符号**: `pytorch_sse_reduce4_i32`, `_mm_hadd_epi32`, `_mm_add_epi32`, `pytorch_q8gemm_ukernel_2x4c8__sse2`, `sub_zero_point`, `_mm_sub_epi16`, `_mm_srl_epi64`, `_mm_loadl_epi64`, `_mm_loadu_ps`, `_mm_mul_ps`, `_mm_packs_epi32`, `_mm_load_si128`
