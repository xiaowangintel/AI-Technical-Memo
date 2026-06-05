# mp8x25-neon.c — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `aten/src/ATen/native/quantized/cpu/qnnpack/src/q8dwconv/mp8x25-neon.c`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements ATen native operators, checks, helper kernels, and backend dispatch behavior. This specific file centers on `mp8x25-neon.c`. The file header highlights: "All rights reserved. This source code is licensed under the BSD-style license found in the LICENSE file in the root directory of this source tree.."
- **Purpose (CN)**: 实现 ATen 原生算子、运行时检查、辅助内核以及后端分发行为。 该文件具体围绕 `mp8x25-neon.c` 展开。 文件头部注释也概括了其核心职责。

## Line-by-Line Analysis / 逐行分析
### Lines 1-30 / 第 1-30 行

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
0011: #include <qnnpack/q8dwconv.h>
0012: 
0013: void pytorch_q8dwconv_ukernel_mp8x25__neon(
0014:     size_t channels,
0015:     size_t output_width,
0016:     const uint8_t** input,
0017:     const void* weights,
0018:     int32_t* outacc32,
0019:     uint8_t* output,
0020:     size_t input_stride,
0021:     size_t output_increment,
0022:     const union pytorch_qnnp_conv_quantization_params
0023:         quantization_params[restrict static 1]) {
0024:   const uint8x8_t vinput_zero_point =
0025:       vld1_dup_u8((const uint8_t*)&quantization_params->neon.input_zero_point);
0026:   const uint8x8_t vkernel_zero_point =
0027:       vdup_n_u8(quantization_params->neon.kernel_zero_points[0]);
0028:   const float32x4_t requantization_scale_v =
0029:       vdupq_n_f32(quantization_params->neon.requantization_scales[0]);
0030: #ifdef __aarch64__
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends. Key symbols: `pytorch_q8dwconv_ukernel_mp8x25__neon`, `vld1_dup_u8`, `vdup_n_u8`, `vdupq_n_f32`.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端。关键符号：`pytorch_q8dwconv_ukernel_mp8x25__neon`, `vld1_dup_u8`, `vdup_n_u8`, `vdupq_n_f32`。

### Lines 31-64 / 第 31-64 行

```c
0031:   const int16x8_t voutput_zero_point =
0032:       vld1q_dup_s16(&quantization_params->neon.output_zero_point);
0033:   const uint8x8_t voutput_min = vld1_dup_u8(&quantization_params->neon.output_min);
0034:   const uint8x8_t voutput_max = vld1_dup_u8(&quantization_params->neon.output_max);
0035: #else
0036:   const float32x4_t vfmin = vdupq_n_f32(quantization_params->neon.vfmin);
0037:   const float32x4_t vfmax = vdupq_n_f32(quantization_params->neon.vfmax);
0038:   const float32x4_t vfmagic = vdupq_n_f32(quantization_params->neon.vfmagic);
0039:   const int32x4_t vimagic = vdupq_n_s32(quantization_params->neon.vimagic);
0040: #endif
0041: 
0042:   do {
0043:     uint8_t* output_start = output;
0044:     int32_t* outacc = outacc32;
0045:     const void* w = weights;
0046:     {
0047:       const uint8_t* i0 = input[0];
0048:       const uint8_t* i1 = input[1];
0049:       const uint8_t* i2 = input[2];
0050:       const uint8_t* i3 = input[3];
0051:       const uint8_t* i4 = input[4];
0052:       const uint8_t* i5 = input[5];
0053:       const uint8_t* i6 = input[6];
0054:       const uint8_t* i7 = input[7];
0055:       const uint8_t* i8 = input[8];
0056:       const uint8_t* i9 = input[9];
0057: 
0058:       size_t c = channels;
0059:       for (; c >= 8; c -= 8) {
0060:         int32x4_t vaccX1_lo = vld1q_s32(w);
0061:         w = (void*)((uintptr_t)w + sizeof(int32x4_t));
0062:         int32x4_t vaccX1_hi = vld1q_s32(w);
0063:         w = (void*)((uintptr_t)w + sizeof(int32x4_t));
0064: 
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends; iterates over tensors, descriptors, options, or runtime state. Key symbols: `vld1q_dup_s16`.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端；遍历张量、描述符、配置项或运行时状态。关键符号：`vld1q_dup_s16`。

### Lines 65-102 / 第 65-102 行

```c
0065:         const uint8x8_t vk0 = vld1_u8(w);
0066:         w += 8;
0067:         const uint8x8_t vi0 = vld1_u8(i0);
0068:         i0 += 8;
0069:         const int16x8_t vxk0 =
0070:             vreinterpretq_s16_u16(vsubl_u8(vk0, vkernel_zero_point));
0071:         const int16x8_t vxi0 =
0072:             vreinterpretq_s16_u16(vsubl_u8(vi0, vinput_zero_point));
0073:         int32x4_t vaccX0_lo = vmull_s16(vget_low_s16(vxk0), vget_low_s16(vxi0));
0074:         int32x4_t vaccX0_hi =
0075:             vmull_s16(vget_high_s16(vxk0), vget_high_s16(vxi0));
0076: 
0077:         const uint8x8_t vk1 = vld1_u8(w);
0078:         w += 8;
0079:         const uint8x8_t vi1 = vld1_u8(i1);
0080:         i1 += 8;
0081:         const int16x8_t vxk1 =
0082:             vreinterpretq_s16_u16(vsubl_u8(vk1, vkernel_zero_point));
0083:         const int16x8_t vxi1 =
0084:             vreinterpretq_s16_u16(vsubl_u8(vi1, vinput_zero_point));
0085:         vaccX1_lo =
0086:             vmlal_s16(vaccX1_lo, vget_low_s16(vxk1), vget_low_s16(vxi1));
0087:         vaccX1_hi =
0088:             vmlal_s16(vaccX1_hi, vget_high_s16(vxk1), vget_high_s16(vxi1));
0089: 
0090:         const uint8x8_t vk2 = vld1_u8(w);
0091:         w += 8;
0092:         const uint8x8_t vi2 = vld1_u8(i2);
0093:         i2 += 8;
0094:         const int16x8_t vxk2 =
0095:             vreinterpretq_s16_u16(vsubl_u8(vk2, vkernel_zero_point));
0096:         const int16x8_t vxi2 =
0097:             vreinterpretq_s16_u16(vsubl_u8(vi2, vinput_zero_point));
0098:         vaccX0_lo =
0099:             vmlal_s16(vaccX0_lo, vget_low_s16(vxk2), vget_low_s16(vxi2));
0100:         vaccX0_hi =
0101:             vmlal_s16(vaccX0_hi, vget_high_s16(vxk2), vget_high_s16(vxi2));
0102: 
```

- **EN:** This block implements local helper logic for `mp8x25-neon`. Key symbols: `vreinterpretq_s16_u16`, `vmull_s16`, `vmlal_s16`.
- **CN:** 该代码块实现与 `mp8x25-neon` 相关的局部辅助逻辑。关键符号：`vreinterpretq_s16_u16`, `vmull_s16`, `vmlal_s16`。

### Lines 103-141 / 第 103-141 行

```c
0103:         const uint8x8_t vk3 = vld1_u8(w);
0104:         w += 8;
0105:         const uint8x8_t vi3 = vld1_u8(i3);
0106:         i3 += 8;
0107:         const int16x8_t vxk3 =
0108:             vreinterpretq_s16_u16(vsubl_u8(vk3, vkernel_zero_point));
0109:         const int16x8_t vxi3 =
0110:             vreinterpretq_s16_u16(vsubl_u8(vi3, vinput_zero_point));
0111:         vaccX1_lo =
0112:             vmlal_s16(vaccX1_lo, vget_low_s16(vxk3), vget_low_s16(vxi3));
0113:         vaccX1_hi =
0114:             vmlal_s16(vaccX1_hi, vget_high_s16(vxk3), vget_high_s16(vxi3));
0115: 
0116:         const uint8x8_t vk4 = vld1_u8(w);
0117:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0118:         const uint8x8_t vi4 = vld1_u8(i4);
0119:         i4 += 8;
0120:         const int16x8_t vxk4 =
0121:             vreinterpretq_s16_u16(vsubl_u8(vk4, vkernel_zero_point));
0122:         const int16x8_t vxi4 =
0123:             vreinterpretq_s16_u16(vsubl_u8(vi4, vinput_zero_point));
0124:         vaccX0_lo =
0125:             vmlal_s16(vaccX0_lo, vget_low_s16(vxk4), vget_low_s16(vxi4));
0126:         vaccX0_hi =
0127:             vmlal_s16(vaccX0_hi, vget_high_s16(vxk4), vget_high_s16(vxi4));
0128: 
0129:         const uint8x8_t vk5 = vld1_u8(w);
0130:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0131:         const uint8x8_t vi5 = vld1_u8(i5);
0132:         i5 += 8;
0133:         const int16x8_t vxk5 =
0134:             vreinterpretq_s16_u16(vsubl_u8(vk5, vkernel_zero_point));
0135:         const int16x8_t vxi5 =
0136:             vreinterpretq_s16_u16(vsubl_u8(vi5, vinput_zero_point));
0137:         vaccX1_lo =
0138:             vmlal_s16(vaccX1_lo, vget_low_s16(vxk5), vget_low_s16(vxi5));
0139:         vaccX1_hi =
0140:             vmlal_s16(vaccX1_hi, vget_high_s16(vxk5), vget_high_s16(vxi5));
0141: 
```

- **EN:** This block implements local helper logic for `mp8x25-neon`. Key symbols: `vreinterpretq_s16_u16`, `vmlal_s16`.
- **CN:** 该代码块实现与 `mp8x25-neon` 相关的局部辅助逻辑。关键符号：`vreinterpretq_s16_u16`, `vmlal_s16`。

### Lines 142-180 / 第 142-180 行

```c
0142:         const uint8x8_t vk6 = vld1_u8(w);
0143:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0144:         const uint8x8_t vi6 = vld1_u8(i6);
0145:         i6 += 8;
0146:         const int16x8_t vxk6 =
0147:             vreinterpretq_s16_u16(vsubl_u8(vk6, vkernel_zero_point));
0148:         const int16x8_t vxi6 =
0149:             vreinterpretq_s16_u16(vsubl_u8(vi6, vinput_zero_point));
0150:         vaccX0_lo =
0151:             vmlal_s16(vaccX0_lo, vget_low_s16(vxk6), vget_low_s16(vxi6));
0152:         vaccX0_hi =
0153:             vmlal_s16(vaccX0_hi, vget_high_s16(vxk6), vget_high_s16(vxi6));
0154: 
0155:         const uint8x8_t vk7 = vld1_u8(w);
0156:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0157:         const uint8x8_t vi7 = vld1_u8(i7);
0158:         i7 += 8;
0159:         const int16x8_t vxk7 =
0160:             vreinterpretq_s16_u16(vsubl_u8(vk7, vkernel_zero_point));
0161:         const int16x8_t vxi7 =
0162:             vreinterpretq_s16_u16(vsubl_u8(vi7, vinput_zero_point));
0163:         vaccX1_lo =
0164:             vmlal_s16(vaccX1_lo, vget_low_s16(vxk7), vget_low_s16(vxi7));
0165:         vaccX1_hi =
0166:             vmlal_s16(vaccX1_hi, vget_high_s16(vxk7), vget_high_s16(vxi7));
0167: 
0168:         const uint8x8_t vk8 = vld1_u8(w);
0169:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0170:         const uint8x8_t vi8 = vld1_u8(i8);
0171:         i8 += 8;
0172:         const int16x8_t vxk8 =
0173:             vreinterpretq_s16_u16(vsubl_u8(vk8, vkernel_zero_point));
0174:         const int16x8_t vxi8 =
0175:             vreinterpretq_s16_u16(vsubl_u8(vi8, vinput_zero_point));
0176:         vaccX0_lo =
0177:             vmlal_s16(vaccX0_lo, vget_low_s16(vxk8), vget_low_s16(vxi8));
0178:         vaccX0_hi =
0179:             vmlal_s16(vaccX0_hi, vget_high_s16(vxk8), vget_high_s16(vxi8));
0180: 
```

- **EN:** This block implements local helper logic for `mp8x25-neon`. Key symbols: `vreinterpretq_s16_u16`, `vmlal_s16`.
- **CN:** 该代码块实现与 `mp8x25-neon` 相关的局部辅助逻辑。关键符号：`vreinterpretq_s16_u16`, `vmlal_s16`。

### Lines 181-215 / 第 181-215 行

```c
0181:         const uint8x8_t vk9 = vld1_u8(w);
0182:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0183:         const uint8x8_t vi9 = vld1_u8(i9);
0184:         i9 += 8;
0185:         const int16x8_t vxk9 =
0186:             vreinterpretq_s16_u16(vsubl_u8(vk9, vkernel_zero_point));
0187:         const int16x8_t vxi9 =
0188:             vreinterpretq_s16_u16(vsubl_u8(vi9, vinput_zero_point));
0189:         vaccX1_lo =
0190:             vmlal_s16(vaccX1_lo, vget_low_s16(vxk9), vget_low_s16(vxi9));
0191:         vaccX1_hi =
0192:             vmlal_s16(vaccX1_hi, vget_high_s16(vxk9), vget_high_s16(vxi9));
0193: 
0194:         int32x4_t vacc_lo = vaddq_s32(vaccX0_lo, vaccX1_lo);
0195:         int32x4_t vacc_hi = vaddq_s32(vaccX0_hi, vaccX1_hi);
0196: 
0197:         vst1q_s32(outacc, vacc_lo);
0198:         outacc += 4;
0199:         vst1q_s32(outacc, vacc_hi);
0200:         outacc += 4;
0201:       }
0202:       if (c != 0) {
0203:         const size_t c_predecrement = 8 - c;
0204:         const int64x1_t vi_shift = vmov_n_s64(-8 * c_predecrement);
0205:         i0 -= c_predecrement;
0206:         i1 -= c_predecrement;
0207:         i2 -= c_predecrement;
0208:         i3 -= c_predecrement;
0209:         i4 -= c_predecrement;
0210:         i5 -= c_predecrement;
0211:         i6 -= c_predecrement;
0212:         i7 -= c_predecrement;
0213:         i8 -= c_predecrement;
0214:         i9 -= c_predecrement;
0215: 
```

- **EN:** This block handles conditional branches and special cases. Key symbols: `vreinterpretq_s16_u16`, `vmlal_s16`, `vst1q_s32`.
- **CN:** 该代码块处理条件分支与特殊情况。关键符号：`vreinterpretq_s16_u16`, `vmlal_s16`, `vst1q_s32`。

### Lines 216-245 / 第 216-245 行

```c
0216:         int32x4_t vaccX1_lo = vld1q_s32(w);
0217:         w = (void*)((uintptr_t)w + sizeof(int32x4_t));
0218:         int32x4_t vaccX1_hi = vld1q_s32(w);
0219:         w = (void*)((uintptr_t)w + sizeof(int32x4_t));
0220: 
0221:         const uint8x8_t vk0 = vld1_u8(w);
0222:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0223:         const uint8x8_t vi0 = vreinterpret_u8_u64(
0224:             vshl_u64(vreinterpret_u64_u8(vld1_u8(i0)), vi_shift));
0225:         const int16x8_t vxk0 =
0226:             vreinterpretq_s16_u16(vsubl_u8(vk0, vkernel_zero_point));
0227:         const int16x8_t vxi0 =
0228:             vreinterpretq_s16_u16(vsubl_u8(vi0, vinput_zero_point));
0229:         int32x4_t vaccX0_lo = vmull_s16(vget_low_s16(vxk0), vget_low_s16(vxi0));
0230:         int32x4_t vaccX0_hi =
0231:             vmull_s16(vget_high_s16(vxk0), vget_high_s16(vxi0));
0232: 
0233:         const uint8x8_t vk1 = vld1_u8(w);
0234:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0235:         const uint8x8_t vi1 = vreinterpret_u8_u64(
0236:             vshl_u64(vreinterpret_u64_u8(vld1_u8(i1)), vi_shift));
0237:         const int16x8_t vxk1 =
0238:             vreinterpretq_s16_u16(vsubl_u8(vk1, vkernel_zero_point));
0239:         const int16x8_t vxi1 =
0240:             vreinterpretq_s16_u16(vsubl_u8(vi1, vinput_zero_point));
0241:         vaccX1_lo =
0242:             vmlal_s16(vaccX1_lo, vget_low_s16(vxk1), vget_low_s16(vxi1));
0243:         vaccX1_hi =
0244:             vmlal_s16(vaccX1_hi, vget_high_s16(vxk1), vget_high_s16(vxi1));
0245: 
```

- **EN:** This block implements local helper logic for `mp8x25-neon`. Key symbols: `vshl_u64`, `vreinterpretq_s16_u16`, `vmull_s16`, `vmlal_s16`.
- **CN:** 该代码块实现与 `mp8x25-neon` 相关的局部辅助逻辑。关键符号：`vshl_u64`, `vreinterpretq_s16_u16`, `vmull_s16`, `vmlal_s16`。

### Lines 246-284 / 第 246-284 行

```c
0246:         const uint8x8_t vk2 = vld1_u8(w);
0247:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0248:         const uint8x8_t vi2 = vreinterpret_u8_u64(
0249:             vshl_u64(vreinterpret_u64_u8(vld1_u8(i2)), vi_shift));
0250:         const int16x8_t vxk2 =
0251:             vreinterpretq_s16_u16(vsubl_u8(vk2, vkernel_zero_point));
0252:         const int16x8_t vxi2 =
0253:             vreinterpretq_s16_u16(vsubl_u8(vi2, vinput_zero_point));
0254:         vaccX0_lo =
0255:             vmlal_s16(vaccX0_lo, vget_low_s16(vxk2), vget_low_s16(vxi2));
0256:         vaccX0_hi =
0257:             vmlal_s16(vaccX0_hi, vget_high_s16(vxk2), vget_high_s16(vxi2));
0258: 
0259:         const uint8x8_t vk3 = vld1_u8(w);
0260:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0261:         const uint8x8_t vi3 = vreinterpret_u8_u64(
0262:             vshl_u64(vreinterpret_u64_u8(vld1_u8(i3)), vi_shift));
0263:         const int16x8_t vxk3 =
0264:             vreinterpretq_s16_u16(vsubl_u8(vk3, vkernel_zero_point));
0265:         const int16x8_t vxi3 =
0266:             vreinterpretq_s16_u16(vsubl_u8(vi3, vinput_zero_point));
0267:         vaccX1_lo =
0268:             vmlal_s16(vaccX1_lo, vget_low_s16(vxk3), vget_low_s16(vxi3));
0269:         vaccX1_hi =
0270:             vmlal_s16(vaccX1_hi, vget_high_s16(vxk3), vget_high_s16(vxi3));
0271: 
0272:         const uint8x8_t vk4 = vld1_u8(w);
0273:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0274:         const uint8x8_t vi4 = vreinterpret_u8_u64(
0275:             vshl_u64(vreinterpret_u64_u8(vld1_u8(i4)), vi_shift));
0276:         const int16x8_t vxk4 =
0277:             vreinterpretq_s16_u16(vsubl_u8(vk4, vkernel_zero_point));
0278:         const int16x8_t vxi4 =
0279:             vreinterpretq_s16_u16(vsubl_u8(vi4, vinput_zero_point));
0280:         vaccX0_lo =
0281:             vmlal_s16(vaccX0_lo, vget_low_s16(vxk4), vget_low_s16(vxi4));
0282:         vaccX0_hi =
0283:             vmlal_s16(vaccX0_hi, vget_high_s16(vxk4), vget_high_s16(vxi4));
0284: 
```

- **EN:** This block implements local helper logic for `mp8x25-neon`. Key symbols: `vshl_u64`, `vreinterpretq_s16_u16`, `vmlal_s16`.
- **CN:** 该代码块实现与 `mp8x25-neon` 相关的局部辅助逻辑。关键符号：`vshl_u64`, `vreinterpretq_s16_u16`, `vmlal_s16`。

### Lines 285-323 / 第 285-323 行

```c
0285:         const uint8x8_t vk5 = vld1_u8(w);
0286:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0287:         const uint8x8_t vi5 = vreinterpret_u8_u64(
0288:             vshl_u64(vreinterpret_u64_u8(vld1_u8(i5)), vi_shift));
0289:         const int16x8_t vxk5 =
0290:             vreinterpretq_s16_u16(vsubl_u8(vk5, vkernel_zero_point));
0291:         const int16x8_t vxi5 =
0292:             vreinterpretq_s16_u16(vsubl_u8(vi5, vinput_zero_point));
0293:         vaccX1_lo =
0294:             vmlal_s16(vaccX1_lo, vget_low_s16(vxk5), vget_low_s16(vxi5));
0295:         vaccX1_hi =
0296:             vmlal_s16(vaccX1_hi, vget_high_s16(vxk5), vget_high_s16(vxi5));
0297: 
0298:         const uint8x8_t vk6 = vld1_u8(w);
0299:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0300:         const uint8x8_t vi6 = vreinterpret_u8_u64(
0301:             vshl_u64(vreinterpret_u64_u8(vld1_u8(i6)), vi_shift));
0302:         const int16x8_t vxk6 =
0303:             vreinterpretq_s16_u16(vsubl_u8(vk6, vkernel_zero_point));
0304:         const int16x8_t vxi6 =
0305:             vreinterpretq_s16_u16(vsubl_u8(vi6, vinput_zero_point));
0306:         vaccX0_lo =
0307:             vmlal_s16(vaccX0_lo, vget_low_s16(vxk6), vget_low_s16(vxi6));
0308:         vaccX0_hi =
0309:             vmlal_s16(vaccX0_hi, vget_high_s16(vxk6), vget_high_s16(vxi6));
0310: 
0311:         const uint8x8_t vk7 = vld1_u8(w);
0312:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0313:         const uint8x8_t vi7 = vreinterpret_u8_u64(
0314:             vshl_u64(vreinterpret_u64_u8(vld1_u8(i7)), vi_shift));
0315:         const int16x8_t vxk7 =
0316:             vreinterpretq_s16_u16(vsubl_u8(vk7, vkernel_zero_point));
0317:         const int16x8_t vxi7 =
0318:             vreinterpretq_s16_u16(vsubl_u8(vi7, vinput_zero_point));
0319:         vaccX1_lo =
0320:             vmlal_s16(vaccX1_lo, vget_low_s16(vxk7), vget_low_s16(vxi7));
0321:         vaccX1_hi =
0322:             vmlal_s16(vaccX1_hi, vget_high_s16(vxk7), vget_high_s16(vxi7));
0323: 
```

- **EN:** This block implements local helper logic for `mp8x25-neon`. Key symbols: `vshl_u64`, `vreinterpretq_s16_u16`, `vmlal_s16`.
- **CN:** 该代码块实现与 `mp8x25-neon` 相关的局部辅助逻辑。关键符号：`vshl_u64`, `vreinterpretq_s16_u16`, `vmlal_s16`。

### Lines 324-372 / 第 324-372 行

```c
0324:         const uint8x8_t vk8 = vld1_u8(w);
0325:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0326:         const uint8x8_t vi8 = vreinterpret_u8_u64(
0327:             vshl_u64(vreinterpret_u64_u8(vld1_u8(i8)), vi_shift));
0328:         const int16x8_t vxk8 =
0329:             vreinterpretq_s16_u16(vsubl_u8(vk8, vkernel_zero_point));
0330:         const int16x8_t vxi8 =
0331:             vreinterpretq_s16_u16(vsubl_u8(vi8, vinput_zero_point));
0332:         vaccX0_lo =
0333:             vmlal_s16(vaccX0_lo, vget_low_s16(vxk8), vget_low_s16(vxi8));
0334:         vaccX0_hi =
0335:             vmlal_s16(vaccX0_hi, vget_high_s16(vxk8), vget_high_s16(vxi8));
0336: 
0337:         const uint8x8_t vk9 = vld1_u8(w);
0338:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0339:         const uint8x8_t vi9 = vreinterpret_u8_u64(
0340:             vshl_u64(vreinterpret_u64_u8(vld1_u8(i9)), vi_shift));
0341:         const int16x8_t vxk9 =
0342:             vreinterpretq_s16_u16(vsubl_u8(vk9, vkernel_zero_point));
0343:         const int16x8_t vxi9 =
0344:             vreinterpretq_s16_u16(vsubl_u8(vi9, vinput_zero_point));
0345:         vaccX1_lo =
0346:             vmlal_s16(vaccX1_lo, vget_low_s16(vxk9), vget_low_s16(vxi9));
0347:         vaccX1_hi =
0348:             vmlal_s16(vaccX1_hi, vget_high_s16(vxk9), vget_high_s16(vxi9));
0349: 
0350:         int32x4_t vacc_lo = vaddq_s32(vaccX0_lo, vaccX1_lo);
0351:         int32x4_t vacc_hi = vaddq_s32(vaccX0_hi, vaccX1_hi);
0352: 
0353:         vst1q_s32(outacc, vacc_lo);
0354:         outacc += 4;
0355:         vst1q_s32(outacc, vacc_hi);
0356:         outacc += 4;
0357:       }
0358:     }
0359:     {
0360:       const uint8_t* i0 = input[10];
0361:       const uint8_t* i1 = input[11];
0362:       const uint8_t* i2 = input[12];
0363:       const uint8_t* i3 = input[13];
0364:       const uint8_t* i4 = input[14];
0365:       const uint8_t* i5 = input[15];
0366:       const uint8_t* i6 = input[16];
0367:       const uint8_t* i7 = input[17];
0368:       const uint8_t* i8 = input[18];
0369:       const uint8_t* i9 = input[19];
0370:       output = output_start;
0371:       outacc = outacc32;
0372: 
```

- **EN:** This block implements local helper logic for `mp8x25-neon`. Key symbols: `vshl_u64`, `vreinterpretq_s16_u16`, `vmlal_s16`, `vst1q_s32`.
- **CN:** 该代码块实现与 `mp8x25-neon` 相关的局部辅助逻辑。关键符号：`vshl_u64`, `vreinterpretq_s16_u16`, `vmlal_s16`, `vst1q_s32`。

### Lines 373-411 / 第 373-411 行

```c
0373:       size_t c = channels;
0374:       for (; c >= 8; c -= 8) {
0375:         const uint8x8_t vk0 = vld1_u8(w);
0376:         w += 8;
0377:         const uint8x8_t vi0 = vld1_u8(i0);
0378:         i0 += 8;
0379:         const int16x8_t vxk0 =
0380:             vreinterpretq_s16_u16(vsubl_u8(vk0, vkernel_zero_point));
0381:         const int16x8_t vxi0 =
0382:             vreinterpretq_s16_u16(vsubl_u8(vi0, vinput_zero_point));
0383:         int32x4_t vaccX0_lo = vmull_s16(vget_low_s16(vxk0), vget_low_s16(vxi0));
0384:         int32x4_t vaccX0_hi =
0385:             vmull_s16(vget_high_s16(vxk0), vget_high_s16(vxi0));
0386: 
0387:         const uint8x8_t vk1 = vld1_u8(w);
0388:         w += 8;
0389:         const uint8x8_t vi1 = vld1_u8(i1);
0390:         i1 += 8;
0391:         const int16x8_t vxk1 =
0392:             vreinterpretq_s16_u16(vsubl_u8(vk1, vkernel_zero_point));
0393:         const int16x8_t vxi1 =
0394:             vreinterpretq_s16_u16(vsubl_u8(vi1, vinput_zero_point));
0395:         int32x4_t vaccX1_lo = vmull_s16(vget_low_s16(vxk1), vget_low_s16(vxi1));
0396:         int32x4_t vaccX1_hi =
0397:             vmull_s16(vget_high_s16(vxk1), vget_high_s16(vxi1));
0398: 
0399:         const uint8x8_t vk2 = vld1_u8(w);
0400:         w += 8;
0401:         const uint8x8_t vi2 = vld1_u8(i2);
0402:         i2 += 8;
0403:         const int16x8_t vxk2 =
0404:             vreinterpretq_s16_u16(vsubl_u8(vk2, vkernel_zero_point));
0405:         const int16x8_t vxi2 =
0406:             vreinterpretq_s16_u16(vsubl_u8(vi2, vinput_zero_point));
0407:         vaccX0_lo =
0408:             vmlal_s16(vaccX0_lo, vget_low_s16(vxk2), vget_low_s16(vxi2));
0409:         vaccX0_hi =
0410:             vmlal_s16(vaccX0_hi, vget_high_s16(vxk2), vget_high_s16(vxi2));
0411: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `vreinterpretq_s16_u16`, `vmull_s16`, `vmlal_s16`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`vreinterpretq_s16_u16`, `vmull_s16`, `vmlal_s16`。

### Lines 412-450 / 第 412-450 行

```c
0412:         const uint8x8_t vk3 = vld1_u8(w);
0413:         w += 8;
0414:         const uint8x8_t vi3 = vld1_u8(i3);
0415:         i3 += 8;
0416:         const int16x8_t vxk3 =
0417:             vreinterpretq_s16_u16(vsubl_u8(vk3, vkernel_zero_point));
0418:         const int16x8_t vxi3 =
0419:             vreinterpretq_s16_u16(vsubl_u8(vi3, vinput_zero_point));
0420:         vaccX1_lo =
0421:             vmlal_s16(vaccX1_lo, vget_low_s16(vxk3), vget_low_s16(vxi3));
0422:         vaccX1_hi =
0423:             vmlal_s16(vaccX1_hi, vget_high_s16(vxk3), vget_high_s16(vxi3));
0424: 
0425:         const uint8x8_t vk4 = vld1_u8(w);
0426:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0427:         const uint8x8_t vi4 = vld1_u8(i4);
0428:         i4 += 8;
0429:         const int16x8_t vxk4 =
0430:             vreinterpretq_s16_u16(vsubl_u8(vk4, vkernel_zero_point));
0431:         const int16x8_t vxi4 =
0432:             vreinterpretq_s16_u16(vsubl_u8(vi4, vinput_zero_point));
0433:         vaccX0_lo =
0434:             vmlal_s16(vaccX0_lo, vget_low_s16(vxk4), vget_low_s16(vxi4));
0435:         vaccX0_hi =
0436:             vmlal_s16(vaccX0_hi, vget_high_s16(vxk4), vget_high_s16(vxi4));
0437: 
0438:         const uint8x8_t vk5 = vld1_u8(w);
0439:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0440:         const uint8x8_t vi5 = vld1_u8(i5);
0441:         i5 += 8;
0442:         const int16x8_t vxk5 =
0443:             vreinterpretq_s16_u16(vsubl_u8(vk5, vkernel_zero_point));
0444:         const int16x8_t vxi5 =
0445:             vreinterpretq_s16_u16(vsubl_u8(vi5, vinput_zero_point));
0446:         vaccX1_lo =
0447:             vmlal_s16(vaccX1_lo, vget_low_s16(vxk5), vget_low_s16(vxi5));
0448:         vaccX1_hi =
0449:             vmlal_s16(vaccX1_hi, vget_high_s16(vxk5), vget_high_s16(vxi5));
0450: 
```

- **EN:** This block implements local helper logic for `mp8x25-neon`. Key symbols: `vreinterpretq_s16_u16`, `vmlal_s16`.
- **CN:** 该代码块实现与 `mp8x25-neon` 相关的局部辅助逻辑。关键符号：`vreinterpretq_s16_u16`, `vmlal_s16`。

### Lines 451-489 / 第 451-489 行

```c
0451:         const uint8x8_t vk6 = vld1_u8(w);
0452:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0453:         const uint8x8_t vi6 = vld1_u8(i6);
0454:         i6 += 8;
0455:         const int16x8_t vxk6 =
0456:             vreinterpretq_s16_u16(vsubl_u8(vk6, vkernel_zero_point));
0457:         const int16x8_t vxi6 =
0458:             vreinterpretq_s16_u16(vsubl_u8(vi6, vinput_zero_point));
0459:         vaccX0_lo =
0460:             vmlal_s16(vaccX0_lo, vget_low_s16(vxk6), vget_low_s16(vxi6));
0461:         vaccX0_hi =
0462:             vmlal_s16(vaccX0_hi, vget_high_s16(vxk6), vget_high_s16(vxi6));
0463: 
0464:         const uint8x8_t vk7 = vld1_u8(w);
0465:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0466:         const uint8x8_t vi7 = vld1_u8(i7);
0467:         i7 += 8;
0468:         const int16x8_t vxk7 =
0469:             vreinterpretq_s16_u16(vsubl_u8(vk7, vkernel_zero_point));
0470:         const int16x8_t vxi7 =
0471:             vreinterpretq_s16_u16(vsubl_u8(vi7, vinput_zero_point));
0472:         vaccX1_lo =
0473:             vmlal_s16(vaccX1_lo, vget_low_s16(vxk7), vget_low_s16(vxi7));
0474:         vaccX1_hi =
0475:             vmlal_s16(vaccX1_hi, vget_high_s16(vxk7), vget_high_s16(vxi7));
0476: 
0477:         const uint8x8_t vk8 = vld1_u8(w);
0478:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0479:         const uint8x8_t vi8 = vld1_u8(i8);
0480:         i8 += 8;
0481:         const int16x8_t vxk8 =
0482:             vreinterpretq_s16_u16(vsubl_u8(vk8, vkernel_zero_point));
0483:         const int16x8_t vxi8 =
0484:             vreinterpretq_s16_u16(vsubl_u8(vi8, vinput_zero_point));
0485:         vaccX0_lo =
0486:             vmlal_s16(vaccX0_lo, vget_low_s16(vxk8), vget_low_s16(vxi8));
0487:         vaccX0_hi =
0488:             vmlal_s16(vaccX0_hi, vget_high_s16(vxk8), vget_high_s16(vxi8));
0489: 
```

- **EN:** This block implements local helper logic for `mp8x25-neon`. Key symbols: `vreinterpretq_s16_u16`, `vmlal_s16`.
- **CN:** 该代码块实现与 `mp8x25-neon` 相关的局部辅助逻辑。关键符号：`vreinterpretq_s16_u16`, `vmlal_s16`。

### Lines 490-528 / 第 490-528 行

```c
0490:         const uint8x8_t vk9 = vld1_u8(w);
0491:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0492:         const uint8x8_t vi9 = vld1_u8(i9);
0493:         i9 += 8;
0494:         const int16x8_t vxk9 =
0495:             vreinterpretq_s16_u16(vsubl_u8(vk9, vkernel_zero_point));
0496:         const int16x8_t vxi9 =
0497:             vreinterpretq_s16_u16(vsubl_u8(vi9, vinput_zero_point));
0498:         vaccX1_lo =
0499:             vmlal_s16(vaccX1_lo, vget_low_s16(vxk9), vget_low_s16(vxi9));
0500:         vaccX1_hi =
0501:             vmlal_s16(vaccX1_hi, vget_high_s16(vxk9), vget_high_s16(vxi9));
0502: 
0503:         int32x4_t vacc_lo = vaddq_s32(vaccX0_lo, vaccX1_lo);
0504:         int32x4_t vacc_hi = vaddq_s32(vaccX0_hi, vaccX1_hi);
0505: 
0506:         const int32x4_t vacc_lo_old = vld1q_s32(outacc);
0507:         const int32x4_t vacc_hi_old = vld1q_s32(outacc + 4);
0508:         vacc_lo = vaddq_s32(vacc_lo, vacc_lo_old);
0509:         vacc_hi = vaddq_s32(vacc_hi, vacc_hi_old);
0510:         vst1q_s32(outacc, vacc_lo);
0511:         outacc += 4;
0512:         vst1q_s32(outacc, vacc_hi);
0513:         outacc += 4;
0514:       }
0515:       if (c != 0) {
0516:         const size_t c_predecrement = 8 - c;
0517:         const int64x1_t vi_shift = vmov_n_s64(-8 * c_predecrement);
0518:         i0 -= c_predecrement;
0519:         i1 -= c_predecrement;
0520:         i2 -= c_predecrement;
0521:         i3 -= c_predecrement;
0522:         i4 -= c_predecrement;
0523:         i5 -= c_predecrement;
0524:         i6 -= c_predecrement;
0525:         i7 -= c_predecrement;
0526:         i8 -= c_predecrement;
0527:         i9 -= c_predecrement;
0528: 
```

- **EN:** This block handles conditional branches and special cases. Key symbols: `vreinterpretq_s16_u16`, `vmlal_s16`, `vst1q_s32`.
- **CN:** 该代码块处理条件分支与特殊情况。关键符号：`vreinterpretq_s16_u16`, `vmlal_s16`, `vst1q_s32`。

### Lines 529-565 / 第 529-565 行

```c
0529:         const uint8x8_t vk0 = vld1_u8(w);
0530:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0531:         const uint8x8_t vi0 = vreinterpret_u8_u64(
0532:             vshl_u64(vreinterpret_u64_u8(vld1_u8(i0)), vi_shift));
0533:         const int16x8_t vxk0 =
0534:             vreinterpretq_s16_u16(vsubl_u8(vk0, vkernel_zero_point));
0535:         const int16x8_t vxi0 =
0536:             vreinterpretq_s16_u16(vsubl_u8(vi0, vinput_zero_point));
0537:         int32x4_t vaccX0_lo = vmull_s16(vget_low_s16(vxk0), vget_low_s16(vxi0));
0538:         int32x4_t vaccX0_hi =
0539:             vmull_s16(vget_high_s16(vxk0), vget_high_s16(vxi0));
0540: 
0541:         const uint8x8_t vk1 = vld1_u8(w);
0542:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0543:         const uint8x8_t vi1 = vreinterpret_u8_u64(
0544:             vshl_u64(vreinterpret_u64_u8(vld1_u8(i1)), vi_shift));
0545:         const int16x8_t vxk1 =
0546:             vreinterpretq_s16_u16(vsubl_u8(vk1, vkernel_zero_point));
0547:         const int16x8_t vxi1 =
0548:             vreinterpretq_s16_u16(vsubl_u8(vi1, vinput_zero_point));
0549:         int32x4_t vaccX1_lo = vmull_s16(vget_low_s16(vxk1), vget_low_s16(vxi1));
0550:         int32x4_t vaccX1_hi =
0551:             vmull_s16(vget_high_s16(vxk1), vget_high_s16(vxi1));
0552: 
0553:         const uint8x8_t vk2 = vld1_u8(w);
0554:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0555:         const uint8x8_t vi2 = vreinterpret_u8_u64(
0556:             vshl_u64(vreinterpret_u64_u8(vld1_u8(i2)), vi_shift));
0557:         const int16x8_t vxk2 =
0558:             vreinterpretq_s16_u16(vsubl_u8(vk2, vkernel_zero_point));
0559:         const int16x8_t vxi2 =
0560:             vreinterpretq_s16_u16(vsubl_u8(vi2, vinput_zero_point));
0561:         vaccX0_lo =
0562:             vmlal_s16(vaccX0_lo, vget_low_s16(vxk2), vget_low_s16(vxi2));
0563:         vaccX0_hi =
0564:             vmlal_s16(vaccX0_hi, vget_high_s16(vxk2), vget_high_s16(vxi2));
0565: 
```

- **EN:** This block implements local helper logic for `mp8x25-neon`. Key symbols: `vshl_u64`, `vreinterpretq_s16_u16`, `vmull_s16`, `vmlal_s16`.
- **CN:** 该代码块实现与 `mp8x25-neon` 相关的局部辅助逻辑。关键符号：`vshl_u64`, `vreinterpretq_s16_u16`, `vmull_s16`, `vmlal_s16`。

### Lines 566-604 / 第 566-604 行

```c
0566:         const uint8x8_t vk3 = vld1_u8(w);
0567:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0568:         const uint8x8_t vi3 = vreinterpret_u8_u64(
0569:             vshl_u64(vreinterpret_u64_u8(vld1_u8(i3)), vi_shift));
0570:         const int16x8_t vxk3 =
0571:             vreinterpretq_s16_u16(vsubl_u8(vk3, vkernel_zero_point));
0572:         const int16x8_t vxi3 =
0573:             vreinterpretq_s16_u16(vsubl_u8(vi3, vinput_zero_point));
0574:         vaccX1_lo =
0575:             vmlal_s16(vaccX1_lo, vget_low_s16(vxk3), vget_low_s16(vxi3));
0576:         vaccX1_hi =
0577:             vmlal_s16(vaccX1_hi, vget_high_s16(vxk3), vget_high_s16(vxi3));
0578: 
0579:         const uint8x8_t vk4 = vld1_u8(w);
0580:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0581:         const uint8x8_t vi4 = vreinterpret_u8_u64(
0582:             vshl_u64(vreinterpret_u64_u8(vld1_u8(i4)), vi_shift));
0583:         const int16x8_t vxk4 =
0584:             vreinterpretq_s16_u16(vsubl_u8(vk4, vkernel_zero_point));
0585:         const int16x8_t vxi4 =
0586:             vreinterpretq_s16_u16(vsubl_u8(vi4, vinput_zero_point));
0587:         vaccX0_lo =
0588:             vmlal_s16(vaccX0_lo, vget_low_s16(vxk4), vget_low_s16(vxi4));
0589:         vaccX0_hi =
0590:             vmlal_s16(vaccX0_hi, vget_high_s16(vxk4), vget_high_s16(vxi4));
0591: 
0592:         const uint8x8_t vk5 = vld1_u8(w);
0593:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0594:         const uint8x8_t vi5 = vreinterpret_u8_u64(
0595:             vshl_u64(vreinterpret_u64_u8(vld1_u8(i5)), vi_shift));
0596:         const int16x8_t vxk5 =
0597:             vreinterpretq_s16_u16(vsubl_u8(vk5, vkernel_zero_point));
0598:         const int16x8_t vxi5 =
0599:             vreinterpretq_s16_u16(vsubl_u8(vi5, vinput_zero_point));
0600:         vaccX1_lo =
0601:             vmlal_s16(vaccX1_lo, vget_low_s16(vxk5), vget_low_s16(vxi5));
0602:         vaccX1_hi =
0603:             vmlal_s16(vaccX1_hi, vget_high_s16(vxk5), vget_high_s16(vxi5));
0604: 
```

- **EN:** This block implements local helper logic for `mp8x25-neon`. Key symbols: `vshl_u64`, `vreinterpretq_s16_u16`, `vmlal_s16`.
- **CN:** 该代码块实现与 `mp8x25-neon` 相关的局部辅助逻辑。关键符号：`vshl_u64`, `vreinterpretq_s16_u16`, `vmlal_s16`。

### Lines 605-643 / 第 605-643 行

```c
0605:         const uint8x8_t vk6 = vld1_u8(w);
0606:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0607:         const uint8x8_t vi6 = vreinterpret_u8_u64(
0608:             vshl_u64(vreinterpret_u64_u8(vld1_u8(i6)), vi_shift));
0609:         const int16x8_t vxk6 =
0610:             vreinterpretq_s16_u16(vsubl_u8(vk6, vkernel_zero_point));
0611:         const int16x8_t vxi6 =
0612:             vreinterpretq_s16_u16(vsubl_u8(vi6, vinput_zero_point));
0613:         vaccX0_lo =
0614:             vmlal_s16(vaccX0_lo, vget_low_s16(vxk6), vget_low_s16(vxi6));
0615:         vaccX0_hi =
0616:             vmlal_s16(vaccX0_hi, vget_high_s16(vxk6), vget_high_s16(vxi6));
0617: 
0618:         const uint8x8_t vk7 = vld1_u8(w);
0619:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0620:         const uint8x8_t vi7 = vreinterpret_u8_u64(
0621:             vshl_u64(vreinterpret_u64_u8(vld1_u8(i7)), vi_shift));
0622:         const int16x8_t vxk7 =
0623:             vreinterpretq_s16_u16(vsubl_u8(vk7, vkernel_zero_point));
0624:         const int16x8_t vxi7 =
0625:             vreinterpretq_s16_u16(vsubl_u8(vi7, vinput_zero_point));
0626:         vaccX1_lo =
0627:             vmlal_s16(vaccX1_lo, vget_low_s16(vxk7), vget_low_s16(vxi7));
0628:         vaccX1_hi =
0629:             vmlal_s16(vaccX1_hi, vget_high_s16(vxk7), vget_high_s16(vxi7));
0630: 
0631:         const uint8x8_t vk8 = vld1_u8(w);
0632:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0633:         const uint8x8_t vi8 = vreinterpret_u8_u64(
0634:             vshl_u64(vreinterpret_u64_u8(vld1_u8(i8)), vi_shift));
0635:         const int16x8_t vxk8 =
0636:             vreinterpretq_s16_u16(vsubl_u8(vk8, vkernel_zero_point));
0637:         const int16x8_t vxi8 =
0638:             vreinterpretq_s16_u16(vsubl_u8(vi8, vinput_zero_point));
0639:         vaccX0_lo =
0640:             vmlal_s16(vaccX0_lo, vget_low_s16(vxk8), vget_low_s16(vxi8));
0641:         vaccX0_hi =
0642:             vmlal_s16(vaccX0_hi, vget_high_s16(vxk8), vget_high_s16(vxi8));
0643: 
```

- **EN:** This block implements local helper logic for `mp8x25-neon`. Key symbols: `vshl_u64`, `vreinterpretq_s16_u16`, `vmlal_s16`.
- **CN:** 该代码块实现与 `mp8x25-neon` 相关的局部辅助逻辑。关键符号：`vshl_u64`, `vreinterpretq_s16_u16`, `vmlal_s16`。

### Lines 644-680 / 第 644-680 行

```c
0644:         const uint8x8_t vk9 = vld1_u8(w);
0645:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0646:         const uint8x8_t vi9 = vreinterpret_u8_u64(
0647:             vshl_u64(vreinterpret_u64_u8(vld1_u8(i9)), vi_shift));
0648:         const int16x8_t vxk9 =
0649:             vreinterpretq_s16_u16(vsubl_u8(vk9, vkernel_zero_point));
0650:         const int16x8_t vxi9 =
0651:             vreinterpretq_s16_u16(vsubl_u8(vi9, vinput_zero_point));
0652:         vaccX1_lo =
0653:             vmlal_s16(vaccX1_lo, vget_low_s16(vxk9), vget_low_s16(vxi9));
0654:         vaccX1_hi =
0655:             vmlal_s16(vaccX1_hi, vget_high_s16(vxk9), vget_high_s16(vxi9));
0656: 
0657:         int32x4_t vacc_lo = vaddq_s32(vaccX0_lo, vaccX1_lo);
0658:         int32x4_t vacc_hi = vaddq_s32(vaccX0_hi, vaccX1_hi);
0659: 
0660:         const int32x4_t vacc_lo_old = vld1q_s32(outacc);
0661:         const int32x4_t vacc_hi_old = vld1q_s32(outacc + 4);
0662:         vacc_lo = vaddq_s32(vacc_lo, vacc_lo_old);
0663:         vacc_hi = vaddq_s32(vacc_hi, vacc_hi_old);
0664:         vst1q_s32(outacc, vacc_lo);
0665:         outacc += 4;
0666:         vst1q_s32(outacc, vacc_hi);
0667:         outacc += 4;
0668:       }
0669:     }
0670: 
0671:     {
0672:       const uint8_t* i0 = input[20];
0673:       const uint8_t* i1 = input[21];
0674:       const uint8_t* i2 = input[22];
0675:       const uint8_t* i3 = input[23];
0676:       const uint8_t* i4 = input[24];
0677:       input = (const uint8_t**)((uintptr_t)input + input_stride);
0678:       output = output_start;
0679:       outacc = outacc32;
0680: 
```

- **EN:** This block implements local helper logic for `mp8x25-neon`. Key symbols: `vshl_u64`, `vreinterpretq_s16_u16`, `vmlal_s16`, `vst1q_s32`.
- **CN:** 该代码块实现与 `mp8x25-neon` 相关的局部辅助逻辑。关键符号：`vshl_u64`, `vreinterpretq_s16_u16`, `vmlal_s16`, `vst1q_s32`。

### Lines 681-719 / 第 681-719 行

```c
0681:       size_t c = channels;
0682:       for (; c >= 8; c -= 8) {
0683:         const uint8x8_t vk0 = vld1_u8(w);
0684:         w += 8;
0685:         const uint8x8_t vi0 = vld1_u8(i0);
0686:         i0 += 8;
0687:         const int16x8_t vxk0 =
0688:             vreinterpretq_s16_u16(vsubl_u8(vk0, vkernel_zero_point));
0689:         const int16x8_t vxi0 =
0690:             vreinterpretq_s16_u16(vsubl_u8(vi0, vinput_zero_point));
0691:         int32x4_t vaccX0_lo = vmull_s16(vget_low_s16(vxk0), vget_low_s16(vxi0));
0692:         int32x4_t vaccX0_hi =
0693:             vmull_s16(vget_high_s16(vxk0), vget_high_s16(vxi0));
0694: 
0695:         const uint8x8_t vk1 = vld1_u8(w);
0696:         w += 8;
0697:         const uint8x8_t vi1 = vld1_u8(i1);
0698:         i1 += 8;
0699:         const int16x8_t vxk1 =
0700:             vreinterpretq_s16_u16(vsubl_u8(vk1, vkernel_zero_point));
0701:         const int16x8_t vxi1 =
0702:             vreinterpretq_s16_u16(vsubl_u8(vi1, vinput_zero_point));
0703:         int32x4_t vaccX1_lo = vmull_s16(vget_low_s16(vxk1), vget_low_s16(vxi1));
0704:         int32x4_t vaccX1_hi =
0705:             vmull_s16(vget_high_s16(vxk1), vget_high_s16(vxi1));
0706: 
0707:         const uint8x8_t vk2 = vld1_u8(w);
0708:         w += 8;
0709:         const uint8x8_t vi2 = vld1_u8(i2);
0710:         i2 += 8;
0711:         const int16x8_t vxk2 =
0712:             vreinterpretq_s16_u16(vsubl_u8(vk2, vkernel_zero_point));
0713:         const int16x8_t vxi2 =
0714:             vreinterpretq_s16_u16(vsubl_u8(vi2, vinput_zero_point));
0715:         vaccX0_lo =
0716:             vmlal_s16(vaccX0_lo, vget_low_s16(vxk2), vget_low_s16(vxi2));
0717:         vaccX0_hi =
0718:             vmlal_s16(vaccX0_hi, vget_high_s16(vxk2), vget_high_s16(vxi2));
0719: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `vreinterpretq_s16_u16`, `vmull_s16`, `vmlal_s16`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`vreinterpretq_s16_u16`, `vmull_s16`, `vmlal_s16`。

### Lines 720-755 / 第 720-755 行

```c
0720:         const uint8x8_t vk3 = vld1_u8(w);
0721:         w += 8;
0722:         const uint8x8_t vi3 = vld1_u8(i3);
0723:         i3 += 8;
0724:         const int16x8_t vxk3 =
0725:             vreinterpretq_s16_u16(vsubl_u8(vk3, vkernel_zero_point));
0726:         const int16x8_t vxi3 =
0727:             vreinterpretq_s16_u16(vsubl_u8(vi3, vinput_zero_point));
0728:         vaccX1_lo =
0729:             vmlal_s16(vaccX1_lo, vget_low_s16(vxk3), vget_low_s16(vxi3));
0730:         vaccX1_hi =
0731:             vmlal_s16(vaccX1_hi, vget_high_s16(vxk3), vget_high_s16(vxi3));
0732: 
0733:         const uint8x8_t vk4 = vld1_u8(w);
0734:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0735:         const uint8x8_t vi4 = vld1_u8(i4);
0736:         i4 += 8;
0737:         const int16x8_t vxk4 =
0738:             vreinterpretq_s16_u16(vsubl_u8(vk4, vkernel_zero_point));
0739:         const int16x8_t vxi4 =
0740:             vreinterpretq_s16_u16(vsubl_u8(vi4, vinput_zero_point));
0741:         vaccX0_lo =
0742:             vmlal_s16(vaccX0_lo, vget_low_s16(vxk4), vget_low_s16(vxi4));
0743:         vaccX0_hi =
0744:             vmlal_s16(vaccX0_hi, vget_high_s16(vxk4), vget_high_s16(vxi4));
0745: 
0746:         int32x4_t vacc_lo = vaddq_s32(vaccX0_lo, vaccX1_lo);
0747:         int32x4_t vacc_hi = vaddq_s32(vaccX0_hi, vaccX1_hi);
0748: 
0749:         const int32x4_t vacc_lo_old = vld1q_s32(outacc);
0750:         outacc += 4;
0751:         const int32x4_t vacc_hi_old = vld1q_s32(outacc);
0752:         outacc += 4;
0753:         vacc_lo = vaddq_s32(vacc_lo, vacc_lo_old);
0754:         vacc_hi = vaddq_s32(vacc_hi, vacc_hi_old);
0755: 
```

- **EN:** This block implements local helper logic for `mp8x25-neon`. Key symbols: `vreinterpretq_s16_u16`, `vmlal_s16`.
- **CN:** 该代码块实现与 `mp8x25-neon` 相关的局部辅助逻辑。关键符号：`vreinterpretq_s16_u16`, `vmlal_s16`。

### Lines 756-785 / 第 756-785 行

```c
0756:         const float32x4_t vacc_lo_f =
0757:           vmulq_f32(vcvtq_f32_s32(vacc_lo), requantization_scale_v);
0758:         const float32x4_t vacc_hi_f =
0759:           vmulq_f32(vcvtq_f32_s32(vacc_hi), requantization_scale_v);
0760: 
0761: #ifdef __aarch64__
0762:         vacc_lo = vcvtnq_s32_f32(vacc_lo_f);
0763:         vacc_hi = vcvtnq_s32_f32(vacc_hi_f);
0764: 
0765:         const int16x8_t vacc = vqaddq_s16(
0766:             vqmovn_high_s32(vqmovn_s32(vacc_lo), vacc_hi), voutput_zero_point);
0767: 
0768:         uint8x8_t vout = vqmovun_s16(vacc);
0769:         vout = vmax_u8(vout, voutput_min);
0770:         vout = vmin_u8(vout, voutput_max);
0771: #else
0772:         const float32x4_t vacc_lo_f_clamped =
0773:             vminq_f32(vmaxq_f32(vacc_lo_f, vfmin), vfmax);
0774:         const float32x4_t vacc_hi_f_clamped =
0775:             vminq_f32(vmaxq_f32(vacc_hi_f, vfmin), vfmax);
0776:         vacc_lo = vsubq_s32(
0777:             vreinterpretq_s32_f32(vaddq_f32(vacc_lo_f_clamped, vfmagic)), vimagic);
0778:         vacc_hi = vsubq_s32(
0779:             vreinterpretq_s32_f32(vaddq_f32(vacc_hi_f_clamped, vfmagic)), vimagic);
0780:         const int16x8_t vacc =
0781:             vcombine_s16(vqmovn_s32(vacc_lo), vqmovn_s32(vacc_hi));
0782: 
0783:         uint8x8_t vout = vqmovun_s16(vacc);
0784: #endif
0785: 
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends. Key symbols: `vmulq_f32`, `vqmovn_high_s32`, `vminq_f32`, `vreinterpretq_s32_f32`, `vcombine_s16`.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端。关键符号：`vmulq_f32`, `vqmovn_high_s32`, `vminq_f32`, `vreinterpretq_s32_f32`, `vcombine_s16`。

### Lines 786-821 / 第 786-821 行

```c
0786:         vst1_u8(output, vout);
0787:         output += 8;
0788:       }
0789:       if (c != 0) {
0790:         const size_t c_predecrement = 8 - c;
0791:         const int64x1_t vi_shift = vmov_n_s64(-8 * c_predecrement);
0792:         i0 -= c_predecrement;
0793:         i1 -= c_predecrement;
0794:         i2 -= c_predecrement;
0795:         i3 -= c_predecrement;
0796:         i4 -= c_predecrement;
0797: 
0798:         const uint8x8_t vk0 = vld1_u8(w);
0799:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0800:         const uint8x8_t vi0 = vreinterpret_u8_u64(
0801:             vshl_u64(vreinterpret_u64_u8(vld1_u8(i0)), vi_shift));
0802:         const int16x8_t vxk0 =
0803:             vreinterpretq_s16_u16(vsubl_u8(vk0, vkernel_zero_point));
0804:         const int16x8_t vxi0 =
0805:             vreinterpretq_s16_u16(vsubl_u8(vi0, vinput_zero_point));
0806:         int32x4_t vaccX0_lo = vmull_s16(vget_low_s16(vxk0), vget_low_s16(vxi0));
0807:         int32x4_t vaccX0_hi =
0808:             vmull_s16(vget_high_s16(vxk0), vget_high_s16(vxi0));
0809: 
0810:         const uint8x8_t vk1 = vld1_u8(w);
0811:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0812:         const uint8x8_t vi1 = vreinterpret_u8_u64(
0813:             vshl_u64(vreinterpret_u64_u8(vld1_u8(i1)), vi_shift));
0814:         const int16x8_t vxk1 =
0815:             vreinterpretq_s16_u16(vsubl_u8(vk1, vkernel_zero_point));
0816:         const int16x8_t vxi1 =
0817:             vreinterpretq_s16_u16(vsubl_u8(vi1, vinput_zero_point));
0818:         int32x4_t vaccX1_lo = vmull_s16(vget_low_s16(vxk1), vget_low_s16(vxi1));
0819:         int32x4_t vaccX1_hi =
0820:             vmull_s16(vget_high_s16(vxk1), vget_high_s16(vxi1));
0821: 
```

- **EN:** This block handles conditional branches and special cases. Key symbols: `vst1_u8`, `vshl_u64`, `vreinterpretq_s16_u16`, `vmull_s16`.
- **CN:** 该代码块处理条件分支与特殊情况。关键符号：`vst1_u8`, `vshl_u64`, `vreinterpretq_s16_u16`, `vmull_s16`。

### Lines 822-860 / 第 822-860 行

```c
0822:         const uint8x8_t vk2 = vld1_u8(w);
0823:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0824:         const uint8x8_t vi2 = vreinterpret_u8_u64(
0825:             vshl_u64(vreinterpret_u64_u8(vld1_u8(i2)), vi_shift));
0826:         const int16x8_t vxk2 =
0827:             vreinterpretq_s16_u16(vsubl_u8(vk2, vkernel_zero_point));
0828:         const int16x8_t vxi2 =
0829:             vreinterpretq_s16_u16(vsubl_u8(vi2, vinput_zero_point));
0830:         vaccX0_lo =
0831:             vmlal_s16(vaccX0_lo, vget_low_s16(vxk2), vget_low_s16(vxi2));
0832:         vaccX0_hi =
0833:             vmlal_s16(vaccX0_hi, vget_high_s16(vxk2), vget_high_s16(vxi2));
0834: 
0835:         const uint8x8_t vk3 = vld1_u8(w);
0836:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0837:         const uint8x8_t vi3 = vreinterpret_u8_u64(
0838:             vshl_u64(vreinterpret_u64_u8(vld1_u8(i3)), vi_shift));
0839:         const int16x8_t vxk3 =
0840:             vreinterpretq_s16_u16(vsubl_u8(vk3, vkernel_zero_point));
0841:         const int16x8_t vxi3 =
0842:             vreinterpretq_s16_u16(vsubl_u8(vi3, vinput_zero_point));
0843:         vaccX1_lo =
0844:             vmlal_s16(vaccX1_lo, vget_low_s16(vxk3), vget_low_s16(vxi3));
0845:         vaccX1_hi =
0846:             vmlal_s16(vaccX1_hi, vget_high_s16(vxk3), vget_high_s16(vxi3));
0847: 
0848:         const uint8x8_t vk4 = vld1_u8(w);
0849:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0850:         const uint8x8_t vi4 = vreinterpret_u8_u64(
0851:             vshl_u64(vreinterpret_u64_u8(vld1_u8(i4)), vi_shift));
0852:         const int16x8_t vxk4 =
0853:             vreinterpretq_s16_u16(vsubl_u8(vk4, vkernel_zero_point));
0854:         const int16x8_t vxi4 =
0855:             vreinterpretq_s16_u16(vsubl_u8(vi4, vinput_zero_point));
0856:         vaccX0_lo =
0857:             vmlal_s16(vaccX0_lo, vget_low_s16(vxk4), vget_low_s16(vxi4));
0858:         vaccX0_hi =
0859:             vmlal_s16(vaccX0_hi, vget_high_s16(vxk4), vget_high_s16(vxi4));
0860: 
```

- **EN:** This block implements local helper logic for `mp8x25-neon`. Key symbols: `vshl_u64`, `vreinterpretq_s16_u16`, `vmlal_s16`.
- **CN:** 该代码块实现与 `mp8x25-neon` 相关的局部辅助逻辑。关键符号：`vshl_u64`, `vreinterpretq_s16_u16`, `vmlal_s16`。

### Lines 861-895 / 第 861-895 行

```c
0861:         int32x4_t vacc_lo = vaddq_s32(vaccX0_lo, vaccX1_lo);
0862:         int32x4_t vacc_hi = vaddq_s32(vaccX0_hi, vaccX1_hi);
0863: 
0864:         const int32x4_t vacc_lo_old = vld1q_s32(outacc);
0865:         const int32x4_t vacc_hi_old = vld1q_s32(outacc + 4);
0866:         vacc_lo = vaddq_s32(vacc_lo, vacc_lo_old);
0867:         vacc_hi = vaddq_s32(vacc_hi, vacc_hi_old);
0868: 
0869:         const float32x4_t vacc_lo_f =
0870:           vmulq_f32(vcvtq_f32_s32(vacc_lo), requantization_scale_v);
0871:         const float32x4_t vacc_hi_f =
0872:           vmulq_f32(vcvtq_f32_s32(vacc_hi), requantization_scale_v);
0873: 
0874: #ifdef __aarch64__
0875:         vacc_lo = vcvtnq_s32_f32(vacc_lo_f);
0876:         vacc_hi = vcvtnq_s32_f32(vacc_hi_f);
0877: 
0878:         const int16x8_t vacc = vqaddq_s16(
0879:             vqmovn_high_s32(vqmovn_s32(vacc_lo), vacc_hi), voutput_zero_point);
0880: 
0881:         uint8x8_t vout = vqmovun_s16(vacc);
0882:         vout = vmax_u8(vout, voutput_min);
0883:         vout = vmin_u8(vout, voutput_max);
0884: #else
0885:         const float32x4_t vacc_lo_f_clamped =
0886:             vminq_f32(vmaxq_f32(vacc_lo_f, vfmin), vfmax);
0887:         const float32x4_t vacc_hi_f_clamped =
0888:             vminq_f32(vmaxq_f32(vacc_hi_f, vfmin), vfmax);
0889:         vacc_lo = vsubq_s32(
0890:             vreinterpretq_s32_f32(vaddq_f32(vacc_lo_f_clamped, vfmagic)), vimagic);
0891:         vacc_hi = vsubq_s32(
0892:             vreinterpretq_s32_f32(vaddq_f32(vacc_hi_f_clamped, vfmagic)), vimagic);
0893:         const int16x8_t vacc =
0894:             vcombine_s16(vqmovn_s32(vacc_lo), vqmovn_s32(vacc_hi));
0895: 
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends. Key symbols: `vmulq_f32`, `vqmovn_high_s32`, `vminq_f32`, `vreinterpretq_s32_f32`, `vcombine_s16`.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端。关键符号：`vmulq_f32`, `vqmovn_high_s32`, `vminq_f32`, `vreinterpretq_s32_f32`, `vcombine_s16`。

### Lines 896-924 / 第 896-924 行

```c
0896:         uint8x8_t vout = vqmovun_s16(vacc);
0897: #endif
0898: 
0899:         if (c & 4) {
0900:           vst1_lane_u32(
0901:               __builtin_assume_aligned(output, 1),
0902:               vreinterpret_u32_u8(vout),
0903:               0);
0904:           output += 4;
0905:           vout = vext_u8(vout, vout, 4);
0906:         }
0907:         if (c & 2) {
0908:           vst1_lane_u16(
0909:               __builtin_assume_aligned(output, 1),
0910:               vreinterpret_u16_u8(vout),
0911:               0);
0912:           output += 2;
0913:           vout = vext_u8(vout, vout, 2);
0914:         }
0915:         if (c & 1) {
0916:           vst1_lane_u8(__builtin_assume_aligned(output, 1), vout, 0);
0917:           output++;
0918:         }
0919:       }
0920:     }
0921: 
0922:     output = (uint8_t*)((uintptr_t)output + output_increment);
0923:   } while (--output_width != 0);
0924: }
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends; handles conditional branches and special cases; iterates over tensors, descriptors, options, or runtime state. Key symbols: `vst1_lane_u32`, `vst1_lane_u16`, `vst1_lane_u8`.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端；处理条件分支与特殊情况；遍历张量、描述符、配置项或运行时状态。关键符号：`vst1_lane_u32`, `vst1_lane_u16`, `vst1_lane_u8`。


## Key Concepts / 关键概念
- **ATen native operator logic** — ATen 原生算子逻辑
- **GPU kernel/runtime interaction** — GPU 内核/运行时交互
- **Mixed-precision/autocast logic** — 混合精度/自动混精逻辑
- **Parallel scan/reduction behavior** — 并行扫描/归约行为
- **Core symbols: pytorch_q8dwconv_ukernel_mp8x25__neon, vld1_dup_u8, vdup_n_u8, vdupq_n_f32, vld1q_dup_s16, vreinterpretq_s16_u16, vmull_s16, vmlal_s16** — 核心符号：pytorch_q8dwconv_ukernel_mp8x25__neon、vld1_dup_u8、vdup_n_u8、vdupq_n_f32、vld1q_dup_s16、vreinterpretq_s16_u16、vmull_s16、vmlal_s16

## Dependencies / 依赖关系
- **Internal includes / 内部头文件**: 无
- **External includes / 外部头文件**: `arm_neon.h`, `qnnpack/q8dwconv.h`
- **Namespaces / 命名空间**: 无
- **Representative symbols / 代表性符号**: `pytorch_q8dwconv_ukernel_mp8x25__neon`, `vld1_dup_u8`, `vdup_n_u8`, `vdupq_n_f32`, `vld1q_dup_s16`, `vreinterpretq_s16_u16`, `vmull_s16`, `vmlal_s16`, `vst1q_s32`, `vshl_u64`, `vmulq_f32`, `vqmovn_high_s32`, `...`
