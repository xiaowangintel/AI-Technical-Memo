# mp8x25-neon-per-channel.c — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `aten/src/ATen/native/quantized/cpu/qnnpack/src/q8dwconv/mp8x25-neon-per-channel.c`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements ATen native operators, checks, helper kernels, and backend dispatch behavior. This specific file centers on `mp8x25-neon-per-channel.c`. The file header highlights: "All rights reserved. This source code is licensed under the BSD-style license found in the LICENSE file in the root directory of this source tree.."
- **Purpose (CN)**: 实现 ATen 原生算子、运行时检查、辅助内核以及后端分发行为。 该文件具体围绕 `mp8x25-neon-per-channel.c` 展开。 文件头部注释也概括了其核心职责。

## Line-by-Line Analysis / 逐行分析
### Lines 1-36 / 第 1-36 行

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
0013: void pytorch_q8dwconv_ukernel_mp8x25_per_channel__neon(
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
0026: #ifdef __aarch64__
0027:   const int16x8_t voutput_zero_point =
0028:       vld1q_dup_s16(&quantization_params->neon.output_zero_point);
0029:   const uint8x8_t voutput_min = vld1_dup_u8(&quantization_params->neon.output_min);
0030:   const uint8x8_t voutput_max = vld1_dup_u8(&quantization_params->neon.output_max);
0031: #else
0032:   const float32x4_t vfmin = vdupq_n_f32(quantization_params->neon.vfmin);
0033:   const float32x4_t vfmax = vdupq_n_f32(quantization_params->neon.vfmax);
0034:   const float32x4_t vfmagic = vdupq_n_f32(quantization_params->neon.vfmagic);
0035:   const int32x4_t vimagic = vdupq_n_s32(quantization_params->neon.vimagic);
0036: #endif
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends. Key symbols: `pytorch_q8dwconv_ukernel_mp8x25_per_channel__neon`, `vld1_dup_u8`, `vld1q_dup_s16`.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端。关键符号：`pytorch_q8dwconv_ukernel_mp8x25_per_channel__neon`, `vld1_dup_u8`, `vld1q_dup_s16`。

### Lines 37-76 / 第 37-76 行

```c
0037: 
0038:   do {
0039:     uint8_t* output_start = output;
0040:     int32_t* outacc = outacc32;
0041:     const void* w = weights;
0042:     {
0043:       const uint8_t* i0 = input[0];
0044:       const uint8_t* i1 = input[1];
0045:       const uint8_t* i2 = input[2];
0046:       const uint8_t* i3 = input[3];
0047:       const uint8_t* i4 = input[4];
0048:       const uint8_t* i5 = input[5];
0049:       const uint8_t* i6 = input[6];
0050:       const uint8_t* i7 = input[7];
0051:       const uint8_t* i8 = input[8];
0052:       const uint8_t* i9 = input[9];
0053: 
0054:       size_t c = channels;
0055:       const uint8_t* kernel_zero_points_ptr =
0056:           quantization_params->neon.kernel_zero_points + channels;
0057:       for (; c >= 8; c -= 8) {
0058:         const uint8x8_t vkernel_zero_point =
0059:             vld1_u8(kernel_zero_points_ptr - c);
0060:         int32x4_t vaccX1_lo = vld1q_s32(w);
0061:         w = (void*)((uintptr_t)w + sizeof(int32x4_t));
0062:         int32x4_t vaccX1_hi = vld1q_s32(w);
0063:         w = (void*)((uintptr_t)w + sizeof(int32x4_t));
0064: 
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
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `vld1_u8`, `vreinterpretq_s16_u16`, `vmull_s16`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`vld1_u8`, `vreinterpretq_s16_u16`, `vmull_s16`。

### Lines 77-115 / 第 77-115 行

```c
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
```

- **EN:** This block implements local helper logic for `mp8x25-neon-per-channel`. Key symbols: `vreinterpretq_s16_u16`, `vmlal_s16`.
- **CN:** 该代码块实现与 `mp8x25-neon-per-channel` 相关的局部辅助逻辑。关键符号：`vreinterpretq_s16_u16`, `vmlal_s16`。

### Lines 116-154 / 第 116-154 行

```c
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
```

- **EN:** This block implements local helper logic for `mp8x25-neon-per-channel`. Key symbols: `vreinterpretq_s16_u16`, `vmlal_s16`.
- **CN:** 该代码块实现与 `mp8x25-neon-per-channel` 相关的局部辅助逻辑。关键符号：`vreinterpretq_s16_u16`, `vmlal_s16`。

### Lines 155-193 / 第 155-193 行

```c
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
```

- **EN:** This block implements local helper logic for `mp8x25-neon-per-channel`. Key symbols: `vreinterpretq_s16_u16`, `vmlal_s16`.
- **CN:** 该代码块实现与 `mp8x25-neon-per-channel` 相关的局部辅助逻辑。关键符号：`vreinterpretq_s16_u16`, `vmlal_s16`。

### Lines 194-234 / 第 194-234 行

```c
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
0216:         const uint8x8_t vkernel_zero_point =
0217:             vld1_u8(&quantization_params->neon.kernel_zero_points[channels - c]);
0218:         int32x4_t vaccX1_lo = vld1q_s32(w);
0219:         w = (void*)((uintptr_t)w + sizeof(int32x4_t));
0220:         int32x4_t vaccX1_hi = vld1q_s32(w);
0221:         w = (void*)((uintptr_t)w + sizeof(int32x4_t));
0222: 
0223:         const uint8x8_t vk0 = vld1_u8(w);
0224:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0225:         const uint8x8_t vi0 = vreinterpret_u8_u64(
0226:             vshl_u64(vreinterpret_u64_u8(vld1_u8(i0)), vi_shift));
0227:         const int16x8_t vxk0 =
0228:             vreinterpretq_s16_u16(vsubl_u8(vk0, vkernel_zero_point));
0229:         const int16x8_t vxi0 =
0230:             vreinterpretq_s16_u16(vsubl_u8(vi0, vinput_zero_point));
0231:         int32x4_t vaccX0_lo = vmull_s16(vget_low_s16(vxk0), vget_low_s16(vxi0));
0232:         int32x4_t vaccX0_hi =
0233:             vmull_s16(vget_high_s16(vxk0), vget_high_s16(vxi0));
0234: 
```

- **EN:** This block handles conditional branches and special cases. Key symbols: `vst1q_s32`, `vld1_u8`, `vshl_u64`, `vreinterpretq_s16_u16`, `vmull_s16`.
- **CN:** 该代码块处理条件分支与特殊情况。关键符号：`vst1q_s32`, `vld1_u8`, `vshl_u64`, `vreinterpretq_s16_u16`, `vmull_s16`。

### Lines 235-273 / 第 235-273 行

```c
0235:         const uint8x8_t vk1 = vld1_u8(w);
0236:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0237:         const uint8x8_t vi1 = vreinterpret_u8_u64(
0238:             vshl_u64(vreinterpret_u64_u8(vld1_u8(i1)), vi_shift));
0239:         const int16x8_t vxk1 =
0240:             vreinterpretq_s16_u16(vsubl_u8(vk1, vkernel_zero_point));
0241:         const int16x8_t vxi1 =
0242:             vreinterpretq_s16_u16(vsubl_u8(vi1, vinput_zero_point));
0243:         vaccX1_lo =
0244:             vmlal_s16(vaccX1_lo, vget_low_s16(vxk1), vget_low_s16(vxi1));
0245:         vaccX1_hi =
0246:             vmlal_s16(vaccX1_hi, vget_high_s16(vxk1), vget_high_s16(vxi1));
0247: 
0248:         const uint8x8_t vk2 = vld1_u8(w);
0249:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0250:         const uint8x8_t vi2 = vreinterpret_u8_u64(
0251:             vshl_u64(vreinterpret_u64_u8(vld1_u8(i2)), vi_shift));
0252:         const int16x8_t vxk2 =
0253:             vreinterpretq_s16_u16(vsubl_u8(vk2, vkernel_zero_point));
0254:         const int16x8_t vxi2 =
0255:             vreinterpretq_s16_u16(vsubl_u8(vi2, vinput_zero_point));
0256:         vaccX0_lo =
0257:             vmlal_s16(vaccX0_lo, vget_low_s16(vxk2), vget_low_s16(vxi2));
0258:         vaccX0_hi =
0259:             vmlal_s16(vaccX0_hi, vget_high_s16(vxk2), vget_high_s16(vxi2));
0260: 
0261:         const uint8x8_t vk3 = vld1_u8(w);
0262:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0263:         const uint8x8_t vi3 = vreinterpret_u8_u64(
0264:             vshl_u64(vreinterpret_u64_u8(vld1_u8(i3)), vi_shift));
0265:         const int16x8_t vxk3 =
0266:             vreinterpretq_s16_u16(vsubl_u8(vk3, vkernel_zero_point));
0267:         const int16x8_t vxi3 =
0268:             vreinterpretq_s16_u16(vsubl_u8(vi3, vinput_zero_point));
0269:         vaccX1_lo =
0270:             vmlal_s16(vaccX1_lo, vget_low_s16(vxk3), vget_low_s16(vxi3));
0271:         vaccX1_hi =
0272:             vmlal_s16(vaccX1_hi, vget_high_s16(vxk3), vget_high_s16(vxi3));
0273: 
```

- **EN:** This block implements local helper logic for `mp8x25-neon-per-channel`. Key symbols: `vshl_u64`, `vreinterpretq_s16_u16`, `vmlal_s16`.
- **CN:** 该代码块实现与 `mp8x25-neon-per-channel` 相关的局部辅助逻辑。关键符号：`vshl_u64`, `vreinterpretq_s16_u16`, `vmlal_s16`。

### Lines 274-312 / 第 274-312 行

```c
0274:         const uint8x8_t vk4 = vld1_u8(w);
0275:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0276:         const uint8x8_t vi4 = vreinterpret_u8_u64(
0277:             vshl_u64(vreinterpret_u64_u8(vld1_u8(i4)), vi_shift));
0278:         const int16x8_t vxk4 =
0279:             vreinterpretq_s16_u16(vsubl_u8(vk4, vkernel_zero_point));
0280:         const int16x8_t vxi4 =
0281:             vreinterpretq_s16_u16(vsubl_u8(vi4, vinput_zero_point));
0282:         vaccX0_lo =
0283:             vmlal_s16(vaccX0_lo, vget_low_s16(vxk4), vget_low_s16(vxi4));
0284:         vaccX0_hi =
0285:             vmlal_s16(vaccX0_hi, vget_high_s16(vxk4), vget_high_s16(vxi4));
0286: 
0287:         const uint8x8_t vk5 = vld1_u8(w);
0288:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0289:         const uint8x8_t vi5 = vreinterpret_u8_u64(
0290:             vshl_u64(vreinterpret_u64_u8(vld1_u8(i5)), vi_shift));
0291:         const int16x8_t vxk5 =
0292:             vreinterpretq_s16_u16(vsubl_u8(vk5, vkernel_zero_point));
0293:         const int16x8_t vxi5 =
0294:             vreinterpretq_s16_u16(vsubl_u8(vi5, vinput_zero_point));
0295:         vaccX1_lo =
0296:             vmlal_s16(vaccX1_lo, vget_low_s16(vxk5), vget_low_s16(vxi5));
0297:         vaccX1_hi =
0298:             vmlal_s16(vaccX1_hi, vget_high_s16(vxk5), vget_high_s16(vxi5));
0299: 
0300:         const uint8x8_t vk6 = vld1_u8(w);
0301:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0302:         const uint8x8_t vi6 = vreinterpret_u8_u64(
0303:             vshl_u64(vreinterpret_u64_u8(vld1_u8(i6)), vi_shift));
0304:         const int16x8_t vxk6 =
0305:             vreinterpretq_s16_u16(vsubl_u8(vk6, vkernel_zero_point));
0306:         const int16x8_t vxi6 =
0307:             vreinterpretq_s16_u16(vsubl_u8(vi6, vinput_zero_point));
0308:         vaccX0_lo =
0309:             vmlal_s16(vaccX0_lo, vget_low_s16(vxk6), vget_low_s16(vxi6));
0310:         vaccX0_hi =
0311:             vmlal_s16(vaccX0_hi, vget_high_s16(vxk6), vget_high_s16(vxi6));
0312: 
```

- **EN:** This block implements local helper logic for `mp8x25-neon-per-channel`. Key symbols: `vshl_u64`, `vreinterpretq_s16_u16`, `vmlal_s16`.
- **CN:** 该代码块实现与 `mp8x25-neon-per-channel` 相关的局部辅助逻辑。关键符号：`vshl_u64`, `vreinterpretq_s16_u16`, `vmlal_s16`。

### Lines 313-351 / 第 313-351 行

```c
0313:         const uint8x8_t vk7 = vld1_u8(w);
0314:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0315:         const uint8x8_t vi7 = vreinterpret_u8_u64(
0316:             vshl_u64(vreinterpret_u64_u8(vld1_u8(i7)), vi_shift));
0317:         const int16x8_t vxk7 =
0318:             vreinterpretq_s16_u16(vsubl_u8(vk7, vkernel_zero_point));
0319:         const int16x8_t vxi7 =
0320:             vreinterpretq_s16_u16(vsubl_u8(vi7, vinput_zero_point));
0321:         vaccX1_lo =
0322:             vmlal_s16(vaccX1_lo, vget_low_s16(vxk7), vget_low_s16(vxi7));
0323:         vaccX1_hi =
0324:             vmlal_s16(vaccX1_hi, vget_high_s16(vxk7), vget_high_s16(vxi7));
0325: 
0326:         const uint8x8_t vk8 = vld1_u8(w);
0327:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0328:         const uint8x8_t vi8 = vreinterpret_u8_u64(
0329:             vshl_u64(vreinterpret_u64_u8(vld1_u8(i8)), vi_shift));
0330:         const int16x8_t vxk8 =
0331:             vreinterpretq_s16_u16(vsubl_u8(vk8, vkernel_zero_point));
0332:         const int16x8_t vxi8 =
0333:             vreinterpretq_s16_u16(vsubl_u8(vi8, vinput_zero_point));
0334:         vaccX0_lo =
0335:             vmlal_s16(vaccX0_lo, vget_low_s16(vxk8), vget_low_s16(vxi8));
0336:         vaccX0_hi =
0337:             vmlal_s16(vaccX0_hi, vget_high_s16(vxk8), vget_high_s16(vxi8));
0338: 
0339:         const uint8x8_t vk9 = vld1_u8(w);
0340:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0341:         const uint8x8_t vi9 = vreinterpret_u8_u64(
0342:             vshl_u64(vreinterpret_u64_u8(vld1_u8(i9)), vi_shift));
0343:         const int16x8_t vxk9 =
0344:             vreinterpretq_s16_u16(vsubl_u8(vk9, vkernel_zero_point));
0345:         const int16x8_t vxi9 =
0346:             vreinterpretq_s16_u16(vsubl_u8(vi9, vinput_zero_point));
0347:         vaccX1_lo =
0348:             vmlal_s16(vaccX1_lo, vget_low_s16(vxk9), vget_low_s16(vxi9));
0349:         vaccX1_hi =
0350:             vmlal_s16(vaccX1_hi, vget_high_s16(vxk9), vget_high_s16(vxi9));
0351: 
```

- **EN:** This block implements local helper logic for `mp8x25-neon-per-channel`. Key symbols: `vshl_u64`, `vreinterpretq_s16_u16`, `vmlal_s16`.
- **CN:** 该代码块实现与 `mp8x25-neon-per-channel` 相关的局部辅助逻辑。关键符号：`vshl_u64`, `vreinterpretq_s16_u16`, `vmlal_s16`。

### Lines 352-392 / 第 352-392 行

```c
0352:         int32x4_t vacc_lo = vaddq_s32(vaccX0_lo, vaccX1_lo);
0353:         int32x4_t vacc_hi = vaddq_s32(vaccX0_hi, vaccX1_hi);
0354: 
0355:         vst1q_s32(outacc, vacc_lo);
0356:         outacc += 4;
0357:         vst1q_s32(outacc, vacc_hi);
0358:         outacc += 4;
0359:       }
0360:     }
0361:     {
0362:       const uint8_t* i0 = input[10];
0363:       const uint8_t* i1 = input[11];
0364:       const uint8_t* i2 = input[12];
0365:       const uint8_t* i3 = input[13];
0366:       const uint8_t* i4 = input[14];
0367:       const uint8_t* i5 = input[15];
0368:       const uint8_t* i6 = input[16];
0369:       const uint8_t* i7 = input[17];
0370:       const uint8_t* i8 = input[18];
0371:       const uint8_t* i9 = input[19];
0372:       output = output_start;
0373:       outacc = outacc32;
0374: 
0375:       size_t c = channels;
0376:       const uint8_t* kernel_zero_points_ptr =
0377:           quantization_params->neon.kernel_zero_points + channels;
0378:       for (; c >= 8; c -= 8) {
0379:         const uint8x8_t vkernel_zero_point =
0380:             vld1_u8(kernel_zero_points_ptr - c);
0381:         const uint8x8_t vk0 = vld1_u8(w);
0382:         w += 8;
0383:         const uint8x8_t vi0 = vld1_u8(i0);
0384:         i0 += 8;
0385:         const int16x8_t vxk0 =
0386:             vreinterpretq_s16_u16(vsubl_u8(vk0, vkernel_zero_point));
0387:         const int16x8_t vxi0 =
0388:             vreinterpretq_s16_u16(vsubl_u8(vi0, vinput_zero_point));
0389:         int32x4_t vaccX0_lo = vmull_s16(vget_low_s16(vxk0), vget_low_s16(vxi0));
0390:         int32x4_t vaccX0_hi =
0391:             vmull_s16(vget_high_s16(vxk0), vget_high_s16(vxi0));
0392: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `vst1q_s32`, `vld1_u8`, `vreinterpretq_s16_u16`, `vmull_s16`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`vst1q_s32`, `vld1_u8`, `vreinterpretq_s16_u16`, `vmull_s16`。

### Lines 393-430 / 第 393-430 行

```c
0393:         const uint8x8_t vk1 = vld1_u8(w);
0394:         w += 8;
0395:         const uint8x8_t vi1 = vld1_u8(i1);
0396:         i1 += 8;
0397:         const int16x8_t vxk1 =
0398:             vreinterpretq_s16_u16(vsubl_u8(vk1, vkernel_zero_point));
0399:         const int16x8_t vxi1 =
0400:             vreinterpretq_s16_u16(vsubl_u8(vi1, vinput_zero_point));
0401:         int32x4_t vaccX1_lo = vmull_s16(vget_low_s16(vxk1), vget_low_s16(vxi1));
0402:         int32x4_t vaccX1_hi =
0403:             vmull_s16(vget_high_s16(vxk1), vget_high_s16(vxi1));
0404: 
0405:         const uint8x8_t vk2 = vld1_u8(w);
0406:         w += 8;
0407:         const uint8x8_t vi2 = vld1_u8(i2);
0408:         i2 += 8;
0409:         const int16x8_t vxk2 =
0410:             vreinterpretq_s16_u16(vsubl_u8(vk2, vkernel_zero_point));
0411:         const int16x8_t vxi2 =
0412:             vreinterpretq_s16_u16(vsubl_u8(vi2, vinput_zero_point));
0413:         vaccX0_lo =
0414:             vmlal_s16(vaccX0_lo, vget_low_s16(vxk2), vget_low_s16(vxi2));
0415:         vaccX0_hi =
0416:             vmlal_s16(vaccX0_hi, vget_high_s16(vxk2), vget_high_s16(vxi2));
0417: 
0418:         const uint8x8_t vk3 = vld1_u8(w);
0419:         w += 8;
0420:         const uint8x8_t vi3 = vld1_u8(i3);
0421:         i3 += 8;
0422:         const int16x8_t vxk3 =
0423:             vreinterpretq_s16_u16(vsubl_u8(vk3, vkernel_zero_point));
0424:         const int16x8_t vxi3 =
0425:             vreinterpretq_s16_u16(vsubl_u8(vi3, vinput_zero_point));
0426:         vaccX1_lo =
0427:             vmlal_s16(vaccX1_lo, vget_low_s16(vxk3), vget_low_s16(vxi3));
0428:         vaccX1_hi =
0429:             vmlal_s16(vaccX1_hi, vget_high_s16(vxk3), vget_high_s16(vxi3));
0430: 
```

- **EN:** This block implements local helper logic for `mp8x25-neon-per-channel`. Key symbols: `vreinterpretq_s16_u16`, `vmull_s16`, `vmlal_s16`.
- **CN:** 该代码块实现与 `mp8x25-neon-per-channel` 相关的局部辅助逻辑。关键符号：`vreinterpretq_s16_u16`, `vmull_s16`, `vmlal_s16`。

### Lines 431-469 / 第 431-469 行

```c
0431:         const uint8x8_t vk4 = vld1_u8(w);
0432:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0433:         const uint8x8_t vi4 = vld1_u8(i4);
0434:         i4 += 8;
0435:         const int16x8_t vxk4 =
0436:             vreinterpretq_s16_u16(vsubl_u8(vk4, vkernel_zero_point));
0437:         const int16x8_t vxi4 =
0438:             vreinterpretq_s16_u16(vsubl_u8(vi4, vinput_zero_point));
0439:         vaccX0_lo =
0440:             vmlal_s16(vaccX0_lo, vget_low_s16(vxk4), vget_low_s16(vxi4));
0441:         vaccX0_hi =
0442:             vmlal_s16(vaccX0_hi, vget_high_s16(vxk4), vget_high_s16(vxi4));
0443: 
0444:         const uint8x8_t vk5 = vld1_u8(w);
0445:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0446:         const uint8x8_t vi5 = vld1_u8(i5);
0447:         i5 += 8;
0448:         const int16x8_t vxk5 =
0449:             vreinterpretq_s16_u16(vsubl_u8(vk5, vkernel_zero_point));
0450:         const int16x8_t vxi5 =
0451:             vreinterpretq_s16_u16(vsubl_u8(vi5, vinput_zero_point));
0452:         vaccX1_lo =
0453:             vmlal_s16(vaccX1_lo, vget_low_s16(vxk5), vget_low_s16(vxi5));
0454:         vaccX1_hi =
0455:             vmlal_s16(vaccX1_hi, vget_high_s16(vxk5), vget_high_s16(vxi5));
0456: 
0457:         const uint8x8_t vk6 = vld1_u8(w);
0458:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0459:         const uint8x8_t vi6 = vld1_u8(i6);
0460:         i6 += 8;
0461:         const int16x8_t vxk6 =
0462:             vreinterpretq_s16_u16(vsubl_u8(vk6, vkernel_zero_point));
0463:         const int16x8_t vxi6 =
0464:             vreinterpretq_s16_u16(vsubl_u8(vi6, vinput_zero_point));
0465:         vaccX0_lo =
0466:             vmlal_s16(vaccX0_lo, vget_low_s16(vxk6), vget_low_s16(vxi6));
0467:         vaccX0_hi =
0468:             vmlal_s16(vaccX0_hi, vget_high_s16(vxk6), vget_high_s16(vxi6));
0469: 
```

- **EN:** This block implements local helper logic for `mp8x25-neon-per-channel`. Key symbols: `vreinterpretq_s16_u16`, `vmlal_s16`.
- **CN:** 该代码块实现与 `mp8x25-neon-per-channel` 相关的局部辅助逻辑。关键符号：`vreinterpretq_s16_u16`, `vmlal_s16`。

### Lines 470-508 / 第 470-508 行

```c
0470:         const uint8x8_t vk7 = vld1_u8(w);
0471:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0472:         const uint8x8_t vi7 = vld1_u8(i7);
0473:         i7 += 8;
0474:         const int16x8_t vxk7 =
0475:             vreinterpretq_s16_u16(vsubl_u8(vk7, vkernel_zero_point));
0476:         const int16x8_t vxi7 =
0477:             vreinterpretq_s16_u16(vsubl_u8(vi7, vinput_zero_point));
0478:         vaccX1_lo =
0479:             vmlal_s16(vaccX1_lo, vget_low_s16(vxk7), vget_low_s16(vxi7));
0480:         vaccX1_hi =
0481:             vmlal_s16(vaccX1_hi, vget_high_s16(vxk7), vget_high_s16(vxi7));
0482: 
0483:         const uint8x8_t vk8 = vld1_u8(w);
0484:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0485:         const uint8x8_t vi8 = vld1_u8(i8);
0486:         i8 += 8;
0487:         const int16x8_t vxk8 =
0488:             vreinterpretq_s16_u16(vsubl_u8(vk8, vkernel_zero_point));
0489:         const int16x8_t vxi8 =
0490:             vreinterpretq_s16_u16(vsubl_u8(vi8, vinput_zero_point));
0491:         vaccX0_lo =
0492:             vmlal_s16(vaccX0_lo, vget_low_s16(vxk8), vget_low_s16(vxi8));
0493:         vaccX0_hi =
0494:             vmlal_s16(vaccX0_hi, vget_high_s16(vxk8), vget_high_s16(vxi8));
0495: 
0496:         const uint8x8_t vk9 = vld1_u8(w);
0497:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0498:         const uint8x8_t vi9 = vld1_u8(i9);
0499:         i9 += 8;
0500:         const int16x8_t vxk9 =
0501:             vreinterpretq_s16_u16(vsubl_u8(vk9, vkernel_zero_point));
0502:         const int16x8_t vxi9 =
0503:             vreinterpretq_s16_u16(vsubl_u8(vi9, vinput_zero_point));
0504:         vaccX1_lo =
0505:             vmlal_s16(vaccX1_lo, vget_low_s16(vxk9), vget_low_s16(vxi9));
0506:         vaccX1_hi =
0507:             vmlal_s16(vaccX1_hi, vget_high_s16(vxk9), vget_high_s16(vxi9));
0508: 
```

- **EN:** This block implements local helper logic for `mp8x25-neon-per-channel`. Key symbols: `vreinterpretq_s16_u16`, `vmlal_s16`.
- **CN:** 该代码块实现与 `mp8x25-neon-per-channel` 相关的局部辅助逻辑。关键符号：`vreinterpretq_s16_u16`, `vmlal_s16`。

### Lines 509-548 / 第 509-548 行

```c
0509:         int32x4_t vacc_lo = vaddq_s32(vaccX0_lo, vaccX1_lo);
0510:         int32x4_t vacc_hi = vaddq_s32(vaccX0_hi, vaccX1_hi);
0511: 
0512:         const int32x4_t vacc_lo_old = vld1q_s32(outacc);
0513:         const int32x4_t vacc_hi_old = vld1q_s32(outacc + 4);
0514:         vacc_lo = vaddq_s32(vacc_lo, vacc_lo_old);
0515:         vacc_hi = vaddq_s32(vacc_hi, vacc_hi_old);
0516:         vst1q_s32(outacc, vacc_lo);
0517:         outacc += 4;
0518:         vst1q_s32(outacc, vacc_hi);
0519:         outacc += 4;
0520:       }
0521:       if (c != 0) {
0522:         const size_t c_predecrement = 8 - c;
0523:         const int64x1_t vi_shift = vmov_n_s64(-8 * c_predecrement);
0524:         i0 -= c_predecrement;
0525:         i1 -= c_predecrement;
0526:         i2 -= c_predecrement;
0527:         i3 -= c_predecrement;
0528:         i4 -= c_predecrement;
0529:         i5 -= c_predecrement;
0530:         i6 -= c_predecrement;
0531:         i7 -= c_predecrement;
0532:         i8 -= c_predecrement;
0533:         i9 -= c_predecrement;
0534: 
0535:         const uint8x8_t vkernel_zero_point =
0536:             vld1_u8(&quantization_params->neon.kernel_zero_points[channels - c]);
0537:         const uint8x8_t vk0 = vld1_u8(w);
0538:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0539:         const uint8x8_t vi0 = vreinterpret_u8_u64(
0540:             vshl_u64(vreinterpret_u64_u8(vld1_u8(i0)), vi_shift));
0541:         const int16x8_t vxk0 =
0542:             vreinterpretq_s16_u16(vsubl_u8(vk0, vkernel_zero_point));
0543:         const int16x8_t vxi0 =
0544:             vreinterpretq_s16_u16(vsubl_u8(vi0, vinput_zero_point));
0545:         int32x4_t vaccX0_lo = vmull_s16(vget_low_s16(vxk0), vget_low_s16(vxi0));
0546:         int32x4_t vaccX0_hi =
0547:             vmull_s16(vget_high_s16(vxk0), vget_high_s16(vxi0));
0548: 
```

- **EN:** This block handles conditional branches and special cases. Key symbols: `vst1q_s32`, `vld1_u8`, `vshl_u64`, `vreinterpretq_s16_u16`, `vmull_s16`.
- **CN:** 该代码块处理条件分支与特殊情况。关键符号：`vst1q_s32`, `vld1_u8`, `vshl_u64`, `vreinterpretq_s16_u16`, `vmull_s16`。

### Lines 549-586 / 第 549-586 行

```c
0549:         const uint8x8_t vk1 = vld1_u8(w);
0550:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0551:         const uint8x8_t vi1 = vreinterpret_u8_u64(
0552:             vshl_u64(vreinterpret_u64_u8(vld1_u8(i1)), vi_shift));
0553:         const int16x8_t vxk1 =
0554:             vreinterpretq_s16_u16(vsubl_u8(vk1, vkernel_zero_point));
0555:         const int16x8_t vxi1 =
0556:             vreinterpretq_s16_u16(vsubl_u8(vi1, vinput_zero_point));
0557:         int32x4_t vaccX1_lo = vmull_s16(vget_low_s16(vxk1), vget_low_s16(vxi1));
0558:         int32x4_t vaccX1_hi =
0559:             vmull_s16(vget_high_s16(vxk1), vget_high_s16(vxi1));
0560: 
0561:         const uint8x8_t vk2 = vld1_u8(w);
0562:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0563:         const uint8x8_t vi2 = vreinterpret_u8_u64(
0564:             vshl_u64(vreinterpret_u64_u8(vld1_u8(i2)), vi_shift));
0565:         const int16x8_t vxk2 =
0566:             vreinterpretq_s16_u16(vsubl_u8(vk2, vkernel_zero_point));
0567:         const int16x8_t vxi2 =
0568:             vreinterpretq_s16_u16(vsubl_u8(vi2, vinput_zero_point));
0569:         vaccX0_lo =
0570:             vmlal_s16(vaccX0_lo, vget_low_s16(vxk2), vget_low_s16(vxi2));
0571:         vaccX0_hi =
0572:             vmlal_s16(vaccX0_hi, vget_high_s16(vxk2), vget_high_s16(vxi2));
0573: 
0574:         const uint8x8_t vk3 = vld1_u8(w);
0575:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0576:         const uint8x8_t vi3 = vreinterpret_u8_u64(
0577:             vshl_u64(vreinterpret_u64_u8(vld1_u8(i3)), vi_shift));
0578:         const int16x8_t vxk3 =
0579:             vreinterpretq_s16_u16(vsubl_u8(vk3, vkernel_zero_point));
0580:         const int16x8_t vxi3 =
0581:             vreinterpretq_s16_u16(vsubl_u8(vi3, vinput_zero_point));
0582:         vaccX1_lo =
0583:             vmlal_s16(vaccX1_lo, vget_low_s16(vxk3), vget_low_s16(vxi3));
0584:         vaccX1_hi =
0585:             vmlal_s16(vaccX1_hi, vget_high_s16(vxk3), vget_high_s16(vxi3));
0586: 
```

- **EN:** This block implements local helper logic for `mp8x25-neon-per-channel`. Key symbols: `vshl_u64`, `vreinterpretq_s16_u16`, `vmull_s16`, `vmlal_s16`.
- **CN:** 该代码块实现与 `mp8x25-neon-per-channel` 相关的局部辅助逻辑。关键符号：`vshl_u64`, `vreinterpretq_s16_u16`, `vmull_s16`, `vmlal_s16`。

### Lines 587-625 / 第 587-625 行

```c
0587:         const uint8x8_t vk4 = vld1_u8(w);
0588:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0589:         const uint8x8_t vi4 = vreinterpret_u8_u64(
0590:             vshl_u64(vreinterpret_u64_u8(vld1_u8(i4)), vi_shift));
0591:         const int16x8_t vxk4 =
0592:             vreinterpretq_s16_u16(vsubl_u8(vk4, vkernel_zero_point));
0593:         const int16x8_t vxi4 =
0594:             vreinterpretq_s16_u16(vsubl_u8(vi4, vinput_zero_point));
0595:         vaccX0_lo =
0596:             vmlal_s16(vaccX0_lo, vget_low_s16(vxk4), vget_low_s16(vxi4));
0597:         vaccX0_hi =
0598:             vmlal_s16(vaccX0_hi, vget_high_s16(vxk4), vget_high_s16(vxi4));
0599: 
0600:         const uint8x8_t vk5 = vld1_u8(w);
0601:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0602:         const uint8x8_t vi5 = vreinterpret_u8_u64(
0603:             vshl_u64(vreinterpret_u64_u8(vld1_u8(i5)), vi_shift));
0604:         const int16x8_t vxk5 =
0605:             vreinterpretq_s16_u16(vsubl_u8(vk5, vkernel_zero_point));
0606:         const int16x8_t vxi5 =
0607:             vreinterpretq_s16_u16(vsubl_u8(vi5, vinput_zero_point));
0608:         vaccX1_lo =
0609:             vmlal_s16(vaccX1_lo, vget_low_s16(vxk5), vget_low_s16(vxi5));
0610:         vaccX1_hi =
0611:             vmlal_s16(vaccX1_hi, vget_high_s16(vxk5), vget_high_s16(vxi5));
0612: 
0613:         const uint8x8_t vk6 = vld1_u8(w);
0614:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0615:         const uint8x8_t vi6 = vreinterpret_u8_u64(
0616:             vshl_u64(vreinterpret_u64_u8(vld1_u8(i6)), vi_shift));
0617:         const int16x8_t vxk6 =
0618:             vreinterpretq_s16_u16(vsubl_u8(vk6, vkernel_zero_point));
0619:         const int16x8_t vxi6 =
0620:             vreinterpretq_s16_u16(vsubl_u8(vi6, vinput_zero_point));
0621:         vaccX0_lo =
0622:             vmlal_s16(vaccX0_lo, vget_low_s16(vxk6), vget_low_s16(vxi6));
0623:         vaccX0_hi =
0624:             vmlal_s16(vaccX0_hi, vget_high_s16(vxk6), vget_high_s16(vxi6));
0625: 
```

- **EN:** This block implements local helper logic for `mp8x25-neon-per-channel`. Key symbols: `vshl_u64`, `vreinterpretq_s16_u16`, `vmlal_s16`.
- **CN:** 该代码块实现与 `mp8x25-neon-per-channel` 相关的局部辅助逻辑。关键符号：`vshl_u64`, `vreinterpretq_s16_u16`, `vmlal_s16`。

### Lines 626-664 / 第 626-664 行

```c
0626:         const uint8x8_t vk7 = vld1_u8(w);
0627:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0628:         const uint8x8_t vi7 = vreinterpret_u8_u64(
0629:             vshl_u64(vreinterpret_u64_u8(vld1_u8(i7)), vi_shift));
0630:         const int16x8_t vxk7 =
0631:             vreinterpretq_s16_u16(vsubl_u8(vk7, vkernel_zero_point));
0632:         const int16x8_t vxi7 =
0633:             vreinterpretq_s16_u16(vsubl_u8(vi7, vinput_zero_point));
0634:         vaccX1_lo =
0635:             vmlal_s16(vaccX1_lo, vget_low_s16(vxk7), vget_low_s16(vxi7));
0636:         vaccX1_hi =
0637:             vmlal_s16(vaccX1_hi, vget_high_s16(vxk7), vget_high_s16(vxi7));
0638: 
0639:         const uint8x8_t vk8 = vld1_u8(w);
0640:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0641:         const uint8x8_t vi8 = vreinterpret_u8_u64(
0642:             vshl_u64(vreinterpret_u64_u8(vld1_u8(i8)), vi_shift));
0643:         const int16x8_t vxk8 =
0644:             vreinterpretq_s16_u16(vsubl_u8(vk8, vkernel_zero_point));
0645:         const int16x8_t vxi8 =
0646:             vreinterpretq_s16_u16(vsubl_u8(vi8, vinput_zero_point));
0647:         vaccX0_lo =
0648:             vmlal_s16(vaccX0_lo, vget_low_s16(vxk8), vget_low_s16(vxi8));
0649:         vaccX0_hi =
0650:             vmlal_s16(vaccX0_hi, vget_high_s16(vxk8), vget_high_s16(vxi8));
0651: 
0652:         const uint8x8_t vk9 = vld1_u8(w);
0653:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0654:         const uint8x8_t vi9 = vreinterpret_u8_u64(
0655:             vshl_u64(vreinterpret_u64_u8(vld1_u8(i9)), vi_shift));
0656:         const int16x8_t vxk9 =
0657:             vreinterpretq_s16_u16(vsubl_u8(vk9, vkernel_zero_point));
0658:         const int16x8_t vxi9 =
0659:             vreinterpretq_s16_u16(vsubl_u8(vi9, vinput_zero_point));
0660:         vaccX1_lo =
0661:             vmlal_s16(vaccX1_lo, vget_low_s16(vxk9), vget_low_s16(vxi9));
0662:         vaccX1_hi =
0663:             vmlal_s16(vaccX1_hi, vget_high_s16(vxk9), vget_high_s16(vxi9));
0664: 
```

- **EN:** This block implements local helper logic for `mp8x25-neon-per-channel`. Key symbols: `vshl_u64`, `vreinterpretq_s16_u16`, `vmlal_s16`.
- **CN:** 该代码块实现与 `mp8x25-neon-per-channel` 相关的局部辅助逻辑。关键符号：`vshl_u64`, `vreinterpretq_s16_u16`, `vmlal_s16`。

### Lines 665-694 / 第 665-694 行

```c
0665:         int32x4_t vacc_lo = vaddq_s32(vaccX0_lo, vaccX1_lo);
0666:         int32x4_t vacc_hi = vaddq_s32(vaccX0_hi, vaccX1_hi);
0667: 
0668:         const int32x4_t vacc_lo_old = vld1q_s32(outacc);
0669:         const int32x4_t vacc_hi_old = vld1q_s32(outacc + 4);
0670:         vacc_lo = vaddq_s32(vacc_lo, vacc_lo_old);
0671:         vacc_hi = vaddq_s32(vacc_hi, vacc_hi_old);
0672:         vst1q_s32(outacc, vacc_lo);
0673:         outacc += 4;
0674:         vst1q_s32(outacc, vacc_hi);
0675:         outacc += 4;
0676:       }
0677:     }
0678: 
0679:     {
0680:       const uint8_t* i0 = input[20];
0681:       const uint8_t* i1 = input[21];
0682:       const uint8_t* i2 = input[22];
0683:       const uint8_t* i3 = input[23];
0684:       const uint8_t* i4 = input[24];
0685:       input = (const uint8_t**)((uintptr_t)input + input_stride);
0686:       output = output_start;
0687:       outacc = outacc32;
0688: 
0689:       size_t c = channels;
0690:       const uint8_t* kernel_zero_points_ptr =
0691:           quantization_params->neon.kernel_zero_points + channels;
0692:       const float* requantization_scales_ptr =
0693:           quantization_params->neon.requantization_scales + channels;
0694:       for (; c >= 8; c -= 8) {
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `vst1q_s32`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`vst1q_s32`。

### Lines 695-733 / 第 695-733 行

```c
0695:         const uint8x8_t vkernel_zero_point =
0696:             vld1_u8(kernel_zero_points_ptr - c);
0697:         const uint8x8_t vk0 = vld1_u8(w);
0698:         w += 8;
0699:         const uint8x8_t vi0 = vld1_u8(i0);
0700:         i0 += 8;
0701:         const int16x8_t vxk0 =
0702:             vreinterpretq_s16_u16(vsubl_u8(vk0, vkernel_zero_point));
0703:         const int16x8_t vxi0 =
0704:             vreinterpretq_s16_u16(vsubl_u8(vi0, vinput_zero_point));
0705:         int32x4_t vaccX0_lo = vmull_s16(vget_low_s16(vxk0), vget_low_s16(vxi0));
0706:         int32x4_t vaccX0_hi =
0707:             vmull_s16(vget_high_s16(vxk0), vget_high_s16(vxi0));
0708: 
0709:         const uint8x8_t vk1 = vld1_u8(w);
0710:         w += 8;
0711:         const uint8x8_t vi1 = vld1_u8(i1);
0712:         i1 += 8;
0713:         const int16x8_t vxk1 =
0714:             vreinterpretq_s16_u16(vsubl_u8(vk1, vkernel_zero_point));
0715:         const int16x8_t vxi1 =
0716:             vreinterpretq_s16_u16(vsubl_u8(vi1, vinput_zero_point));
0717:         int32x4_t vaccX1_lo = vmull_s16(vget_low_s16(vxk1), vget_low_s16(vxi1));
0718:         int32x4_t vaccX1_hi =
0719:             vmull_s16(vget_high_s16(vxk1), vget_high_s16(vxi1));
0720: 
0721:         const uint8x8_t vk2 = vld1_u8(w);
0722:         w += 8;
0723:         const uint8x8_t vi2 = vld1_u8(i2);
0724:         i2 += 8;
0725:         const int16x8_t vxk2 =
0726:             vreinterpretq_s16_u16(vsubl_u8(vk2, vkernel_zero_point));
0727:         const int16x8_t vxi2 =
0728:             vreinterpretq_s16_u16(vsubl_u8(vi2, vinput_zero_point));
0729:         vaccX0_lo =
0730:             vmlal_s16(vaccX0_lo, vget_low_s16(vxk2), vget_low_s16(vxi2));
0731:         vaccX0_hi =
0732:             vmlal_s16(vaccX0_hi, vget_high_s16(vxk2), vget_high_s16(vxi2));
0733: 
```

- **EN:** This block implements local helper logic for `mp8x25-neon-per-channel`. Key symbols: `vld1_u8`, `vreinterpretq_s16_u16`, `vmull_s16`, `vmlal_s16`.
- **CN:** 该代码块实现与 `mp8x25-neon-per-channel` 相关的局部辅助逻辑。关键符号：`vld1_u8`, `vreinterpretq_s16_u16`, `vmull_s16`, `vmlal_s16`。

### Lines 734-769 / 第 734-769 行

```c
0734:         const uint8x8_t vk3 = vld1_u8(w);
0735:         w += 8;
0736:         const uint8x8_t vi3 = vld1_u8(i3);
0737:         i3 += 8;
0738:         const int16x8_t vxk3 =
0739:             vreinterpretq_s16_u16(vsubl_u8(vk3, vkernel_zero_point));
0740:         const int16x8_t vxi3 =
0741:             vreinterpretq_s16_u16(vsubl_u8(vi3, vinput_zero_point));
0742:         vaccX1_lo =
0743:             vmlal_s16(vaccX1_lo, vget_low_s16(vxk3), vget_low_s16(vxi3));
0744:         vaccX1_hi =
0745:             vmlal_s16(vaccX1_hi, vget_high_s16(vxk3), vget_high_s16(vxi3));
0746: 
0747:         const uint8x8_t vk4 = vld1_u8(w);
0748:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0749:         const uint8x8_t vi4 = vld1_u8(i4);
0750:         i4 += 8;
0751:         const int16x8_t vxk4 =
0752:             vreinterpretq_s16_u16(vsubl_u8(vk4, vkernel_zero_point));
0753:         const int16x8_t vxi4 =
0754:             vreinterpretq_s16_u16(vsubl_u8(vi4, vinput_zero_point));
0755:         vaccX0_lo =
0756:             vmlal_s16(vaccX0_lo, vget_low_s16(vxk4), vget_low_s16(vxi4));
0757:         vaccX0_hi =
0758:             vmlal_s16(vaccX0_hi, vget_high_s16(vxk4), vget_high_s16(vxi4));
0759: 
0760:         int32x4_t vacc_lo = vaddq_s32(vaccX0_lo, vaccX1_lo);
0761:         int32x4_t vacc_hi = vaddq_s32(vaccX0_hi, vaccX1_hi);
0762: 
0763:         const int32x4_t vacc_lo_old = vld1q_s32(outacc);
0764:         outacc += 4;
0765:         const int32x4_t vacc_hi_old = vld1q_s32(outacc);
0766:         outacc += 4;
0767:         vacc_lo = vaddq_s32(vacc_lo, vacc_lo_old);
0768:         vacc_hi = vaddq_s32(vacc_hi, vacc_hi_old);
0769: 
```

- **EN:** This block implements local helper logic for `mp8x25-neon-per-channel`. Key symbols: `vreinterpretq_s16_u16`, `vmlal_s16`.
- **CN:** 该代码块实现与 `mp8x25-neon-per-channel` 相关的局部辅助逻辑。关键符号：`vreinterpretq_s16_u16`, `vmlal_s16`。

### Lines 770-801 / 第 770-801 行

```c
0770:         const float32x4_t requantization_scale_v_lo =
0771:             vld1q_f32(requantization_scales_ptr - c);
0772:         const float32x4_t requantization_scale_v_hi =
0773:             vld1q_f32(requantization_scales_ptr - c + 4);
0774: 
0775:         const float32x4_t vacc_lo_f =
0776:           vmulq_f32(vcvtq_f32_s32(vacc_lo), requantization_scale_v_lo);
0777:         const float32x4_t vacc_hi_f =
0778:           vmulq_f32(vcvtq_f32_s32(vacc_hi), requantization_scale_v_hi);
0779: 
0780: #ifdef __aarch64__
0781:         vacc_lo = vcvtnq_s32_f32(vacc_lo_f);
0782:         vacc_hi = vcvtnq_s32_f32(vacc_hi_f);
0783: 
0784:         const int16x8_t vacc = vqaddq_s16(
0785:             vqmovn_high_s32(vqmovn_s32(vacc_lo), vacc_hi), voutput_zero_point);
0786: 
0787:         uint8x8_t vout = vqmovun_s16(vacc);
0788:         vout = vmax_u8(vout, voutput_min);
0789:         vout = vmin_u8(vout, voutput_max);
0790: #else
0791:         const float32x4_t vacc_lo_f_clamped =
0792:             vminq_f32(vmaxq_f32(vacc_lo_f, vfmin), vfmax);
0793:         const float32x4_t vacc_hi_f_clamped =
0794:             vminq_f32(vmaxq_f32(vacc_hi_f, vfmin), vfmax);
0795:         vacc_lo = vsubq_s32(
0796:             vreinterpretq_s32_f32(vaddq_f32(vacc_lo_f_clamped, vfmagic)), vimagic);
0797:         vacc_hi = vsubq_s32(
0798:             vreinterpretq_s32_f32(vaddq_f32(vacc_hi_f_clamped, vfmagic)), vimagic);
0799:         const int16x8_t vacc =
0800:             vcombine_s16(vqmovn_s32(vacc_lo), vqmovn_s32(vacc_hi));
0801: 
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends. Key symbols: `vld1q_f32`, `vmulq_f32`, `vqmovn_high_s32`, `vminq_f32`, `vreinterpretq_s32_f32`, `vcombine_s16`.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端。关键符号：`vld1q_f32`, `vmulq_f32`, `vqmovn_high_s32`, `vminq_f32`, `vreinterpretq_s32_f32`, `vcombine_s16`。

### Lines 802-842 / 第 802-842 行

```c
0802:         uint8x8_t vout = vqmovun_s16(vacc);
0803: #endif
0804: 
0805:         vst1_u8(output, vout);
0806:         output += 8;
0807:       }
0808:       if (c != 0) {
0809:         const size_t c_predecrement = 8 - c;
0810:         const int64x1_t vi_shift = vmov_n_s64(-8 * c_predecrement);
0811:         i0 -= c_predecrement;
0812:         i1 -= c_predecrement;
0813:         i2 -= c_predecrement;
0814:         i3 -= c_predecrement;
0815:         i4 -= c_predecrement;
0816: 
0817:         const uint8x8_t vkernel_zero_point =
0818:             vld1_u8(&quantization_params->neon.kernel_zero_points[channels - c]);
0819:         const uint8x8_t vk0 = vld1_u8(w);
0820:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0821:         const uint8x8_t vi0 = vreinterpret_u8_u64(
0822:             vshl_u64(vreinterpret_u64_u8(vld1_u8(i0)), vi_shift));
0823:         const int16x8_t vxk0 =
0824:             vreinterpretq_s16_u16(vsubl_u8(vk0, vkernel_zero_point));
0825:         const int16x8_t vxi0 =
0826:             vreinterpretq_s16_u16(vsubl_u8(vi0, vinput_zero_point));
0827:         int32x4_t vaccX0_lo = vmull_s16(vget_low_s16(vxk0), vget_low_s16(vxi0));
0828:         int32x4_t vaccX0_hi =
0829:             vmull_s16(vget_high_s16(vxk0), vget_high_s16(vxi0));
0830: 
0831:         const uint8x8_t vk1 = vld1_u8(w);
0832:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0833:         const uint8x8_t vi1 = vreinterpret_u8_u64(
0834:             vshl_u64(vreinterpret_u64_u8(vld1_u8(i1)), vi_shift));
0835:         const int16x8_t vxk1 =
0836:             vreinterpretq_s16_u16(vsubl_u8(vk1, vkernel_zero_point));
0837:         const int16x8_t vxi1 =
0838:             vreinterpretq_s16_u16(vsubl_u8(vi1, vinput_zero_point));
0839:         int32x4_t vaccX1_lo = vmull_s16(vget_low_s16(vxk1), vget_low_s16(vxi1));
0840:         int32x4_t vaccX1_hi =
0841:             vmull_s16(vget_high_s16(vxk1), vget_high_s16(vxi1));
0842: 
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends; handles conditional branches and special cases. Key symbols: `vst1_u8`, `vld1_u8`, `vshl_u64`, `vreinterpretq_s16_u16`, `vmull_s16`.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端；处理条件分支与特殊情况。关键符号：`vst1_u8`, `vld1_u8`, `vshl_u64`, `vreinterpretq_s16_u16`, `vmull_s16`。

### Lines 843-881 / 第 843-881 行

```c
0843:         const uint8x8_t vk2 = vld1_u8(w);
0844:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0845:         const uint8x8_t vi2 = vreinterpret_u8_u64(
0846:             vshl_u64(vreinterpret_u64_u8(vld1_u8(i2)), vi_shift));
0847:         const int16x8_t vxk2 =
0848:             vreinterpretq_s16_u16(vsubl_u8(vk2, vkernel_zero_point));
0849:         const int16x8_t vxi2 =
0850:             vreinterpretq_s16_u16(vsubl_u8(vi2, vinput_zero_point));
0851:         vaccX0_lo =
0852:             vmlal_s16(vaccX0_lo, vget_low_s16(vxk2), vget_low_s16(vxi2));
0853:         vaccX0_hi =
0854:             vmlal_s16(vaccX0_hi, vget_high_s16(vxk2), vget_high_s16(vxi2));
0855: 
0856:         const uint8x8_t vk3 = vld1_u8(w);
0857:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0858:         const uint8x8_t vi3 = vreinterpret_u8_u64(
0859:             vshl_u64(vreinterpret_u64_u8(vld1_u8(i3)), vi_shift));
0860:         const int16x8_t vxk3 =
0861:             vreinterpretq_s16_u16(vsubl_u8(vk3, vkernel_zero_point));
0862:         const int16x8_t vxi3 =
0863:             vreinterpretq_s16_u16(vsubl_u8(vi3, vinput_zero_point));
0864:         vaccX1_lo =
0865:             vmlal_s16(vaccX1_lo, vget_low_s16(vxk3), vget_low_s16(vxi3));
0866:         vaccX1_hi =
0867:             vmlal_s16(vaccX1_hi, vget_high_s16(vxk3), vget_high_s16(vxi3));
0868: 
0869:         const uint8x8_t vk4 = vld1_u8(w);
0870:         w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0871:         const uint8x8_t vi4 = vreinterpret_u8_u64(
0872:             vshl_u64(vreinterpret_u64_u8(vld1_u8(i4)), vi_shift));
0873:         const int16x8_t vxk4 =
0874:             vreinterpretq_s16_u16(vsubl_u8(vk4, vkernel_zero_point));
0875:         const int16x8_t vxi4 =
0876:             vreinterpretq_s16_u16(vsubl_u8(vi4, vinput_zero_point));
0877:         vaccX0_lo =
0878:             vmlal_s16(vaccX0_lo, vget_low_s16(vxk4), vget_low_s16(vxi4));
0879:         vaccX0_hi =
0880:             vmlal_s16(vaccX0_hi, vget_high_s16(vxk4), vget_high_s16(vxi4));
0881: 
```

- **EN:** This block implements local helper logic for `mp8x25-neon-per-channel`. Key symbols: `vshl_u64`, `vreinterpretq_s16_u16`, `vmlal_s16`.
- **CN:** 该代码块实现与 `mp8x25-neon-per-channel` 相关的局部辅助逻辑。关键符号：`vshl_u64`, `vreinterpretq_s16_u16`, `vmlal_s16`。

### Lines 882-921 / 第 882-921 行

```c
0882:         int32x4_t vacc_lo = vaddq_s32(vaccX0_lo, vaccX1_lo);
0883:         int32x4_t vacc_hi = vaddq_s32(vaccX0_hi, vaccX1_hi);
0884: 
0885:         const int32x4_t vacc_lo_old = vld1q_s32(outacc);
0886:         const int32x4_t vacc_hi_old = vld1q_s32(outacc + 4);
0887:         vacc_lo = vaddq_s32(vacc_lo, vacc_lo_old);
0888:         vacc_hi = vaddq_s32(vacc_hi, vacc_hi_old);
0889: 
0890:         const float32x4_t requantization_scale_v_lo =
0891:             vld1q_f32(&quantization_params->neon.requantization_scales[channels - c]);
0892:         const float32x4_t requantization_scale_v_hi =
0893:             vld1q_f32(&quantization_params->neon.requantization_scales[channels - c + 4]);
0894: 
0895:         const float32x4_t vacc_lo_f =
0896:           vmulq_f32(vcvtq_f32_s32(vacc_lo), requantization_scale_v_lo);
0897:         const float32x4_t vacc_hi_f =
0898:           vmulq_f32(vcvtq_f32_s32(vacc_hi), requantization_scale_v_hi);
0899: 
0900: #ifdef __aarch64__
0901:         vacc_lo = vcvtnq_s32_f32(vacc_lo_f);
0902:         vacc_hi = vcvtnq_s32_f32(vacc_hi_f);
0903: 
0904:         const int16x8_t vacc = vqaddq_s16(
0905:             vqmovn_high_s32(vqmovn_s32(vacc_lo), vacc_hi), voutput_zero_point);
0906: 
0907:         uint8x8_t vout = vqmovun_s16(vacc);
0908:         vout = vmax_u8(vout, voutput_min);
0909:         vout = vmin_u8(vout, voutput_max);
0910: #else
0911:         const float32x4_t vacc_lo_f_clamped =
0912:             vminq_f32(vmaxq_f32(vacc_lo_f, vfmin), vfmax);
0913:         const float32x4_t vacc_hi_f_clamped =
0914:             vminq_f32(vmaxq_f32(vacc_hi_f, vfmin), vfmax);
0915:         vacc_lo = vsubq_s32(
0916:             vreinterpretq_s32_f32(vaddq_f32(vacc_lo_f_clamped, vfmagic)), vimagic);
0917:         vacc_hi = vsubq_s32(
0918:             vreinterpretq_s32_f32(vaddq_f32(vacc_hi_f_clamped, vfmagic)), vimagic);
0919:         const int16x8_t vacc =
0920:             vcombine_s16(vqmovn_s32(vacc_lo), vqmovn_s32(vacc_hi));
0921: 
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends. Key symbols: `vld1q_f32`, `vmulq_f32`, `vqmovn_high_s32`, `vminq_f32`, `vreinterpretq_s32_f32`, `vcombine_s16`.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端。关键符号：`vld1q_f32`, `vmulq_f32`, `vqmovn_high_s32`, `vminq_f32`, `vreinterpretq_s32_f32`, `vcombine_s16`。

### Lines 922-950 / 第 922-950 行

```c
0922:         uint8x8_t vout = vqmovun_s16(vacc);
0923: #endif
0924: 
0925:         if (c & 4) {
0926:           vst1_lane_u32(
0927:               __builtin_assume_aligned(output, 1),
0928:               vreinterpret_u32_u8(vout),
0929:               0);
0930:           output += 4;
0931:           vout = vext_u8(vout, vout, 4);
0932:         }
0933:         if (c & 2) {
0934:           vst1_lane_u16(
0935:               __builtin_assume_aligned(output, 1),
0936:               vreinterpret_u16_u8(vout),
0937:               0);
0938:           output += 2;
0939:           vout = vext_u8(vout, vout, 2);
0940:         }
0941:         if (c & 1) {
0942:           vst1_lane_u8(__builtin_assume_aligned(output, 1), vout, 0);
0943:           output++;
0944:         }
0945:       }
0946:     }
0947: 
0948:     output = (uint8_t*)((uintptr_t)output + output_increment);
0949:   } while (--output_width != 0);
0950: }
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends; handles conditional branches and special cases; iterates over tensors, descriptors, options, or runtime state. Key symbols: `vst1_lane_u32`, `vst1_lane_u16`, `vst1_lane_u8`.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端；处理条件分支与特殊情况；遍历张量、描述符、配置项或运行时状态。关键符号：`vst1_lane_u32`, `vst1_lane_u16`, `vst1_lane_u8`。


## Key Concepts / 关键概念
- **ATen native operator logic** — ATen 原生算子逻辑
- **GPU kernel/runtime interaction** — GPU 内核/运行时交互
- **Mixed-precision/autocast logic** — 混合精度/自动混精逻辑
- **Parallel scan/reduction behavior** — 并行扫描/归约行为
- **Core symbols: pytorch_q8dwconv_ukernel_mp8x25_per_channel__neon, vld1_dup_u8, vld1q_dup_s16, vld1_u8, vreinterpretq_s16_u16, vmull_s16, vmlal_s16, vst1q_s32** — 核心符号：pytorch_q8dwconv_ukernel_mp8x25_per_channel__neon、vld1_dup_u8、vld1q_dup_s16、vld1_u8、vreinterpretq_s16_u16、vmull_s16、vmlal_s16、vst1q_s32

## Dependencies / 依赖关系
- **Internal includes / 内部头文件**: 无
- **External includes / 外部头文件**: `arm_neon.h`, `qnnpack/q8dwconv.h`
- **Namespaces / 命名空间**: 无
- **Representative symbols / 代表性符号**: `pytorch_q8dwconv_ukernel_mp8x25_per_channel__neon`, `vld1_dup_u8`, `vld1q_dup_s16`, `vld1_u8`, `vreinterpretq_s16_u16`, `vmull_s16`, `vmlal_s16`, `vst1q_s32`, `vshl_u64`, `vld1q_f32`, `vmulq_f32`, `vqmovn_high_s32`, `...`
