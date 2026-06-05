# 6x4-neon.c — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `aten/src/ATen/native/quantized/cpu/qnnpack/src/q8gemm/6x4-neon.c`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements ATen native operators, checks, helper kernels, and backend dispatch behavior. This specific file centers on `6x4-neon.c`. The file header highlights: "All rights reserved. This source code is licensed under the BSD-style license found in the LICENSE file in the root directory of this source tree.." Matrix multiplication and GEMM-style kernels are a central concern.
- **Purpose (CN)**: 实现 ATen 原生算子、运行时检查、辅助内核以及后端分发行为。 该文件具体围绕 `6x4-neon.c` 展开。 文件头部注释也概括了其核心职责。 矩阵乘法与 GEMM 风格内核是该文件的核心关注点。

## Line-by-Line Analysis / 逐行分析
### Lines 1-33 / 第 1-33 行

```c
0001: /*
0002:  * Copyright (c) Facebook, Inc. and its affiliates.
0003:  * All rights reserved.
0004:  *
0005:  * This source code is licensed under the BSD-style license found in the
0006:  * LICENSE file in the root directory of this source tree.
0007:  */
0008: 
0009: #include <arm_neon.h>
0010: 
0011: #include <qnnpack/q8gemm.h>
0012: #include <requantization/runtime-neon.h>
0013: 
0014: void pytorch_q8gemm_ukernel_6x4__neon(
0015:     size_t mr,
0016:     size_t nr,
0017:     size_t k,
0018:     const uint8_t* restrict a,
0019:     size_t a_stride,
0020:     const void* restrict w,
0021:     uint8_t* restrict c,
0022:     size_t c_stride,
0023:     size_t output_channel_index,
0024:     const union pytorch_qnnp_conv_quantization_params
0025:         quantization_params[restrict static 1]) {
0026:   int32x4_t vacc0x0123 = vld1q_s32(w);
0027:   w = (const void*)((uintptr_t)w + 16);
0028:   int32x4_t vacc1x0123 = vacc0x0123;
0029:   int32x4_t vacc2x0123 = vacc0x0123;
0030:   int32x4_t vacc3x0123 = vacc0x0123;
0031:   int32x4_t vacc4x0123 = vacc0x0123;
0032:   int32x4_t vacc5x0123 = vacc0x0123;
0033: 
```

- **EN:** This block implements local helper logic for `6x4-neon`. Key symbols: `pytorch_q8gemm_ukernel_6x4__neon`.
- **CN:** 该代码块实现与 `6x4-neon` 相关的局部辅助逻辑。关键符号：`pytorch_q8gemm_ukernel_6x4__neon`。

### Lines 34-55 / 第 34-55 行

```c
0034:   const uint8_t* a0 = a;
0035:   const uint8_t* a1 = (const uint8_t*)((uintptr_t)a0 + a_stride);
0036:   if (mr < 2) {
0037:     a1 = a0;
0038:   }
0039:   const uint8_t* a2 = (const uint8_t*)((uintptr_t)a1 + a_stride);
0040:   if (mr <= 2) {
0041:     a2 = a1;
0042:   }
0043:   const uint8_t* a3 = (const uint8_t*)((uintptr_t)a2 + a_stride);
0044:   if (mr < 4) {
0045:     a3 = a2;
0046:   }
0047:   const uint8_t* a4 = (const uint8_t*)((uintptr_t)a3 + a_stride);
0048:   if (mr <= 4) {
0049:     a4 = a3;
0050:   };
0051:   const uint8_t* a5 = (const uint8_t*)((uintptr_t)a4 + a_stride);
0052:   if (mr != 6) {
0053:     a5 = a4;
0054:   }
0055: 
```

- **EN:** This block handles conditional branches and special cases. Key symbols: no prominent local symbols.
- **CN:** 该代码块处理条件分支与特殊情况。关键符号：无明显局部符号。

### Lines 56-93 / 第 56-93 行

```c
0056:   const uint8x8_t va_zero_point =
0057:       vld1_dup_u8((const uint8_t*)&quantization_params->neon.input_zero_point);
0058:   uint8x8_t vb_zero_point =
0059:       vld1_u8((const uint8_t*)&quantization_params->neon.kernel_zero_points
0060:           [output_channel_index]);
0061:   // Since only lower 4 values are used in this kernel. We replicate lower 4
0062:   // values in upper 4 values. Still we end up loading 8 values assuming
0063:   // zero point array is always multiple of 8.
0064:   vb_zero_point = vset_lane_u8(vget_lane_u8(vb_zero_point, 0), vb_zero_point, 4);
0065:   vb_zero_point = vset_lane_u8(vget_lane_u8(vb_zero_point, 1), vb_zero_point, 5);
0066:   vb_zero_point = vset_lane_u8(vget_lane_u8(vb_zero_point, 2), vb_zero_point, 6);
0067:   vb_zero_point = vset_lane_u8(vget_lane_u8(vb_zero_point, 3), vb_zero_point, 7);
0068:   for (; k >= 8; k -= 8) {
0069:     const uint8x8_t va0 = vld1_u8(a0);
0070:     a0 += 8;
0071:     const int16x8_t vxa0 =
0072:         vreinterpretq_s16_u16(sub_zero_point(va0, va_zero_point));
0073:     const uint8x8_t va1 = vld1_u8(a1);
0074:     a1 += 8;
0075:     const int16x8_t vxa1 =
0076:         vreinterpretq_s16_u16(sub_zero_point(va1, va_zero_point));
0077:     const uint8x8_t va2 = vld1_u8(a2);
0078:     a2 += 8;
0079:     const int16x8_t vxa2 =
0080:         vreinterpretq_s16_u16(sub_zero_point(va2, va_zero_point));
0081:     const uint8x8_t va3 = vld1_u8(a3);
0082:     a3 += 8;
0083:     const int16x8_t vxa3 =
0084:         vreinterpretq_s16_u16(sub_zero_point(va3, va_zero_point));
0085:     const uint8x8_t va4 = vld1_u8(a4);
0086:     a4 += 8;
0087:     const int16x8_t vxa4 =
0088:         vreinterpretq_s16_u16(sub_zero_point(va4, va_zero_point));
0089:     const uint8x8_t va5 = vld1_u8(a5);
0090:     a5 += 8;
0091:     const int16x8_t vxa5 =
0092:         vreinterpretq_s16_u16(sub_zero_point(va5, va_zero_point));
0093: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `vld1_dup_u8`, `vld1_u8`, `vreinterpretq_s16_u16`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`vld1_dup_u8`, `vld1_u8`, `vreinterpretq_s16_u16`。

### Lines 94-124 / 第 94-124 行

```c
0094:     const uint8x8_t vb0123c01 = vld1_u8(w);
0095:     w = (const void*)((uintptr_t)w + 8);
0096:     const int16x8_t vxb0123c01 =
0097:         vreinterpretq_s16_u16(vsubl_u8(vb0123c01, vb_zero_point));
0098: 
0099:     vacc0x0123 = vmlal_lane_s16(
0100:         vacc0x0123, vget_low_s16(vxb0123c01), vget_low_s16(vxa0), 0);
0101:     vacc1x0123 = vmlal_lane_s16(
0102:         vacc1x0123, vget_low_s16(vxb0123c01), vget_low_s16(vxa1), 0);
0103:     vacc2x0123 = vmlal_lane_s16(
0104:         vacc2x0123, vget_low_s16(vxb0123c01), vget_low_s16(vxa2), 0);
0105:     vacc3x0123 = vmlal_lane_s16(
0106:         vacc3x0123, vget_low_s16(vxb0123c01), vget_low_s16(vxa3), 0);
0107:     vacc4x0123 = vmlal_lane_s16(
0108:         vacc4x0123, vget_low_s16(vxb0123c01), vget_low_s16(vxa4), 0);
0109:     vacc5x0123 = vmlal_lane_s16(
0110:         vacc5x0123, vget_low_s16(vxb0123c01), vget_low_s16(vxa5), 0);
0111: 
0112:     vacc0x0123 = vmlal_lane_s16(
0113:         vacc0x0123, vget_high_s16(vxb0123c01), vget_low_s16(vxa0), 1);
0114:     vacc1x0123 = vmlal_lane_s16(
0115:         vacc1x0123, vget_high_s16(vxb0123c01), vget_low_s16(vxa1), 1);
0116:     vacc2x0123 = vmlal_lane_s16(
0117:         vacc2x0123, vget_high_s16(vxb0123c01), vget_low_s16(vxa2), 1);
0118:     vacc3x0123 = vmlal_lane_s16(
0119:         vacc3x0123, vget_high_s16(vxb0123c01), vget_low_s16(vxa3), 1);
0120:     vacc4x0123 = vmlal_lane_s16(
0121:         vacc4x0123, vget_high_s16(vxb0123c01), vget_low_s16(vxa4), 1);
0122:     vacc5x0123 = vmlal_lane_s16(
0123:         vacc5x0123, vget_high_s16(vxb0123c01), vget_low_s16(vxa5), 1);
0124: 
```

- **EN:** This block implements local helper logic for `6x4-neon`. Key symbols: `vreinterpretq_s16_u16`, `vget_low_s16`, `vget_high_s16`.
- **CN:** 该代码块实现与 `6x4-neon` 相关的局部辅助逻辑。关键符号：`vreinterpretq_s16_u16`, `vget_low_s16`, `vget_high_s16`。

### Lines 125-155 / 第 125-155 行

```c
0125:     const uint8x8_t vb0123c23 = vld1_u8(w);
0126:     w = (const void*)((uintptr_t)w + 8);
0127:     const int16x8_t vxb0123c23 =
0128:         vreinterpretq_s16_u16(vsubl_u8(vb0123c23, vb_zero_point));
0129: 
0130:     vacc0x0123 = vmlal_lane_s16(
0131:         vacc0x0123, vget_low_s16(vxb0123c23), vget_low_s16(vxa0), 2);
0132:     vacc1x0123 = vmlal_lane_s16(
0133:         vacc1x0123, vget_low_s16(vxb0123c23), vget_low_s16(vxa1), 2);
0134:     vacc2x0123 = vmlal_lane_s16(
0135:         vacc2x0123, vget_low_s16(vxb0123c23), vget_low_s16(vxa2), 2);
0136:     vacc3x0123 = vmlal_lane_s16(
0137:         vacc3x0123, vget_low_s16(vxb0123c23), vget_low_s16(vxa3), 2);
0138:     vacc4x0123 = vmlal_lane_s16(
0139:         vacc4x0123, vget_low_s16(vxb0123c23), vget_low_s16(vxa4), 2);
0140:     vacc5x0123 = vmlal_lane_s16(
0141:         vacc5x0123, vget_low_s16(vxb0123c23), vget_low_s16(vxa5), 2);
0142: 
0143:     vacc0x0123 = vmlal_lane_s16(
0144:         vacc0x0123, vget_high_s16(vxb0123c23), vget_low_s16(vxa0), 3);
0145:     vacc1x0123 = vmlal_lane_s16(
0146:         vacc1x0123, vget_high_s16(vxb0123c23), vget_low_s16(vxa1), 3);
0147:     vacc2x0123 = vmlal_lane_s16(
0148:         vacc2x0123, vget_high_s16(vxb0123c23), vget_low_s16(vxa2), 3);
0149:     vacc3x0123 = vmlal_lane_s16(
0150:         vacc3x0123, vget_high_s16(vxb0123c23), vget_low_s16(vxa3), 3);
0151:     vacc4x0123 = vmlal_lane_s16(
0152:         vacc4x0123, vget_high_s16(vxb0123c23), vget_low_s16(vxa4), 3);
0153:     vacc5x0123 = vmlal_lane_s16(
0154:         vacc5x0123, vget_high_s16(vxb0123c23), vget_low_s16(vxa5), 3);
0155: 
```

- **EN:** This block implements local helper logic for `6x4-neon`. Key symbols: `vreinterpretq_s16_u16`, `vget_low_s16`, `vget_high_s16`.
- **CN:** 该代码块实现与 `6x4-neon` 相关的局部辅助逻辑。关键符号：`vreinterpretq_s16_u16`, `vget_low_s16`, `vget_high_s16`。

### Lines 156-186 / 第 156-186 行

```c
0156:     const uint8x8_t vb0123c45 = vld1_u8(w);
0157:     w = (const void*)((uintptr_t)w + 8);
0158:     const int16x8_t vxb0123c45 =
0159:         vreinterpretq_s16_u16(vsubl_u8(vb0123c45, vb_zero_point));
0160: 
0161:     vacc0x0123 = vmlal_lane_s16(
0162:         vacc0x0123, vget_low_s16(vxb0123c45), vget_high_s16(vxa0), 0);
0163:     vacc1x0123 = vmlal_lane_s16(
0164:         vacc1x0123, vget_low_s16(vxb0123c45), vget_high_s16(vxa1), 0);
0165:     vacc2x0123 = vmlal_lane_s16(
0166:         vacc2x0123, vget_low_s16(vxb0123c45), vget_high_s16(vxa2), 0);
0167:     vacc3x0123 = vmlal_lane_s16(
0168:         vacc3x0123, vget_low_s16(vxb0123c45), vget_high_s16(vxa3), 0);
0169:     vacc4x0123 = vmlal_lane_s16(
0170:         vacc4x0123, vget_low_s16(vxb0123c45), vget_high_s16(vxa4), 0);
0171:     vacc5x0123 = vmlal_lane_s16(
0172:         vacc5x0123, vget_low_s16(vxb0123c45), vget_high_s16(vxa5), 0);
0173: 
0174:     vacc0x0123 = vmlal_lane_s16(
0175:         vacc0x0123, vget_high_s16(vxb0123c45), vget_high_s16(vxa0), 1);
0176:     vacc1x0123 = vmlal_lane_s16(
0177:         vacc1x0123, vget_high_s16(vxb0123c45), vget_high_s16(vxa1), 1);
0178:     vacc2x0123 = vmlal_lane_s16(
0179:         vacc2x0123, vget_high_s16(vxb0123c45), vget_high_s16(vxa2), 1);
0180:     vacc3x0123 = vmlal_lane_s16(
0181:         vacc3x0123, vget_high_s16(vxb0123c45), vget_high_s16(vxa3), 1);
0182:     vacc4x0123 = vmlal_lane_s16(
0183:         vacc4x0123, vget_high_s16(vxb0123c45), vget_high_s16(vxa4), 1);
0184:     vacc5x0123 = vmlal_lane_s16(
0185:         vacc5x0123, vget_high_s16(vxb0123c45), vget_high_s16(vxa5), 1);
0186: 
```

- **EN:** This block implements local helper logic for `6x4-neon`. Key symbols: `vreinterpretq_s16_u16`, `vget_low_s16`, `vget_high_s16`.
- **CN:** 该代码块实现与 `6x4-neon` 相关的局部辅助逻辑。关键符号：`vreinterpretq_s16_u16`, `vget_low_s16`, `vget_high_s16`。

### Lines 187-218 / 第 187-218 行

```c
0187:     const uint8x8_t vb0123c67 = vld1_u8(w);
0188:     w = (const void*)((uintptr_t)w + 8);
0189:     const int16x8_t vxb0123c67 =
0190:         vreinterpretq_s16_u16(vsubl_u8(vb0123c67, vb_zero_point));
0191: 
0192:     vacc0x0123 = vmlal_lane_s16(
0193:         vacc0x0123, vget_low_s16(vxb0123c67), vget_high_s16(vxa0), 2);
0194:     vacc1x0123 = vmlal_lane_s16(
0195:         vacc1x0123, vget_low_s16(vxb0123c67), vget_high_s16(vxa1), 2);
0196:     vacc2x0123 = vmlal_lane_s16(
0197:         vacc2x0123, vget_low_s16(vxb0123c67), vget_high_s16(vxa2), 2);
0198:     vacc3x0123 = vmlal_lane_s16(
0199:         vacc3x0123, vget_low_s16(vxb0123c67), vget_high_s16(vxa3), 2);
0200:     vacc4x0123 = vmlal_lane_s16(
0201:         vacc4x0123, vget_low_s16(vxb0123c67), vget_high_s16(vxa4), 2);
0202:     vacc5x0123 = vmlal_lane_s16(
0203:         vacc5x0123, vget_low_s16(vxb0123c67), vget_high_s16(vxa5), 2);
0204: 
0205:     vacc0x0123 = vmlal_lane_s16(
0206:         vacc0x0123, vget_high_s16(vxb0123c67), vget_high_s16(vxa0), 3);
0207:     vacc1x0123 = vmlal_lane_s16(
0208:         vacc1x0123, vget_high_s16(vxb0123c67), vget_high_s16(vxa1), 3);
0209:     vacc2x0123 = vmlal_lane_s16(
0210:         vacc2x0123, vget_high_s16(vxb0123c67), vget_high_s16(vxa2), 3);
0211:     vacc3x0123 = vmlal_lane_s16(
0212:         vacc3x0123, vget_high_s16(vxb0123c67), vget_high_s16(vxa3), 3);
0213:     vacc4x0123 = vmlal_lane_s16(
0214:         vacc4x0123, vget_high_s16(vxb0123c67), vget_high_s16(vxa4), 3);
0215:     vacc5x0123 = vmlal_lane_s16(
0216:         vacc5x0123, vget_high_s16(vxb0123c67), vget_high_s16(vxa5), 3);
0217:   }
0218:   if (k != 0) {
```

- **EN:** This block handles conditional branches and special cases. Key symbols: `vreinterpretq_s16_u16`, `vget_low_s16`, `vget_high_s16`.
- **CN:** 该代码块处理条件分支与特殊情况。关键符号：`vreinterpretq_s16_u16`, `vget_low_s16`, `vget_high_s16`。

### Lines 219-245 / 第 219-245 行

```c
0219:     const size_t a_predecrement = 8 - k;
0220:     const int64x1_t va_shift = vmov_n_s64(-8 * a_predecrement);
0221:     const uint8x8_t va0 = vreinterpret_u8_u64(
0222:         vshl_u64(vreinterpret_u64_u8(vld1_u8(a0 - a_predecrement)), va_shift));
0223:     const int16x8_t vxa0 =
0224:         vreinterpretq_s16_u16(sub_zero_point(va0, va_zero_point));
0225:     const uint8x8_t va1 = vreinterpret_u8_u64(
0226:         vshl_u64(vreinterpret_u64_u8(vld1_u8(a1 - a_predecrement)), va_shift));
0227:     const int16x8_t vxa1 =
0228:         vreinterpretq_s16_u16(sub_zero_point(va1, va_zero_point));
0229:     const uint8x8_t va2 = vreinterpret_u8_u64(
0230:         vshl_u64(vreinterpret_u64_u8(vld1_u8(a2 - a_predecrement)), va_shift));
0231:     const int16x8_t vxa2 =
0232:         vreinterpretq_s16_u16(sub_zero_point(va2, va_zero_point));
0233:     const uint8x8_t va3 = vreinterpret_u8_u64(
0234:         vshl_u64(vreinterpret_u64_u8(vld1_u8(a3 - a_predecrement)), va_shift));
0235:     const int16x8_t vxa3 =
0236:         vreinterpretq_s16_u16(sub_zero_point(va3, va_zero_point));
0237:     const uint8x8_t va4 = vreinterpret_u8_u64(
0238:         vshl_u64(vreinterpret_u64_u8(vld1_u8(a4 - a_predecrement)), va_shift));
0239:     const int16x8_t vxa4 =
0240:         vreinterpretq_s16_u16(sub_zero_point(va4, va_zero_point));
0241:     const uint8x8_t va5 = vreinterpret_u8_u64(
0242:         vshl_u64(vreinterpret_u64_u8(vld1_u8(a5 - a_predecrement)), va_shift));
0243:     const int16x8_t vxa5 =
0244:         vreinterpretq_s16_u16(sub_zero_point(va5, va_zero_point));
0245: 
```

- **EN:** This block implements local helper logic for `6x4-neon`. Key symbols: `vshl_u64`, `vreinterpretq_s16_u16`.
- **CN:** 该代码块实现与 `6x4-neon` 相关的局部辅助逻辑。关键符号：`vshl_u64`, `vreinterpretq_s16_u16`。

### Lines 246-269 / 第 246-269 行

```c
0246:     const uint8x8_t vb0123c0 = vreinterpret_u8_u32(vld1_dup_u32(w));
0247:     w = (const void*)((uintptr_t)w + 4);
0248:     const int16x8_t vxb0123c0 =
0249:         vreinterpretq_s16_u16(vsubl_u8(vb0123c0, vb_zero_point));
0250: 
0251:     vacc0x0123 = vmlal_lane_s16(
0252:         vacc0x0123, vget_low_s16(vxb0123c0), vget_low_s16(vxa0), 0);
0253:     vacc1x0123 = vmlal_lane_s16(
0254:         vacc1x0123, vget_low_s16(vxb0123c0), vget_low_s16(vxa1), 0);
0255:     vacc2x0123 = vmlal_lane_s16(
0256:         vacc2x0123, vget_low_s16(vxb0123c0), vget_low_s16(vxa2), 0);
0257:     vacc3x0123 = vmlal_lane_s16(
0258:         vacc3x0123, vget_low_s16(vxb0123c0), vget_low_s16(vxa3), 0);
0259:     vacc4x0123 = vmlal_lane_s16(
0260:         vacc4x0123, vget_low_s16(vxb0123c0), vget_low_s16(vxa4), 0);
0261:     vacc5x0123 = vmlal_lane_s16(
0262:         vacc5x0123, vget_low_s16(vxb0123c0), vget_low_s16(vxa5), 0);
0263: 
0264:     if (k >= 2) {
0265:       const uint8x8_t vb0123c1 = vreinterpret_u8_u32(vld1_dup_u32(w));
0266:       w = (const void*)((uintptr_t)w + 4);
0267:       const int16x8_t vxb0123c1 =
0268:           vreinterpretq_s16_u16(vsubl_u8(vb0123c1, vb_zero_point));
0269: 
```

- **EN:** This block handles conditional branches and special cases. Key symbols: `vreinterpretq_s16_u16`, `vget_low_s16`.
- **CN:** 该代码块处理条件分支与特殊情况。关键符号：`vreinterpretq_s16_u16`, `vget_low_s16`。

### Lines 270-301 / 第 270-301 行

```c
0270:       vacc0x0123 = vmlal_lane_s16(
0271:           vacc0x0123, vget_low_s16(vxb0123c1), vget_low_s16(vxa0), 1);
0272:       vacc1x0123 = vmlal_lane_s16(
0273:           vacc1x0123, vget_low_s16(vxb0123c1), vget_low_s16(vxa1), 1);
0274:       vacc2x0123 = vmlal_lane_s16(
0275:           vacc2x0123, vget_low_s16(vxb0123c1), vget_low_s16(vxa2), 1);
0276:       vacc3x0123 = vmlal_lane_s16(
0277:           vacc3x0123, vget_low_s16(vxb0123c1), vget_low_s16(vxa3), 1);
0278:       vacc4x0123 = vmlal_lane_s16(
0279:           vacc4x0123, vget_low_s16(vxb0123c1), vget_low_s16(vxa4), 1);
0280:       vacc5x0123 = vmlal_lane_s16(
0281:           vacc5x0123, vget_low_s16(vxb0123c1), vget_low_s16(vxa5), 1);
0282: 
0283:       if (k > 2) {
0284:         const uint8x8_t vb0123c2 = vreinterpret_u8_u32(vld1_dup_u32(w));
0285:         w = (const void*)((uintptr_t)w + 4);
0286:         const int16x8_t vxb0123c2 =
0287:             vreinterpretq_s16_u16(vsubl_u8(vb0123c2, vb_zero_point));
0288: 
0289:         vacc0x0123 = vmlal_lane_s16(
0290:             vacc0x0123, vget_low_s16(vxb0123c2), vget_low_s16(vxa0), 2);
0291:         vacc1x0123 = vmlal_lane_s16(
0292:             vacc1x0123, vget_low_s16(vxb0123c2), vget_low_s16(vxa1), 2);
0293:         vacc2x0123 = vmlal_lane_s16(
0294:             vacc2x0123, vget_low_s16(vxb0123c2), vget_low_s16(vxa2), 2);
0295:         vacc3x0123 = vmlal_lane_s16(
0296:             vacc3x0123, vget_low_s16(vxb0123c2), vget_low_s16(vxa3), 2);
0297:         vacc4x0123 = vmlal_lane_s16(
0298:             vacc4x0123, vget_low_s16(vxb0123c2), vget_low_s16(vxa4), 2);
0299:         vacc5x0123 = vmlal_lane_s16(
0300:             vacc5x0123, vget_low_s16(vxb0123c2), vget_low_s16(vxa5), 2);
0301: 
```

- **EN:** This block handles conditional branches and special cases. Key symbols: `vget_low_s16`, `vreinterpretq_s16_u16`.
- **CN:** 该代码块处理条件分支与特殊情况。关键符号：`vget_low_s16`, `vreinterpretq_s16_u16`。

### Lines 302-321 / 第 302-321 行

```c
0302:         if (k >= 4) {
0303:           const uint8x8_t vb0123c3 = vreinterpret_u8_u32(vld1_dup_u32(w));
0304:           w = (const void*)((uintptr_t)w + 4);
0305:           const int16x8_t vxb0123c3 =
0306:               vreinterpretq_s16_u16(vsubl_u8(vb0123c3, vb_zero_point));
0307: 
0308:           vacc0x0123 = vmlal_lane_s16(
0309:               vacc0x0123, vget_low_s16(vxb0123c3), vget_low_s16(vxa0), 3);
0310:           vacc1x0123 = vmlal_lane_s16(
0311:               vacc1x0123, vget_low_s16(vxb0123c3), vget_low_s16(vxa1), 3);
0312:           vacc2x0123 = vmlal_lane_s16(
0313:               vacc2x0123, vget_low_s16(vxb0123c3), vget_low_s16(vxa2), 3);
0314:           vacc3x0123 = vmlal_lane_s16(
0315:               vacc3x0123, vget_low_s16(vxb0123c3), vget_low_s16(vxa3), 3);
0316:           vacc4x0123 = vmlal_lane_s16(
0317:               vacc4x0123, vget_low_s16(vxb0123c3), vget_low_s16(vxa4), 3);
0318:           vacc5x0123 = vmlal_lane_s16(
0319:               vacc5x0123, vget_low_s16(vxb0123c3), vget_low_s16(vxa5), 3);
0320: 
0321:           if (k > 4) {
```

- **EN:** This block handles conditional branches and special cases. Key symbols: `vreinterpretq_s16_u16`, `vget_low_s16`.
- **CN:** 该代码块处理条件分支与特殊情况。关键符号：`vreinterpretq_s16_u16`, `vget_low_s16`。

### Lines 322-345 / 第 322-345 行

```c
0322:             const uint8x8_t vb0123c4 = vreinterpret_u8_u32(vld1_dup_u32(w));
0323:             w = (const void*)((uintptr_t)w + 4);
0324:             const int16x8_t vxb0123c4 =
0325:                 vreinterpretq_s16_u16(vsubl_u8(vb0123c4, vb_zero_point));
0326: 
0327:             vacc0x0123 = vmlal_lane_s16(
0328:                 vacc0x0123, vget_low_s16(vxb0123c4), vget_high_s16(vxa0), 0);
0329:             vacc1x0123 = vmlal_lane_s16(
0330:                 vacc1x0123, vget_low_s16(vxb0123c4), vget_high_s16(vxa1), 0);
0331:             vacc2x0123 = vmlal_lane_s16(
0332:                 vacc2x0123, vget_low_s16(vxb0123c4), vget_high_s16(vxa2), 0);
0333:             vacc3x0123 = vmlal_lane_s16(
0334:                 vacc3x0123, vget_low_s16(vxb0123c4), vget_high_s16(vxa3), 0);
0335:             vacc4x0123 = vmlal_lane_s16(
0336:                 vacc4x0123, vget_low_s16(vxb0123c4), vget_high_s16(vxa4), 0);
0337:             vacc5x0123 = vmlal_lane_s16(
0338:                 vacc5x0123, vget_low_s16(vxb0123c4), vget_high_s16(vxa5), 0);
0339: 
0340:             if (k >= 6) {
0341:               const uint8x8_t vb0123c5 = vreinterpret_u8_u32(vld1_dup_u32(w));
0342:               w = (const void*)((uintptr_t)w + 4);
0343:               const int16x8_t vxb0123c5 =
0344:                   vreinterpretq_s16_u16(vsubl_u8(vb0123c5, vb_zero_point));
0345: 
```

- **EN:** This block handles conditional branches and special cases. Key symbols: `vreinterpretq_s16_u16`, `vget_low_s16`.
- **CN:** 该代码块处理条件分支与特殊情况。关键符号：`vreinterpretq_s16_u16`, `vget_low_s16`。

### Lines 346-385 / 第 346-385 行

```c
0346:               vacc0x0123 = vmlal_lane_s16(
0347:                   vacc0x0123, vget_low_s16(vxb0123c5), vget_high_s16(vxa0), 1);
0348:               vacc1x0123 = vmlal_lane_s16(
0349:                   vacc1x0123, vget_low_s16(vxb0123c5), vget_high_s16(vxa1), 1);
0350:               vacc2x0123 = vmlal_lane_s16(
0351:                   vacc2x0123, vget_low_s16(vxb0123c5), vget_high_s16(vxa2), 1);
0352:               vacc3x0123 = vmlal_lane_s16(
0353:                   vacc3x0123, vget_low_s16(vxb0123c5), vget_high_s16(vxa3), 1);
0354:               vacc4x0123 = vmlal_lane_s16(
0355:                   vacc4x0123, vget_low_s16(vxb0123c5), vget_high_s16(vxa4), 1);
0356:               vacc5x0123 = vmlal_lane_s16(
0357:                   vacc5x0123, vget_low_s16(vxb0123c5), vget_high_s16(vxa5), 1);
0358: 
0359:               if (k > 6) {
0360:                 const uint8x8_t vb0123c6 = vreinterpret_u8_u32(vld1_dup_u32(w));
0361:                 const int16x8_t vxb0123c6 =
0362:                     vreinterpretq_s16_u16(vsubl_u8(vb0123c6, vb_zero_point));
0363: 
0364:                 vacc0x0123 = vmlal_lane_s16(
0365:                     vacc0x0123,
0366:                     vget_low_s16(vxb0123c6),
0367:                     vget_high_s16(vxa0),
0368:                     2);
0369:                 vacc1x0123 = vmlal_lane_s16(
0370:                     vacc1x0123,
0371:                     vget_low_s16(vxb0123c6),
0372:                     vget_high_s16(vxa1),
0373:                     2);
0374:                 vacc2x0123 = vmlal_lane_s16(
0375:                     vacc2x0123,
0376:                     vget_low_s16(vxb0123c6),
0377:                     vget_high_s16(vxa2),
0378:                     2);
0379:                 vacc3x0123 = vmlal_lane_s16(
0380:                     vacc3x0123,
0381:                     vget_low_s16(vxb0123c6),
0382:                     vget_high_s16(vxa3),
0383:                     2);
0384:                 vacc4x0123 = vmlal_lane_s16(
0385:                     vacc4x0123,
```

- **EN:** This block handles conditional branches and special cases. Key symbols: `vget_low_s16`, `vreinterpretq_s16_u16`.
- **CN:** 该代码块处理条件分支与特殊情况。关键符号：`vget_low_s16`, `vreinterpretq_s16_u16`。

### Lines 386-406 / 第 386-406 行

```c
0386:                     vget_low_s16(vxb0123c6),
0387:                     vget_high_s16(vxa4),
0388:                     2);
0389:                 vacc5x0123 = vmlal_lane_s16(
0390:                     vacc5x0123,
0391:                     vget_low_s16(vxb0123c6),
0392:                     vget_high_s16(vxa5),
0393:                     2);
0394:               }
0395:             }
0396:           }
0397:         }
0398:       }
0399:     }
0400:   }
0401: 
0402:   const float32x4_t requantization_scale_v =
0403:       vld1q_f32(
0404:           &quantization_params->neon.requantization_scales[
0405:               output_channel_index]);
0406: 
```

- **EN:** This block implements local helper logic for `6x4-neon`. Key symbols: `vget_low_s16`, `vld1q_f32`.
- **CN:** 该代码块实现与 `6x4-neon` 相关的局部辅助逻辑。关键符号：`vget_low_s16`, `vld1q_f32`。

### Lines 407-429 / 第 407-429 行

```c
0407:   const float32x4_t vacc0x0123_f =
0408:     vmulq_f32(vcvtq_f32_s32(vacc0x0123), requantization_scale_v);
0409:   const float32x4_t vacc1x0123_f =
0410:     vmulq_f32(vcvtq_f32_s32(vacc1x0123), requantization_scale_v);
0411:   const float32x4_t vacc2x0123_f =
0412:     vmulq_f32(vcvtq_f32_s32(vacc2x0123), requantization_scale_v);
0413:   const float32x4_t vacc3x0123_f =
0414:     vmulq_f32(vcvtq_f32_s32(vacc3x0123), requantization_scale_v);
0415:   const float32x4_t vacc4x0123_f =
0416:     vmulq_f32(vcvtq_f32_s32(vacc4x0123), requantization_scale_v);
0417:   const float32x4_t vacc5x0123_f =
0418:     vmulq_f32(vcvtq_f32_s32(vacc5x0123), requantization_scale_v);
0419: 
0420: #ifdef __aarch64__
0421:   const int16x8_t voutput_zero_point =
0422:       vld1q_dup_s16(&quantization_params->neon.output_zero_point);
0423:   vacc0x0123 = vcvtnq_s32_f32(vacc0x0123_f);
0424:   vacc1x0123 = vcvtnq_s32_f32(vacc1x0123_f);
0425:   vacc2x0123 = vcvtnq_s32_f32(vacc2x0123_f);
0426:   vacc3x0123 = vcvtnq_s32_f32(vacc3x0123_f);
0427:   vacc4x0123 = vcvtnq_s32_f32(vacc4x0123_f);
0428:   vacc5x0123 = vcvtnq_s32_f32(vacc5x0123_f);
0429: 
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends. Key symbols: `vmulq_f32`, `vld1q_dup_s16`.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端。关键符号：`vmulq_f32`, `vld1q_dup_s16`。

### Lines 430-455 / 第 430-455 行

```c
0430:   const int16x8_t vacc01x0123 = vqaddq_s16(
0431:       vqmovn_high_s32(vqmovn_s32(vacc0x0123), vacc1x0123), voutput_zero_point);
0432:   const int16x8_t vacc23x0123 = vqaddq_s16(
0433:       vqmovn_high_s32(vqmovn_s32(vacc2x0123), vacc3x0123), voutput_zero_point);
0434:   const int16x8_t vacc45x0123 = vqaddq_s16(
0435:       vqmovn_high_s32(vqmovn_s32(vacc4x0123), vacc5x0123), voutput_zero_point);
0436: 
0437:   uint8x16_t vout0123x0123 =
0438:       vqmovun_high_s16(vqmovun_s16(vacc01x0123), vacc23x0123);
0439:   uint8x8_t vout45x0123 = vqmovun_s16(vacc45x0123);
0440: 
0441:   const uint8x16_t voutput_min =
0442:       vld1q_dup_u8(&quantization_params->neon.output_min);
0443:   const uint8x16_t voutput_max =
0444:       vld1q_dup_u8(&quantization_params->neon.output_max);
0445: 
0446:   vout0123x0123 = vmaxq_u8(vout0123x0123, voutput_min);
0447:   vout45x0123 = vmax_u8(vout45x0123, vget_low_u8(voutput_min));
0448:   vout0123x0123 = vminq_u8(vout0123x0123, voutput_max);
0449:   vout45x0123 = vmin_u8(vout45x0123, vget_low_u8(voutput_max));
0450: #else
0451:   const float32x4_t vfmin = vdupq_n_f32(quantization_params->neon.vfmin);
0452:   const float32x4_t vfmax = vdupq_n_f32(quantization_params->neon.vfmax);
0453:   const float32x4_t vfmagic = vdupq_n_f32(quantization_params->neon.vfmagic);
0454:   const int32x4_t vimagic = vdupq_n_s32(quantization_params->neon.vimagic);
0455: 
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends. Key symbols: `vqmovn_high_s32`, `vqmovun_high_s16`, `vld1q_dup_u8`.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端。关键符号：`vqmovn_high_s32`, `vqmovun_high_s16`, `vld1q_dup_u8`。

### Lines 456-481 / 第 456-481 行

```c
0456:   const float32x4_t vacc0x0123_f_clamped =
0457:       vminq_f32(vmaxq_f32(vacc0x0123_f, vfmin), vfmax);
0458:   const float32x4_t vacc1x0123_f_clamped =
0459:       vminq_f32(vmaxq_f32(vacc1x0123_f, vfmin), vfmax);
0460:   const float32x4_t vacc2x0123_f_clamped =
0461:       vminq_f32(vmaxq_f32(vacc2x0123_f, vfmin), vfmax);
0462:   const float32x4_t vacc3x0123_f_clamped =
0463:       vminq_f32(vmaxq_f32(vacc3x0123_f, vfmin), vfmax);
0464:   const float32x4_t vacc4x0123_f_clamped =
0465:       vminq_f32(vmaxq_f32(vacc4x0123_f, vfmin), vfmax);
0466:   const float32x4_t vacc5x0123_f_clamped =
0467:       vminq_f32(vmaxq_f32(vacc5x0123_f, vfmin), vfmax);
0468: 
0469:   vacc0x0123 = vsubq_s32(
0470:       vreinterpretq_s32_f32(vaddq_f32(vacc0x0123_f_clamped, vfmagic)), vimagic);
0471:   vacc1x0123 = vsubq_s32(
0472:       vreinterpretq_s32_f32(vaddq_f32(vacc1x0123_f_clamped, vfmagic)), vimagic);
0473:   vacc2x0123 = vsubq_s32(
0474:       vreinterpretq_s32_f32(vaddq_f32(vacc2x0123_f_clamped, vfmagic)), vimagic);
0475:   vacc3x0123 = vsubq_s32(
0476:       vreinterpretq_s32_f32(vaddq_f32(vacc3x0123_f_clamped, vfmagic)), vimagic);
0477:   vacc4x0123 = vsubq_s32(
0478:       vreinterpretq_s32_f32(vaddq_f32(vacc4x0123_f_clamped, vfmagic)), vimagic);
0479:   vacc5x0123 = vsubq_s32(
0480:       vreinterpretq_s32_f32(vaddq_f32(vacc5x0123_f_clamped, vfmagic)), vimagic);
0481: 
```

- **EN:** This block implements local helper logic for `6x4-neon`. Key symbols: `vminq_f32`, `vreinterpretq_s32_f32`.
- **CN:** 该代码块实现与 `6x4-neon` 相关的局部辅助逻辑。关键符号：`vminq_f32`, `vreinterpretq_s32_f32`。

### Lines 482-504 / 第 482-504 行

```c
0482:   const int16x8_t vacc01x0123 =
0483:       vcombine_s16(vqmovn_s32(vacc0x0123), vqmovn_s32(vacc1x0123));
0484:   const int16x8_t vacc23x0123 =
0485:       vcombine_s16(vqmovn_s32(vacc2x0123), vqmovn_s32(vacc3x0123));
0486:   const int16x8_t vacc45x0123 =
0487:       vcombine_s16(vqmovn_s32(vacc4x0123), vqmovn_s32(vacc5x0123));
0488: 
0489:   uint8x16_t vout0123x0123 =
0490:       vcombine_u8(vqmovun_s16(vacc01x0123), vqmovun_s16(vacc23x0123));
0491:   uint8x8_t vout45x0123 = vqmovun_s16(vacc45x0123);
0492: #endif
0493: 
0494:   uint8_t* c0 = c;
0495:   uint8_t* c1 = (uint8_t*)((uintptr_t)c0 + c_stride);
0496:   if (mr < 2) {
0497:     c1 = c0;
0498:   }
0499:   uint8_t* c2 = (uint8_t*)((uintptr_t)c1 + c_stride);
0500:   if (mr <= 2) {
0501:     c2 = c1;
0502:   }
0503:   uint8_t* c3 = (uint8_t*)((uintptr_t)c2 + c_stride);
0504:   if (mr < 4) {
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends; handles conditional branches and special cases. Key symbols: `vcombine_s16`, `vcombine_u8`.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端；处理条件分支与特殊情况。关键符号：`vcombine_s16`, `vcombine_u8`。

### Lines 505-537 / 第 505-537 行

```c
0505:     c3 = c2;
0506:   }
0507:   uint8_t* c4 = (uint8_t*)((uintptr_t)c3 + c_stride);
0508:   if (mr <= 4) {
0509:     c4 = c3;
0510:   }
0511:   uint8_t* c5 = (uint8_t*)((uintptr_t)c4 + c_stride);
0512:   if (mr != 6) {
0513:     c5 = c4;
0514:   }
0515:   if (nr == 4) {
0516:     vst1q_lane_u32(
0517:         __builtin_assume_aligned(c0, 1),
0518:         vreinterpretq_u32_u8(vout0123x0123),
0519:         0);
0520:     vst1q_lane_u32(
0521:         __builtin_assume_aligned(c1, 1),
0522:         vreinterpretq_u32_u8(vout0123x0123),
0523:         1);
0524:     vst1q_lane_u32(
0525:         __builtin_assume_aligned(c2, 1),
0526:         vreinterpretq_u32_u8(vout0123x0123),
0527:         2);
0528:     vst1q_lane_u32(
0529:         __builtin_assume_aligned(c3, 1),
0530:         vreinterpretq_u32_u8(vout0123x0123),
0531:         3);
0532:     vst1_lane_u32(
0533:         __builtin_assume_aligned(c4, 1), vreinterpret_u32_u8(vout45x0123), 0);
0534:     vst1_lane_u32(
0535:         __builtin_assume_aligned(c5, 1), vreinterpret_u32_u8(vout45x0123), 1);
0536:   } else {
0537:     if (nr >= 2) {
```

- **EN:** This block handles conditional branches and special cases. Key symbols: `vst1q_lane_u32`, `vst1_lane_u32`.
- **CN:** 该代码块处理条件分支与特殊情况。关键符号：`vst1q_lane_u32`, `vst1_lane_u32`。

### Lines 538-568 / 第 538-568 行

```c
0538:       vst1q_lane_u16(
0539:           __builtin_assume_aligned(c0, 1),
0540:           vreinterpretq_u16_u8(vout0123x0123),
0541:           0);
0542:       c0 += 2;
0543:       vst1q_lane_u16(
0544:           __builtin_assume_aligned(c1, 1),
0545:           vreinterpretq_u16_u8(vout0123x0123),
0546:           2);
0547:       c1 += 2;
0548:       vst1q_lane_u16(
0549:           __builtin_assume_aligned(c2, 1),
0550:           vreinterpretq_u16_u8(vout0123x0123),
0551:           4);
0552:       c2 += 2;
0553:       vst1q_lane_u16(
0554:           __builtin_assume_aligned(c3, 1),
0555:           vreinterpretq_u16_u8(vout0123x0123),
0556:           6);
0557:       c3 += 2;
0558:       vst1_lane_u16(
0559:           __builtin_assume_aligned(c4, 1), vreinterpret_u16_u8(vout45x0123), 0);
0560:       c4 += 2;
0561:       vst1_lane_u16(
0562:           __builtin_assume_aligned(c5, 1), vreinterpret_u16_u8(vout45x0123), 2);
0563:       c5 += 2;
0564:       vout0123x0123 = vextq_u8(vout0123x0123, vout0123x0123, 2);
0565:       vout45x0123 = vext_u8(vout45x0123, vout45x0123, 2);
0566:       nr -= 2;
0567:     }
0568:     if (nr != 0) {
```

- **EN:** This block handles conditional branches and special cases. Key symbols: `vst1q_lane_u16`, `vst1_lane_u16`.
- **CN:** 该代码块处理条件分支与特殊情况。关键符号：`vst1q_lane_u16`, `vst1_lane_u16`。

### Lines 569-577 / 第 569-577 行

```c
0569:       vst1q_lane_u8(__builtin_assume_aligned(c0, 1), vout0123x0123, 0);
0570:       vst1q_lane_u8(__builtin_assume_aligned(c1, 1), vout0123x0123, 4);
0571:       vst1q_lane_u8(__builtin_assume_aligned(c2, 1), vout0123x0123, 8);
0572:       vst1q_lane_u8(__builtin_assume_aligned(c3, 1), vout0123x0123, 12);
0573:       vst1_lane_u8(__builtin_assume_aligned(c4, 1), vout45x0123, 0);
0574:       vst1_lane_u8(__builtin_assume_aligned(c5, 1), vout45x0123, 4);
0575:     }
0576:   }
0577: }
```

- **EN:** This block implements local helper logic for `6x4-neon`. Key symbols: `vst1q_lane_u8`, `vst1_lane_u8`.
- **CN:** 该代码块实现与 `6x4-neon` 相关的局部辅助逻辑。关键符号：`vst1q_lane_u8`, `vst1_lane_u8`。


## Key Concepts / 关键概念
- **ATen native operator logic** — ATen 原生算子逻辑
- **GPU kernel/runtime interaction** — GPU 内核/运行时交互
- **Linear algebra backend integration** — 线性代数后端集成
- **Mixed-precision/autocast logic** — 混合精度/自动混精逻辑
- **Parallel scan/reduction behavior** — 并行扫描/归约行为
- **Core symbols: pytorch_q8gemm_ukernel_6x4__neon, vld1_dup_u8, vld1_u8, vreinterpretq_s16_u16, vget_low_s16, vget_high_s16, vshl_u64, vld1q_f32** — 核心符号：pytorch_q8gemm_ukernel_6x4__neon、vld1_dup_u8、vld1_u8、vreinterpretq_s16_u16、vget_low_s16、vget_high_s16、vshl_u64、vld1q_f32

## Dependencies / 依赖关系
- **Internal includes / 内部头文件**: 无
- **External includes / 外部头文件**: `arm_neon.h`, `qnnpack/q8gemm.h`, `requantization/runtime-neon.h`
- **Namespaces / 命名空间**: 无
- **Representative symbols / 代表性符号**: `pytorch_q8gemm_ukernel_6x4__neon`, `vld1_dup_u8`, `vld1_u8`, `vreinterpretq_s16_u16`, `vget_low_s16`, `vget_high_s16`, `vshl_u64`, `vld1q_f32`, `vmulq_f32`, `vld1q_dup_s16`, `vqmovn_high_s32`, `vqmovun_high_s16`, `...`
