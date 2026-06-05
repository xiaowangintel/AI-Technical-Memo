# up8x9-neon.c — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `aten/src/ATen/native/quantized/cpu/qnnpack/src/q8dwconv/up8x9-neon.c`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements ATen native operators, checks, helper kernels, and backend dispatch behavior. This specific file centers on `up8x9-neon.c`. The file header highlights: "All rights reserved. This source code is licensed under the BSD-style license found in the LICENSE file in the root directory of this source tree.."
- **Purpose (CN)**: 实现 ATen 原生算子、运行时检查、辅助内核以及后端分发行为。 该文件具体围绕 `up8x9-neon.c` 展开。 文件头部注释也概括了其核心职责。

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
0012: #include <requantization/runtime-neon.h>
0013: 
0014: void pytorch_q8dwconv_ukernel_up8x9__neon(
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
0026:   const uint8x8_t vkernel_zero_point =
0027:       vdup_n_u8(quantization_params->neon.kernel_zero_points[0]);
0028:   const float32x4_t requantization_scale_v =
0029:       vdupq_n_f32(quantization_params->neon.requantization_scales[0]);
0030: #ifdef __aarch64__
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends. Key symbols: `pytorch_q8dwconv_ukernel_up8x9__neon`, `vld1_dup_u8`, `vdup_n_u8`, `vdupq_n_f32`.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端。关键符号：`pytorch_q8dwconv_ukernel_up8x9__neon`, `vld1_dup_u8`, `vdup_n_u8`, `vdupq_n_f32`。

### Lines 31-64 / 第 31-64 行

```c
0031:   const int16x8_t voutput_zero_point =
0032:       vld1q_dup_s16(&quantization_params->neon.output_zero_point);
0033:   const uint8x8_t voutput_min =
0034:       vld1_dup_u8(&quantization_params->neon.output_min);
0035:   const uint8x8_t voutput_max =
0036:       vld1_dup_u8(&quantization_params->neon.output_max);
0037: #else
0038:   const float32x4_t vfmin = vdupq_n_f32(quantization_params->neon.vfmin);
0039:   const float32x4_t vfmax = vdupq_n_f32(quantization_params->neon.vfmax);
0040:   const float32x4_t vfmagic = vdupq_n_f32(quantization_params->neon.vfmagic);
0041:   const int32x4_t vimagic = vdupq_n_s32(quantization_params->neon.vimagic);
0042: #endif
0043: 
0044: #ifdef __aarch64__
0045:   /* Larger number of registers on AArch64 make it possible to process few
0046:    * pixels at a time */
0047:   if (input_stride == 3 * sizeof(void*)) {
0048:     for (; output_width >= 3; output_width -= 3) {
0049:       const uint8_t* i00 = input[0];
0050:       const uint8_t* i10 = input[1];
0051:       const uint8_t* i20 = input[2];
0052:       const uint8_t* i01 = input[3];
0053:       const uint8_t* i11 = input[4];
0054:       const uint8_t* i21 = input[5];
0055:       const uint8_t* i02 = input[6];
0056:       const uint8_t* i12 = input[7];
0057:       const uint8_t* i22 = input[8];
0058:       const uint8_t* i03 = input[9];
0059:       const uint8_t* i13 = input[10];
0060:       const uint8_t* i23 = input[11];
0061:       const uint8_t* i04 = input[12];
0062:       const uint8_t* i14 = input[13];
0063:       const uint8_t* i24 = input[14];
0064: 
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends; handles conditional branches and special cases; iterates over tensors, descriptors, options, or runtime state. Key symbols: `vld1q_dup_s16`, `vld1_dup_u8`.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端；处理条件分支与特殊情况；遍历张量、描述符、配置项或运行时状态。关键符号：`vld1q_dup_s16`, `vld1_dup_u8`。

### Lines 65-108 / 第 65-108 行

```c
0065:       uint8_t* output0 = output;
0066:       uint8_t* output1 = output0 + channels + output_increment;
0067:       uint8_t* output2 = output1 + channels + output_increment;
0068: 
0069:       input += 9;
0070: 
0071:       size_t c = channels;
0072:       const void* w = weights;
0073:       for (; c >= 8; c -= 8) {
0074:         int32x4_t vacc0_lo = vld1q_s32(w);
0075:         w = (void*)((uintptr_t)w + sizeof(int32x4_t));
0076:         int32x4_t vacc0_hi = vld1q_s32(w);
0077:         w = (void*)((uintptr_t)w + sizeof(int32x4_t));
0078:         int32x4_t vacc1_lo = vacc0_lo;
0079:         int32x4_t vacc2_lo = vacc0_lo;
0080:         int32x4_t vacc1_hi = vacc0_hi;
0081:         int32x4_t vacc2_hi = vacc0_hi;
0082: 
0083:         const uint8x8_t vk00 = vld1_u8(w);
0084:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0085:         const uint8x8_t vi00 = vld1_u8(i00);
0086:         i00 += 8;
0087:         const uint8x8_t vi01 = vld1_u8(i01);
0088:         i01 += 8;
0089:         const uint8x8_t vi02 = vld1_u8(i02);
0090:         i02 += 8;
0091:         const int16x8_t vxk00 =
0092:             vreinterpretq_s16_u16(vsubl_u8(vk00, vkernel_zero_point));
0093:         const int16x8_t vxi00 =
0094:             vreinterpretq_s16_u16(sub_zero_point(vi00, va_zero_point));
0095:         const int16x8_t vxi01 =
0096:             vreinterpretq_s16_u16(sub_zero_point(vi01, va_zero_point));
0097:         const int16x8_t vxi02 =
0098:             vreinterpretq_s16_u16(sub_zero_point(vi02, va_zero_point));
0099:         vacc0_lo =
0100:             vmlal_s16(vacc0_lo, vget_low_s16(vxk00), vget_low_s16(vxi00));
0101:         vacc0_hi = vmlal_high_s16(vacc0_hi, vxk00, vxi00);
0102:         vacc1_lo =
0103:             vmlal_s16(vacc1_lo, vget_low_s16(vxk00), vget_low_s16(vxi01));
0104:         vacc1_hi = vmlal_high_s16(vacc1_hi, vxk00, vxi01);
0105:         vacc2_lo =
0106:             vmlal_s16(vacc2_lo, vget_low_s16(vxk00), vget_low_s16(vxi02));
0107:         vacc2_hi = vmlal_high_s16(vacc2_hi, vxk00, vxi02);
0108: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `vreinterpretq_s16_u16`, `vmlal_s16`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`vreinterpretq_s16_u16`, `vmlal_s16`。

### Lines 109-160 / 第 109-160 行

```c
0109:         const uint8x8_t vk10 = vld1_u8(w);
0110:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0111:         const uint8x8_t vi10 = vld1_u8(i10);
0112:         i10 += 8;
0113:         const uint8x8_t vi11 = vld1_u8(i11);
0114:         i11 += 8;
0115:         const uint8x8_t vi12 = vld1_u8(i12);
0116:         i12 += 8;
0117:         const int16x8_t vxk10 =
0118:             vreinterpretq_s16_u16(vsubl_u8(vk10, vkernel_zero_point));
0119:         const int16x8_t vxi10 =
0120:             vreinterpretq_s16_u16(sub_zero_point(vi10, va_zero_point));
0121:         const int16x8_t vxi11 =
0122:             vreinterpretq_s16_u16(sub_zero_point(vi11, va_zero_point));
0123:         const int16x8_t vxi12 =
0124:             vreinterpretq_s16_u16(sub_zero_point(vi12, va_zero_point));
0125:         vacc0_lo =
0126:             vmlal_s16(vacc0_lo, vget_low_s16(vxk10), vget_low_s16(vxi10));
0127:         vacc0_hi = vmlal_high_s16(vacc0_hi, vxk10, vxi10);
0128:         vacc1_lo =
0129:             vmlal_s16(vacc1_lo, vget_low_s16(vxk10), vget_low_s16(vxi11));
0130:         vacc1_hi = vmlal_high_s16(vacc1_hi, vxk10, vxi11);
0131:         vacc2_lo =
0132:             vmlal_s16(vacc2_lo, vget_low_s16(vxk10), vget_low_s16(vxi12));
0133:         vacc2_hi = vmlal_high_s16(vacc2_hi, vxk10, vxi12);
0134: 
0135:         const uint8x8_t vk20 = vld1_u8(w);
0136:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0137:         const uint8x8_t vi20 = vld1_u8(i20);
0138:         i20 += 8;
0139:         const uint8x8_t vi21 = vld1_u8(i21);
0140:         i21 += 8;
0141:         const uint8x8_t vi22 = vld1_u8(i22);
0142:         i22 += 8;
0143:         const int16x8_t vxk20 =
0144:             vreinterpretq_s16_u16(vsubl_u8(vk20, vkernel_zero_point));
0145:         const int16x8_t vxi20 =
0146:             vreinterpretq_s16_u16(sub_zero_point(vi20, va_zero_point));
0147:         const int16x8_t vxi21 =
0148:             vreinterpretq_s16_u16(sub_zero_point(vi21, va_zero_point));
0149:         const int16x8_t vxi22 =
0150:             vreinterpretq_s16_u16(sub_zero_point(vi22, va_zero_point));
0151:         vacc0_lo =
0152:             vmlal_s16(vacc0_lo, vget_low_s16(vxk20), vget_low_s16(vxi20));
0153:         vacc0_hi = vmlal_high_s16(vacc0_hi, vxk20, vxi20);
0154:         vacc1_lo =
0155:             vmlal_s16(vacc1_lo, vget_low_s16(vxk20), vget_low_s16(vxi21));
0156:         vacc1_hi = vmlal_high_s16(vacc1_hi, vxk20, vxi21);
0157:         vacc2_lo =
0158:             vmlal_s16(vacc2_lo, vget_low_s16(vxk20), vget_low_s16(vxi22));
0159:         vacc2_hi = vmlal_high_s16(vacc2_hi, vxk20, vxi22);
0160: 
```

- **EN:** This block implements local helper logic for `up8x9-neon`. Key symbols: `vreinterpretq_s16_u16`, `vmlal_s16`.
- **CN:** 该代码块实现与 `up8x9-neon` 相关的局部辅助逻辑。关键符号：`vreinterpretq_s16_u16`, `vmlal_s16`。

### Lines 161-196 / 第 161-196 行

```c
0161:         const uint8x8_t vk01 = vld1_u8(w);
0162:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0163:         const uint8x8_t vi03 = vld1_u8(i03);
0164:         i03 += 8;
0165:         const int16x8_t vxk01 =
0166:             vreinterpretq_s16_u16(vsubl_u8(vk01, vkernel_zero_point));
0167:         const int16x8_t vxi03 =
0168:             vreinterpretq_s16_u16(sub_zero_point(vi03, va_zero_point));
0169:         vacc0_lo =
0170:             vmlal_s16(vacc0_lo, vget_low_s16(vxk01), vget_low_s16(vxi01));
0171:         vacc0_hi = vmlal_high_s16(vacc0_hi, vxk01, vxi01);
0172:         vacc1_lo =
0173:             vmlal_s16(vacc1_lo, vget_low_s16(vxk01), vget_low_s16(vxi02));
0174:         vacc1_hi = vmlal_high_s16(vacc1_hi, vxk01, vxi02);
0175:         vacc2_lo =
0176:             vmlal_s16(vacc2_lo, vget_low_s16(vxk01), vget_low_s16(vxi03));
0177:         vacc2_hi = vmlal_high_s16(vacc2_hi, vxk01, vxi03);
0178: 
0179:         const uint8x8_t vk11 = vld1_u8(w);
0180:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0181:         const uint8x8_t vi13 = vld1_u8(i13);
0182:         i13 += 8;
0183:         const int16x8_t vxk11 =
0184:             vreinterpretq_s16_u16(vsubl_u8(vk11, vkernel_zero_point));
0185:         const int16x8_t vxi13 =
0186:             vreinterpretq_s16_u16(sub_zero_point(vi13, va_zero_point));
0187:         vacc0_lo =
0188:             vmlal_s16(vacc0_lo, vget_low_s16(vxk11), vget_low_s16(vxi11));
0189:         vacc0_hi = vmlal_high_s16(vacc0_hi, vxk11, vxi11);
0190:         vacc1_lo =
0191:             vmlal_s16(vacc1_lo, vget_low_s16(vxk11), vget_low_s16(vxi12));
0192:         vacc1_hi = vmlal_high_s16(vacc1_hi, vxk11, vxi12);
0193:         vacc2_lo =
0194:             vmlal_s16(vacc2_lo, vget_low_s16(vxk11), vget_low_s16(vxi13));
0195:         vacc2_hi = vmlal_high_s16(vacc2_hi, vxk11, vxi13);
0196: 
```

- **EN:** This block implements local helper logic for `up8x9-neon`. Key symbols: `vreinterpretq_s16_u16`, `vmlal_s16`.
- **CN:** 该代码块实现与 `up8x9-neon` 相关的局部辅助逻辑。关键符号：`vreinterpretq_s16_u16`, `vmlal_s16`。

### Lines 197-232 / 第 197-232 行

```c
0197:         const uint8x8_t vk21 = vld1_u8(w);
0198:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0199:         const uint8x8_t vi23 = vld1_u8(i23);
0200:         i23 += 8;
0201:         const int16x8_t vxk21 =
0202:             vreinterpretq_s16_u16(vsubl_u8(vk21, vkernel_zero_point));
0203:         const int16x8_t vxi23 =
0204:             vreinterpretq_s16_u16(sub_zero_point(vi23, va_zero_point));
0205:         vacc0_lo =
0206:             vmlal_s16(vacc0_lo, vget_low_s16(vxk21), vget_low_s16(vxi21));
0207:         vacc0_hi = vmlal_high_s16(vacc0_hi, vxk21, vxi21);
0208:         vacc1_lo =
0209:             vmlal_s16(vacc1_lo, vget_low_s16(vxk21), vget_low_s16(vxi22));
0210:         vacc1_hi = vmlal_high_s16(vacc1_hi, vxk21, vxi22);
0211:         vacc2_lo =
0212:             vmlal_s16(vacc2_lo, vget_low_s16(vxk21), vget_low_s16(vxi23));
0213:         vacc2_hi = vmlal_high_s16(vacc2_hi, vxk21, vxi23);
0214: 
0215:         const uint8x8_t vk02 = vld1_u8(w);
0216:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0217:         const uint8x8_t vi04 = vld1_u8(i04);
0218:         i04 += 8;
0219:         const int16x8_t vxk02 =
0220:             vreinterpretq_s16_u16(vsubl_u8(vk02, vkernel_zero_point));
0221:         const int16x8_t vxi04 =
0222:             vreinterpretq_s16_u16(sub_zero_point(vi04, va_zero_point));
0223:         vacc0_lo =
0224:             vmlal_s16(vacc0_lo, vget_low_s16(vxk02), vget_low_s16(vxi02));
0225:         vacc0_hi = vmlal_high_s16(vacc0_hi, vxk02, vxi02);
0226:         vacc1_lo =
0227:             vmlal_s16(vacc1_lo, vget_low_s16(vxk02), vget_low_s16(vxi03));
0228:         vacc1_hi = vmlal_high_s16(vacc1_hi, vxk02, vxi03);
0229:         vacc2_lo =
0230:             vmlal_s16(vacc2_lo, vget_low_s16(vxk02), vget_low_s16(vxi04));
0231:         vacc2_hi = vmlal_high_s16(vacc2_hi, vxk02, vxi04);
0232: 
```

- **EN:** This block implements local helper logic for `up8x9-neon`. Key symbols: `vreinterpretq_s16_u16`, `vmlal_s16`.
- **CN:** 该代码块实现与 `up8x9-neon` 相关的局部辅助逻辑。关键符号：`vreinterpretq_s16_u16`, `vmlal_s16`。

### Lines 233-268 / 第 233-268 行

```c
0233:         const uint8x8_t vk12 = vld1_u8(w);
0234:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0235:         const uint8x8_t vi14 = vld1_u8(i14);
0236:         i14 += 8;
0237:         const int16x8_t vxk12 =
0238:             vreinterpretq_s16_u16(vsubl_u8(vk12, vkernel_zero_point));
0239:         const int16x8_t vxi14 =
0240:             vreinterpretq_s16_u16(sub_zero_point(vi14, va_zero_point));
0241:         vacc0_lo =
0242:             vmlal_s16(vacc0_lo, vget_low_s16(vxk12), vget_low_s16(vxi12));
0243:         vacc0_hi = vmlal_high_s16(vacc0_hi, vxk12, vxi12);
0244:         vacc1_lo =
0245:             vmlal_s16(vacc1_lo, vget_low_s16(vxk12), vget_low_s16(vxi13));
0246:         vacc1_hi = vmlal_high_s16(vacc1_hi, vxk12, vxi13);
0247:         vacc2_lo =
0248:             vmlal_s16(vacc2_lo, vget_low_s16(vxk12), vget_low_s16(vxi14));
0249:         vacc2_hi = vmlal_high_s16(vacc2_hi, vxk12, vxi14);
0250: 
0251:         const uint8x8_t vk22 = vld1_u8(w);
0252:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0253:         const uint8x8_t vi24 = vld1_u8(i24);
0254:         i24 += 8;
0255:         const int16x8_t vxk22 =
0256:             vreinterpretq_s16_u16(vsubl_u8(vk22, vkernel_zero_point));
0257:         const int16x8_t vxi24 =
0258:             vreinterpretq_s16_u16(sub_zero_point(vi24, va_zero_point));
0259:         vacc0_lo =
0260:             vmlal_s16(vacc0_lo, vget_low_s16(vxk22), vget_low_s16(vxi22));
0261:         vacc0_hi = vmlal_high_s16(vacc0_hi, vxk22, vxi22);
0262:         vacc1_lo =
0263:             vmlal_s16(vacc1_lo, vget_low_s16(vxk22), vget_low_s16(vxi23));
0264:         vacc1_hi = vmlal_high_s16(vacc1_hi, vxk22, vxi23);
0265:         vacc2_lo =
0266:             vmlal_s16(vacc2_lo, vget_low_s16(vxk22), vget_low_s16(vxi24));
0267:         vacc2_hi = vmlal_high_s16(vacc2_hi, vxk22, vxi24);
0268: 
```

- **EN:** This block implements local helper logic for `up8x9-neon`. Key symbols: `vreinterpretq_s16_u16`, `vmlal_s16`.
- **CN:** 该代码块实现与 `up8x9-neon` 相关的局部辅助逻辑。关键符号：`vreinterpretq_s16_u16`, `vmlal_s16`。

### Lines 269-300 / 第 269-300 行

```c
0269:         vacc0_lo = vcvtnq_s32_f32(
0270:             vmulq_f32(vcvtq_f32_s32(vacc0_lo), requantization_scale_v));
0271:         vacc0_hi = vcvtnq_s32_f32(
0272:             vmulq_f32(vcvtq_f32_s32(vacc0_hi), requantization_scale_v));
0273:         vacc1_lo = vcvtnq_s32_f32(
0274:             vmulq_f32(vcvtq_f32_s32(vacc1_lo), requantization_scale_v));
0275:         vacc1_hi = vcvtnq_s32_f32(
0276:             vmulq_f32(vcvtq_f32_s32(vacc1_hi), requantization_scale_v));
0277:         vacc2_lo = vcvtnq_s32_f32(
0278:             vmulq_f32(vcvtq_f32_s32(vacc2_lo), requantization_scale_v));
0279:         vacc2_hi = vcvtnq_s32_f32(
0280:             vmulq_f32(vcvtq_f32_s32(vacc2_hi), requantization_scale_v));
0281: 
0282:         const int16x8_t vacc0 = vqaddq_s16(
0283:             vqmovn_high_s32(vqmovn_s32(vacc0_lo), vacc0_hi),
0284:             voutput_zero_point);
0285:         const int16x8_t vacc1 = vqaddq_s16(
0286:             vqmovn_high_s32(vqmovn_s32(vacc1_lo), vacc1_hi),
0287:             voutput_zero_point);
0288:         const int16x8_t vacc2 = vqaddq_s16(
0289:             vqmovn_high_s32(vqmovn_s32(vacc2_lo), vacc2_hi),
0290:             voutput_zero_point);
0291:         uint8x8_t vout0 = vqmovun_s16(vacc0);
0292:         uint8x8_t vout1 = vqmovun_s16(vacc1);
0293:         uint8x8_t vout2 = vqmovun_s16(vacc2);
0294:         vout0 = vmax_u8(vout0, voutput_min);
0295:         vout1 = vmax_u8(vout1, voutput_min);
0296:         vout2 = vmax_u8(vout2, voutput_min);
0297:         vout0 = vmin_u8(vout0, voutput_max);
0298:         vout1 = vmin_u8(vout1, voutput_max);
0299:         vout2 = vmin_u8(vout2, voutput_max);
0300: 
```

- **EN:** This block implements local helper logic for `up8x9-neon`. Key symbols: `vmulq_f32`, `vqmovn_high_s32`.
- **CN:** 该代码块实现与 `up8x9-neon` 相关的局部辅助逻辑。关键符号：`vmulq_f32`, `vqmovn_high_s32`。

### Lines 301-335 / 第 301-335 行

```c
0301:         vst1_u8(output0, vout0);
0302:         output0 += 8;
0303:         vst1_u8(output1, vout1);
0304:         output1 += 8;
0305:         vst1_u8(output2, vout2);
0306:         output2 += 8;
0307:       }
0308:       if (c != 0) {
0309:         const size_t c_predecrement = 8 - c;
0310:         const int64x1_t vi_shift = vmov_n_s64(-8 * c_predecrement);
0311:         i00 -= c_predecrement;
0312:         i10 -= c_predecrement;
0313:         i20 -= c_predecrement;
0314:         i01 -= c_predecrement;
0315:         i11 -= c_predecrement;
0316:         i21 -= c_predecrement;
0317:         i02 -= c_predecrement;
0318:         i12 -= c_predecrement;
0319:         i22 -= c_predecrement;
0320:         i03 -= c_predecrement;
0321:         i13 -= c_predecrement;
0322:         i23 -= c_predecrement;
0323:         i04 -= c_predecrement;
0324:         i14 -= c_predecrement;
0325:         i24 -= c_predecrement;
0326: 
0327:         int32x4_t vacc0_lo = vld1q_s32(w);
0328:         w = (void*)((uintptr_t)w + sizeof(int32x4_t));
0329:         int32x4_t vacc0_hi = vld1q_s32(w);
0330:         w = (void*)((uintptr_t)w + sizeof(int32x4_t));
0331:         int32x4_t vacc1_lo = vacc0_lo;
0332:         int32x4_t vacc2_lo = vacc0_lo;
0333:         int32x4_t vacc1_hi = vacc0_hi;
0334:         int32x4_t vacc2_hi = vacc0_hi;
0335: 
```

- **EN:** This block handles conditional branches and special cases. Key symbols: `vst1_u8`.
- **CN:** 该代码块处理条件分支与特殊情况。关键符号：`vst1_u8`。

### Lines 336-387 / 第 336-387 行

```c
0336:         const uint8x8_t vk00 = vld1_u8(w);
0337:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0338:         const uint8x8_t vi00 = vreinterpret_u8_u64(
0339:             vshl_u64(vreinterpret_u64_u8(vld1_u8(i00)), vi_shift));
0340:         const uint8x8_t vi01 = vreinterpret_u8_u64(
0341:             vshl_u64(vreinterpret_u64_u8(vld1_u8(i01)), vi_shift));
0342:         const uint8x8_t vi02 = vreinterpret_u8_u64(
0343:             vshl_u64(vreinterpret_u64_u8(vld1_u8(i02)), vi_shift));
0344:         const int16x8_t vxk00 =
0345:             vreinterpretq_s16_u16(vsubl_u8(vk00, vkernel_zero_point));
0346:         const int16x8_t vxi00 =
0347:             vreinterpretq_s16_u16(sub_zero_point(vi00, va_zero_point));
0348:         const int16x8_t vxi01 =
0349:             vreinterpretq_s16_u16(sub_zero_point(vi01, va_zero_point));
0350:         const int16x8_t vxi02 =
0351:             vreinterpretq_s16_u16(sub_zero_point(vi02, va_zero_point));
0352:         vacc0_lo =
0353:             vmlal_s16(vacc0_lo, vget_low_s16(vxk00), vget_low_s16(vxi00));
0354:         vacc0_hi = vmlal_high_s16(vacc0_hi, vxk00, vxi00);
0355:         vacc1_lo =
0356:             vmlal_s16(vacc1_lo, vget_low_s16(vxk00), vget_low_s16(vxi01));
0357:         vacc1_hi = vmlal_high_s16(vacc1_hi, vxk00, vxi01);
0358:         vacc2_lo =
0359:             vmlal_s16(vacc2_lo, vget_low_s16(vxk00), vget_low_s16(vxi02));
0360:         vacc2_hi = vmlal_high_s16(vacc2_hi, vxk00, vxi02);
0361: 
0362:         const uint8x8_t vk10 = vld1_u8(w);
0363:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0364:         const uint8x8_t vi10 = vreinterpret_u8_u64(
0365:             vshl_u64(vreinterpret_u64_u8(vld1_u8(i10)), vi_shift));
0366:         const uint8x8_t vi11 = vreinterpret_u8_u64(
0367:             vshl_u64(vreinterpret_u64_u8(vld1_u8(i11)), vi_shift));
0368:         const uint8x8_t vi12 = vreinterpret_u8_u64(
0369:             vshl_u64(vreinterpret_u64_u8(vld1_u8(i12)), vi_shift));
0370:         const int16x8_t vxk10 =
0371:             vreinterpretq_s16_u16(vsubl_u8(vk10, vkernel_zero_point));
0372:         const int16x8_t vxi10 =
0373:             vreinterpretq_s16_u16(sub_zero_point(vi10, va_zero_point));
0374:         const int16x8_t vxi11 =
0375:             vreinterpretq_s16_u16(sub_zero_point(vi11, va_zero_point));
0376:         const int16x8_t vxi12 =
0377:             vreinterpretq_s16_u16(sub_zero_point(vi12, va_zero_point));
0378:         vacc0_lo =
0379:             vmlal_s16(vacc0_lo, vget_low_s16(vxk10), vget_low_s16(vxi10));
0380:         vacc0_hi = vmlal_high_s16(vacc0_hi, vxk10, vxi10);
0381:         vacc1_lo =
0382:             vmlal_s16(vacc1_lo, vget_low_s16(vxk10), vget_low_s16(vxi11));
0383:         vacc1_hi = vmlal_high_s16(vacc1_hi, vxk10, vxi11);
0384:         vacc2_lo =
0385:             vmlal_s16(vacc2_lo, vget_low_s16(vxk10), vget_low_s16(vxi12));
0386:         vacc2_hi = vmlal_high_s16(vacc2_hi, vxk10, vxi12);
0387: 
```

- **EN:** This block implements local helper logic for `up8x9-neon`. Key symbols: `vshl_u64`, `vreinterpretq_s16_u16`, `vmlal_s16`.
- **CN:** 该代码块实现与 `up8x9-neon` 相关的局部辅助逻辑。关键符号：`vshl_u64`, `vreinterpretq_s16_u16`, `vmlal_s16`。

### Lines 388-431 / 第 388-431 行

```c
0388:         const uint8x8_t vk20 = vld1_u8(w);
0389:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0390:         const uint8x8_t vi20 = vreinterpret_u8_u64(
0391:             vshl_u64(vreinterpret_u64_u8(vld1_u8(i20)), vi_shift));
0392:         const uint8x8_t vi21 = vreinterpret_u8_u64(
0393:             vshl_u64(vreinterpret_u64_u8(vld1_u8(i21)), vi_shift));
0394:         const uint8x8_t vi22 = vreinterpret_u8_u64(
0395:             vshl_u64(vreinterpret_u64_u8(vld1_u8(i22)), vi_shift));
0396:         const int16x8_t vxk20 =
0397:             vreinterpretq_s16_u16(vsubl_u8(vk20, vkernel_zero_point));
0398:         const int16x8_t vxi20 =
0399:             vreinterpretq_s16_u16(sub_zero_point(vi20, va_zero_point));
0400:         const int16x8_t vxi21 =
0401:             vreinterpretq_s16_u16(sub_zero_point(vi21, va_zero_point));
0402:         const int16x8_t vxi22 =
0403:             vreinterpretq_s16_u16(sub_zero_point(vi22, va_zero_point));
0404:         vacc0_lo =
0405:             vmlal_s16(vacc0_lo, vget_low_s16(vxk20), vget_low_s16(vxi20));
0406:         vacc0_hi = vmlal_high_s16(vacc0_hi, vxk20, vxi20);
0407:         vacc1_lo =
0408:             vmlal_s16(vacc1_lo, vget_low_s16(vxk20), vget_low_s16(vxi21));
0409:         vacc1_hi = vmlal_high_s16(vacc1_hi, vxk20, vxi21);
0410:         vacc2_lo =
0411:             vmlal_s16(vacc2_lo, vget_low_s16(vxk20), vget_low_s16(vxi22));
0412:         vacc2_hi = vmlal_high_s16(vacc2_hi, vxk20, vxi22);
0413: 
0414:         const uint8x8_t vk01 = vld1_u8(w);
0415:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0416:         const uint8x8_t vi03 = vreinterpret_u8_u64(
0417:             vshl_u64(vreinterpret_u64_u8(vld1_u8(i03)), vi_shift));
0418:         const int16x8_t vxk01 =
0419:             vreinterpretq_s16_u16(vsubl_u8(vk01, vkernel_zero_point));
0420:         const int16x8_t vxi03 =
0421:             vreinterpretq_s16_u16(sub_zero_point(vi03, va_zero_point));
0422:         vacc0_lo =
0423:             vmlal_s16(vacc0_lo, vget_low_s16(vxk01), vget_low_s16(vxi01));
0424:         vacc0_hi = vmlal_high_s16(vacc0_hi, vxk01, vxi01);
0425:         vacc1_lo =
0426:             vmlal_s16(vacc1_lo, vget_low_s16(vxk01), vget_low_s16(vxi02));
0427:         vacc1_hi = vmlal_high_s16(vacc1_hi, vxk01, vxi02);
0428:         vacc2_lo =
0429:             vmlal_s16(vacc2_lo, vget_low_s16(vxk01), vget_low_s16(vxi03));
0430:         vacc2_hi = vmlal_high_s16(vacc2_hi, vxk01, vxi03);
0431: 
```

- **EN:** This block implements local helper logic for `up8x9-neon`. Key symbols: `vshl_u64`, `vreinterpretq_s16_u16`, `vmlal_s16`.
- **CN:** 该代码块实现与 `up8x9-neon` 相关的局部辅助逻辑。关键符号：`vshl_u64`, `vreinterpretq_s16_u16`, `vmlal_s16`。

### Lines 432-467 / 第 432-467 行

```c
0432:         const uint8x8_t vk11 = vld1_u8(w);
0433:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0434:         const uint8x8_t vi13 = vreinterpret_u8_u64(
0435:             vshl_u64(vreinterpret_u64_u8(vld1_u8(i13)), vi_shift));
0436:         const int16x8_t vxk11 =
0437:             vreinterpretq_s16_u16(vsubl_u8(vk11, vkernel_zero_point));
0438:         const int16x8_t vxi13 =
0439:             vreinterpretq_s16_u16(sub_zero_point(vi13, va_zero_point));
0440:         vacc0_lo =
0441:             vmlal_s16(vacc0_lo, vget_low_s16(vxk11), vget_low_s16(vxi11));
0442:         vacc0_hi = vmlal_high_s16(vacc0_hi, vxk11, vxi11);
0443:         vacc1_lo =
0444:             vmlal_s16(vacc1_lo, vget_low_s16(vxk11), vget_low_s16(vxi12));
0445:         vacc1_hi = vmlal_high_s16(vacc1_hi, vxk11, vxi12);
0446:         vacc2_lo =
0447:             vmlal_s16(vacc2_lo, vget_low_s16(vxk11), vget_low_s16(vxi13));
0448:         vacc2_hi = vmlal_high_s16(vacc2_hi, vxk11, vxi13);
0449: 
0450:         const uint8x8_t vk21 = vld1_u8(w);
0451:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0452:         const uint8x8_t vi23 = vreinterpret_u8_u64(
0453:             vshl_u64(vreinterpret_u64_u8(vld1_u8(i23)), vi_shift));
0454:         const int16x8_t vxk21 =
0455:             vreinterpretq_s16_u16(vsubl_u8(vk21, vkernel_zero_point));
0456:         const int16x8_t vxi23 =
0457:             vreinterpretq_s16_u16(sub_zero_point(vi23, va_zero_point));
0458:         vacc0_lo =
0459:             vmlal_s16(vacc0_lo, vget_low_s16(vxk21), vget_low_s16(vxi21));
0460:         vacc0_hi = vmlal_high_s16(vacc0_hi, vxk21, vxi21);
0461:         vacc1_lo =
0462:             vmlal_s16(vacc1_lo, vget_low_s16(vxk21), vget_low_s16(vxi22));
0463:         vacc1_hi = vmlal_high_s16(vacc1_hi, vxk21, vxi22);
0464:         vacc2_lo =
0465:             vmlal_s16(vacc2_lo, vget_low_s16(vxk21), vget_low_s16(vxi23));
0466:         vacc2_hi = vmlal_high_s16(vacc2_hi, vxk21, vxi23);
0467: 
```

- **EN:** This block implements local helper logic for `up8x9-neon`. Key symbols: `vshl_u64`, `vreinterpretq_s16_u16`, `vmlal_s16`.
- **CN:** 该代码块实现与 `up8x9-neon` 相关的局部辅助逻辑。关键符号：`vshl_u64`, `vreinterpretq_s16_u16`, `vmlal_s16`。

### Lines 468-503 / 第 468-503 行

```c
0468:         const uint8x8_t vk02 = vld1_u8(w);
0469:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0470:         const uint8x8_t vi04 = vreinterpret_u8_u64(
0471:             vshl_u64(vreinterpret_u64_u8(vld1_u8(i04)), vi_shift));
0472:         const int16x8_t vxk02 =
0473:             vreinterpretq_s16_u16(vsubl_u8(vk02, vkernel_zero_point));
0474:         const int16x8_t vxi04 =
0475:             vreinterpretq_s16_u16(sub_zero_point(vi04, va_zero_point));
0476:         vacc0_lo =
0477:             vmlal_s16(vacc0_lo, vget_low_s16(vxk02), vget_low_s16(vxi02));
0478:         vacc0_hi = vmlal_high_s16(vacc0_hi, vxk02, vxi02);
0479:         vacc1_lo =
0480:             vmlal_s16(vacc1_lo, vget_low_s16(vxk02), vget_low_s16(vxi03));
0481:         vacc1_hi = vmlal_high_s16(vacc1_hi, vxk02, vxi03);
0482:         vacc2_lo =
0483:             vmlal_s16(vacc2_lo, vget_low_s16(vxk02), vget_low_s16(vxi04));
0484:         vacc2_hi = vmlal_high_s16(vacc2_hi, vxk02, vxi04);
0485: 
0486:         const uint8x8_t vk12 = vld1_u8(w);
0487:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0488:         const uint8x8_t vi14 = vreinterpret_u8_u64(
0489:             vshl_u64(vreinterpret_u64_u8(vld1_u8(i14)), vi_shift));
0490:         const int16x8_t vxk12 =
0491:             vreinterpretq_s16_u16(vsubl_u8(vk12, vkernel_zero_point));
0492:         const int16x8_t vxi14 =
0493:             vreinterpretq_s16_u16(sub_zero_point(vi14, va_zero_point));
0494:         vacc0_lo =
0495:             vmlal_s16(vacc0_lo, vget_low_s16(vxk12), vget_low_s16(vxi12));
0496:         vacc0_hi = vmlal_high_s16(vacc0_hi, vxk12, vxi12);
0497:         vacc1_lo =
0498:             vmlal_s16(vacc1_lo, vget_low_s16(vxk12), vget_low_s16(vxi13));
0499:         vacc1_hi = vmlal_high_s16(vacc1_hi, vxk12, vxi13);
0500:         vacc2_lo =
0501:             vmlal_s16(vacc2_lo, vget_low_s16(vxk12), vget_low_s16(vxi14));
0502:         vacc2_hi = vmlal_high_s16(vacc2_hi, vxk12, vxi14);
0503: 
```

- **EN:** This block implements local helper logic for `up8x9-neon`. Key symbols: `vshl_u64`, `vreinterpretq_s16_u16`, `vmlal_s16`.
- **CN:** 该代码块实现与 `up8x9-neon` 相关的局部辅助逻辑。关键符号：`vshl_u64`, `vreinterpretq_s16_u16`, `vmlal_s16`。

### Lines 504-534 / 第 504-534 行

```c
0504:         const uint8x8_t vk22 = vld1_u8(w);
0505:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0506:         const uint8x8_t vi24 = vreinterpret_u8_u64(
0507:             vshl_u64(vreinterpret_u64_u8(vld1_u8(i24)), vi_shift));
0508:         const int16x8_t vxk22 =
0509:             vreinterpretq_s16_u16(vsubl_u8(vk22, vkernel_zero_point));
0510:         const int16x8_t vxi24 =
0511:             vreinterpretq_s16_u16(sub_zero_point(vi24, va_zero_point));
0512:         vacc0_lo =
0513:             vmlal_s16(vacc0_lo, vget_low_s16(vxk22), vget_low_s16(vxi22));
0514:         vacc0_hi = vmlal_high_s16(vacc0_hi, vxk22, vxi22);
0515:         vacc1_lo =
0516:             vmlal_s16(vacc1_lo, vget_low_s16(vxk22), vget_low_s16(vxi23));
0517:         vacc1_hi = vmlal_high_s16(vacc1_hi, vxk22, vxi23);
0518:         vacc2_lo =
0519:             vmlal_s16(vacc2_lo, vget_low_s16(vxk22), vget_low_s16(vxi24));
0520:         vacc2_hi = vmlal_high_s16(vacc2_hi, vxk22, vxi24);
0521: 
0522:         vacc0_lo = vcvtnq_s32_f32(
0523:             vmulq_f32(vcvtq_f32_s32(vacc0_lo), requantization_scale_v));
0524:         vacc0_hi = vcvtnq_s32_f32(
0525:             vmulq_f32(vcvtq_f32_s32(vacc0_hi), requantization_scale_v));
0526:         vacc1_lo = vcvtnq_s32_f32(
0527:             vmulq_f32(vcvtq_f32_s32(vacc1_lo), requantization_scale_v));
0528:         vacc1_hi = vcvtnq_s32_f32(
0529:             vmulq_f32(vcvtq_f32_s32(vacc1_hi), requantization_scale_v));
0530:         vacc2_lo = vcvtnq_s32_f32(
0531:             vmulq_f32(vcvtq_f32_s32(vacc2_lo), requantization_scale_v));
0532:         vacc2_hi = vcvtnq_s32_f32(
0533:             vmulq_f32(vcvtq_f32_s32(vacc2_hi), requantization_scale_v));
0534: 
```

- **EN:** This block implements local helper logic for `up8x9-neon`. Key symbols: `vshl_u64`, `vreinterpretq_s16_u16`, `vmlal_s16`, `vmulq_f32`.
- **CN:** 该代码块实现与 `up8x9-neon` 相关的局部辅助逻辑。关键符号：`vshl_u64`, `vreinterpretq_s16_u16`, `vmlal_s16`, `vmulq_f32`。

### Lines 535-574 / 第 535-574 行

```c
0535:         const int16x8_t vacc0 = vqaddq_s16(
0536:             vqmovn_high_s32(vqmovn_s32(vacc0_lo), vacc0_hi),
0537:             voutput_zero_point);
0538:         const int16x8_t vacc1 = vqaddq_s16(
0539:             vqmovn_high_s32(vqmovn_s32(vacc1_lo), vacc1_hi),
0540:             voutput_zero_point);
0541:         const int16x8_t vacc2 = vqaddq_s16(
0542:             vqmovn_high_s32(vqmovn_s32(vacc2_lo), vacc2_hi),
0543:             voutput_zero_point);
0544:         uint8x8_t vout0 = vqmovun_s16(vacc0);
0545:         uint8x8_t vout1 = vqmovun_s16(vacc1);
0546:         uint8x8_t vout2 = vqmovun_s16(vacc2);
0547:         vout0 = vmax_u8(vout0, voutput_min);
0548:         vout1 = vmax_u8(vout1, voutput_min);
0549:         vout2 = vmax_u8(vout2, voutput_min);
0550:         vout0 = vmin_u8(vout0, voutput_max);
0551:         vout1 = vmin_u8(vout1, voutput_max);
0552:         vout2 = vmin_u8(vout2, voutput_max);
0553: 
0554:         if (c & 4) {
0555:           vst1_lane_u32(
0556:               __builtin_assume_aligned(output0, 1),
0557:               vreinterpret_u32_u8(vout0),
0558:               0);
0559:           output0 += 4;
0560:           vst1_lane_u32(
0561:               __builtin_assume_aligned(output1, 1),
0562:               vreinterpret_u32_u8(vout1),
0563:               0);
0564:           output1 += 4;
0565:           vst1_lane_u32(
0566:               __builtin_assume_aligned(output2, 1),
0567:               vreinterpret_u32_u8(vout2),
0568:               0);
0569:           output2 += 4;
0570:           vout0 = vext_u8(vout0, vout0, 4);
0571:           vout1 = vext_u8(vout1, vout1, 4);
0572:           vout2 = vext_u8(vout2, vout2, 4);
0573:         }
0574:         if (c & 2) {
```

- **EN:** This block handles conditional branches and special cases. Key symbols: `vqmovn_high_s32`, `vst1_lane_u32`.
- **CN:** 该代码块处理条件分支与特殊情况。关键符号：`vqmovn_high_s32`, `vst1_lane_u32`。

### Lines 575-606 / 第 575-606 行

```c
0575:           vst1_lane_u16(
0576:               __builtin_assume_aligned(output0, 1),
0577:               vreinterpret_u16_u8(vout0),
0578:               0);
0579:           output0 += 2;
0580:           vst1_lane_u16(
0581:               __builtin_assume_aligned(output1, 1),
0582:               vreinterpret_u16_u8(vout1),
0583:               0);
0584:           output1 += 2;
0585:           vst1_lane_u16(
0586:               __builtin_assume_aligned(output2, 1),
0587:               vreinterpret_u16_u8(vout2),
0588:               0);
0589:           output2 += 2;
0590:           vout0 = vext_u8(vout0, vout0, 2);
0591:           vout1 = vext_u8(vout1, vout1, 2);
0592:           vout2 = vext_u8(vout2, vout2, 2);
0593:         }
0594:         if (c & 1) {
0595:           vst1_lane_u8(__builtin_assume_aligned(output0, 1), vout0, 0);
0596:           output0++;
0597:           vst1_lane_u8(__builtin_assume_aligned(output1, 1), vout1, 0);
0598:           output1++;
0599:           vst1_lane_u8(__builtin_assume_aligned(output2, 1), vout2, 0);
0600:           output2++;
0601:         }
0602:       }
0603: 
0604:       output = (uint8_t*)((uintptr_t)output2 + output_increment);
0605:     }
0606:     if (output_width == 0) {
```

- **EN:** This block handles conditional branches and special cases. Key symbols: `vst1_lane_u16`, `vst1_lane_u8`.
- **CN:** 该代码块处理条件分支与特殊情况。关键符号：`vst1_lane_u16`, `vst1_lane_u8`。

### Lines 607-643 / 第 607-643 行

```c
0607:       return;
0608:     }
0609:   }
0610: #endif
0611: 
0612:   do {
0613:     const uint8_t* i0 = input[0];
0614:     const uint8_t* i1 = input[1];
0615:     const uint8_t* i2 = input[2];
0616:     const uint8_t* i3 = input[3];
0617:     const uint8_t* i4 = input[4];
0618:     const uint8_t* i5 = input[5];
0619:     const uint8_t* i6 = input[6];
0620:     const uint8_t* i7 = input[7];
0621:     const uint8_t* i8 = input[8];
0622: 
0623:     input = (const uint8_t**)((uintptr_t)input + input_stride);
0624: 
0625:     size_t c = channels;
0626:     const void* w = weights;
0627:     for (; c >= 8; c -= 8) {
0628:       int32x4_t vaccX1_lo = vld1q_s32(w);
0629:       w = (void*)((uintptr_t)w + sizeof(int32x4_t));
0630:       int32x4_t vaccX1_hi = vld1q_s32(w);
0631:       w = (void*)((uintptr_t)w + sizeof(int32x4_t));
0632: 
0633:       const uint8x8_t vk0 = vld1_u8(w);
0634:       w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0635:       const uint8x8_t vi0 = vld1_u8(i0);
0636:       i0 += 8;
0637:       const int16x8_t vxk0 =
0638:           vreinterpretq_s16_u16(vsubl_u8(vk0, vkernel_zero_point));
0639:       const int16x8_t vxi0 =
0640:           vreinterpretq_s16_u16(sub_zero_point(vi0, va_zero_point));
0641:       int32x4_t vaccX0_lo = vmull_s16(vget_low_s16(vxk0), vget_low_s16(vxi0));
0642:       int32x4_t vaccX0_hi = vmull_s16(vget_high_s16(vxk0), vget_high_s16(vxi0));
0643: 
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends; iterates over tensors, descriptors, options, or runtime state. Key symbols: `vreinterpretq_s16_u16`.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端；遍历张量、描述符、配置项或运行时状态。关键符号：`vreinterpretq_s16_u16`。

### Lines 644-679 / 第 644-679 行

```c
0644:       const uint8x8_t vk1 = vld1_u8(w);
0645:       w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0646:       const uint8x8_t vi1 = vld1_u8(i1);
0647:       i1 += 8;
0648:       const int16x8_t vxk1 =
0649:           vreinterpretq_s16_u16(vsubl_u8(vk1, vkernel_zero_point));
0650:       const int16x8_t vxi1 =
0651:           vreinterpretq_s16_u16(sub_zero_point(vi1, va_zero_point));
0652:       vaccX1_lo = vmlal_s16(vaccX1_lo, vget_low_s16(vxk1), vget_low_s16(vxi1));
0653:       vaccX1_hi =
0654:           vmlal_s16(vaccX1_hi, vget_high_s16(vxk1), vget_high_s16(vxi1));
0655: 
0656:       const uint8x8_t vk2 = vld1_u8(w);
0657:       w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0658:       const uint8x8_t vi2 = vld1_u8(i2);
0659:       i2 += 8;
0660:       const int16x8_t vxk2 =
0661:           vreinterpretq_s16_u16(vsubl_u8(vk2, vkernel_zero_point));
0662:       const int16x8_t vxi2 =
0663:           vreinterpretq_s16_u16(sub_zero_point(vi2, va_zero_point));
0664:       vaccX0_lo = vmlal_s16(vaccX0_lo, vget_low_s16(vxk2), vget_low_s16(vxi2));
0665:       vaccX0_hi =
0666:           vmlal_s16(vaccX0_hi, vget_high_s16(vxk2), vget_high_s16(vxi2));
0667: 
0668:       const uint8x8_t vk3 = vld1_u8(w);
0669:       w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0670:       const uint8x8_t vi3 = vld1_u8(i3);
0671:       i3 += 8;
0672:       const int16x8_t vxk3 =
0673:           vreinterpretq_s16_u16(vsubl_u8(vk3, vkernel_zero_point));
0674:       const int16x8_t vxi3 =
0675:           vreinterpretq_s16_u16(sub_zero_point(vi3, va_zero_point));
0676:       vaccX1_lo = vmlal_s16(vaccX1_lo, vget_low_s16(vxk3), vget_low_s16(vxi3));
0677:       vaccX1_hi =
0678:           vmlal_s16(vaccX1_hi, vget_high_s16(vxk3), vget_high_s16(vxi3));
0679: 
```

- **EN:** This block implements local helper logic for `up8x9-neon`. Key symbols: `vreinterpretq_s16_u16`, `vmlal_s16`.
- **CN:** 该代码块实现与 `up8x9-neon` 相关的局部辅助逻辑。关键符号：`vreinterpretq_s16_u16`, `vmlal_s16`。

### Lines 680-715 / 第 680-715 行

```c
0680:       const uint8x8_t vk4 = vld1_u8(w);
0681:       w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0682:       const uint8x8_t vi4 = vld1_u8(i4);
0683:       i4 += 8;
0684:       const int16x8_t vxk4 =
0685:           vreinterpretq_s16_u16(vsubl_u8(vk4, vkernel_zero_point));
0686:       const int16x8_t vxi4 =
0687:           vreinterpretq_s16_u16(sub_zero_point(vi4, va_zero_point));
0688:       vaccX0_lo = vmlal_s16(vaccX0_lo, vget_low_s16(vxk4), vget_low_s16(vxi4));
0689:       vaccX0_hi =
0690:           vmlal_s16(vaccX0_hi, vget_high_s16(vxk4), vget_high_s16(vxi4));
0691: 
0692:       const uint8x8_t vk5 = vld1_u8(w);
0693:       w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0694:       const uint8x8_t vi5 = vld1_u8(i5);
0695:       i5 += 8;
0696:       const int16x8_t vxk5 =
0697:           vreinterpretq_s16_u16(vsubl_u8(vk5, vkernel_zero_point));
0698:       const int16x8_t vxi5 =
0699:           vreinterpretq_s16_u16(sub_zero_point(vi5, va_zero_point));
0700:       vaccX1_lo = vmlal_s16(vaccX1_lo, vget_low_s16(vxk5), vget_low_s16(vxi5));
0701:       vaccX1_hi =
0702:           vmlal_s16(vaccX1_hi, vget_high_s16(vxk5), vget_high_s16(vxi5));
0703: 
0704:       const uint8x8_t vk6 = vld1_u8(w);
0705:       w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0706:       const uint8x8_t vi6 = vld1_u8(i6);
0707:       i6 += 8;
0708:       const int16x8_t vxk6 =
0709:           vreinterpretq_s16_u16(vsubl_u8(vk6, vkernel_zero_point));
0710:       const int16x8_t vxi6 =
0711:           vreinterpretq_s16_u16(sub_zero_point(vi6, va_zero_point));
0712:       vaccX0_lo = vmlal_s16(vaccX0_lo, vget_low_s16(vxk6), vget_low_s16(vxi6));
0713:       vaccX0_hi =
0714:           vmlal_s16(vaccX0_hi, vget_high_s16(vxk6), vget_high_s16(vxi6));
0715: 
```

- **EN:** This block implements local helper logic for `up8x9-neon`. Key symbols: `vreinterpretq_s16_u16`, `vmlal_s16`.
- **CN:** 该代码块实现与 `up8x9-neon` 相关的局部辅助逻辑。关键符号：`vreinterpretq_s16_u16`, `vmlal_s16`。

### Lines 716-747 / 第 716-747 行

```c
0716:       const uint8x8_t vk7 = vld1_u8(w);
0717:       w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0718:       const uint8x8_t vi7 = vld1_u8(i7);
0719:       i7 += 8;
0720:       const int16x8_t vxk7 =
0721:           vreinterpretq_s16_u16(vsubl_u8(vk7, vkernel_zero_point));
0722:       const int16x8_t vxi7 =
0723:           vreinterpretq_s16_u16(sub_zero_point(vi7, va_zero_point));
0724:       vaccX1_lo = vmlal_s16(vaccX1_lo, vget_low_s16(vxk7), vget_low_s16(vxi7));
0725:       vaccX1_hi =
0726:           vmlal_s16(vaccX1_hi, vget_high_s16(vxk7), vget_high_s16(vxi7));
0727: 
0728:       const uint8x8_t vk8 = vld1_u8(w);
0729:       w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0730:       const uint8x8_t vi8 = vld1_u8(i8);
0731:       i8 += 8;
0732:       const int16x8_t vxk8 =
0733:           vreinterpretq_s16_u16(vsubl_u8(vk8, vkernel_zero_point));
0734:       const int16x8_t vxi8 =
0735:           vreinterpretq_s16_u16(sub_zero_point(vi8, va_zero_point));
0736:       vaccX0_lo = vmlal_s16(vaccX0_lo, vget_low_s16(vxk8), vget_low_s16(vxi8));
0737:       vaccX0_hi =
0738:           vmlal_s16(vaccX0_hi, vget_high_s16(vxk8), vget_high_s16(vxi8));
0739: 
0740:       int32x4_t vacc_lo = vaddq_s32(vaccX0_lo, vaccX1_lo);
0741:       int32x4_t vacc_hi = vaddq_s32(vaccX0_hi, vaccX1_hi);
0742: 
0743:       const float32x4_t vacc_lo_f =
0744:         vmulq_f32(vcvtq_f32_s32(vacc_lo), requantization_scale_v);
0745:       const float32x4_t vacc_hi_f =
0746:         vmulq_f32(vcvtq_f32_s32(vacc_hi), requantization_scale_v);
0747: 
```

- **EN:** This block implements local helper logic for `up8x9-neon`. Key symbols: `vreinterpretq_s16_u16`, `vmlal_s16`, `vmulq_f32`.
- **CN:** 该代码块实现与 `up8x9-neon` 相关的局部辅助逻辑。关键符号：`vreinterpretq_s16_u16`, `vmlal_s16`, `vmulq_f32`。

### Lines 748-788 / 第 748-788 行

```c
0748: #ifdef __aarch64__
0749:       vacc_lo = vcvtnq_s32_f32(vacc_lo_f);
0750:       vacc_hi = vcvtnq_s32_f32(vacc_hi_f);
0751: 
0752:       const int16x8_t vacc = vqaddq_s16(
0753:           vqmovn_high_s32(vqmovn_s32(vacc_lo), vacc_hi), voutput_zero_point);
0754: 
0755:       uint8x8_t vout = vqmovun_s16(vacc);
0756:       vout = vmax_u8(vout, voutput_min);
0757:       vout = vmin_u8(vout, voutput_max);
0758: #else
0759:       const float32x4_t vacc_lo_f_clamped =
0760:           vminq_f32(vmaxq_f32(vacc_lo_f, vfmin), vfmax);
0761:       const float32x4_t vacc_hi_f_clamped =
0762:           vminq_f32(vmaxq_f32(vacc_hi_f, vfmin), vfmax);
0763:       vacc_lo = vsubq_s32(
0764:           vreinterpretq_s32_f32(vaddq_f32(vacc_lo_f_clamped, vfmagic)), vimagic);
0765:       vacc_hi = vsubq_s32(
0766:           vreinterpretq_s32_f32(vaddq_f32(vacc_hi_f_clamped, vfmagic)), vimagic);
0767:       const int16x8_t vacc =
0768:           vcombine_s16(vqmovn_s32(vacc_lo), vqmovn_s32(vacc_hi));
0769: 
0770:       uint8x8_t vout = vqmovun_s16(vacc);
0771: #endif
0772: 
0773:       vst1_u8(output, vout);
0774:       output += 8;
0775:     }
0776:     if (c != 0) {
0777:       const size_t c_predecrement = 8 - c;
0778:       const int64x1_t vi_shift = vmov_n_s64(-8 * c_predecrement);
0779:       i0 -= c_predecrement;
0780:       i1 -= c_predecrement;
0781:       i2 -= c_predecrement;
0782:       i3 -= c_predecrement;
0783:       i4 -= c_predecrement;
0784:       i5 -= c_predecrement;
0785:       i6 -= c_predecrement;
0786:       i7 -= c_predecrement;
0787:       i8 -= c_predecrement;
0788: 
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends; handles conditional branches and special cases. Key symbols: `vqmovn_high_s32`, `vminq_f32`, `vreinterpretq_s32_f32`, `vcombine_s16`, `vst1_u8`.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端；处理条件分支与特殊情况。关键符号：`vqmovn_high_s32`, `vminq_f32`, `vreinterpretq_s32_f32`, `vcombine_s16`, `vst1_u8`。

### Lines 789-828 / 第 789-828 行

```c
0789:       int32x4_t vaccX1_lo = vld1q_s32(w);
0790:       w = (void*)((uintptr_t)w + sizeof(int32x4_t));
0791:       int32x4_t vaccX1_hi = vld1q_s32(w);
0792:       w = (void*)((uintptr_t)w + sizeof(int32x4_t));
0793: 
0794:       const uint8x8_t vk0 = vld1_u8(w);
0795:       w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0796:       const uint8x8_t vi0 = vreinterpret_u8_u64(
0797:           vshl_u64(vreinterpret_u64_u8(vld1_u8(i0)), vi_shift));
0798:       const int16x8_t vxk0 =
0799:           vreinterpretq_s16_u16(vsubl_u8(vk0, vkernel_zero_point));
0800:       const int16x8_t vxi0 =
0801:           vreinterpretq_s16_u16(sub_zero_point(vi0, va_zero_point));
0802:       int32x4_t vaccX0_lo = vmull_s16(vget_low_s16(vxk0), vget_low_s16(vxi0));
0803:       int32x4_t vaccX0_hi = vmull_s16(vget_high_s16(vxk0), vget_high_s16(vxi0));
0804: 
0805:       const uint8x8_t vk1 = vld1_u8(w);
0806:       w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0807:       const uint8x8_t vi1 = vreinterpret_u8_u64(
0808:           vshl_u64(vreinterpret_u64_u8(vld1_u8(i1)), vi_shift));
0809:       const int16x8_t vxk1 =
0810:           vreinterpretq_s16_u16(vsubl_u8(vk1, vkernel_zero_point));
0811:       const int16x8_t vxi1 =
0812:           vreinterpretq_s16_u16(sub_zero_point(vi1, va_zero_point));
0813:       vaccX1_lo = vmlal_s16(vaccX1_lo, vget_low_s16(vxk1), vget_low_s16(vxi1));
0814:       vaccX1_hi =
0815:           vmlal_s16(vaccX1_hi, vget_high_s16(vxk1), vget_high_s16(vxi1));
0816: 
0817:       const uint8x8_t vk2 = vld1_u8(w);
0818:       w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0819:       const uint8x8_t vi2 = vreinterpret_u8_u64(
0820:           vshl_u64(vreinterpret_u64_u8(vld1_u8(i2)), vi_shift));
0821:       const int16x8_t vxk2 =
0822:           vreinterpretq_s16_u16(vsubl_u8(vk2, vkernel_zero_point));
0823:       const int16x8_t vxi2 =
0824:           vreinterpretq_s16_u16(sub_zero_point(vi2, va_zero_point));
0825:       vaccX0_lo = vmlal_s16(vaccX0_lo, vget_low_s16(vxk2), vget_low_s16(vxi2));
0826:       vaccX0_hi =
0827:           vmlal_s16(vaccX0_hi, vget_high_s16(vxk2), vget_high_s16(vxi2));
0828: 
```

- **EN:** This block implements local helper logic for `up8x9-neon`. Key symbols: `vshl_u64`, `vreinterpretq_s16_u16`, `vmlal_s16`.
- **CN:** 该代码块实现与 `up8x9-neon` 相关的局部辅助逻辑。关键符号：`vshl_u64`, `vreinterpretq_s16_u16`, `vmlal_s16`。

### Lines 829-864 / 第 829-864 行

```c
0829:       const uint8x8_t vk3 = vld1_u8(w);
0830:       w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0831:       const uint8x8_t vi3 = vreinterpret_u8_u64(
0832:           vshl_u64(vreinterpret_u64_u8(vld1_u8(i3)), vi_shift));
0833:       const int16x8_t vxk3 =
0834:           vreinterpretq_s16_u16(vsubl_u8(vk3, vkernel_zero_point));
0835:       const int16x8_t vxi3 =
0836:           vreinterpretq_s16_u16(sub_zero_point(vi3, va_zero_point));
0837:       vaccX1_lo = vmlal_s16(vaccX1_lo, vget_low_s16(vxk3), vget_low_s16(vxi3));
0838:       vaccX1_hi =
0839:           vmlal_s16(vaccX1_hi, vget_high_s16(vxk3), vget_high_s16(vxi3));
0840: 
0841:       const uint8x8_t vk4 = vld1_u8(w);
0842:       w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0843:       const uint8x8_t vi4 = vreinterpret_u8_u64(
0844:           vshl_u64(vreinterpret_u64_u8(vld1_u8(i4)), vi_shift));
0845:       const int16x8_t vxk4 =
0846:           vreinterpretq_s16_u16(vsubl_u8(vk4, vkernel_zero_point));
0847:       const int16x8_t vxi4 =
0848:           vreinterpretq_s16_u16(sub_zero_point(vi4, va_zero_point));
0849:       vaccX0_lo = vmlal_s16(vaccX0_lo, vget_low_s16(vxk4), vget_low_s16(vxi4));
0850:       vaccX0_hi =
0851:           vmlal_s16(vaccX0_hi, vget_high_s16(vxk4), vget_high_s16(vxi4));
0852: 
0853:       const uint8x8_t vk5 = vld1_u8(w);
0854:       w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0855:       const uint8x8_t vi5 = vreinterpret_u8_u64(
0856:           vshl_u64(vreinterpret_u64_u8(vld1_u8(i5)), vi_shift));
0857:       const int16x8_t vxk5 =
0858:           vreinterpretq_s16_u16(vsubl_u8(vk5, vkernel_zero_point));
0859:       const int16x8_t vxi5 =
0860:           vreinterpretq_s16_u16(sub_zero_point(vi5, va_zero_point));
0861:       vaccX1_lo = vmlal_s16(vaccX1_lo, vget_low_s16(vxk5), vget_low_s16(vxi5));
0862:       vaccX1_hi =
0863:           vmlal_s16(vaccX1_hi, vget_high_s16(vxk5), vget_high_s16(vxi5));
0864: 
```

- **EN:** This block implements local helper logic for `up8x9-neon`. Key symbols: `vshl_u64`, `vreinterpretq_s16_u16`, `vmlal_s16`.
- **CN:** 该代码块实现与 `up8x9-neon` 相关的局部辅助逻辑。关键符号：`vshl_u64`, `vreinterpretq_s16_u16`, `vmlal_s16`。

### Lines 865-899 / 第 865-899 行

```c
0865:       const uint8x8_t vk6 = vld1_u8(w);
0866:       w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0867:       const uint8x8_t vi6 = vreinterpret_u8_u64(
0868:           vshl_u64(vreinterpret_u64_u8(vld1_u8(i6)), vi_shift));
0869:       const int16x8_t vxk6 =
0870:           vreinterpretq_s16_u16(vsubl_u8(vk6, vkernel_zero_point));
0871:       const int16x8_t vxi6 =
0872:           vreinterpretq_s16_u16(sub_zero_point(vi6, va_zero_point));
0873:       vaccX0_lo = vmlal_s16(vaccX0_lo, vget_low_s16(vxk6), vget_low_s16(vxi6));
0874:       vaccX0_hi =
0875:           vmlal_s16(vaccX0_hi, vget_high_s16(vxk6), vget_high_s16(vxi6));
0876: 
0877:       const uint8x8_t vk7 = vld1_u8(w);
0878:       w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0879:       const uint8x8_t vi7 = vreinterpret_u8_u64(
0880:           vshl_u64(vreinterpret_u64_u8(vld1_u8(i7)), vi_shift));
0881:       const int16x8_t vxk7 =
0882:           vreinterpretq_s16_u16(vsubl_u8(vk7, vkernel_zero_point));
0883:       const int16x8_t vxi7 =
0884:           vreinterpretq_s16_u16(sub_zero_point(vi7, va_zero_point));
0885:       vaccX1_lo = vmlal_s16(vaccX1_lo, vget_low_s16(vxk7), vget_low_s16(vxi7));
0886:       vaccX1_hi =
0887:           vmlal_s16(vaccX1_hi, vget_high_s16(vxk7), vget_high_s16(vxi7));
0888: 
0889:       const uint8x8_t vk8 = vld1_u8(w);
0890:       const uint8x8_t vi8 = vreinterpret_u8_u64(
0891:           vshl_u64(vreinterpret_u64_u8(vld1_u8(i8)), vi_shift));
0892:       const int16x8_t vxk8 =
0893:           vreinterpretq_s16_u16(vsubl_u8(vk8, vkernel_zero_point));
0894:       const int16x8_t vxi8 =
0895:           vreinterpretq_s16_u16(sub_zero_point(vi8, va_zero_point));
0896:       vaccX0_lo = vmlal_s16(vaccX0_lo, vget_low_s16(vxk8), vget_low_s16(vxi8));
0897:       vaccX0_hi =
0898:           vmlal_s16(vaccX0_hi, vget_high_s16(vxk8), vget_high_s16(vxi8));
0899: 
```

- **EN:** This block implements local helper logic for `up8x9-neon`. Key symbols: `vshl_u64`, `vreinterpretq_s16_u16`, `vmlal_s16`.
- **CN:** 该代码块实现与 `up8x9-neon` 相关的局部辅助逻辑。关键符号：`vshl_u64`, `vreinterpretq_s16_u16`, `vmlal_s16`。

### Lines 900-929 / 第 900-929 行

```c
0900:       int32x4_t vacc_lo = vaddq_s32(vaccX0_lo, vaccX1_lo);
0901:       int32x4_t vacc_hi = vaddq_s32(vaccX0_hi, vaccX1_hi);
0902: 
0903:       const float32x4_t vacc_lo_f =
0904:         vmulq_f32(vcvtq_f32_s32(vacc_lo), requantization_scale_v);
0905:       const float32x4_t vacc_hi_f =
0906:         vmulq_f32(vcvtq_f32_s32(vacc_hi), requantization_scale_v);
0907: 
0908: #ifdef __aarch64__
0909:       vacc_lo = vcvtnq_s32_f32(vacc_lo_f);
0910:       vacc_hi = vcvtnq_s32_f32(vacc_hi_f);
0911: 
0912:       const int16x8_t vacc = vqaddq_s16(
0913:           vqmovn_high_s32(vqmovn_s32(vacc_lo), vacc_hi), voutput_zero_point);
0914: 
0915:       uint8x8_t vout = vqmovun_s16(vacc);
0916:       vout = vmax_u8(vout, voutput_min);
0917:       vout = vmin_u8(vout, voutput_max);
0918: #else
0919:       const float32x4_t vacc_lo_f_clamped =
0920:           vminq_f32(vmaxq_f32(vacc_lo_f, vfmin), vfmax);
0921:       const float32x4_t vacc_hi_f_clamped =
0922:           vminq_f32(vmaxq_f32(vacc_hi_f, vfmin), vfmax);
0923:       vacc_lo = vsubq_s32(
0924:           vreinterpretq_s32_f32(vaddq_f32(vacc_lo_f_clamped, vfmagic)), vimagic);
0925:       vacc_hi = vsubq_s32(
0926:           vreinterpretq_s32_f32(vaddq_f32(vacc_hi_f_clamped, vfmagic)), vimagic);
0927:       const int16x8_t vacc =
0928:           vcombine_s16(vqmovn_s32(vacc_lo), vqmovn_s32(vacc_hi));
0929: 
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends. Key symbols: `vmulq_f32`, `vqmovn_high_s32`, `vminq_f32`, `vreinterpretq_s32_f32`, `vcombine_s16`.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端。关键符号：`vmulq_f32`, `vqmovn_high_s32`, `vminq_f32`, `vreinterpretq_s32_f32`, `vcombine_s16`。

### Lines 930-953 / 第 930-953 行

```c
0930:       uint8x8_t vout = vqmovun_s16(vacc);
0931: #endif
0932: 
0933:       if (c & 4) {
0934:         vst1_lane_u32(
0935:             __builtin_assume_aligned(output, 1), vreinterpret_u32_u8(vout), 0);
0936:         output += 4;
0937:         vout = vext_u8(vout, vout, 4);
0938:       }
0939:       if (c & 2) {
0940:         vst1_lane_u16(
0941:             __builtin_assume_aligned(output, 1), vreinterpret_u16_u8(vout), 0);
0942:         output += 2;
0943:         vout = vext_u8(vout, vout, 2);
0944:       }
0945:       if (c & 1) {
0946:         vst1_lane_u8(__builtin_assume_aligned(output, 1), vout, 0);
0947:         output++;
0948:       }
0949:     }
0950: 
0951:     output = (uint8_t*)((uintptr_t)output + output_increment);
0952:   } while (--output_width != 0);
0953: }
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends; handles conditional branches and special cases; iterates over tensors, descriptors, options, or runtime state. Key symbols: `vst1_lane_u32`, `vst1_lane_u16`, `vst1_lane_u8`.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端；处理条件分支与特殊情况；遍历张量、描述符、配置项或运行时状态。关键符号：`vst1_lane_u32`, `vst1_lane_u16`, `vst1_lane_u8`。


## Key Concepts / 关键概念
- **ATen native operator logic** — ATen 原生算子逻辑
- **Dispatch and backend routing** — 调度与后端路由
- **GPU kernel/runtime interaction** — GPU 内核/运行时交互
- **Mixed-precision/autocast logic** — 混合精度/自动混精逻辑
- **Parallel scan/reduction behavior** — 并行扫描/归约行为
- **Core symbols: pytorch_q8dwconv_ukernel_up8x9__neon, vld1_dup_u8, vdup_n_u8, vdupq_n_f32, vld1q_dup_s16, vreinterpretq_s16_u16, vmlal_s16, vmulq_f32** — 核心符号：pytorch_q8dwconv_ukernel_up8x9__neon、vld1_dup_u8、vdup_n_u8、vdupq_n_f32、vld1q_dup_s16、vreinterpretq_s16_u16、vmlal_s16、vmulq_f32

## Dependencies / 依赖关系
- **Internal includes / 内部头文件**: 无
- **External includes / 外部头文件**: `arm_neon.h`, `qnnpack/q8dwconv.h`, `requantization/runtime-neon.h`
- **Namespaces / 命名空间**: 无
- **Representative symbols / 代表性符号**: `pytorch_q8dwconv_ukernel_up8x9__neon`, `vld1_dup_u8`, `vdup_n_u8`, `vdupq_n_f32`, `vld1q_dup_s16`, `vreinterpretq_s16_u16`, `vmlal_s16`, `vmulq_f32`, `vqmovn_high_s32`, `vst1_u8`, `vshl_u64`, `vst1_lane_u32`, `...`
