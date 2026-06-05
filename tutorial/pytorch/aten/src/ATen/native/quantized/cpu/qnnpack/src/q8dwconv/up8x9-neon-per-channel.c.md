# up8x9-neon-per-channel.c — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `aten/src/ATen/native/quantized/cpu/qnnpack/src/q8dwconv/up8x9-neon-per-channel.c`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements ATen native operators, checks, helper kernels, and backend dispatch behavior. This specific file centers on `up8x9-neon-per-channel.c`. The file header highlights: "All rights reserved. This source code is licensed under the BSD-style license found in the LICENSE file in the root directory of this source tree.."
- **Purpose (CN)**: 实现 ATen 原生算子、运行时检查、辅助内核以及后端分发行为。 该文件具体围绕 `up8x9-neon-per-channel.c` 展开。 文件头部注释也概括了其核心职责。

## Line-by-Line Analysis / 逐行分析
### Lines 1-38 / 第 1-38 行

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
0012: #include <requantization/runtime-neon.h>
0013: 
0014: void pytorch_q8dwconv_ukernel_up8x9_per_channel__neon(
0015:     size_t channels,
0016:     size_t output_width,
0017:     const uint8_t** input,
0018:     const void* weights,
0019:     uint8_t* output,
0020:     size_t input_stride,
0021:     size_t output_increment,
0022:     const union pytorch_qnnp_conv_quantization_params
0023:         quantization_params[restrict static 1]) {
0024:   const uint8x8_t va_zero_point =
0025:       vld1_dup_u8((const uint8_t*)&quantization_params->neon.input_zero_point);
0026: #ifdef __aarch64__
0027:   const int16x8_t voutput_zero_point =
0028:       vld1q_dup_s16(&quantization_params->neon.output_zero_point);
0029:   const uint8x8_t voutput_min =
0030:       vld1_dup_u8(&quantization_params->neon.output_min);
0031:   const uint8x8_t voutput_max =
0032:       vld1_dup_u8(&quantization_params->neon.output_max);
0033: #else
0034:   const float32x4_t vfmin = vdupq_n_f32(quantization_params->neon.vfmin);
0035:   const float32x4_t vfmax = vdupq_n_f32(quantization_params->neon.vfmax);
0036:   const float32x4_t vfmagic = vdupq_n_f32(quantization_params->neon.vfmagic);
0037:   const int32x4_t vimagic = vdupq_n_s32(quantization_params->neon.vimagic);
0038: #endif
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends. Key symbols: `pytorch_q8dwconv_ukernel_up8x9_per_channel__neon`, `vld1_dup_u8`, `vld1q_dup_s16`.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端。关键符号：`pytorch_q8dwconv_ukernel_up8x9_per_channel__neon`, `vld1_dup_u8`, `vld1q_dup_s16`。

### Lines 39-77 / 第 39-77 行

```c
0039: 
0040: #ifdef __aarch64__
0041:   /* Larger number of registers on AArch64 make it possible to process few
0042:    * pixels at a time */
0043:   if (input_stride == 3 * sizeof(void*)) {
0044:     for (; output_width >= 3; output_width -= 3) {
0045:       /*
0046:        * Following 15 values represent:
0047:        * -------------------------
0048:        *| 00 | 01 | 02 | 03 | 04 |
0049:        * -------------------------
0050:        *| 10 | 11 | 12 | 13 | 14 |
0051:        * -------------------------
0052:        *| 20 | 21 | 22 | 23 | 24 |
0053:        * -------------------------
0054:        *  Thus:
0055:        *  acc0 = 00 + 10 + 20 + 01 + 11 + 21 + 02 + 12 + 22
0056:        *  acc1 = 01 + 11 + 21 + 02 + 12 + 22 + 03 + 13 + 23
0057:        *  acc2 = 02 + 12 + 22 + 03 + 13 + 23 + 04 + 14 + 24
0058:        *
0059:        *  For channel wise:
0060:        *  We may have to do one less output for per perhaps? Need to look at the perf.
0061:        */
0062:       const uint8_t* i00 = input[0];
0063:       const uint8_t* i10 = input[1];
0064:       const uint8_t* i20 = input[2];
0065:       const uint8_t* i01 = input[3];
0066:       const uint8_t* i11 = input[4];
0067:       const uint8_t* i21 = input[5];
0068:       const uint8_t* i02 = input[6];
0069:       const uint8_t* i12 = input[7];
0070:       const uint8_t* i22 = input[8];
0071:       const uint8_t* i03 = input[9];
0072:       const uint8_t* i13 = input[10];
0073:       const uint8_t* i23 = input[11];
0074:       const uint8_t* i04 = input[12];
0075:       const uint8_t* i14 = input[13];
0076:       const uint8_t* i24 = input[14];
0077: 
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends; handles conditional branches and special cases; iterates over tensors, descriptors, options, or runtime state. Key symbols: no prominent local symbols.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端；处理条件分支与特殊情况；遍历张量、描述符、配置项或运行时状态。关键符号：无明显局部符号。

### Lines 78-123 / 第 78-123 行

```c
0078:       uint8_t* output0 = output;
0079:       uint8_t* output1 = output0 + channels + output_increment;
0080:       uint8_t* output2 = output1 + channels + output_increment;
0081: 
0082:       input += 9;
0083: 
0084:       size_t c = channels;
0085:       const void* w = weights;
0086:       for (; c >= 8; c -= 8) {
0087:         const uint8x8_t vkernel_zero_point =
0088:             vld1_u8(&quantization_params->neon.kernel_zero_points[channels - c]);
0089:         int32x4_t vacc0_lo = vld1q_s32(w);
0090:         w = (void*)((uintptr_t)w + sizeof(int32x4_t));
0091:         int32x4_t vacc0_hi = vld1q_s32(w);
0092:         w = (void*)((uintptr_t)w + sizeof(int32x4_t));
0093:         int32x4_t vacc1_lo = vacc0_lo;
0094:         int32x4_t vacc2_lo = vacc0_lo;
0095:         int32x4_t vacc1_hi = vacc0_hi;
0096:         int32x4_t vacc2_hi = vacc0_hi;
0097: 
0098:         const uint8x8_t vk00 = vld1_u8(w);
0099:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0100:         const uint8x8_t vi00 = vld1_u8(i00);
0101:         i00 += 8;
0102:         const uint8x8_t vi01 = vld1_u8(i01);
0103:         i01 += 8;
0104:         const uint8x8_t vi02 = vld1_u8(i02);
0105:         i02 += 8;
0106:         const int16x8_t vxk00 =
0107:             vreinterpretq_s16_u16(vsubl_u8(vk00, vkernel_zero_point));
0108:         const int16x8_t vxi00 =
0109:             vreinterpretq_s16_u16(sub_zero_point(vi00, va_zero_point));
0110:         const int16x8_t vxi01 =
0111:             vreinterpretq_s16_u16(sub_zero_point(vi01, va_zero_point));
0112:         const int16x8_t vxi02 =
0113:             vreinterpretq_s16_u16(sub_zero_point(vi02, va_zero_point));
0114:         vacc0_lo =
0115:             vmlal_s16(vacc0_lo, vget_low_s16(vxk00), vget_low_s16(vxi00));
0116:         vacc0_hi = vmlal_high_s16(vacc0_hi, vxk00, vxi00);
0117:         vacc1_lo =
0118:             vmlal_s16(vacc1_lo, vget_low_s16(vxk00), vget_low_s16(vxi01));
0119:         vacc1_hi = vmlal_high_s16(vacc1_hi, vxk00, vxi01);
0120:         vacc2_lo =
0121:             vmlal_s16(vacc2_lo, vget_low_s16(vxk00), vget_low_s16(vxi02));
0122:         vacc2_hi = vmlal_high_s16(vacc2_hi, vxk00, vxi02);
0123: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `vld1_u8`, `vreinterpretq_s16_u16`, `vmlal_s16`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`vld1_u8`, `vreinterpretq_s16_u16`, `vmlal_s16`。

### Lines 124-175 / 第 124-175 行

```c
0124:         const uint8x8_t vk10 = vld1_u8(w);
0125:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0126:         const uint8x8_t vi10 = vld1_u8(i10);
0127:         i10 += 8;
0128:         const uint8x8_t vi11 = vld1_u8(i11);
0129:         i11 += 8;
0130:         const uint8x8_t vi12 = vld1_u8(i12);
0131:         i12 += 8;
0132:         const int16x8_t vxk10 =
0133:             vreinterpretq_s16_u16(vsubl_u8(vk10, vkernel_zero_point));
0134:         const int16x8_t vxi10 =
0135:             vreinterpretq_s16_u16(sub_zero_point(vi10, va_zero_point));
0136:         const int16x8_t vxi11 =
0137:             vreinterpretq_s16_u16(sub_zero_point(vi11, va_zero_point));
0138:         const int16x8_t vxi12 =
0139:             vreinterpretq_s16_u16(sub_zero_point(vi12, va_zero_point));
0140:         vacc0_lo =
0141:             vmlal_s16(vacc0_lo, vget_low_s16(vxk10), vget_low_s16(vxi10));
0142:         vacc0_hi = vmlal_high_s16(vacc0_hi, vxk10, vxi10);
0143:         vacc1_lo =
0144:             vmlal_s16(vacc1_lo, vget_low_s16(vxk10), vget_low_s16(vxi11));
0145:         vacc1_hi = vmlal_high_s16(vacc1_hi, vxk10, vxi11);
0146:         vacc2_lo =
0147:             vmlal_s16(vacc2_lo, vget_low_s16(vxk10), vget_low_s16(vxi12));
0148:         vacc2_hi = vmlal_high_s16(vacc2_hi, vxk10, vxi12);
0149: 
0150:         const uint8x8_t vk20 = vld1_u8(w);
0151:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0152:         const uint8x8_t vi20 = vld1_u8(i20);
0153:         i20 += 8;
0154:         const uint8x8_t vi21 = vld1_u8(i21);
0155:         i21 += 8;
0156:         const uint8x8_t vi22 = vld1_u8(i22);
0157:         i22 += 8;
0158:         const int16x8_t vxk20 =
0159:             vreinterpretq_s16_u16(vsubl_u8(vk20, vkernel_zero_point));
0160:         const int16x8_t vxi20 =
0161:             vreinterpretq_s16_u16(sub_zero_point(vi20, va_zero_point));
0162:         const int16x8_t vxi21 =
0163:             vreinterpretq_s16_u16(sub_zero_point(vi21, va_zero_point));
0164:         const int16x8_t vxi22 =
0165:             vreinterpretq_s16_u16(sub_zero_point(vi22, va_zero_point));
0166:         vacc0_lo =
0167:             vmlal_s16(vacc0_lo, vget_low_s16(vxk20), vget_low_s16(vxi20));
0168:         vacc0_hi = vmlal_high_s16(vacc0_hi, vxk20, vxi20);
0169:         vacc1_lo =
0170:             vmlal_s16(vacc1_lo, vget_low_s16(vxk20), vget_low_s16(vxi21));
0171:         vacc1_hi = vmlal_high_s16(vacc1_hi, vxk20, vxi21);
0172:         vacc2_lo =
0173:             vmlal_s16(vacc2_lo, vget_low_s16(vxk20), vget_low_s16(vxi22));
0174:         vacc2_hi = vmlal_high_s16(vacc2_hi, vxk20, vxi22);
0175: 
```

- **EN:** This block implements local helper logic for `up8x9-neon-per-channel`. Key symbols: `vreinterpretq_s16_u16`, `vmlal_s16`.
- **CN:** 该代码块实现与 `up8x9-neon-per-channel` 相关的局部辅助逻辑。关键符号：`vreinterpretq_s16_u16`, `vmlal_s16`。

### Lines 176-211 / 第 176-211 行

```c
0176:         const uint8x8_t vk01 = vld1_u8(w);
0177:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0178:         const uint8x8_t vi03 = vld1_u8(i03);
0179:         i03 += 8;
0180:         const int16x8_t vxk01 =
0181:             vreinterpretq_s16_u16(vsubl_u8(vk01, vkernel_zero_point));
0182:         const int16x8_t vxi03 =
0183:             vreinterpretq_s16_u16(sub_zero_point(vi03, va_zero_point));
0184:         vacc0_lo =
0185:             vmlal_s16(vacc0_lo, vget_low_s16(vxk01), vget_low_s16(vxi01));
0186:         vacc0_hi = vmlal_high_s16(vacc0_hi, vxk01, vxi01);
0187:         vacc1_lo =
0188:             vmlal_s16(vacc1_lo, vget_low_s16(vxk01), vget_low_s16(vxi02));
0189:         vacc1_hi = vmlal_high_s16(vacc1_hi, vxk01, vxi02);
0190:         vacc2_lo =
0191:             vmlal_s16(vacc2_lo, vget_low_s16(vxk01), vget_low_s16(vxi03));
0192:         vacc2_hi = vmlal_high_s16(vacc2_hi, vxk01, vxi03);
0193: 
0194:         const uint8x8_t vk11 = vld1_u8(w);
0195:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0196:         const uint8x8_t vi13 = vld1_u8(i13);
0197:         i13 += 8;
0198:         const int16x8_t vxk11 =
0199:             vreinterpretq_s16_u16(vsubl_u8(vk11, vkernel_zero_point));
0200:         const int16x8_t vxi13 =
0201:             vreinterpretq_s16_u16(sub_zero_point(vi13, va_zero_point));
0202:         vacc0_lo =
0203:             vmlal_s16(vacc0_lo, vget_low_s16(vxk11), vget_low_s16(vxi11));
0204:         vacc0_hi = vmlal_high_s16(vacc0_hi, vxk11, vxi11);
0205:         vacc1_lo =
0206:             vmlal_s16(vacc1_lo, vget_low_s16(vxk11), vget_low_s16(vxi12));
0207:         vacc1_hi = vmlal_high_s16(vacc1_hi, vxk11, vxi12);
0208:         vacc2_lo =
0209:             vmlal_s16(vacc2_lo, vget_low_s16(vxk11), vget_low_s16(vxi13));
0210:         vacc2_hi = vmlal_high_s16(vacc2_hi, vxk11, vxi13);
0211: 
```

- **EN:** This block implements local helper logic for `up8x9-neon-per-channel`. Key symbols: `vreinterpretq_s16_u16`, `vmlal_s16`.
- **CN:** 该代码块实现与 `up8x9-neon-per-channel` 相关的局部辅助逻辑。关键符号：`vreinterpretq_s16_u16`, `vmlal_s16`。

### Lines 212-247 / 第 212-247 行

```c
0212:         const uint8x8_t vk21 = vld1_u8(w);
0213:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0214:         const uint8x8_t vi23 = vld1_u8(i23);
0215:         i23 += 8;
0216:         const int16x8_t vxk21 =
0217:             vreinterpretq_s16_u16(vsubl_u8(vk21, vkernel_zero_point));
0218:         const int16x8_t vxi23 =
0219:             vreinterpretq_s16_u16(sub_zero_point(vi23, va_zero_point));
0220:         vacc0_lo =
0221:             vmlal_s16(vacc0_lo, vget_low_s16(vxk21), vget_low_s16(vxi21));
0222:         vacc0_hi = vmlal_high_s16(vacc0_hi, vxk21, vxi21);
0223:         vacc1_lo =
0224:             vmlal_s16(vacc1_lo, vget_low_s16(vxk21), vget_low_s16(vxi22));
0225:         vacc1_hi = vmlal_high_s16(vacc1_hi, vxk21, vxi22);
0226:         vacc2_lo =
0227:             vmlal_s16(vacc2_lo, vget_low_s16(vxk21), vget_low_s16(vxi23));
0228:         vacc2_hi = vmlal_high_s16(vacc2_hi, vxk21, vxi23);
0229: 
0230:         const uint8x8_t vk02 = vld1_u8(w);
0231:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0232:         const uint8x8_t vi04 = vld1_u8(i04);
0233:         i04 += 8;
0234:         const int16x8_t vxk02 =
0235:             vreinterpretq_s16_u16(vsubl_u8(vk02, vkernel_zero_point));
0236:         const int16x8_t vxi04 =
0237:             vreinterpretq_s16_u16(sub_zero_point(vi04, va_zero_point));
0238:         vacc0_lo =
0239:             vmlal_s16(vacc0_lo, vget_low_s16(vxk02), vget_low_s16(vxi02));
0240:         vacc0_hi = vmlal_high_s16(vacc0_hi, vxk02, vxi02);
0241:         vacc1_lo =
0242:             vmlal_s16(vacc1_lo, vget_low_s16(vxk02), vget_low_s16(vxi03));
0243:         vacc1_hi = vmlal_high_s16(vacc1_hi, vxk02, vxi03);
0244:         vacc2_lo =
0245:             vmlal_s16(vacc2_lo, vget_low_s16(vxk02), vget_low_s16(vxi04));
0246:         vacc2_hi = vmlal_high_s16(vacc2_hi, vxk02, vxi04);
0247: 
```

- **EN:** This block implements local helper logic for `up8x9-neon-per-channel`. Key symbols: `vreinterpretq_s16_u16`, `vmlal_s16`.
- **CN:** 该代码块实现与 `up8x9-neon-per-channel` 相关的局部辅助逻辑。关键符号：`vreinterpretq_s16_u16`, `vmlal_s16`。

### Lines 248-283 / 第 248-283 行

```c
0248:         const uint8x8_t vk12 = vld1_u8(w);
0249:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0250:         const uint8x8_t vi14 = vld1_u8(i14);
0251:         i14 += 8;
0252:         const int16x8_t vxk12 =
0253:             vreinterpretq_s16_u16(vsubl_u8(vk12, vkernel_zero_point));
0254:         const int16x8_t vxi14 =
0255:             vreinterpretq_s16_u16(sub_zero_point(vi14, va_zero_point));
0256:         vacc0_lo =
0257:             vmlal_s16(vacc0_lo, vget_low_s16(vxk12), vget_low_s16(vxi12));
0258:         vacc0_hi = vmlal_high_s16(vacc0_hi, vxk12, vxi12);
0259:         vacc1_lo =
0260:             vmlal_s16(vacc1_lo, vget_low_s16(vxk12), vget_low_s16(vxi13));
0261:         vacc1_hi = vmlal_high_s16(vacc1_hi, vxk12, vxi13);
0262:         vacc2_lo =
0263:             vmlal_s16(vacc2_lo, vget_low_s16(vxk12), vget_low_s16(vxi14));
0264:         vacc2_hi = vmlal_high_s16(vacc2_hi, vxk12, vxi14);
0265: 
0266:         const uint8x8_t vk22 = vld1_u8(w);
0267:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0268:         const uint8x8_t vi24 = vld1_u8(i24);
0269:         i24 += 8;
0270:         const int16x8_t vxk22 =
0271:             vreinterpretq_s16_u16(vsubl_u8(vk22, vkernel_zero_point));
0272:         const int16x8_t vxi24 =
0273:             vreinterpretq_s16_u16(sub_zero_point(vi24, va_zero_point));
0274:         vacc0_lo =
0275:             vmlal_s16(vacc0_lo, vget_low_s16(vxk22), vget_low_s16(vxi22));
0276:         vacc0_hi = vmlal_high_s16(vacc0_hi, vxk22, vxi22);
0277:         vacc1_lo =
0278:             vmlal_s16(vacc1_lo, vget_low_s16(vxk22), vget_low_s16(vxi23));
0279:         vacc1_hi = vmlal_high_s16(vacc1_hi, vxk22, vxi23);
0280:         vacc2_lo =
0281:             vmlal_s16(vacc2_lo, vget_low_s16(vxk22), vget_low_s16(vxi24));
0282:         vacc2_hi = vmlal_high_s16(vacc2_hi, vxk22, vxi24);
0283: 
```

- **EN:** This block implements local helper logic for `up8x9-neon-per-channel`. Key symbols: `vreinterpretq_s16_u16`, `vmlal_s16`.
- **CN:** 该代码块实现与 `up8x9-neon-per-channel` 相关的局部辅助逻辑。关键符号：`vreinterpretq_s16_u16`, `vmlal_s16`。

### Lines 284-320 / 第 284-320 行

```c
0284:         const float32x4_t requantization_scale_v_lo =
0285:             vld1q_f32(&quantization_params->neon.requantization_scales[channels - c]);
0286:         const float32x4_t requantization_scale_v_hi =
0287:             vld1q_f32(&quantization_params->neon.requantization_scales[channels - c + 4]);
0288: 
0289:         vacc0_lo = vcvtnq_s32_f32(
0290:             vmulq_f32(vcvtq_f32_s32(vacc0_lo), requantization_scale_v_lo));
0291:         vacc0_hi = vcvtnq_s32_f32(
0292:             vmulq_f32(vcvtq_f32_s32(vacc0_hi), requantization_scale_v_hi));
0293:         vacc1_lo = vcvtnq_s32_f32(
0294:             vmulq_f32(vcvtq_f32_s32(vacc1_lo), requantization_scale_v_lo));
0295:         vacc1_hi = vcvtnq_s32_f32(
0296:             vmulq_f32(vcvtq_f32_s32(vacc1_hi), requantization_scale_v_hi));
0297:         vacc2_lo = vcvtnq_s32_f32(
0298:             vmulq_f32(vcvtq_f32_s32(vacc2_lo), requantization_scale_v_lo));
0299:         vacc2_hi = vcvtnq_s32_f32(
0300:             vmulq_f32(vcvtq_f32_s32(vacc2_hi), requantization_scale_v_hi));
0301: 
0302:         const int16x8_t vacc0 = vqaddq_s16(
0303:             vqmovn_high_s32(vqmovn_s32(vacc0_lo), vacc0_hi),
0304:             voutput_zero_point);
0305:         const int16x8_t vacc1 = vqaddq_s16(
0306:             vqmovn_high_s32(vqmovn_s32(vacc1_lo), vacc1_hi),
0307:             voutput_zero_point);
0308:         const int16x8_t vacc2 = vqaddq_s16(
0309:             vqmovn_high_s32(vqmovn_s32(vacc2_lo), vacc2_hi),
0310:             voutput_zero_point);
0311:         uint8x8_t vout0 = vqmovun_s16(vacc0);
0312:         uint8x8_t vout1 = vqmovun_s16(vacc1);
0313:         uint8x8_t vout2 = vqmovun_s16(vacc2);
0314:         vout0 = vmax_u8(vout0, voutput_min);
0315:         vout1 = vmax_u8(vout1, voutput_min);
0316:         vout2 = vmax_u8(vout2, voutput_min);
0317:         vout0 = vmin_u8(vout0, voutput_max);
0318:         vout1 = vmin_u8(vout1, voutput_max);
0319:         vout2 = vmin_u8(vout2, voutput_max);
0320: 
```

- **EN:** This block implements local helper logic for `up8x9-neon-per-channel`. Key symbols: `vld1q_f32`, `vmulq_f32`, `vqmovn_high_s32`.
- **CN:** 该代码块实现与 `up8x9-neon-per-channel` 相关的局部辅助逻辑。关键符号：`vld1q_f32`, `vmulq_f32`, `vqmovn_high_s32`。

### Lines 321-357 / 第 321-357 行

```c
0321:         vst1_u8(output0, vout0);
0322:         output0 += 8;
0323:         vst1_u8(output1, vout1);
0324:         output1 += 8;
0325:         vst1_u8(output2, vout2);
0326:         output2 += 8;
0327:       }
0328:       if (c != 0) {
0329:         const size_t c_predecrement = 8 - c;
0330:         const int64x1_t vi_shift = vmov_n_s64(-8 * c_predecrement);
0331:         i00 -= c_predecrement;
0332:         i10 -= c_predecrement;
0333:         i20 -= c_predecrement;
0334:         i01 -= c_predecrement;
0335:         i11 -= c_predecrement;
0336:         i21 -= c_predecrement;
0337:         i02 -= c_predecrement;
0338:         i12 -= c_predecrement;
0339:         i22 -= c_predecrement;
0340:         i03 -= c_predecrement;
0341:         i13 -= c_predecrement;
0342:         i23 -= c_predecrement;
0343:         i04 -= c_predecrement;
0344:         i14 -= c_predecrement;
0345:         i24 -= c_predecrement;
0346: 
0347:         const uint8x8_t vkernel_zero_point =
0348:             vld1_u8(&quantization_params->neon.kernel_zero_points[channels - c]);
0349:         int32x4_t vacc0_lo = vld1q_s32(w);
0350:         w = (void*)((uintptr_t)w + sizeof(int32x4_t));
0351:         int32x4_t vacc0_hi = vld1q_s32(w);
0352:         w = (void*)((uintptr_t)w + sizeof(int32x4_t));
0353:         int32x4_t vacc1_lo = vacc0_lo;
0354:         int32x4_t vacc2_lo = vacc0_lo;
0355:         int32x4_t vacc1_hi = vacc0_hi;
0356:         int32x4_t vacc2_hi = vacc0_hi;
0357: 
```

- **EN:** This block handles conditional branches and special cases. Key symbols: `vst1_u8`, `vld1_u8`.
- **CN:** 该代码块处理条件分支与特殊情况。关键符号：`vst1_u8`, `vld1_u8`。

### Lines 358-409 / 第 358-409 行

```c
0358:         const uint8x8_t vk00 = vld1_u8(w);
0359:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0360:         const uint8x8_t vi00 = vreinterpret_u8_u64(
0361:             vshl_u64(vreinterpret_u64_u8(vld1_u8(i00)), vi_shift));
0362:         const uint8x8_t vi01 = vreinterpret_u8_u64(
0363:             vshl_u64(vreinterpret_u64_u8(vld1_u8(i01)), vi_shift));
0364:         const uint8x8_t vi02 = vreinterpret_u8_u64(
0365:             vshl_u64(vreinterpret_u64_u8(vld1_u8(i02)), vi_shift));
0366:         const int16x8_t vxk00 =
0367:             vreinterpretq_s16_u16(vsubl_u8(vk00, vkernel_zero_point));
0368:         const int16x8_t vxi00 =
0369:             vreinterpretq_s16_u16(sub_zero_point(vi00, va_zero_point));
0370:         const int16x8_t vxi01 =
0371:             vreinterpretq_s16_u16(sub_zero_point(vi01, va_zero_point));
0372:         const int16x8_t vxi02 =
0373:             vreinterpretq_s16_u16(sub_zero_point(vi02, va_zero_point));
0374:         vacc0_lo =
0375:             vmlal_s16(vacc0_lo, vget_low_s16(vxk00), vget_low_s16(vxi00));
0376:         vacc0_hi = vmlal_high_s16(vacc0_hi, vxk00, vxi00);
0377:         vacc1_lo =
0378:             vmlal_s16(vacc1_lo, vget_low_s16(vxk00), vget_low_s16(vxi01));
0379:         vacc1_hi = vmlal_high_s16(vacc1_hi, vxk00, vxi01);
0380:         vacc2_lo =
0381:             vmlal_s16(vacc2_lo, vget_low_s16(vxk00), vget_low_s16(vxi02));
0382:         vacc2_hi = vmlal_high_s16(vacc2_hi, vxk00, vxi02);
0383: 
0384:         const uint8x8_t vk10 = vld1_u8(w);
0385:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0386:         const uint8x8_t vi10 = vreinterpret_u8_u64(
0387:             vshl_u64(vreinterpret_u64_u8(vld1_u8(i10)), vi_shift));
0388:         const uint8x8_t vi11 = vreinterpret_u8_u64(
0389:             vshl_u64(vreinterpret_u64_u8(vld1_u8(i11)), vi_shift));
0390:         const uint8x8_t vi12 = vreinterpret_u8_u64(
0391:             vshl_u64(vreinterpret_u64_u8(vld1_u8(i12)), vi_shift));
0392:         const int16x8_t vxk10 =
0393:             vreinterpretq_s16_u16(vsubl_u8(vk10, vkernel_zero_point));
0394:         const int16x8_t vxi10 =
0395:             vreinterpretq_s16_u16(sub_zero_point(vi10, va_zero_point));
0396:         const int16x8_t vxi11 =
0397:             vreinterpretq_s16_u16(sub_zero_point(vi11, va_zero_point));
0398:         const int16x8_t vxi12 =
0399:             vreinterpretq_s16_u16(sub_zero_point(vi12, va_zero_point));
0400:         vacc0_lo =
0401:             vmlal_s16(vacc0_lo, vget_low_s16(vxk10), vget_low_s16(vxi10));
0402:         vacc0_hi = vmlal_high_s16(vacc0_hi, vxk10, vxi10);
0403:         vacc1_lo =
0404:             vmlal_s16(vacc1_lo, vget_low_s16(vxk10), vget_low_s16(vxi11));
0405:         vacc1_hi = vmlal_high_s16(vacc1_hi, vxk10, vxi11);
0406:         vacc2_lo =
0407:             vmlal_s16(vacc2_lo, vget_low_s16(vxk10), vget_low_s16(vxi12));
0408:         vacc2_hi = vmlal_high_s16(vacc2_hi, vxk10, vxi12);
0409: 
```

- **EN:** This block implements local helper logic for `up8x9-neon-per-channel`. Key symbols: `vshl_u64`, `vreinterpretq_s16_u16`, `vmlal_s16`.
- **CN:** 该代码块实现与 `up8x9-neon-per-channel` 相关的局部辅助逻辑。关键符号：`vshl_u64`, `vreinterpretq_s16_u16`, `vmlal_s16`。

### Lines 410-453 / 第 410-453 行

```c
0410:         const uint8x8_t vk20 = vld1_u8(w);
0411:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0412:         const uint8x8_t vi20 = vreinterpret_u8_u64(
0413:             vshl_u64(vreinterpret_u64_u8(vld1_u8(i20)), vi_shift));
0414:         const uint8x8_t vi21 = vreinterpret_u8_u64(
0415:             vshl_u64(vreinterpret_u64_u8(vld1_u8(i21)), vi_shift));
0416:         const uint8x8_t vi22 = vreinterpret_u8_u64(
0417:             vshl_u64(vreinterpret_u64_u8(vld1_u8(i22)), vi_shift));
0418:         const int16x8_t vxk20 =
0419:             vreinterpretq_s16_u16(vsubl_u8(vk20, vkernel_zero_point));
0420:         const int16x8_t vxi20 =
0421:             vreinterpretq_s16_u16(sub_zero_point(vi20, va_zero_point));
0422:         const int16x8_t vxi21 =
0423:             vreinterpretq_s16_u16(sub_zero_point(vi21, va_zero_point));
0424:         const int16x8_t vxi22 =
0425:             vreinterpretq_s16_u16(sub_zero_point(vi22, va_zero_point));
0426:         vacc0_lo =
0427:             vmlal_s16(vacc0_lo, vget_low_s16(vxk20), vget_low_s16(vxi20));
0428:         vacc0_hi = vmlal_high_s16(vacc0_hi, vxk20, vxi20);
0429:         vacc1_lo =
0430:             vmlal_s16(vacc1_lo, vget_low_s16(vxk20), vget_low_s16(vxi21));
0431:         vacc1_hi = vmlal_high_s16(vacc1_hi, vxk20, vxi21);
0432:         vacc2_lo =
0433:             vmlal_s16(vacc2_lo, vget_low_s16(vxk20), vget_low_s16(vxi22));
0434:         vacc2_hi = vmlal_high_s16(vacc2_hi, vxk20, vxi22);
0435: 
0436:         const uint8x8_t vk01 = vld1_u8(w);
0437:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0438:         const uint8x8_t vi03 = vreinterpret_u8_u64(
0439:             vshl_u64(vreinterpret_u64_u8(vld1_u8(i03)), vi_shift));
0440:         const int16x8_t vxk01 =
0441:             vreinterpretq_s16_u16(vsubl_u8(vk01, vkernel_zero_point));
0442:         const int16x8_t vxi03 =
0443:             vreinterpretq_s16_u16(sub_zero_point(vi03, va_zero_point));
0444:         vacc0_lo =
0445:             vmlal_s16(vacc0_lo, vget_low_s16(vxk01), vget_low_s16(vxi01));
0446:         vacc0_hi = vmlal_high_s16(vacc0_hi, vxk01, vxi01);
0447:         vacc1_lo =
0448:             vmlal_s16(vacc1_lo, vget_low_s16(vxk01), vget_low_s16(vxi02));
0449:         vacc1_hi = vmlal_high_s16(vacc1_hi, vxk01, vxi02);
0450:         vacc2_lo =
0451:             vmlal_s16(vacc2_lo, vget_low_s16(vxk01), vget_low_s16(vxi03));
0452:         vacc2_hi = vmlal_high_s16(vacc2_hi, vxk01, vxi03);
0453: 
```

- **EN:** This block implements local helper logic for `up8x9-neon-per-channel`. Key symbols: `vshl_u64`, `vreinterpretq_s16_u16`, `vmlal_s16`.
- **CN:** 该代码块实现与 `up8x9-neon-per-channel` 相关的局部辅助逻辑。关键符号：`vshl_u64`, `vreinterpretq_s16_u16`, `vmlal_s16`。

### Lines 454-489 / 第 454-489 行

```c
0454:         const uint8x8_t vk11 = vld1_u8(w);
0455:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0456:         const uint8x8_t vi13 = vreinterpret_u8_u64(
0457:             vshl_u64(vreinterpret_u64_u8(vld1_u8(i13)), vi_shift));
0458:         const int16x8_t vxk11 =
0459:             vreinterpretq_s16_u16(vsubl_u8(vk11, vkernel_zero_point));
0460:         const int16x8_t vxi13 =
0461:             vreinterpretq_s16_u16(sub_zero_point(vi13, va_zero_point));
0462:         vacc0_lo =
0463:             vmlal_s16(vacc0_lo, vget_low_s16(vxk11), vget_low_s16(vxi11));
0464:         vacc0_hi = vmlal_high_s16(vacc0_hi, vxk11, vxi11);
0465:         vacc1_lo =
0466:             vmlal_s16(vacc1_lo, vget_low_s16(vxk11), vget_low_s16(vxi12));
0467:         vacc1_hi = vmlal_high_s16(vacc1_hi, vxk11, vxi12);
0468:         vacc2_lo =
0469:             vmlal_s16(vacc2_lo, vget_low_s16(vxk11), vget_low_s16(vxi13));
0470:         vacc2_hi = vmlal_high_s16(vacc2_hi, vxk11, vxi13);
0471: 
0472:         const uint8x8_t vk21 = vld1_u8(w);
0473:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0474:         const uint8x8_t vi23 = vreinterpret_u8_u64(
0475:             vshl_u64(vreinterpret_u64_u8(vld1_u8(i23)), vi_shift));
0476:         const int16x8_t vxk21 =
0477:             vreinterpretq_s16_u16(vsubl_u8(vk21, vkernel_zero_point));
0478:         const int16x8_t vxi23 =
0479:             vreinterpretq_s16_u16(sub_zero_point(vi23, va_zero_point));
0480:         vacc0_lo =
0481:             vmlal_s16(vacc0_lo, vget_low_s16(vxk21), vget_low_s16(vxi21));
0482:         vacc0_hi = vmlal_high_s16(vacc0_hi, vxk21, vxi21);
0483:         vacc1_lo =
0484:             vmlal_s16(vacc1_lo, vget_low_s16(vxk21), vget_low_s16(vxi22));
0485:         vacc1_hi = vmlal_high_s16(vacc1_hi, vxk21, vxi22);
0486:         vacc2_lo =
0487:             vmlal_s16(vacc2_lo, vget_low_s16(vxk21), vget_low_s16(vxi23));
0488:         vacc2_hi = vmlal_high_s16(vacc2_hi, vxk21, vxi23);
0489: 
```

- **EN:** This block implements local helper logic for `up8x9-neon-per-channel`. Key symbols: `vshl_u64`, `vreinterpretq_s16_u16`, `vmlal_s16`.
- **CN:** 该代码块实现与 `up8x9-neon-per-channel` 相关的局部辅助逻辑。关键符号：`vshl_u64`, `vreinterpretq_s16_u16`, `vmlal_s16`。

### Lines 490-525 / 第 490-525 行

```c
0490:         const uint8x8_t vk02 = vld1_u8(w);
0491:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0492:         const uint8x8_t vi04 = vreinterpret_u8_u64(
0493:             vshl_u64(vreinterpret_u64_u8(vld1_u8(i04)), vi_shift));
0494:         const int16x8_t vxk02 =
0495:             vreinterpretq_s16_u16(vsubl_u8(vk02, vkernel_zero_point));
0496:         const int16x8_t vxi04 =
0497:             vreinterpretq_s16_u16(sub_zero_point(vi04, va_zero_point));
0498:         vacc0_lo =
0499:             vmlal_s16(vacc0_lo, vget_low_s16(vxk02), vget_low_s16(vxi02));
0500:         vacc0_hi = vmlal_high_s16(vacc0_hi, vxk02, vxi02);
0501:         vacc1_lo =
0502:             vmlal_s16(vacc1_lo, vget_low_s16(vxk02), vget_low_s16(vxi03));
0503:         vacc1_hi = vmlal_high_s16(vacc1_hi, vxk02, vxi03);
0504:         vacc2_lo =
0505:             vmlal_s16(vacc2_lo, vget_low_s16(vxk02), vget_low_s16(vxi04));
0506:         vacc2_hi = vmlal_high_s16(vacc2_hi, vxk02, vxi04);
0507: 
0508:         const uint8x8_t vk12 = vld1_u8(w);
0509:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0510:         const uint8x8_t vi14 = vreinterpret_u8_u64(
0511:             vshl_u64(vreinterpret_u64_u8(vld1_u8(i14)), vi_shift));
0512:         const int16x8_t vxk12 =
0513:             vreinterpretq_s16_u16(vsubl_u8(vk12, vkernel_zero_point));
0514:         const int16x8_t vxi14 =
0515:             vreinterpretq_s16_u16(sub_zero_point(vi14, va_zero_point));
0516:         vacc0_lo =
0517:             vmlal_s16(vacc0_lo, vget_low_s16(vxk12), vget_low_s16(vxi12));
0518:         vacc0_hi = vmlal_high_s16(vacc0_hi, vxk12, vxi12);
0519:         vacc1_lo =
0520:             vmlal_s16(vacc1_lo, vget_low_s16(vxk12), vget_low_s16(vxi13));
0521:         vacc1_hi = vmlal_high_s16(vacc1_hi, vxk12, vxi13);
0522:         vacc2_lo =
0523:             vmlal_s16(vacc2_lo, vget_low_s16(vxk12), vget_low_s16(vxi14));
0524:         vacc2_hi = vmlal_high_s16(vacc2_hi, vxk12, vxi14);
0525: 
```

- **EN:** This block implements local helper logic for `up8x9-neon-per-channel`. Key symbols: `vshl_u64`, `vreinterpretq_s16_u16`, `vmlal_s16`.
- **CN:** 该代码块实现与 `up8x9-neon-per-channel` 相关的局部辅助逻辑。关键符号：`vshl_u64`, `vreinterpretq_s16_u16`, `vmlal_s16`。

### Lines 526-561 / 第 526-561 行

```c
0526:         const uint8x8_t vk22 = vld1_u8(w);
0527:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0528:         const uint8x8_t vi24 = vreinterpret_u8_u64(
0529:             vshl_u64(vreinterpret_u64_u8(vld1_u8(i24)), vi_shift));
0530:         const int16x8_t vxk22 =
0531:             vreinterpretq_s16_u16(vsubl_u8(vk22, vkernel_zero_point));
0532:         const int16x8_t vxi24 =
0533:             vreinterpretq_s16_u16(sub_zero_point(vi24, va_zero_point));
0534:         vacc0_lo =
0535:             vmlal_s16(vacc0_lo, vget_low_s16(vxk22), vget_low_s16(vxi22));
0536:         vacc0_hi = vmlal_high_s16(vacc0_hi, vxk22, vxi22);
0537:         vacc1_lo =
0538:             vmlal_s16(vacc1_lo, vget_low_s16(vxk22), vget_low_s16(vxi23));
0539:         vacc1_hi = vmlal_high_s16(vacc1_hi, vxk22, vxi23);
0540:         vacc2_lo =
0541:             vmlal_s16(vacc2_lo, vget_low_s16(vxk22), vget_low_s16(vxi24));
0542:         vacc2_hi = vmlal_high_s16(vacc2_hi, vxk22, vxi24);
0543: 
0544:         const float32x4_t requantization_scale_v_lo =
0545:             vld1q_f32(&quantization_params->neon.requantization_scales[channels - c]);
0546:         const float32x4_t requantization_scale_v_hi =
0547:             vld1q_f32(&quantization_params->neon.requantization_scales[channels - c + 4]);
0548: 
0549:         vacc0_lo = vcvtnq_s32_f32(
0550:             vmulq_f32(vcvtq_f32_s32(vacc0_lo), requantization_scale_v_lo));
0551:         vacc0_hi = vcvtnq_s32_f32(
0552:             vmulq_f32(vcvtq_f32_s32(vacc0_hi), requantization_scale_v_hi));
0553:         vacc1_lo = vcvtnq_s32_f32(
0554:             vmulq_f32(vcvtq_f32_s32(vacc1_lo), requantization_scale_v_lo));
0555:         vacc1_hi = vcvtnq_s32_f32(
0556:             vmulq_f32(vcvtq_f32_s32(vacc1_hi), requantization_scale_v_hi));
0557:         vacc2_lo = vcvtnq_s32_f32(
0558:             vmulq_f32(vcvtq_f32_s32(vacc2_lo), requantization_scale_v_lo));
0559:         vacc2_hi = vcvtnq_s32_f32(
0560:             vmulq_f32(vcvtq_f32_s32(vacc2_hi), requantization_scale_v_hi));
0561: 
```

- **EN:** This block implements local helper logic for `up8x9-neon-per-channel`. Key symbols: `vshl_u64`, `vreinterpretq_s16_u16`, `vmlal_s16`, `vld1q_f32`, `vmulq_f32`.
- **CN:** 该代码块实现与 `up8x9-neon-per-channel` 相关的局部辅助逻辑。关键符号：`vshl_u64`, `vreinterpretq_s16_u16`, `vmlal_s16`, `vld1q_f32`, `vmulq_f32`。

### Lines 562-601 / 第 562-601 行

```c
0562:         const int16x8_t vacc0 = vqaddq_s16(
0563:             vqmovn_high_s32(vqmovn_s32(vacc0_lo), vacc0_hi),
0564:             voutput_zero_point);
0565:         const int16x8_t vacc1 = vqaddq_s16(
0566:             vqmovn_high_s32(vqmovn_s32(vacc1_lo), vacc1_hi),
0567:             voutput_zero_point);
0568:         const int16x8_t vacc2 = vqaddq_s16(
0569:             vqmovn_high_s32(vqmovn_s32(vacc2_lo), vacc2_hi),
0570:             voutput_zero_point);
0571:         uint8x8_t vout0 = vqmovun_s16(vacc0);
0572:         uint8x8_t vout1 = vqmovun_s16(vacc1);
0573:         uint8x8_t vout2 = vqmovun_s16(vacc2);
0574:         vout0 = vmax_u8(vout0, voutput_min);
0575:         vout1 = vmax_u8(vout1, voutput_min);
0576:         vout2 = vmax_u8(vout2, voutput_min);
0577:         vout0 = vmin_u8(vout0, voutput_max);
0578:         vout1 = vmin_u8(vout1, voutput_max);
0579:         vout2 = vmin_u8(vout2, voutput_max);
0580: 
0581:         if (c & 4) {
0582:           vst1_lane_u32(
0583:               __builtin_assume_aligned(output0, 1),
0584:               vreinterpret_u32_u8(vout0),
0585:               0);
0586:           output0 += 4;
0587:           vst1_lane_u32(
0588:               __builtin_assume_aligned(output1, 1),
0589:               vreinterpret_u32_u8(vout1),
0590:               0);
0591:           output1 += 4;
0592:           vst1_lane_u32(
0593:               __builtin_assume_aligned(output2, 1),
0594:               vreinterpret_u32_u8(vout2),
0595:               0);
0596:           output2 += 4;
0597:           vout0 = vext_u8(vout0, vout0, 4);
0598:           vout1 = vext_u8(vout1, vout1, 4);
0599:           vout2 = vext_u8(vout2, vout2, 4);
0600:         }
0601:         if (c & 2) {
```

- **EN:** This block handles conditional branches and special cases. Key symbols: `vqmovn_high_s32`, `vst1_lane_u32`.
- **CN:** 该代码块处理条件分支与特殊情况。关键符号：`vqmovn_high_s32`, `vst1_lane_u32`。

### Lines 602-633 / 第 602-633 行

```c
0602:           vst1_lane_u16(
0603:               __builtin_assume_aligned(output0, 1),
0604:               vreinterpret_u16_u8(vout0),
0605:               0);
0606:           output0 += 2;
0607:           vst1_lane_u16(
0608:               __builtin_assume_aligned(output1, 1),
0609:               vreinterpret_u16_u8(vout1),
0610:               0);
0611:           output1 += 2;
0612:           vst1_lane_u16(
0613:               __builtin_assume_aligned(output2, 1),
0614:               vreinterpret_u16_u8(vout2),
0615:               0);
0616:           output2 += 2;
0617:           vout0 = vext_u8(vout0, vout0, 2);
0618:           vout1 = vext_u8(vout1, vout1, 2);
0619:           vout2 = vext_u8(vout2, vout2, 2);
0620:         }
0621:         if (c & 1) {
0622:           vst1_lane_u8(__builtin_assume_aligned(output0, 1), vout0, 0);
0623:           output0++;
0624:           vst1_lane_u8(__builtin_assume_aligned(output1, 1), vout1, 0);
0625:           output1++;
0626:           vst1_lane_u8(__builtin_assume_aligned(output2, 1), vout2, 0);
0627:           output2++;
0628:         }
0629:       }
0630: 
0631:       output = (uint8_t*)((uintptr_t)output2 + output_increment);
0632:     }
0633:     if (output_width == 0) {
```

- **EN:** This block handles conditional branches and special cases. Key symbols: `vst1_lane_u16`, `vst1_lane_u8`.
- **CN:** 该代码块处理条件分支与特殊情况。关键符号：`vst1_lane_u16`, `vst1_lane_u8`。

### Lines 634-672 / 第 634-672 行

```c
0634:       return;
0635:     }
0636:   }
0637: #endif
0638: 
0639:   do {
0640:     const uint8_t* i0 = input[0];
0641:     const uint8_t* i1 = input[1];
0642:     const uint8_t* i2 = input[2];
0643:     const uint8_t* i3 = input[3];
0644:     const uint8_t* i4 = input[4];
0645:     const uint8_t* i5 = input[5];
0646:     const uint8_t* i6 = input[6];
0647:     const uint8_t* i7 = input[7];
0648:     const uint8_t* i8 = input[8];
0649: 
0650:     input = (const uint8_t**)((uintptr_t)input + input_stride);
0651: 
0652:     size_t c = channels;
0653:     const void* w = weights;
0654:     for (; c >= 8; c -= 8) {
0655:       const uint8x8_t vkernel_zero_point =
0656:           vld1_u8(&quantization_params->neon.kernel_zero_points[channels - c]);
0657:       int32x4_t vaccX1_lo = vld1q_s32(w);
0658:       w = (void*)((uintptr_t)w + sizeof(int32x4_t));
0659:       int32x4_t vaccX1_hi = vld1q_s32(w);
0660:       w = (void*)((uintptr_t)w + sizeof(int32x4_t));
0661: 
0662:       const uint8x8_t vk0 = vld1_u8(w);
0663:       w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0664:       const uint8x8_t vi0 = vld1_u8(i0);
0665:       i0 += 8;
0666:       const int16x8_t vxk0 =
0667:           vreinterpretq_s16_u16(vsubl_u8(vk0, vkernel_zero_point));
0668:       const int16x8_t vxi0 =
0669:           vreinterpretq_s16_u16(sub_zero_point(vi0, va_zero_point));
0670:       int32x4_t vaccX0_lo = vmull_s16(vget_low_s16(vxk0), vget_low_s16(vxi0));
0671:       int32x4_t vaccX0_hi = vmull_s16(vget_high_s16(vxk0), vget_high_s16(vxi0));
0672: 
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends; iterates over tensors, descriptors, options, or runtime state. Key symbols: `vld1_u8`, `vreinterpretq_s16_u16`.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端；遍历张量、描述符、配置项或运行时状态。关键符号：`vld1_u8`, `vreinterpretq_s16_u16`。

### Lines 673-708 / 第 673-708 行

```c
0673:       const uint8x8_t vk1 = vld1_u8(w);
0674:       w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0675:       const uint8x8_t vi1 = vld1_u8(i1);
0676:       i1 += 8;
0677:       const int16x8_t vxk1 =
0678:           vreinterpretq_s16_u16(vsubl_u8(vk1, vkernel_zero_point));
0679:       const int16x8_t vxi1 =
0680:           vreinterpretq_s16_u16(sub_zero_point(vi1, va_zero_point));
0681:       vaccX1_lo = vmlal_s16(vaccX1_lo, vget_low_s16(vxk1), vget_low_s16(vxi1));
0682:       vaccX1_hi =
0683:           vmlal_s16(vaccX1_hi, vget_high_s16(vxk1), vget_high_s16(vxi1));
0684: 
0685:       const uint8x8_t vk2 = vld1_u8(w);
0686:       w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0687:       const uint8x8_t vi2 = vld1_u8(i2);
0688:       i2 += 8;
0689:       const int16x8_t vxk2 =
0690:           vreinterpretq_s16_u16(vsubl_u8(vk2, vkernel_zero_point));
0691:       const int16x8_t vxi2 =
0692:           vreinterpretq_s16_u16(sub_zero_point(vi2, va_zero_point));
0693:       vaccX0_lo = vmlal_s16(vaccX0_lo, vget_low_s16(vxk2), vget_low_s16(vxi2));
0694:       vaccX0_hi =
0695:           vmlal_s16(vaccX0_hi, vget_high_s16(vxk2), vget_high_s16(vxi2));
0696: 
0697:       const uint8x8_t vk3 = vld1_u8(w);
0698:       w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0699:       const uint8x8_t vi3 = vld1_u8(i3);
0700:       i3 += 8;
0701:       const int16x8_t vxk3 =
0702:           vreinterpretq_s16_u16(vsubl_u8(vk3, vkernel_zero_point));
0703:       const int16x8_t vxi3 =
0704:           vreinterpretq_s16_u16(sub_zero_point(vi3, va_zero_point));
0705:       vaccX1_lo = vmlal_s16(vaccX1_lo, vget_low_s16(vxk3), vget_low_s16(vxi3));
0706:       vaccX1_hi =
0707:           vmlal_s16(vaccX1_hi, vget_high_s16(vxk3), vget_high_s16(vxi3));
0708: 
```

- **EN:** This block implements local helper logic for `up8x9-neon-per-channel`. Key symbols: `vreinterpretq_s16_u16`, `vmlal_s16`.
- **CN:** 该代码块实现与 `up8x9-neon-per-channel` 相关的局部辅助逻辑。关键符号：`vreinterpretq_s16_u16`, `vmlal_s16`。

### Lines 709-744 / 第 709-744 行

```c
0709:       const uint8x8_t vk4 = vld1_u8(w);
0710:       w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0711:       const uint8x8_t vi4 = vld1_u8(i4);
0712:       i4 += 8;
0713:       const int16x8_t vxk4 =
0714:           vreinterpretq_s16_u16(vsubl_u8(vk4, vkernel_zero_point));
0715:       const int16x8_t vxi4 =
0716:           vreinterpretq_s16_u16(sub_zero_point(vi4, va_zero_point));
0717:       vaccX0_lo = vmlal_s16(vaccX0_lo, vget_low_s16(vxk4), vget_low_s16(vxi4));
0718:       vaccX0_hi =
0719:           vmlal_s16(vaccX0_hi, vget_high_s16(vxk4), vget_high_s16(vxi4));
0720: 
0721:       const uint8x8_t vk5 = vld1_u8(w);
0722:       w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0723:       const uint8x8_t vi5 = vld1_u8(i5);
0724:       i5 += 8;
0725:       const int16x8_t vxk5 =
0726:           vreinterpretq_s16_u16(vsubl_u8(vk5, vkernel_zero_point));
0727:       const int16x8_t vxi5 =
0728:           vreinterpretq_s16_u16(sub_zero_point(vi5, va_zero_point));
0729:       vaccX1_lo = vmlal_s16(vaccX1_lo, vget_low_s16(vxk5), vget_low_s16(vxi5));
0730:       vaccX1_hi =
0731:           vmlal_s16(vaccX1_hi, vget_high_s16(vxk5), vget_high_s16(vxi5));
0732: 
0733:       const uint8x8_t vk6 = vld1_u8(w);
0734:       w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0735:       const uint8x8_t vi6 = vld1_u8(i6);
0736:       i6 += 8;
0737:       const int16x8_t vxk6 =
0738:           vreinterpretq_s16_u16(vsubl_u8(vk6, vkernel_zero_point));
0739:       const int16x8_t vxi6 =
0740:           vreinterpretq_s16_u16(sub_zero_point(vi6, va_zero_point));
0741:       vaccX0_lo = vmlal_s16(vaccX0_lo, vget_low_s16(vxk6), vget_low_s16(vxi6));
0742:       vaccX0_hi =
0743:           vmlal_s16(vaccX0_hi, vget_high_s16(vxk6), vget_high_s16(vxi6));
0744: 
```

- **EN:** This block implements local helper logic for `up8x9-neon-per-channel`. Key symbols: `vreinterpretq_s16_u16`, `vmlal_s16`.
- **CN:** 该代码块实现与 `up8x9-neon-per-channel` 相关的局部辅助逻辑。关键符号：`vreinterpretq_s16_u16`, `vmlal_s16`。

### Lines 745-776 / 第 745-776 行

```c
0745:       const uint8x8_t vk7 = vld1_u8(w);
0746:       w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0747:       const uint8x8_t vi7 = vld1_u8(i7);
0748:       i7 += 8;
0749:       const int16x8_t vxk7 =
0750:           vreinterpretq_s16_u16(vsubl_u8(vk7, vkernel_zero_point));
0751:       const int16x8_t vxi7 =
0752:           vreinterpretq_s16_u16(sub_zero_point(vi7, va_zero_point));
0753:       vaccX1_lo = vmlal_s16(vaccX1_lo, vget_low_s16(vxk7), vget_low_s16(vxi7));
0754:       vaccX1_hi =
0755:           vmlal_s16(vaccX1_hi, vget_high_s16(vxk7), vget_high_s16(vxi7));
0756: 
0757:       const uint8x8_t vk8 = vld1_u8(w);
0758:       w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0759:       const uint8x8_t vi8 = vld1_u8(i8);
0760:       i8 += 8;
0761:       const int16x8_t vxk8 =
0762:           vreinterpretq_s16_u16(vsubl_u8(vk8, vkernel_zero_point));
0763:       const int16x8_t vxi8 =
0764:           vreinterpretq_s16_u16(sub_zero_point(vi8, va_zero_point));
0765:       vaccX0_lo = vmlal_s16(vaccX0_lo, vget_low_s16(vxk8), vget_low_s16(vxi8));
0766:       vaccX0_hi =
0767:           vmlal_s16(vaccX0_hi, vget_high_s16(vxk8), vget_high_s16(vxi8));
0768: 
0769:       int32x4_t vacc_lo = vaddq_s32(vaccX0_lo, vaccX1_lo);
0770:       int32x4_t vacc_hi = vaddq_s32(vaccX0_hi, vaccX1_hi);
0771: 
0772:       const float32x4_t requantization_scale_v_lo =
0773:           vld1q_f32(&quantization_params->neon.requantization_scales[channels - c]);
0774:       const float32x4_t requantization_scale_v_hi =
0775:           vld1q_f32(&quantization_params->neon.requantization_scales[channels - c + 4]);
0776: 
```

- **EN:** This block implements local helper logic for `up8x9-neon-per-channel`. Key symbols: `vreinterpretq_s16_u16`, `vmlal_s16`, `vld1q_f32`.
- **CN:** 该代码块实现与 `up8x9-neon-per-channel` 相关的局部辅助逻辑。关键符号：`vreinterpretq_s16_u16`, `vmlal_s16`, `vld1q_f32`。

### Lines 777-806 / 第 777-806 行

```c
0777:       const float32x4_t vacc_lo_f =
0778:         vmulq_f32(vcvtq_f32_s32(vacc_lo), requantization_scale_v_lo);
0779:       const float32x4_t vacc_hi_f =
0780:         vmulq_f32(vcvtq_f32_s32(vacc_hi), requantization_scale_v_hi);
0781: 
0782: #ifdef __aarch64__
0783:       vacc_lo = vcvtnq_s32_f32(vacc_lo_f);
0784:       vacc_hi = vcvtnq_s32_f32(vacc_hi_f);
0785: 
0786:       const int16x8_t vacc = vqaddq_s16(
0787:           vqmovn_high_s32(vqmovn_s32(vacc_lo), vacc_hi), voutput_zero_point);
0788: 
0789:       uint8x8_t vout = vqmovun_s16(vacc);
0790:       vout = vmax_u8(vout, voutput_min);
0791:       vout = vmin_u8(vout, voutput_max);
0792: #else
0793:       const float32x4_t vacc_lo_f_clamped =
0794:           vminq_f32(vmaxq_f32(vacc_lo_f, vfmin), vfmax);
0795:       const float32x4_t vacc_hi_f_clamped =
0796:           vminq_f32(vmaxq_f32(vacc_hi_f, vfmin), vfmax);
0797:       vacc_lo = vsubq_s32(
0798:           vreinterpretq_s32_f32(vaddq_f32(vacc_lo_f_clamped, vfmagic)), vimagic);
0799:       vacc_hi = vsubq_s32(
0800:           vreinterpretq_s32_f32(vaddq_f32(vacc_hi_f_clamped, vfmagic)), vimagic);
0801:       const int16x8_t vacc =
0802:           vcombine_s16(vqmovn_s32(vacc_lo), vqmovn_s32(vacc_hi));
0803: 
0804:       uint8x8_t vout = vqmovun_s16(vacc);
0805: #endif
0806: 
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends. Key symbols: `vmulq_f32`, `vqmovn_high_s32`, `vminq_f32`, `vreinterpretq_s32_f32`, `vcombine_s16`.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端。关键符号：`vmulq_f32`, `vqmovn_high_s32`, `vminq_f32`, `vreinterpretq_s32_f32`, `vcombine_s16`。

### Lines 807-840 / 第 807-840 行

```c
0807:       vst1_u8(output, vout);
0808:       output += 8;
0809:     }
0810:     if (c != 0) {
0811:       const size_t c_predecrement = 8 - c;
0812:       const int64x1_t vi_shift = vmov_n_s64(-8 * c_predecrement);
0813:       i0 -= c_predecrement;
0814:       i1 -= c_predecrement;
0815:       i2 -= c_predecrement;
0816:       i3 -= c_predecrement;
0817:       i4 -= c_predecrement;
0818:       i5 -= c_predecrement;
0819:       i6 -= c_predecrement;
0820:       i7 -= c_predecrement;
0821:       i8 -= c_predecrement;
0822: 
0823:       const uint8x8_t vkernel_zero_point =
0824:           vld1_u8(&quantization_params->neon.kernel_zero_points[channels - c]);
0825:       int32x4_t vaccX1_lo = vld1q_s32(w);
0826:       w = (void*)((uintptr_t)w + sizeof(int32x4_t));
0827:       int32x4_t vaccX1_hi = vld1q_s32(w);
0828:       w = (void*)((uintptr_t)w + sizeof(int32x4_t));
0829: 
0830:       const uint8x8_t vk0 = vld1_u8(w);
0831:       w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0832:       const uint8x8_t vi0 = vreinterpret_u8_u64(
0833:           vshl_u64(vreinterpret_u64_u8(vld1_u8(i0)), vi_shift));
0834:       const int16x8_t vxk0 =
0835:           vreinterpretq_s16_u16(vsubl_u8(vk0, vkernel_zero_point));
0836:       const int16x8_t vxi0 =
0837:           vreinterpretq_s16_u16(sub_zero_point(vi0, va_zero_point));
0838:       int32x4_t vaccX0_lo = vmull_s16(vget_low_s16(vxk0), vget_low_s16(vxi0));
0839:       int32x4_t vaccX0_hi = vmull_s16(vget_high_s16(vxk0), vget_high_s16(vxi0));
0840: 
```

- **EN:** This block handles conditional branches and special cases. Key symbols: `vst1_u8`, `vld1_u8`, `vshl_u64`, `vreinterpretq_s16_u16`.
- **CN:** 该代码块处理条件分支与特殊情况。关键符号：`vst1_u8`, `vld1_u8`, `vshl_u64`, `vreinterpretq_s16_u16`。

### Lines 841-876 / 第 841-876 行

```c
0841:       const uint8x8_t vk1 = vld1_u8(w);
0842:       w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0843:       const uint8x8_t vi1 = vreinterpret_u8_u64(
0844:           vshl_u64(vreinterpret_u64_u8(vld1_u8(i1)), vi_shift));
0845:       const int16x8_t vxk1 =
0846:           vreinterpretq_s16_u16(vsubl_u8(vk1, vkernel_zero_point));
0847:       const int16x8_t vxi1 =
0848:           vreinterpretq_s16_u16(sub_zero_point(vi1, va_zero_point));
0849:       vaccX1_lo = vmlal_s16(vaccX1_lo, vget_low_s16(vxk1), vget_low_s16(vxi1));
0850:       vaccX1_hi =
0851:           vmlal_s16(vaccX1_hi, vget_high_s16(vxk1), vget_high_s16(vxi1));
0852: 
0853:       const uint8x8_t vk2 = vld1_u8(w);
0854:       w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0855:       const uint8x8_t vi2 = vreinterpret_u8_u64(
0856:           vshl_u64(vreinterpret_u64_u8(vld1_u8(i2)), vi_shift));
0857:       const int16x8_t vxk2 =
0858:           vreinterpretq_s16_u16(vsubl_u8(vk2, vkernel_zero_point));
0859:       const int16x8_t vxi2 =
0860:           vreinterpretq_s16_u16(sub_zero_point(vi2, va_zero_point));
0861:       vaccX0_lo = vmlal_s16(vaccX0_lo, vget_low_s16(vxk2), vget_low_s16(vxi2));
0862:       vaccX0_hi =
0863:           vmlal_s16(vaccX0_hi, vget_high_s16(vxk2), vget_high_s16(vxi2));
0864: 
0865:       const uint8x8_t vk3 = vld1_u8(w);
0866:       w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0867:       const uint8x8_t vi3 = vreinterpret_u8_u64(
0868:           vshl_u64(vreinterpret_u64_u8(vld1_u8(i3)), vi_shift));
0869:       const int16x8_t vxk3 =
0870:           vreinterpretq_s16_u16(vsubl_u8(vk3, vkernel_zero_point));
0871:       const int16x8_t vxi3 =
0872:           vreinterpretq_s16_u16(sub_zero_point(vi3, va_zero_point));
0873:       vaccX1_lo = vmlal_s16(vaccX1_lo, vget_low_s16(vxk3), vget_low_s16(vxi3));
0874:       vaccX1_hi =
0875:           vmlal_s16(vaccX1_hi, vget_high_s16(vxk3), vget_high_s16(vxi3));
0876: 
```

- **EN:** This block implements local helper logic for `up8x9-neon-per-channel`. Key symbols: `vshl_u64`, `vreinterpretq_s16_u16`, `vmlal_s16`.
- **CN:** 该代码块实现与 `up8x9-neon-per-channel` 相关的局部辅助逻辑。关键符号：`vshl_u64`, `vreinterpretq_s16_u16`, `vmlal_s16`。

### Lines 877-912 / 第 877-912 行

```c
0877:       const uint8x8_t vk4 = vld1_u8(w);
0878:       w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0879:       const uint8x8_t vi4 = vreinterpret_u8_u64(
0880:           vshl_u64(vreinterpret_u64_u8(vld1_u8(i4)), vi_shift));
0881:       const int16x8_t vxk4 =
0882:           vreinterpretq_s16_u16(vsubl_u8(vk4, vkernel_zero_point));
0883:       const int16x8_t vxi4 =
0884:           vreinterpretq_s16_u16(sub_zero_point(vi4, va_zero_point));
0885:       vaccX0_lo = vmlal_s16(vaccX0_lo, vget_low_s16(vxk4), vget_low_s16(vxi4));
0886:       vaccX0_hi =
0887:           vmlal_s16(vaccX0_hi, vget_high_s16(vxk4), vget_high_s16(vxi4));
0888: 
0889:       const uint8x8_t vk5 = vld1_u8(w);
0890:       w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0891:       const uint8x8_t vi5 = vreinterpret_u8_u64(
0892:           vshl_u64(vreinterpret_u64_u8(vld1_u8(i5)), vi_shift));
0893:       const int16x8_t vxk5 =
0894:           vreinterpretq_s16_u16(vsubl_u8(vk5, vkernel_zero_point));
0895:       const int16x8_t vxi5 =
0896:           vreinterpretq_s16_u16(sub_zero_point(vi5, va_zero_point));
0897:       vaccX1_lo = vmlal_s16(vaccX1_lo, vget_low_s16(vxk5), vget_low_s16(vxi5));
0898:       vaccX1_hi =
0899:           vmlal_s16(vaccX1_hi, vget_high_s16(vxk5), vget_high_s16(vxi5));
0900: 
0901:       const uint8x8_t vk6 = vld1_u8(w);
0902:       w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0903:       const uint8x8_t vi6 = vreinterpret_u8_u64(
0904:           vshl_u64(vreinterpret_u64_u8(vld1_u8(i6)), vi_shift));
0905:       const int16x8_t vxk6 =
0906:           vreinterpretq_s16_u16(vsubl_u8(vk6, vkernel_zero_point));
0907:       const int16x8_t vxi6 =
0908:           vreinterpretq_s16_u16(sub_zero_point(vi6, va_zero_point));
0909:       vaccX0_lo = vmlal_s16(vaccX0_lo, vget_low_s16(vxk6), vget_low_s16(vxi6));
0910:       vaccX0_hi =
0911:           vmlal_s16(vaccX0_hi, vget_high_s16(vxk6), vget_high_s16(vxi6));
0912: 
```

- **EN:** This block implements local helper logic for `up8x9-neon-per-channel`. Key symbols: `vshl_u64`, `vreinterpretq_s16_u16`, `vmlal_s16`.
- **CN:** 该代码块实现与 `up8x9-neon-per-channel` 相关的局部辅助逻辑。关键符号：`vshl_u64`, `vreinterpretq_s16_u16`, `vmlal_s16`。

### Lines 913-943 / 第 913-943 行

```c
0913:       const uint8x8_t vk7 = vld1_u8(w);
0914:       w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0915:       const uint8x8_t vi7 = vreinterpret_u8_u64(
0916:           vshl_u64(vreinterpret_u64_u8(vld1_u8(i7)), vi_shift));
0917:       const int16x8_t vxk7 =
0918:           vreinterpretq_s16_u16(vsubl_u8(vk7, vkernel_zero_point));
0919:       const int16x8_t vxi7 =
0920:           vreinterpretq_s16_u16(sub_zero_point(vi7, va_zero_point));
0921:       vaccX1_lo = vmlal_s16(vaccX1_lo, vget_low_s16(vxk7), vget_low_s16(vxi7));
0922:       vaccX1_hi =
0923:           vmlal_s16(vaccX1_hi, vget_high_s16(vxk7), vget_high_s16(vxi7));
0924: 
0925:       const uint8x8_t vk8 = vld1_u8(w);
0926:       const uint8x8_t vi8 = vreinterpret_u8_u64(
0927:           vshl_u64(vreinterpret_u64_u8(vld1_u8(i8)), vi_shift));
0928:       const int16x8_t vxk8 =
0929:           vreinterpretq_s16_u16(vsubl_u8(vk8, vkernel_zero_point));
0930:       const int16x8_t vxi8 =
0931:           vreinterpretq_s16_u16(sub_zero_point(vi8, va_zero_point));
0932:       vaccX0_lo = vmlal_s16(vaccX0_lo, vget_low_s16(vxk8), vget_low_s16(vxi8));
0933:       vaccX0_hi =
0934:           vmlal_s16(vaccX0_hi, vget_high_s16(vxk8), vget_high_s16(vxi8));
0935: 
0936:       int32x4_t vacc_lo = vaddq_s32(vaccX0_lo, vaccX1_lo);
0937:       int32x4_t vacc_hi = vaddq_s32(vaccX0_hi, vaccX1_hi);
0938: 
0939:       const float32x4_t requantization_scale_v_lo =
0940:           vld1q_f32(&quantization_params->neon.requantization_scales[channels - c]);
0941:       const float32x4_t requantization_scale_v_hi =
0942:           vld1q_f32(&quantization_params->neon.requantization_scales[channels - c + 4]);
0943: 
```

- **EN:** This block implements local helper logic for `up8x9-neon-per-channel`. Key symbols: `vshl_u64`, `vreinterpretq_s16_u16`, `vmlal_s16`, `vld1q_f32`.
- **CN:** 该代码块实现与 `up8x9-neon-per-channel` 相关的局部辅助逻辑。关键符号：`vshl_u64`, `vreinterpretq_s16_u16`, `vmlal_s16`, `vld1q_f32`。

### Lines 944-973 / 第 944-973 行

```c
0944:       const float32x4_t vacc_lo_f =
0945:         vmulq_f32(vcvtq_f32_s32(vacc_lo), requantization_scale_v_lo);
0946:       const float32x4_t vacc_hi_f =
0947:         vmulq_f32(vcvtq_f32_s32(vacc_hi), requantization_scale_v_hi);
0948: 
0949: #ifdef __aarch64__
0950:       vacc_lo = vcvtnq_s32_f32(vacc_lo_f);
0951:       vacc_hi = vcvtnq_s32_f32(vacc_hi_f);
0952: 
0953:       const int16x8_t vacc = vqaddq_s16(
0954:           vqmovn_high_s32(vqmovn_s32(vacc_lo), vacc_hi), voutput_zero_point);
0955: 
0956:       uint8x8_t vout = vqmovun_s16(vacc);
0957:       vout = vmax_u8(vout, voutput_min);
0958:       vout = vmin_u8(vout, voutput_max);
0959: #else
0960:       const float32x4_t vacc_lo_f_clamped =
0961:           vminq_f32(vmaxq_f32(vacc_lo_f, vfmin), vfmax);
0962:       const float32x4_t vacc_hi_f_clamped =
0963:           vminq_f32(vmaxq_f32(vacc_hi_f, vfmin), vfmax);
0964:       vacc_lo = vsubq_s32(
0965:           vreinterpretq_s32_f32(vaddq_f32(vacc_lo_f_clamped, vfmagic)), vimagic);
0966:       vacc_hi = vsubq_s32(
0967:           vreinterpretq_s32_f32(vaddq_f32(vacc_hi_f_clamped, vfmagic)), vimagic);
0968:       const int16x8_t vacc =
0969:           vcombine_s16(vqmovn_s32(vacc_lo), vqmovn_s32(vacc_hi));
0970: 
0971:       uint8x8_t vout = vqmovun_s16(vacc);
0972: #endif
0973: 
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends. Key symbols: `vmulq_f32`, `vqmovn_high_s32`, `vminq_f32`, `vreinterpretq_s32_f32`, `vcombine_s16`.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端。关键符号：`vmulq_f32`, `vqmovn_high_s32`, `vminq_f32`, `vreinterpretq_s32_f32`, `vcombine_s16`。

### Lines 974-994 / 第 974-994 行

```c
0974:       if (c & 4) {
0975:         vst1_lane_u32(
0976:             __builtin_assume_aligned(output, 1), vreinterpret_u32_u8(vout), 0);
0977:         output += 4;
0978:         vout = vext_u8(vout, vout, 4);
0979:       }
0980:       if (c & 2) {
0981:         vst1_lane_u16(
0982:             __builtin_assume_aligned(output, 1), vreinterpret_u16_u8(vout), 0);
0983:         output += 2;
0984:         vout = vext_u8(vout, vout, 2);
0985:       }
0986:       if (c & 1) {
0987:         vst1_lane_u8(__builtin_assume_aligned(output, 1), vout, 0);
0988:         output++;
0989:       }
0990:     }
0991: 
0992:     output = (uint8_t*)((uintptr_t)output + output_increment);
0993:   } while (--output_width != 0);
0994: }
```

- **EN:** This block handles conditional branches and special cases; iterates over tensors, descriptors, options, or runtime state. Key symbols: `vst1_lane_u32`, `vst1_lane_u16`, `vst1_lane_u8`.
- **CN:** 该代码块处理条件分支与特殊情况；遍历张量、描述符、配置项或运行时状态。关键符号：`vst1_lane_u32`, `vst1_lane_u16`, `vst1_lane_u8`。


## Key Concepts / 关键概念
- **ATen native operator logic** — ATen 原生算子逻辑
- **Dispatch and backend routing** — 调度与后端路由
- **GPU kernel/runtime interaction** — GPU 内核/运行时交互
- **Mixed-precision/autocast logic** — 混合精度/自动混精逻辑
- **Parallel scan/reduction behavior** — 并行扫描/归约行为
- **Core symbols: pytorch_q8dwconv_ukernel_up8x9_per_channel__neon, vld1_dup_u8, vld1q_dup_s16, vld1_u8, vreinterpretq_s16_u16, vmlal_s16, vld1q_f32, vmulq_f32** — 核心符号：pytorch_q8dwconv_ukernel_up8x9_per_channel__neon、vld1_dup_u8、vld1q_dup_s16、vld1_u8、vreinterpretq_s16_u16、vmlal_s16、vld1q_f32、vmulq_f32

## Dependencies / 依赖关系
- **Internal includes / 内部头文件**: 无
- **External includes / 外部头文件**: `arm_neon.h`, `qnnpack/q8dwconv.h`, `requantization/runtime-neon.h`
- **Namespaces / 命名空间**: 无
- **Representative symbols / 代表性符号**: `pytorch_q8dwconv_ukernel_up8x9_per_channel__neon`, `vld1_dup_u8`, `vld1q_dup_s16`, `vld1_u8`, `vreinterpretq_s16_u16`, `vmlal_s16`, `vld1q_f32`, `vmulq_f32`, `vqmovn_high_s32`, `vst1_u8`, `vshl_u64`, `vst1_lane_u32`, `...`
