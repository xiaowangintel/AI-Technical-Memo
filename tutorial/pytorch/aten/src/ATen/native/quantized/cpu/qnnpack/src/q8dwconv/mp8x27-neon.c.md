# mp8x27-neon.c — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `aten/src/ATen/native/quantized/cpu/qnnpack/src/q8dwconv/mp8x27-neon.c`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements ATen native operators, checks, helper kernels, and backend dispatch behavior. This specific file centers on `mp8x27-neon.c`. The file header highlights: "All rights reserved. This source code is licensed under the BSD-style license found in the LICENSE file in the root directory of this source tree.."
- **Purpose (CN)**: 实现 ATen 原生算子、运行时检查、辅助内核以及后端分发行为。 该文件具体围绕 `mp8x27-neon.c` 展开。 文件头部注释也概括了其核心职责。

## Line-by-Line Analysis / 逐行分析
### Lines 1-43 / 第 1-43 行

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
0013: /**
0014:  * Kernel for performing depthwise convolution with a kernel of weights of
0015:  * size 27 (ex. 3x3x3). The general strategy is
0016:  * For each output row:
0017:  *   For each output pixel in that row:
0018:  *     For each of the three groupings of 9 of the 27 weights (ex. one yz slice for a 3x3x3 kernel):
0019:  *       For each grouping of 8 channels:
0020:  *          Load input pixel values from the indirection buffer and the weights,
0021:  *          multiply and add them, and keep track of a running total of these
0022:  *          products and the bias in a temporary buffer
0023:  *       Perform requantization to obtain final output
0024:  *     Advance indirection buffer to next pixel in the row
0025:  *   Advance indirection buffer to next row
0026:  */
0027: void pytorch_q8dwconv_ukernel_mp8x27__neon(
0028:     size_t channels,
0029:     size_t output_height,
0030:     size_t output_width,
0031:     const uint8_t** input,
0032:     const void* weights,
0033:     int32_t* outacc32,
0034:     uint8_t* output,
0035:     size_t input_row_stride, // Stride between rows in the indirection buffer
0036:     size_t input_col_stride, // Stride between columns in the indirection buffer
0037:     size_t
0038:         output_increment, // Padding between output pixels in the output buffer
0039:     const union pytorch_qnnp_conv_quantization_params
0040:         quantization_params[restrict static 1]) {
0041:   const uint8x8_t vinput_zero_point =
0042:       vld1_dup_u8((const uint8_t*)&quantization_params->neon.input_zero_point);
0043: #ifdef __aarch64__
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends; manages memory allocation, buffers, or ownership boundaries. Key symbols: `pytorch_q8dwconv_ukernel_mp8x27__neon`, `vld1_dup_u8`.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端；管理内存分配、缓冲区或所有权边界。关键符号：`pytorch_q8dwconv_ukernel_mp8x27__neon`, `vld1_dup_u8`。

### Lines 44-75 / 第 44-75 行

```c
0044:   const int16x8_t voutput_zero_point =
0045:       vld1q_dup_s16(&quantization_params->neon.output_zero_point);
0046:   const uint8x8_t voutput_min = vld1_dup_u8(&quantization_params->neon.output_min);
0047:   const uint8x8_t voutput_max = vld1_dup_u8(&quantization_params->neon.output_max);
0048: #else
0049:   const float32x4_t vfmin = vdupq_n_f32(quantization_params->neon.vfmin);
0050:   const float32x4_t vfmax = vdupq_n_f32(quantization_params->neon.vfmax);
0051:   const float32x4_t vfmagic = vdupq_n_f32(quantization_params->neon.vfmagic);
0052:   const int32x4_t vimagic = vdupq_n_s32(quantization_params->neon.vimagic);
0053: #endif
0054: 
0055:   for (size_t output_y = 0; output_y < output_height; output_y++) {
0056:     const uint8_t** input_row_start = input;
0057:     for (size_t output_x = 0; output_x < output_width; output_x++) {
0058:       uint8_t* output_start = output;
0059:       int32_t* outacc = outacc32;
0060:       const void* w = weights;
0061:       {
0062:         const uint8_t* i0 = input[0];
0063:         const uint8_t* i1 = input[1];
0064:         const uint8_t* i2 = input[2];
0065:         const uint8_t* i3 = input[3];
0066:         const uint8_t* i4 = input[4];
0067:         const uint8_t* i5 = input[5];
0068:         const uint8_t* i6 = input[6];
0069:         const uint8_t* i7 = input[7];
0070:         const uint8_t* i8 = input[8];
0071: 
0072:         size_t c = channels;
0073:         const uint8_t* kernel_zero_points_ptr =
0074:             quantization_params->neon.kernel_zero_points + channels;
0075:         for (; c >= 8; c -= 8) {
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends; iterates over tensors, descriptors, options, or runtime state. Key symbols: `vld1q_dup_s16`.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端；遍历张量、描述符、配置项或运行时状态。关键符号：`vld1q_dup_s16`。

### Lines 76-108 / 第 76-108 行

```c
0076:           const uint8x8_t vkernel_zero_point =
0077:               vld1_u8(kernel_zero_points_ptr - c);
0078:           int32x4_t vaccX1_lo = vld1q_s32(w);
0079:           w = (void*)((uintptr_t)w + sizeof(int32x4_t));
0080:           int32x4_t vaccX1_hi = vld1q_s32(w);
0081:           w = (void*)((uintptr_t)w + sizeof(int32x4_t));
0082: 
0083:           const uint8x8_t vk0 = vld1_u8(w);
0084:           w += 8;
0085:           const uint8x8_t vi0 = vld1_u8(i0);
0086:           i0 += 8;
0087:           const int16x8_t vxk0 =
0088:               vreinterpretq_s16_u16(vsubl_u8(vk0, vkernel_zero_point));
0089:           const int16x8_t vxi0 =
0090:               vreinterpretq_s16_u16(vsubl_u8(vi0, vinput_zero_point));
0091:           int32x4_t vaccX0_lo =
0092:               vmull_s16(vget_low_s16(vxk0), vget_low_s16(vxi0));
0093:           int32x4_t vaccX0_hi =
0094:               vmull_s16(vget_high_s16(vxk0), vget_high_s16(vxi0));
0095: 
0096:           const uint8x8_t vk1 = vld1_u8(w);
0097:           w += 8;
0098:           const uint8x8_t vi1 = vld1_u8(i1);
0099:           i1 += 8;
0100:           const int16x8_t vxk1 =
0101:               vreinterpretq_s16_u16(vsubl_u8(vk1, vkernel_zero_point));
0102:           const int16x8_t vxi1 =
0103:               vreinterpretq_s16_u16(vsubl_u8(vi1, vinput_zero_point));
0104:           vaccX1_lo =
0105:               vmlal_s16(vaccX1_lo, vget_low_s16(vxk1), vget_low_s16(vxi1));
0106:           vaccX1_hi =
0107:               vmlal_s16(vaccX1_hi, vget_high_s16(vxk1), vget_high_s16(vxi1));
0108: 
```

- **EN:** This block implements local helper logic for `mp8x27-neon`. Key symbols: `vld1_u8`, `vreinterpretq_s16_u16`, `vmull_s16`, `vmlal_s16`.
- **CN:** 该代码块实现与 `mp8x27-neon` 相关的局部辅助逻辑。关键符号：`vld1_u8`, `vreinterpretq_s16_u16`, `vmull_s16`, `vmlal_s16`。

### Lines 109-147 / 第 109-147 行

```c
0109:           const uint8x8_t vk2 = vld1_u8(w);
0110:           w += 8;
0111:           const uint8x8_t vi2 = vld1_u8(i2);
0112:           i2 += 8;
0113:           const int16x8_t vxk2 =
0114:               vreinterpretq_s16_u16(vsubl_u8(vk2, vkernel_zero_point));
0115:           const int16x8_t vxi2 =
0116:               vreinterpretq_s16_u16(vsubl_u8(vi2, vinput_zero_point));
0117:           vaccX0_lo =
0118:               vmlal_s16(vaccX0_lo, vget_low_s16(vxk2), vget_low_s16(vxi2));
0119:           vaccX0_hi =
0120:               vmlal_s16(vaccX0_hi, vget_high_s16(vxk2), vget_high_s16(vxi2));
0121: 
0122:           const uint8x8_t vk3 = vld1_u8(w);
0123:           w += 8;
0124:           const uint8x8_t vi3 = vld1_u8(i3);
0125:           i3 += 8;
0126:           const int16x8_t vxk3 =
0127:               vreinterpretq_s16_u16(vsubl_u8(vk3, vkernel_zero_point));
0128:           const int16x8_t vxi3 =
0129:               vreinterpretq_s16_u16(vsubl_u8(vi3, vinput_zero_point));
0130:           vaccX1_lo =
0131:               vmlal_s16(vaccX1_lo, vget_low_s16(vxk3), vget_low_s16(vxi3));
0132:           vaccX1_hi =
0133:               vmlal_s16(vaccX1_hi, vget_high_s16(vxk3), vget_high_s16(vxi3));
0134: 
0135:           const uint8x8_t vk4 = vld1_u8(w);
0136:           w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0137:           const uint8x8_t vi4 = vld1_u8(i4);
0138:           i4 += 8;
0139:           const int16x8_t vxk4 =
0140:               vreinterpretq_s16_u16(vsubl_u8(vk4, vkernel_zero_point));
0141:           const int16x8_t vxi4 =
0142:               vreinterpretq_s16_u16(vsubl_u8(vi4, vinput_zero_point));
0143:           vaccX0_lo =
0144:               vmlal_s16(vaccX0_lo, vget_low_s16(vxk4), vget_low_s16(vxi4));
0145:           vaccX0_hi =
0146:               vmlal_s16(vaccX0_hi, vget_high_s16(vxk4), vget_high_s16(vxi4));
0147: 
```

- **EN:** This block implements local helper logic for `mp8x27-neon`. Key symbols: `vreinterpretq_s16_u16`, `vmlal_s16`.
- **CN:** 该代码块实现与 `mp8x27-neon` 相关的局部辅助逻辑。关键符号：`vreinterpretq_s16_u16`, `vmlal_s16`。

### Lines 148-186 / 第 148-186 行

```c
0148:           const uint8x8_t vk5 = vld1_u8(w);
0149:           w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0150:           const uint8x8_t vi5 = vld1_u8(i5);
0151:           i5 += 8;
0152:           const int16x8_t vxk5 =
0153:               vreinterpretq_s16_u16(vsubl_u8(vk5, vkernel_zero_point));
0154:           const int16x8_t vxi5 =
0155:               vreinterpretq_s16_u16(vsubl_u8(vi5, vinput_zero_point));
0156:           vaccX1_lo =
0157:               vmlal_s16(vaccX1_lo, vget_low_s16(vxk5), vget_low_s16(vxi5));
0158:           vaccX1_hi =
0159:               vmlal_s16(vaccX1_hi, vget_high_s16(vxk5), vget_high_s16(vxi5));
0160: 
0161:           const uint8x8_t vk6 = vld1_u8(w);
0162:           w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0163:           const uint8x8_t vi6 = vld1_u8(i6);
0164:           i6 += 8;
0165:           const int16x8_t vxk6 =
0166:               vreinterpretq_s16_u16(vsubl_u8(vk6, vkernel_zero_point));
0167:           const int16x8_t vxi6 =
0168:               vreinterpretq_s16_u16(vsubl_u8(vi6, vinput_zero_point));
0169:           vaccX0_lo =
0170:               vmlal_s16(vaccX0_lo, vget_low_s16(vxk6), vget_low_s16(vxi6));
0171:           vaccX0_hi =
0172:               vmlal_s16(vaccX0_hi, vget_high_s16(vxk6), vget_high_s16(vxi6));
0173: 
0174:           const uint8x8_t vk7 = vld1_u8(w);
0175:           w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0176:           const uint8x8_t vi7 = vld1_u8(i7);
0177:           i7 += 8;
0178:           const int16x8_t vxk7 =
0179:               vreinterpretq_s16_u16(vsubl_u8(vk7, vkernel_zero_point));
0180:           const int16x8_t vxi7 =
0181:               vreinterpretq_s16_u16(vsubl_u8(vi7, vinput_zero_point));
0182:           vaccX1_lo =
0183:               vmlal_s16(vaccX1_lo, vget_low_s16(vxk7), vget_low_s16(vxi7));
0184:           vaccX1_hi =
0185:               vmlal_s16(vaccX1_hi, vget_high_s16(vxk7), vget_high_s16(vxi7));
0186: 
```

- **EN:** This block implements local helper logic for `mp8x27-neon`. Key symbols: `vreinterpretq_s16_u16`, `vmlal_s16`.
- **CN:** 该代码块实现与 `mp8x27-neon` 相关的局部辅助逻辑。关键符号：`vreinterpretq_s16_u16`, `vmlal_s16`。

### Lines 187-220 / 第 187-220 行

```c
0187:           const uint8x8_t vk8 = vld1_u8(w);
0188:           w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0189:           const uint8x8_t vi8 = vld1_u8(i8);
0190:           i8 += 8;
0191:           const int16x8_t vxk8 =
0192:               vreinterpretq_s16_u16(vsubl_u8(vk8, vkernel_zero_point));
0193:           const int16x8_t vxi8 =
0194:               vreinterpretq_s16_u16(vsubl_u8(vi8, vinput_zero_point));
0195:           vaccX0_lo =
0196:               vmlal_s16(vaccX0_lo, vget_low_s16(vxk8), vget_low_s16(vxi8));
0197:           vaccX0_hi =
0198:               vmlal_s16(vaccX0_hi, vget_high_s16(vxk8), vget_high_s16(vxi8));
0199: 
0200:           int32x4_t vacc_lo = vaddq_s32(vaccX0_lo, vaccX1_lo);
0201:           int32x4_t vacc_hi = vaddq_s32(vaccX0_hi, vaccX1_hi);
0202: 
0203:           vst1q_s32(outacc, vacc_lo);
0204:           outacc += 4;
0205:           vst1q_s32(outacc, vacc_hi);
0206:           outacc += 4;
0207:         }
0208:         if (c != 0) {
0209:           const size_t c_predecrement = 8 - c;
0210:           const int64x1_t vi_shift = vmov_n_s64(-8 * c_predecrement);
0211:           i0 -= c_predecrement;
0212:           i1 -= c_predecrement;
0213:           i2 -= c_predecrement;
0214:           i3 -= c_predecrement;
0215:           i4 -= c_predecrement;
0216:           i5 -= c_predecrement;
0217:           i6 -= c_predecrement;
0218:           i7 -= c_predecrement;
0219:           i8 -= c_predecrement;
0220: 
```

- **EN:** This block handles conditional branches and special cases. Key symbols: `vreinterpretq_s16_u16`, `vmlal_s16`, `vst1q_s32`.
- **CN:** 该代码块处理条件分支与特殊情况。关键符号：`vreinterpretq_s16_u16`, `vmlal_s16`, `vst1q_s32`。

### Lines 221-253 / 第 221-253 行

```c
0221:           const uint8x8_t vkernel_zero_point = vld1_u8(
0222:               &quantization_params->neon.kernel_zero_points[channels - c]);
0223:           int32x4_t vaccX1_lo = vld1q_s32(w);
0224:           w = (void*)((uintptr_t)w + sizeof(int32x4_t));
0225:           int32x4_t vaccX1_hi = vld1q_s32(w);
0226:           w = (void*)((uintptr_t)w + sizeof(int32x4_t));
0227: 
0228:           const uint8x8_t vk0 = vld1_u8(w);
0229:           w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0230:           const uint8x8_t vi0 = vreinterpret_u8_u64(
0231:               vshl_u64(vreinterpret_u64_u8(vld1_u8(i0)), vi_shift));
0232:           const int16x8_t vxk0 =
0233:               vreinterpretq_s16_u16(vsubl_u8(vk0, vkernel_zero_point));
0234:           const int16x8_t vxi0 =
0235:               vreinterpretq_s16_u16(vsubl_u8(vi0, vinput_zero_point));
0236:           int32x4_t vaccX0_lo =
0237:               vmull_s16(vget_low_s16(vxk0), vget_low_s16(vxi0));
0238:           int32x4_t vaccX0_hi =
0239:               vmull_s16(vget_high_s16(vxk0), vget_high_s16(vxi0));
0240: 
0241:           const uint8x8_t vk1 = vld1_u8(w);
0242:           w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0243:           const uint8x8_t vi1 = vreinterpret_u8_u64(
0244:               vshl_u64(vreinterpret_u64_u8(vld1_u8(i1)), vi_shift));
0245:           const int16x8_t vxk1 =
0246:               vreinterpretq_s16_u16(vsubl_u8(vk1, vkernel_zero_point));
0247:           const int16x8_t vxi1 =
0248:               vreinterpretq_s16_u16(vsubl_u8(vi1, vinput_zero_point));
0249:           vaccX1_lo =
0250:               vmlal_s16(vaccX1_lo, vget_low_s16(vxk1), vget_low_s16(vxi1));
0251:           vaccX1_hi =
0252:               vmlal_s16(vaccX1_hi, vget_high_s16(vxk1), vget_high_s16(vxi1));
0253: 
```

- **EN:** This block implements local helper logic for `mp8x27-neon`. Key symbols: `vshl_u64`, `vreinterpretq_s16_u16`, `vmull_s16`, `vmlal_s16`.
- **CN:** 该代码块实现与 `mp8x27-neon` 相关的局部辅助逻辑。关键符号：`vshl_u64`, `vreinterpretq_s16_u16`, `vmull_s16`, `vmlal_s16`。

### Lines 254-292 / 第 254-292 行

```c
0254:           const uint8x8_t vk2 = vld1_u8(w);
0255:           w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0256:           const uint8x8_t vi2 = vreinterpret_u8_u64(
0257:               vshl_u64(vreinterpret_u64_u8(vld1_u8(i2)), vi_shift));
0258:           const int16x8_t vxk2 =
0259:               vreinterpretq_s16_u16(vsubl_u8(vk2, vkernel_zero_point));
0260:           const int16x8_t vxi2 =
0261:               vreinterpretq_s16_u16(vsubl_u8(vi2, vinput_zero_point));
0262:           vaccX0_lo =
0263:               vmlal_s16(vaccX0_lo, vget_low_s16(vxk2), vget_low_s16(vxi2));
0264:           vaccX0_hi =
0265:               vmlal_s16(vaccX0_hi, vget_high_s16(vxk2), vget_high_s16(vxi2));
0266: 
0267:           const uint8x8_t vk3 = vld1_u8(w);
0268:           w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0269:           const uint8x8_t vi3 = vreinterpret_u8_u64(
0270:               vshl_u64(vreinterpret_u64_u8(vld1_u8(i3)), vi_shift));
0271:           const int16x8_t vxk3 =
0272:               vreinterpretq_s16_u16(vsubl_u8(vk3, vkernel_zero_point));
0273:           const int16x8_t vxi3 =
0274:               vreinterpretq_s16_u16(vsubl_u8(vi3, vinput_zero_point));
0275:           vaccX1_lo =
0276:               vmlal_s16(vaccX1_lo, vget_low_s16(vxk3), vget_low_s16(vxi3));
0277:           vaccX1_hi =
0278:               vmlal_s16(vaccX1_hi, vget_high_s16(vxk3), vget_high_s16(vxi3));
0279: 
0280:           const uint8x8_t vk4 = vld1_u8(w);
0281:           w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0282:           const uint8x8_t vi4 = vreinterpret_u8_u64(
0283:               vshl_u64(vreinterpret_u64_u8(vld1_u8(i4)), vi_shift));
0284:           const int16x8_t vxk4 =
0285:               vreinterpretq_s16_u16(vsubl_u8(vk4, vkernel_zero_point));
0286:           const int16x8_t vxi4 =
0287:               vreinterpretq_s16_u16(vsubl_u8(vi4, vinput_zero_point));
0288:           vaccX0_lo =
0289:               vmlal_s16(vaccX0_lo, vget_low_s16(vxk4), vget_low_s16(vxi4));
0290:           vaccX0_hi =
0291:               vmlal_s16(vaccX0_hi, vget_high_s16(vxk4), vget_high_s16(vxi4));
0292: 
```

- **EN:** This block implements local helper logic for `mp8x27-neon`. Key symbols: `vshl_u64`, `vreinterpretq_s16_u16`, `vmlal_s16`.
- **CN:** 该代码块实现与 `mp8x27-neon` 相关的局部辅助逻辑。关键符号：`vshl_u64`, `vreinterpretq_s16_u16`, `vmlal_s16`。

### Lines 293-331 / 第 293-331 行

```c
0293:           const uint8x8_t vk5 = vld1_u8(w);
0294:           w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0295:           const uint8x8_t vi5 = vreinterpret_u8_u64(
0296:               vshl_u64(vreinterpret_u64_u8(vld1_u8(i5)), vi_shift));
0297:           const int16x8_t vxk5 =
0298:               vreinterpretq_s16_u16(vsubl_u8(vk5, vkernel_zero_point));
0299:           const int16x8_t vxi5 =
0300:               vreinterpretq_s16_u16(vsubl_u8(vi5, vinput_zero_point));
0301:           vaccX1_lo =
0302:               vmlal_s16(vaccX1_lo, vget_low_s16(vxk5), vget_low_s16(vxi5));
0303:           vaccX1_hi =
0304:               vmlal_s16(vaccX1_hi, vget_high_s16(vxk5), vget_high_s16(vxi5));
0305: 
0306:           const uint8x8_t vk6 = vld1_u8(w);
0307:           w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0308:           const uint8x8_t vi6 = vreinterpret_u8_u64(
0309:               vshl_u64(vreinterpret_u64_u8(vld1_u8(i6)), vi_shift));
0310:           const int16x8_t vxk6 =
0311:               vreinterpretq_s16_u16(vsubl_u8(vk6, vkernel_zero_point));
0312:           const int16x8_t vxi6 =
0313:               vreinterpretq_s16_u16(vsubl_u8(vi6, vinput_zero_point));
0314:           vaccX0_lo =
0315:               vmlal_s16(vaccX0_lo, vget_low_s16(vxk6), vget_low_s16(vxi6));
0316:           vaccX0_hi =
0317:               vmlal_s16(vaccX0_hi, vget_high_s16(vxk6), vget_high_s16(vxi6));
0318: 
0319:           const uint8x8_t vk7 = vld1_u8(w);
0320:           w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0321:           const uint8x8_t vi7 = vreinterpret_u8_u64(
0322:               vshl_u64(vreinterpret_u64_u8(vld1_u8(i7)), vi_shift));
0323:           const int16x8_t vxk7 =
0324:               vreinterpretq_s16_u16(vsubl_u8(vk7, vkernel_zero_point));
0325:           const int16x8_t vxi7 =
0326:               vreinterpretq_s16_u16(vsubl_u8(vi7, vinput_zero_point));
0327:           vaccX1_lo =
0328:               vmlal_s16(vaccX1_lo, vget_low_s16(vxk7), vget_low_s16(vxi7));
0329:           vaccX1_hi =
0330:               vmlal_s16(vaccX1_hi, vget_high_s16(vxk7), vget_high_s16(vxi7));
0331: 
```

- **EN:** This block implements local helper logic for `mp8x27-neon`. Key symbols: `vshl_u64`, `vreinterpretq_s16_u16`, `vmlal_s16`.
- **CN:** 该代码块实现与 `mp8x27-neon` 相关的局部辅助逻辑。关键符号：`vshl_u64`, `vreinterpretq_s16_u16`, `vmlal_s16`。

### Lines 332-366 / 第 332-366 行

```c
0332:           const uint8x8_t vk8 = vld1_u8(w);
0333:           w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0334:           const uint8x8_t vi8 = vreinterpret_u8_u64(
0335:               vshl_u64(vreinterpret_u64_u8(vld1_u8(i8)), vi_shift));
0336:           const int16x8_t vxk8 =
0337:               vreinterpretq_s16_u16(vsubl_u8(vk8, vkernel_zero_point));
0338:           const int16x8_t vxi8 =
0339:               vreinterpretq_s16_u16(vsubl_u8(vi8, vinput_zero_point));
0340:           vaccX0_lo =
0341:               vmlal_s16(vaccX0_lo, vget_low_s16(vxk8), vget_low_s16(vxi8));
0342:           vaccX0_hi =
0343:               vmlal_s16(vaccX0_hi, vget_high_s16(vxk8), vget_high_s16(vxi8));
0344: 
0345:           int32x4_t vacc_lo = vaddq_s32(vaccX0_lo, vaccX1_lo);
0346:           int32x4_t vacc_hi = vaddq_s32(vaccX0_hi, vaccX1_hi);
0347: 
0348:           vst1q_s32(outacc, vacc_lo);
0349:           outacc += 4;
0350:           vst1q_s32(outacc, vacc_hi);
0351:           outacc += 4;
0352:         }
0353:       }
0354:       {
0355:         const uint8_t* i0 = input[9];
0356:         const uint8_t* i1 = input[10];
0357:         const uint8_t* i2 = input[11];
0358:         const uint8_t* i3 = input[12];
0359:         const uint8_t* i4 = input[13];
0360:         const uint8_t* i5 = input[14];
0361:         const uint8_t* i6 = input[15];
0362:         const uint8_t* i7 = input[16];
0363:         const uint8_t* i8 = input[17];
0364:         output = output_start;
0365:         outacc = outacc32;
0366: 
```

- **EN:** This block implements local helper logic for `mp8x27-neon`. Key symbols: `vshl_u64`, `vreinterpretq_s16_u16`, `vmlal_s16`, `vst1q_s32`.
- **CN:** 该代码块实现与 `mp8x27-neon` 相关的局部辅助逻辑。关键符号：`vshl_u64`, `vreinterpretq_s16_u16`, `vmlal_s16`, `vst1q_s32`。

### Lines 367-398 / 第 367-398 行

```c
0367:         size_t c = channels;
0368:         const uint8_t* kernel_zero_points_ptr =
0369:             quantization_params->neon.kernel_zero_points + channels;
0370:         for (; c >= 8; c -= 8) {
0371:           const uint8x8_t vkernel_zero_point =
0372:               vld1_u8(kernel_zero_points_ptr - c);
0373:           const uint8x8_t vk0 = vld1_u8(w);
0374:           w += 8;
0375:           const uint8x8_t vi0 = vld1_u8(i0);
0376:           i0 += 8;
0377:           const int16x8_t vxk0 =
0378:               vreinterpretq_s16_u16(vsubl_u8(vk0, vkernel_zero_point));
0379:           const int16x8_t vxi0 =
0380:               vreinterpretq_s16_u16(vsubl_u8(vi0, vinput_zero_point));
0381:           int32x4_t vaccX0_lo =
0382:               vmull_s16(vget_low_s16(vxk0), vget_low_s16(vxi0));
0383:           int32x4_t vaccX0_hi =
0384:               vmull_s16(vget_high_s16(vxk0), vget_high_s16(vxi0));
0385: 
0386:           const uint8x8_t vk1 = vld1_u8(w);
0387:           w += 8;
0388:           const uint8x8_t vi1 = vld1_u8(i1);
0389:           i1 += 8;
0390:           const int16x8_t vxk1 =
0391:               vreinterpretq_s16_u16(vsubl_u8(vk1, vkernel_zero_point));
0392:           const int16x8_t vxi1 =
0393:               vreinterpretq_s16_u16(vsubl_u8(vi1, vinput_zero_point));
0394:           int32x4_t vaccX1_lo =
0395:               vmull_s16(vget_low_s16(vxk1), vget_low_s16(vxi1));
0396:           int32x4_t vaccX1_hi =
0397:               vmull_s16(vget_high_s16(vxk1), vget_high_s16(vxi1));
0398: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `vld1_u8`, `vreinterpretq_s16_u16`, `vmull_s16`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`vld1_u8`, `vreinterpretq_s16_u16`, `vmull_s16`。

### Lines 399-437 / 第 399-437 行

```c
0399:           const uint8x8_t vk2 = vld1_u8(w);
0400:           w += 8;
0401:           const uint8x8_t vi2 = vld1_u8(i2);
0402:           i2 += 8;
0403:           const int16x8_t vxk2 =
0404:               vreinterpretq_s16_u16(vsubl_u8(vk2, vkernel_zero_point));
0405:           const int16x8_t vxi2 =
0406:               vreinterpretq_s16_u16(vsubl_u8(vi2, vinput_zero_point));
0407:           vaccX0_lo =
0408:               vmlal_s16(vaccX0_lo, vget_low_s16(vxk2), vget_low_s16(vxi2));
0409:           vaccX0_hi =
0410:               vmlal_s16(vaccX0_hi, vget_high_s16(vxk2), vget_high_s16(vxi2));
0411: 
0412:           const uint8x8_t vk3 = vld1_u8(w);
0413:           w += 8;
0414:           const uint8x8_t vi3 = vld1_u8(i3);
0415:           i3 += 8;
0416:           const int16x8_t vxk3 =
0417:               vreinterpretq_s16_u16(vsubl_u8(vk3, vkernel_zero_point));
0418:           const int16x8_t vxi3 =
0419:               vreinterpretq_s16_u16(vsubl_u8(vi3, vinput_zero_point));
0420:           vaccX1_lo =
0421:               vmlal_s16(vaccX1_lo, vget_low_s16(vxk3), vget_low_s16(vxi3));
0422:           vaccX1_hi =
0423:               vmlal_s16(vaccX1_hi, vget_high_s16(vxk3), vget_high_s16(vxi3));
0424: 
0425:           const uint8x8_t vk4 = vld1_u8(w);
0426:           w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0427:           const uint8x8_t vi4 = vld1_u8(i4);
0428:           i4 += 8;
0429:           const int16x8_t vxk4 =
0430:               vreinterpretq_s16_u16(vsubl_u8(vk4, vkernel_zero_point));
0431:           const int16x8_t vxi4 =
0432:               vreinterpretq_s16_u16(vsubl_u8(vi4, vinput_zero_point));
0433:           vaccX0_lo =
0434:               vmlal_s16(vaccX0_lo, vget_low_s16(vxk4), vget_low_s16(vxi4));
0435:           vaccX0_hi =
0436:               vmlal_s16(vaccX0_hi, vget_high_s16(vxk4), vget_high_s16(vxi4));
0437: 
```

- **EN:** This block implements local helper logic for `mp8x27-neon`. Key symbols: `vreinterpretq_s16_u16`, `vmlal_s16`.
- **CN:** 该代码块实现与 `mp8x27-neon` 相关的局部辅助逻辑。关键符号：`vreinterpretq_s16_u16`, `vmlal_s16`。

### Lines 438-476 / 第 438-476 行

```c
0438:           const uint8x8_t vk5 = vld1_u8(w);
0439:           w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0440:           const uint8x8_t vi5 = vld1_u8(i5);
0441:           i5 += 8;
0442:           const int16x8_t vxk5 =
0443:               vreinterpretq_s16_u16(vsubl_u8(vk5, vkernel_zero_point));
0444:           const int16x8_t vxi5 =
0445:               vreinterpretq_s16_u16(vsubl_u8(vi5, vinput_zero_point));
0446:           vaccX1_lo =
0447:               vmlal_s16(vaccX1_lo, vget_low_s16(vxk5), vget_low_s16(vxi5));
0448:           vaccX1_hi =
0449:               vmlal_s16(vaccX1_hi, vget_high_s16(vxk5), vget_high_s16(vxi5));
0450: 
0451:           const uint8x8_t vk6 = vld1_u8(w);
0452:           w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0453:           const uint8x8_t vi6 = vld1_u8(i6);
0454:           i6 += 8;
0455:           const int16x8_t vxk6 =
0456:               vreinterpretq_s16_u16(vsubl_u8(vk6, vkernel_zero_point));
0457:           const int16x8_t vxi6 =
0458:               vreinterpretq_s16_u16(vsubl_u8(vi6, vinput_zero_point));
0459:           vaccX0_lo =
0460:               vmlal_s16(vaccX0_lo, vget_low_s16(vxk6), vget_low_s16(vxi6));
0461:           vaccX0_hi =
0462:               vmlal_s16(vaccX0_hi, vget_high_s16(vxk6), vget_high_s16(vxi6));
0463: 
0464:           const uint8x8_t vk7 = vld1_u8(w);
0465:           w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0466:           const uint8x8_t vi7 = vld1_u8(i7);
0467:           i7 += 8;
0468:           const int16x8_t vxk7 =
0469:               vreinterpretq_s16_u16(vsubl_u8(vk7, vkernel_zero_point));
0470:           const int16x8_t vxi7 =
0471:               vreinterpretq_s16_u16(vsubl_u8(vi7, vinput_zero_point));
0472:           vaccX1_lo =
0473:               vmlal_s16(vaccX1_lo, vget_low_s16(vxk7), vget_low_s16(vxi7));
0474:           vaccX1_hi =
0475:               vmlal_s16(vaccX1_hi, vget_high_s16(vxk7), vget_high_s16(vxi7));
0476: 
```

- **EN:** This block implements local helper logic for `mp8x27-neon`. Key symbols: `vreinterpretq_s16_u16`, `vmlal_s16`.
- **CN:** 该代码块实现与 `mp8x27-neon` 相关的局部辅助逻辑。关键符号：`vreinterpretq_s16_u16`, `vmlal_s16`。

### Lines 477-514 / 第 477-514 行

```c
0477:           const uint8x8_t vk8 = vld1_u8(w);
0478:           w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0479:           const uint8x8_t vi8 = vld1_u8(i8);
0480:           i8 += 8;
0481:           const int16x8_t vxk8 =
0482:               vreinterpretq_s16_u16(vsubl_u8(vk8, vkernel_zero_point));
0483:           const int16x8_t vxi8 =
0484:               vreinterpretq_s16_u16(vsubl_u8(vi8, vinput_zero_point));
0485:           vaccX0_lo =
0486:               vmlal_s16(vaccX0_lo, vget_low_s16(vxk8), vget_low_s16(vxi8));
0487:           vaccX0_hi =
0488:               vmlal_s16(vaccX0_hi, vget_high_s16(vxk8), vget_high_s16(vxi8));
0489: 
0490:           int32x4_t vacc_lo = vaddq_s32(vaccX0_lo, vaccX1_lo);
0491:           int32x4_t vacc_hi = vaddq_s32(vaccX0_hi, vaccX1_hi);
0492: 
0493:           const int32x4_t vacc_lo_old = vld1q_s32(outacc);
0494:           const int32x4_t vacc_hi_old = vld1q_s32(outacc + 4);
0495:           vacc_lo = vaddq_s32(vacc_lo, vacc_lo_old);
0496:           vacc_hi = vaddq_s32(vacc_hi, vacc_hi_old);
0497:           vst1q_s32(outacc, vacc_lo);
0498:           outacc += 4;
0499:           vst1q_s32(outacc, vacc_hi);
0500:           outacc += 4;
0501:         }
0502:         if (c != 0) {
0503:           const size_t c_predecrement = 8 - c;
0504:           const int64x1_t vi_shift = vmov_n_s64(-8 * c_predecrement);
0505:           i0 -= c_predecrement;
0506:           i1 -= c_predecrement;
0507:           i2 -= c_predecrement;
0508:           i3 -= c_predecrement;
0509:           i4 -= c_predecrement;
0510:           i5 -= c_predecrement;
0511:           i6 -= c_predecrement;
0512:           i7 -= c_predecrement;
0513:           i8 -= c_predecrement;
0514: 
```

- **EN:** This block handles conditional branches and special cases. Key symbols: `vreinterpretq_s16_u16`, `vmlal_s16`, `vst1q_s32`.
- **CN:** 该代码块处理条件分支与特殊情况。关键符号：`vreinterpretq_s16_u16`, `vmlal_s16`, `vst1q_s32`。

### Lines 515-555 / 第 515-555 行

```c
0515:           const uint8x8_t vkernel_zero_point = vld1_u8(
0516:               &quantization_params->neon.kernel_zero_points[channels - c]);
0517:           const uint8x8_t vk0 = vld1_u8(w);
0518:           w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0519:           const uint8x8_t vi0 = vreinterpret_u8_u64(
0520:               vshl_u64(vreinterpret_u64_u8(vld1_u8(i0)), vi_shift));
0521:           const int16x8_t vxk0 =
0522:               vreinterpretq_s16_u16(vsubl_u8(vk0, vkernel_zero_point));
0523:           const int16x8_t vxi0 =
0524:               vreinterpretq_s16_u16(vsubl_u8(vi0, vinput_zero_point));
0525:           int32x4_t vaccX0_lo =
0526:               vmull_s16(vget_low_s16(vxk0), vget_low_s16(vxi0));
0527:           int32x4_t vaccX0_hi =
0528:               vmull_s16(vget_high_s16(vxk0), vget_high_s16(vxi0));
0529: 
0530:           const uint8x8_t vk1 = vld1_u8(w);
0531:           w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0532:           const uint8x8_t vi1 = vreinterpret_u8_u64(
0533:               vshl_u64(vreinterpret_u64_u8(vld1_u8(i1)), vi_shift));
0534:           const int16x8_t vxk1 =
0535:               vreinterpretq_s16_u16(vsubl_u8(vk1, vkernel_zero_point));
0536:           const int16x8_t vxi1 =
0537:               vreinterpretq_s16_u16(vsubl_u8(vi1, vinput_zero_point));
0538:           int32x4_t vaccX1_lo =
0539:               vmull_s16(vget_low_s16(vxk1), vget_low_s16(vxi1));
0540:           int32x4_t vaccX1_hi =
0541:               vmull_s16(vget_high_s16(vxk1), vget_high_s16(vxi1));
0542: 
0543:           const uint8x8_t vk2 = vld1_u8(w);
0544:           w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0545:           const uint8x8_t vi2 = vreinterpret_u8_u64(
0546:               vshl_u64(vreinterpret_u64_u8(vld1_u8(i2)), vi_shift));
0547:           const int16x8_t vxk2 =
0548:               vreinterpretq_s16_u16(vsubl_u8(vk2, vkernel_zero_point));
0549:           const int16x8_t vxi2 =
0550:               vreinterpretq_s16_u16(vsubl_u8(vi2, vinput_zero_point));
0551:           vaccX0_lo =
0552:               vmlal_s16(vaccX0_lo, vget_low_s16(vxk2), vget_low_s16(vxi2));
0553:           vaccX0_hi =
0554:               vmlal_s16(vaccX0_hi, vget_high_s16(vxk2), vget_high_s16(vxi2));
0555: 
```

- **EN:** This block implements local helper logic for `mp8x27-neon`. Key symbols: `vshl_u64`, `vreinterpretq_s16_u16`, `vmull_s16`, `vmlal_s16`.
- **CN:** 该代码块实现与 `mp8x27-neon` 相关的局部辅助逻辑。关键符号：`vshl_u64`, `vreinterpretq_s16_u16`, `vmull_s16`, `vmlal_s16`。

### Lines 556-594 / 第 556-594 行

```c
0556:           const uint8x8_t vk3 = vld1_u8(w);
0557:           w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0558:           const uint8x8_t vi3 = vreinterpret_u8_u64(
0559:               vshl_u64(vreinterpret_u64_u8(vld1_u8(i3)), vi_shift));
0560:           const int16x8_t vxk3 =
0561:               vreinterpretq_s16_u16(vsubl_u8(vk3, vkernel_zero_point));
0562:           const int16x8_t vxi3 =
0563:               vreinterpretq_s16_u16(vsubl_u8(vi3, vinput_zero_point));
0564:           vaccX1_lo =
0565:               vmlal_s16(vaccX1_lo, vget_low_s16(vxk3), vget_low_s16(vxi3));
0566:           vaccX1_hi =
0567:               vmlal_s16(vaccX1_hi, vget_high_s16(vxk3), vget_high_s16(vxi3));
0568: 
0569:           const uint8x8_t vk4 = vld1_u8(w);
0570:           w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0571:           const uint8x8_t vi4 = vreinterpret_u8_u64(
0572:               vshl_u64(vreinterpret_u64_u8(vld1_u8(i4)), vi_shift));
0573:           const int16x8_t vxk4 =
0574:               vreinterpretq_s16_u16(vsubl_u8(vk4, vkernel_zero_point));
0575:           const int16x8_t vxi4 =
0576:               vreinterpretq_s16_u16(vsubl_u8(vi4, vinput_zero_point));
0577:           vaccX0_lo =
0578:               vmlal_s16(vaccX0_lo, vget_low_s16(vxk4), vget_low_s16(vxi4));
0579:           vaccX0_hi =
0580:               vmlal_s16(vaccX0_hi, vget_high_s16(vxk4), vget_high_s16(vxi4));
0581: 
0582:           const uint8x8_t vk5 = vld1_u8(w);
0583:           w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0584:           const uint8x8_t vi5 = vreinterpret_u8_u64(
0585:               vshl_u64(vreinterpret_u64_u8(vld1_u8(i5)), vi_shift));
0586:           const int16x8_t vxk5 =
0587:               vreinterpretq_s16_u16(vsubl_u8(vk5, vkernel_zero_point));
0588:           const int16x8_t vxi5 =
0589:               vreinterpretq_s16_u16(vsubl_u8(vi5, vinput_zero_point));
0590:           vaccX1_lo =
0591:               vmlal_s16(vaccX1_lo, vget_low_s16(vxk5), vget_low_s16(vxi5));
0592:           vaccX1_hi =
0593:               vmlal_s16(vaccX1_hi, vget_high_s16(vxk5), vget_high_s16(vxi5));
0594: 
```

- **EN:** This block implements local helper logic for `mp8x27-neon`. Key symbols: `vshl_u64`, `vreinterpretq_s16_u16`, `vmlal_s16`.
- **CN:** 该代码块实现与 `mp8x27-neon` 相关的局部辅助逻辑。关键符号：`vshl_u64`, `vreinterpretq_s16_u16`, `vmlal_s16`。

### Lines 595-633 / 第 595-633 行

```c
0595:           const uint8x8_t vk6 = vld1_u8(w);
0596:           w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0597:           const uint8x8_t vi6 = vreinterpret_u8_u64(
0598:               vshl_u64(vreinterpret_u64_u8(vld1_u8(i6)), vi_shift));
0599:           const int16x8_t vxk6 =
0600:               vreinterpretq_s16_u16(vsubl_u8(vk6, vkernel_zero_point));
0601:           const int16x8_t vxi6 =
0602:               vreinterpretq_s16_u16(vsubl_u8(vi6, vinput_zero_point));
0603:           vaccX0_lo =
0604:               vmlal_s16(vaccX0_lo, vget_low_s16(vxk6), vget_low_s16(vxi6));
0605:           vaccX0_hi =
0606:               vmlal_s16(vaccX0_hi, vget_high_s16(vxk6), vget_high_s16(vxi6));
0607: 
0608:           const uint8x8_t vk7 = vld1_u8(w);
0609:           w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0610:           const uint8x8_t vi7 = vreinterpret_u8_u64(
0611:               vshl_u64(vreinterpret_u64_u8(vld1_u8(i7)), vi_shift));
0612:           const int16x8_t vxk7 =
0613:               vreinterpretq_s16_u16(vsubl_u8(vk7, vkernel_zero_point));
0614:           const int16x8_t vxi7 =
0615:               vreinterpretq_s16_u16(vsubl_u8(vi7, vinput_zero_point));
0616:           vaccX1_lo =
0617:               vmlal_s16(vaccX1_lo, vget_low_s16(vxk7), vget_low_s16(vxi7));
0618:           vaccX1_hi =
0619:               vmlal_s16(vaccX1_hi, vget_high_s16(vxk7), vget_high_s16(vxi7));
0620: 
0621:           const uint8x8_t vk8 = vld1_u8(w);
0622:           w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0623:           const uint8x8_t vi8 = vreinterpret_u8_u64(
0624:               vshl_u64(vreinterpret_u64_u8(vld1_u8(i8)), vi_shift));
0625:           const int16x8_t vxk8 =
0626:               vreinterpretq_s16_u16(vsubl_u8(vk8, vkernel_zero_point));
0627:           const int16x8_t vxi8 =
0628:               vreinterpretq_s16_u16(vsubl_u8(vi8, vinput_zero_point));
0629:           vaccX0_lo =
0630:               vmlal_s16(vaccX0_lo, vget_low_s16(vxk8), vget_low_s16(vxi8));
0631:           vaccX0_hi =
0632:               vmlal_s16(vaccX0_hi, vget_high_s16(vxk8), vget_high_s16(vxi8));
0633: 
```

- **EN:** This block implements local helper logic for `mp8x27-neon`. Key symbols: `vshl_u64`, `vreinterpretq_s16_u16`, `vmlal_s16`.
- **CN:** 该代码块实现与 `mp8x27-neon` 相关的局部辅助逻辑。关键符号：`vshl_u64`, `vreinterpretq_s16_u16`, `vmlal_s16`。

### Lines 634-667 / 第 634-667 行

```c
0634:           int32x4_t vacc_lo = vaddq_s32(vaccX0_lo, vaccX1_lo);
0635:           int32x4_t vacc_hi = vaddq_s32(vaccX0_hi, vaccX1_hi);
0636: 
0637:           const int32x4_t vacc_lo_old = vld1q_s32(outacc);
0638:           const int32x4_t vacc_hi_old = vld1q_s32(outacc + 4);
0639:           vacc_lo = vaddq_s32(vacc_lo, vacc_lo_old);
0640:           vacc_hi = vaddq_s32(vacc_hi, vacc_hi_old);
0641:           vst1q_s32(outacc, vacc_lo);
0642:           outacc += 4;
0643:           vst1q_s32(outacc, vacc_hi);
0644:           outacc += 4;
0645:         }
0646:       }
0647: 
0648:       {
0649:         const uint8_t* i0 = input[18];
0650:         const uint8_t* i1 = input[19];
0651:         const uint8_t* i2 = input[20];
0652:         const uint8_t* i3 = input[21];
0653:         const uint8_t* i4 = input[22];
0654:         const uint8_t* i5 = input[23];
0655:         const uint8_t* i6 = input[24];
0656:         const uint8_t* i7 = input[25];
0657:         const uint8_t* i8 = input[26];
0658:         input = (const uint8_t**)((uintptr_t)input + input_col_stride);
0659:         output = output_start;
0660:         outacc = outacc32;
0661: 
0662:         size_t c = channels;
0663:         const uint8_t* kernel_zero_points_ptr =
0664:             quantization_params->neon.kernel_zero_points + channels;
0665:         const float* requantization_scales_ptr =
0666:             quantization_params->neon.requantization_scales + channels;
0667:         for (; c >= 8; c -= 8) {
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `vst1q_s32`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`vst1q_s32`。

### Lines 668-708 / 第 668-708 行

```c
0668:           const uint8x8_t vkernel_zero_point =
0669:               vld1_u8(kernel_zero_points_ptr - c);
0670:           const uint8x8_t vk0 = vld1_u8(w);
0671:           w += 8;
0672:           const uint8x8_t vi0 = vld1_u8(i0);
0673:           i0 += 8;
0674:           const int16x8_t vxk0 =
0675:               vreinterpretq_s16_u16(vsubl_u8(vk0, vkernel_zero_point));
0676:           const int16x8_t vxi0 =
0677:               vreinterpretq_s16_u16(vsubl_u8(vi0, vinput_zero_point));
0678:           int32x4_t vaccX0_lo =
0679:               vmull_s16(vget_low_s16(vxk0), vget_low_s16(vxi0));
0680:           int32x4_t vaccX0_hi =
0681:               vmull_s16(vget_high_s16(vxk0), vget_high_s16(vxi0));
0682: 
0683:           const uint8x8_t vk1 = vld1_u8(w);
0684:           w += 8;
0685:           const uint8x8_t vi1 = vld1_u8(i1);
0686:           i1 += 8;
0687:           const int16x8_t vxk1 =
0688:               vreinterpretq_s16_u16(vsubl_u8(vk1, vkernel_zero_point));
0689:           const int16x8_t vxi1 =
0690:               vreinterpretq_s16_u16(vsubl_u8(vi1, vinput_zero_point));
0691:           int32x4_t vaccX1_lo =
0692:               vmull_s16(vget_low_s16(vxk1), vget_low_s16(vxi1));
0693:           int32x4_t vaccX1_hi =
0694:               vmull_s16(vget_high_s16(vxk1), vget_high_s16(vxi1));
0695: 
0696:           const uint8x8_t vk2 = vld1_u8(w);
0697:           w += 8;
0698:           const uint8x8_t vi2 = vld1_u8(i2);
0699:           i2 += 8;
0700:           const int16x8_t vxk2 =
0701:               vreinterpretq_s16_u16(vsubl_u8(vk2, vkernel_zero_point));
0702:           const int16x8_t vxi2 =
0703:               vreinterpretq_s16_u16(vsubl_u8(vi2, vinput_zero_point));
0704:           vaccX0_lo =
0705:               vmlal_s16(vaccX0_lo, vget_low_s16(vxk2), vget_low_s16(vxi2));
0706:           vaccX0_hi =
0707:               vmlal_s16(vaccX0_hi, vget_high_s16(vxk2), vget_high_s16(vxi2));
0708: 
```

- **EN:** This block implements local helper logic for `mp8x27-neon`. Key symbols: `vld1_u8`, `vreinterpretq_s16_u16`, `vmull_s16`, `vmlal_s16`.
- **CN:** 该代码块实现与 `mp8x27-neon` 相关的局部辅助逻辑。关键符号：`vld1_u8`, `vreinterpretq_s16_u16`, `vmull_s16`, `vmlal_s16`。

### Lines 709-747 / 第 709-747 行

```c
0709:           const uint8x8_t vk3 = vld1_u8(w);
0710:           w += 8;
0711:           const uint8x8_t vi3 = vld1_u8(i3);
0712:           i3 += 8;
0713:           const int16x8_t vxk3 =
0714:               vreinterpretq_s16_u16(vsubl_u8(vk3, vkernel_zero_point));
0715:           const int16x8_t vxi3 =
0716:               vreinterpretq_s16_u16(vsubl_u8(vi3, vinput_zero_point));
0717:           vaccX1_lo =
0718:               vmlal_s16(vaccX1_lo, vget_low_s16(vxk3), vget_low_s16(vxi3));
0719:           vaccX1_hi =
0720:               vmlal_s16(vaccX1_hi, vget_high_s16(vxk3), vget_high_s16(vxi3));
0721: 
0722:           const uint8x8_t vk4 = vld1_u8(w);
0723:           w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0724:           const uint8x8_t vi4 = vld1_u8(i4);
0725:           i4 += 8;
0726:           const int16x8_t vxk4 =
0727:               vreinterpretq_s16_u16(vsubl_u8(vk4, vkernel_zero_point));
0728:           const int16x8_t vxi4 =
0729:               vreinterpretq_s16_u16(vsubl_u8(vi4, vinput_zero_point));
0730:           vaccX0_lo =
0731:               vmlal_s16(vaccX0_lo, vget_low_s16(vxk4), vget_low_s16(vxi4));
0732:           vaccX0_hi =
0733:               vmlal_s16(vaccX0_hi, vget_high_s16(vxk4), vget_high_s16(vxi4));
0734: 
0735:           const uint8x8_t vk5 = vld1_u8(w);
0736:           w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0737:           const uint8x8_t vi5 = vld1_u8(i5);
0738:           i5 += 8;
0739:           const int16x8_t vxk5 =
0740:               vreinterpretq_s16_u16(vsubl_u8(vk5, vkernel_zero_point));
0741:           const int16x8_t vxi5 =
0742:               vreinterpretq_s16_u16(vsubl_u8(vi5, vinput_zero_point));
0743:           vaccX1_lo =
0744:               vmlal_s16(vaccX1_lo, vget_low_s16(vxk5), vget_low_s16(vxi5));
0745:           vaccX1_hi =
0746:               vmlal_s16(vaccX1_hi, vget_high_s16(vxk5), vget_high_s16(vxi5));
0747: 
```

- **EN:** This block implements local helper logic for `mp8x27-neon`. Key symbols: `vreinterpretq_s16_u16`, `vmlal_s16`.
- **CN:** 该代码块实现与 `mp8x27-neon` 相关的局部辅助逻辑。关键符号：`vreinterpretq_s16_u16`, `vmlal_s16`。

### Lines 748-786 / 第 748-786 行

```c
0748:           const uint8x8_t vk6 = vld1_u8(w);
0749:           w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0750:           const uint8x8_t vi6 = vld1_u8(i6);
0751:           i6 += 8;
0752:           const int16x8_t vxk6 =
0753:               vreinterpretq_s16_u16(vsubl_u8(vk6, vkernel_zero_point));
0754:           const int16x8_t vxi6 =
0755:               vreinterpretq_s16_u16(vsubl_u8(vi6, vinput_zero_point));
0756:           vaccX0_lo =
0757:               vmlal_s16(vaccX0_lo, vget_low_s16(vxk6), vget_low_s16(vxi6));
0758:           vaccX0_hi =
0759:               vmlal_s16(vaccX0_hi, vget_high_s16(vxk6), vget_high_s16(vxi6));
0760: 
0761:           const uint8x8_t vk7 = vld1_u8(w);
0762:           w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0763:           const uint8x8_t vi7 = vld1_u8(i7);
0764:           i7 += 8;
0765:           const int16x8_t vxk7 =
0766:               vreinterpretq_s16_u16(vsubl_u8(vk7, vkernel_zero_point));
0767:           const int16x8_t vxi7 =
0768:               vreinterpretq_s16_u16(vsubl_u8(vi7, vinput_zero_point));
0769:           vaccX1_lo =
0770:               vmlal_s16(vaccX1_lo, vget_low_s16(vxk7), vget_low_s16(vxi7));
0771:           vaccX1_hi =
0772:               vmlal_s16(vaccX1_hi, vget_high_s16(vxk7), vget_high_s16(vxi7));
0773: 
0774:           const uint8x8_t vk8 = vld1_u8(w);
0775:           w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0776:           const uint8x8_t vi8 = vld1_u8(i8);
0777:           i8 += 8;
0778:           const int16x8_t vxk8 =
0779:               vreinterpretq_s16_u16(vsubl_u8(vk8, vkernel_zero_point));
0780:           const int16x8_t vxi8 =
0781:               vreinterpretq_s16_u16(vsubl_u8(vi8, vinput_zero_point));
0782:           vaccX0_lo =
0783:               vmlal_s16(vaccX0_lo, vget_low_s16(vxk8), vget_low_s16(vxi8));
0784:           vaccX0_hi =
0785:               vmlal_s16(vaccX0_hi, vget_high_s16(vxk8), vget_high_s16(vxi8));
0786: 
```

- **EN:** This block implements local helper logic for `mp8x27-neon`. Key symbols: `vreinterpretq_s16_u16`, `vmlal_s16`.
- **CN:** 该代码块实现与 `mp8x27-neon` 相关的局部辅助逻辑。关键符号：`vreinterpretq_s16_u16`, `vmlal_s16`。

### Lines 787-831 / 第 787-831 行

```c
0787:           int32x4_t vacc_lo = vaddq_s32(vaccX0_lo, vaccX1_lo);
0788:           int32x4_t vacc_hi = vaddq_s32(vaccX0_hi, vaccX1_hi);
0789: 
0790:           const int32x4_t vacc_lo_old = vld1q_s32(outacc);
0791:           outacc += 4;
0792:           const int32x4_t vacc_hi_old = vld1q_s32(outacc);
0793:           outacc += 4;
0794:           vacc_lo = vaddq_s32(vacc_lo, vacc_lo_old);
0795:           vacc_hi = vaddq_s32(vacc_hi, vacc_hi_old);
0796: 
0797:           const float32x4_t requantization_scale_v_lo =
0798:               vld1q_f32(requantization_scales_ptr - c);
0799:           const float32x4_t requantization_scale_v_hi =
0800:               vld1q_f32(requantization_scales_ptr - c + 4);
0801: 
0802:           const float32x4_t vacc_lo_f =
0803:               vmulq_f32(vcvtq_f32_s32(vacc_lo), requantization_scale_v_lo);
0804:           const float32x4_t vacc_hi_f =
0805:               vmulq_f32(vcvtq_f32_s32(vacc_hi), requantization_scale_v_hi);
0806: 
0807: #ifdef __aarch64__
0808:           vacc_lo = vcvtnq_s32_f32(vacc_lo_f);
0809:           vacc_hi = vcvtnq_s32_f32(vacc_hi_f);
0810: 
0811:           const int16x8_t vacc = vqaddq_s16(
0812:               vqmovn_high_s32(vqmovn_s32(vacc_lo), vacc_hi),
0813:               voutput_zero_point);
0814: 
0815:           uint8x8_t vout = vqmovun_s16(vacc);
0816:           vout = vmax_u8(vout, voutput_min);
0817:           vout = vmin_u8(vout, voutput_max);
0818: #else
0819:           const float32x4_t vacc_lo_f_clamped =
0820:               vminq_f32(vmaxq_f32(vacc_lo_f, vfmin), vfmax);
0821:           const float32x4_t vacc_hi_f_clamped =
0822:               vminq_f32(vmaxq_f32(vacc_hi_f, vfmin), vfmax);
0823:           vacc_lo = vsubq_s32(
0824:               vreinterpretq_s32_f32(vaddq_f32(vacc_lo_f_clamped, vfmagic)),
0825:               vimagic);
0826:           vacc_hi = vsubq_s32(
0827:               vreinterpretq_s32_f32(vaddq_f32(vacc_hi_f_clamped, vfmagic)),
0828:               vimagic);
0829:           const int16x8_t vacc =
0830:               vcombine_s16(vqmovn_s32(vacc_lo), vqmovn_s32(vacc_hi));
0831: 
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends. Key symbols: `vld1q_f32`, `vmulq_f32`, `vqmovn_high_s32`, `vminq_f32`, `vreinterpretq_s32_f32`, `vcombine_s16`.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端。关键符号：`vld1q_f32`, `vmulq_f32`, `vqmovn_high_s32`, `vminq_f32`, `vreinterpretq_s32_f32`, `vcombine_s16`。

### Lines 832-865 / 第 832-865 行

```c
0832:           uint8x8_t vout = vqmovun_s16(vacc);
0833: #endif
0834: 
0835:           vst1_u8(output, vout);
0836:           output += 8;
0837:         }
0838:         if (c != 0) {
0839:           const size_t c_predecrement = 8 - c;
0840:           const int64x1_t vi_shift = vmov_n_s64(-8 * c_predecrement);
0841:           i0 -= c_predecrement;
0842:           i1 -= c_predecrement;
0843:           i2 -= c_predecrement;
0844:           i3 -= c_predecrement;
0845:           i4 -= c_predecrement;
0846:           i5 -= c_predecrement;
0847:           i6 -= c_predecrement;
0848:           i7 -= c_predecrement;
0849:           i8 -= c_predecrement;
0850: 
0851:           const uint8x8_t vkernel_zero_point = vld1_u8(
0852:               &quantization_params->neon.kernel_zero_points[channels - c]);
0853:           const uint8x8_t vk0 = vld1_u8(w);
0854:           w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0855:           const uint8x8_t vi0 = vreinterpret_u8_u64(
0856:               vshl_u64(vreinterpret_u64_u8(vld1_u8(i0)), vi_shift));
0857:           const int16x8_t vxk0 =
0858:               vreinterpretq_s16_u16(vsubl_u8(vk0, vkernel_zero_point));
0859:           const int16x8_t vxi0 =
0860:               vreinterpretq_s16_u16(vsubl_u8(vi0, vinput_zero_point));
0861:           int32x4_t vaccX0_lo =
0862:               vmull_s16(vget_low_s16(vxk0), vget_low_s16(vxi0));
0863:           int32x4_t vaccX0_hi =
0864:               vmull_s16(vget_high_s16(vxk0), vget_high_s16(vxi0));
0865: 
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends; handles conditional branches and special cases. Key symbols: `vst1_u8`, `vshl_u64`, `vreinterpretq_s16_u16`, `vmull_s16`.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端；处理条件分支与特殊情况。关键符号：`vst1_u8`, `vshl_u64`, `vreinterpretq_s16_u16`, `vmull_s16`。

### Lines 866-904 / 第 866-904 行

```c
0866:           const uint8x8_t vk1 = vld1_u8(w);
0867:           w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0868:           const uint8x8_t vi1 = vreinterpret_u8_u64(
0869:               vshl_u64(vreinterpret_u64_u8(vld1_u8(i1)), vi_shift));
0870:           const int16x8_t vxk1 =
0871:               vreinterpretq_s16_u16(vsubl_u8(vk1, vkernel_zero_point));
0872:           const int16x8_t vxi1 =
0873:               vreinterpretq_s16_u16(vsubl_u8(vi1, vinput_zero_point));
0874:           int32x4_t vaccX1_lo =
0875:               vmull_s16(vget_low_s16(vxk1), vget_low_s16(vxi1));
0876:           int32x4_t vaccX1_hi =
0877:               vmull_s16(vget_high_s16(vxk1), vget_high_s16(vxi1));
0878: 
0879:           const uint8x8_t vk2 = vld1_u8(w);
0880:           w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0881:           const uint8x8_t vi2 = vreinterpret_u8_u64(
0882:               vshl_u64(vreinterpret_u64_u8(vld1_u8(i2)), vi_shift));
0883:           const int16x8_t vxk2 =
0884:               vreinterpretq_s16_u16(vsubl_u8(vk2, vkernel_zero_point));
0885:           const int16x8_t vxi2 =
0886:               vreinterpretq_s16_u16(vsubl_u8(vi2, vinput_zero_point));
0887:           vaccX0_lo =
0888:               vmlal_s16(vaccX0_lo, vget_low_s16(vxk2), vget_low_s16(vxi2));
0889:           vaccX0_hi =
0890:               vmlal_s16(vaccX0_hi, vget_high_s16(vxk2), vget_high_s16(vxi2));
0891: 
0892:           const uint8x8_t vk3 = vld1_u8(w);
0893:           w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0894:           const uint8x8_t vi3 = vreinterpret_u8_u64(
0895:               vshl_u64(vreinterpret_u64_u8(vld1_u8(i3)), vi_shift));
0896:           const int16x8_t vxk3 =
0897:               vreinterpretq_s16_u16(vsubl_u8(vk3, vkernel_zero_point));
0898:           const int16x8_t vxi3 =
0899:               vreinterpretq_s16_u16(vsubl_u8(vi3, vinput_zero_point));
0900:           vaccX1_lo =
0901:               vmlal_s16(vaccX1_lo, vget_low_s16(vxk3), vget_low_s16(vxi3));
0902:           vaccX1_hi =
0903:               vmlal_s16(vaccX1_hi, vget_high_s16(vxk3), vget_high_s16(vxi3));
0904: 
```

- **EN:** This block implements local helper logic for `mp8x27-neon`. Key symbols: `vshl_u64`, `vreinterpretq_s16_u16`, `vmull_s16`, `vmlal_s16`.
- **CN:** 该代码块实现与 `mp8x27-neon` 相关的局部辅助逻辑。关键符号：`vshl_u64`, `vreinterpretq_s16_u16`, `vmull_s16`, `vmlal_s16`。

### Lines 905-943 / 第 905-943 行

```c
0905:           const uint8x8_t vk4 = vld1_u8(w);
0906:           w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0907:           const uint8x8_t vi4 = vreinterpret_u8_u64(
0908:               vshl_u64(vreinterpret_u64_u8(vld1_u8(i4)), vi_shift));
0909:           const int16x8_t vxk4 =
0910:               vreinterpretq_s16_u16(vsubl_u8(vk4, vkernel_zero_point));
0911:           const int16x8_t vxi4 =
0912:               vreinterpretq_s16_u16(vsubl_u8(vi4, vinput_zero_point));
0913:           vaccX0_lo =
0914:               vmlal_s16(vaccX0_lo, vget_low_s16(vxk4), vget_low_s16(vxi4));
0915:           vaccX0_hi =
0916:               vmlal_s16(vaccX0_hi, vget_high_s16(vxk4), vget_high_s16(vxi4));
0917: 
0918:           const uint8x8_t vk5 = vld1_u8(w);
0919:           w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0920:           const uint8x8_t vi5 = vreinterpret_u8_u64(
0921:               vshl_u64(vreinterpret_u64_u8(vld1_u8(i5)), vi_shift));
0922:           const int16x8_t vxk5 =
0923:               vreinterpretq_s16_u16(vsubl_u8(vk5, vkernel_zero_point));
0924:           const int16x8_t vxi5 =
0925:               vreinterpretq_s16_u16(vsubl_u8(vi5, vinput_zero_point));
0926:           vaccX1_lo =
0927:               vmlal_s16(vaccX1_lo, vget_low_s16(vxk5), vget_low_s16(vxi5));
0928:           vaccX1_hi =
0929:               vmlal_s16(vaccX1_hi, vget_high_s16(vxk5), vget_high_s16(vxi5));
0930: 
0931:           const uint8x8_t vk6 = vld1_u8(w);
0932:           w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0933:           const uint8x8_t vi6 = vreinterpret_u8_u64(
0934:               vshl_u64(vreinterpret_u64_u8(vld1_u8(i6)), vi_shift));
0935:           const int16x8_t vxk6 =
0936:               vreinterpretq_s16_u16(vsubl_u8(vk6, vkernel_zero_point));
0937:           const int16x8_t vxi6 =
0938:               vreinterpretq_s16_u16(vsubl_u8(vi6, vinput_zero_point));
0939:           vaccX0_lo =
0940:               vmlal_s16(vaccX0_lo, vget_low_s16(vxk6), vget_low_s16(vxi6));
0941:           vaccX0_hi =
0942:               vmlal_s16(vaccX0_hi, vget_high_s16(vxk6), vget_high_s16(vxi6));
0943: 
```

- **EN:** This block implements local helper logic for `mp8x27-neon`. Key symbols: `vshl_u64`, `vreinterpretq_s16_u16`, `vmlal_s16`.
- **CN:** 该代码块实现与 `mp8x27-neon` 相关的局部辅助逻辑。关键符号：`vshl_u64`, `vreinterpretq_s16_u16`, `vmlal_s16`。

### Lines 944-977 / 第 944-977 行

```c
0944:           const uint8x8_t vk7 = vld1_u8(w);
0945:           w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0946:           const uint8x8_t vi7 = vreinterpret_u8_u64(
0947:               vshl_u64(vreinterpret_u64_u8(vld1_u8(i7)), vi_shift));
0948:           const int16x8_t vxk7 =
0949:               vreinterpretq_s16_u16(vsubl_u8(vk7, vkernel_zero_point));
0950:           const int16x8_t vxi7 =
0951:               vreinterpretq_s16_u16(vsubl_u8(vi7, vinput_zero_point));
0952:           vaccX1_lo =
0953:               vmlal_s16(vaccX1_lo, vget_low_s16(vxk7), vget_low_s16(vxi7));
0954:           vaccX1_hi =
0955:               vmlal_s16(vaccX1_hi, vget_high_s16(vxk7), vget_high_s16(vxi7));
0956: 
0957:           const uint8x8_t vk8 = vld1_u8(w);
0958:           w = (void*)((uintptr_t)w + sizeof(uint8x8_t));
0959:           const uint8x8_t vi8 = vreinterpret_u8_u64(
0960:               vshl_u64(vreinterpret_u64_u8(vld1_u8(i8)), vi_shift));
0961:           const int16x8_t vxk8 =
0962:               vreinterpretq_s16_u16(vsubl_u8(vk8, vkernel_zero_point));
0963:           const int16x8_t vxi8 =
0964:               vreinterpretq_s16_u16(vsubl_u8(vi8, vinput_zero_point));
0965:           vaccX0_lo =
0966:               vmlal_s16(vaccX0_lo, vget_low_s16(vxk8), vget_low_s16(vxi8));
0967:           vaccX0_hi =
0968:               vmlal_s16(vaccX0_hi, vget_high_s16(vxk8), vget_high_s16(vxi8));
0969: 
0970:           int32x4_t vacc_lo = vaddq_s32(vaccX0_lo, vaccX1_lo);
0971:           int32x4_t vacc_hi = vaddq_s32(vaccX0_hi, vaccX1_hi);
0972: 
0973:           const int32x4_t vacc_lo_old = vld1q_s32(outacc);
0974:           const int32x4_t vacc_hi_old = vld1q_s32(outacc + 4);
0975:           vacc_lo = vaddq_s32(vacc_lo, vacc_lo_old);
0976:           vacc_hi = vaddq_s32(vacc_hi, vacc_hi_old);
0977: 
```

- **EN:** This block implements local helper logic for `mp8x27-neon`. Key symbols: `vshl_u64`, `vreinterpretq_s16_u16`, `vmlal_s16`.
- **CN:** 该代码块实现与 `mp8x27-neon` 相关的局部辅助逻辑。关键符号：`vshl_u64`, `vreinterpretq_s16_u16`, `vmlal_s16`。

### Lines 978-1013 / 第 978-1013 行

```c
0978:           const float32x4_t requantization_scale_v_lo = vld1q_f32(
0979:               &quantization_params->neon.requantization_scales[channels - c]);
0980:           const float32x4_t requantization_scale_v_hi =
0981:               vld1q_f32(&quantization_params->neon
0982:                              .requantization_scales[channels - c + 4]);
0983: 
0984:           const float32x4_t vacc_lo_f =
0985:               vmulq_f32(vcvtq_f32_s32(vacc_lo), requantization_scale_v_lo);
0986:           const float32x4_t vacc_hi_f =
0987:               vmulq_f32(vcvtq_f32_s32(vacc_hi), requantization_scale_v_hi);
0988: 
0989: #ifdef __aarch64__
0990:           vacc_lo = vcvtnq_s32_f32(vacc_lo_f);
0991:           vacc_hi = vcvtnq_s32_f32(vacc_hi_f);
0992: 
0993:           const int16x8_t vacc = vqaddq_s16(
0994:               vqmovn_high_s32(vqmovn_s32(vacc_lo), vacc_hi),
0995:               voutput_zero_point);
0996: 
0997:           uint8x8_t vout = vqmovun_s16(vacc);
0998:           vout = vmax_u8(vout, voutput_min);
0999:           vout = vmin_u8(vout, voutput_max);
1000: #else
1001:           const float32x4_t vacc_lo_f_clamped =
1002:               vminq_f32(vmaxq_f32(vacc_lo_f, vfmin), vfmax);
1003:           const float32x4_t vacc_hi_f_clamped =
1004:               vminq_f32(vmaxq_f32(vacc_hi_f, vfmin), vfmax);
1005:           vacc_lo = vsubq_s32(
1006:               vreinterpretq_s32_f32(vaddq_f32(vacc_lo_f_clamped, vfmagic)),
1007:               vimagic);
1008:           vacc_hi = vsubq_s32(
1009:               vreinterpretq_s32_f32(vaddq_f32(vacc_hi_f_clamped, vfmagic)),
1010:               vimagic);
1011:           const int16x8_t vacc =
1012:               vcombine_s16(vqmovn_s32(vacc_lo), vqmovn_s32(vacc_hi));
1013: 
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends. Key symbols: `vld1q_f32`, `vmulq_f32`, `vqmovn_high_s32`, `vminq_f32`, `vreinterpretq_s32_f32`, `vcombine_s16`.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端。关键符号：`vld1q_f32`, `vmulq_f32`, `vqmovn_high_s32`, `vminq_f32`, `vreinterpretq_s32_f32`, `vcombine_s16`。

### Lines 1014-1044 / 第 1014-1044 行

```c
1014:           uint8x8_t vout = vqmovun_s16(vacc);
1015: #endif
1016: 
1017:           if (c & 4) {
1018:             vst1_lane_u32(
1019:                 __builtin_assume_aligned(output, 1),
1020:                 vreinterpret_u32_u8(vout),
1021:                 0);
1022:             output += 4;
1023:             vout = vext_u8(vout, vout, 4);
1024:           }
1025:           if (c & 2) {
1026:             vst1_lane_u16(
1027:                 __builtin_assume_aligned(output, 1),
1028:                 vreinterpret_u16_u8(vout),
1029:                 0);
1030:             output += 2;
1031:             vout = vext_u8(vout, vout, 2);
1032:           }
1033:           if (c & 1) {
1034:             vst1_lane_u8(__builtin_assume_aligned(output, 1), vout, 0);
1035:             output++;
1036:           }
1037:         }
1038:       }
1039: 
1040:       output = (uint8_t*)((uintptr_t)output + output_increment);
1041:     }
1042:     input = (const uint8_t**)((uintptr_t)input_row_start + input_row_stride);
1043:   }
1044: }
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends; handles conditional branches and special cases. Key symbols: `vst1_lane_u32`, `vst1_lane_u16`, `vst1_lane_u8`.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端；处理条件分支与特殊情况。关键符号：`vst1_lane_u32`, `vst1_lane_u16`, `vst1_lane_u8`。


## Key Concepts / 关键概念
- **ATen native operator logic** — ATen 原生算子逻辑
- **GPU kernel/runtime interaction** — GPU 内核/运行时交互
- **Mixed-precision/autocast logic** — 混合精度/自动混精逻辑
- **Parallel scan/reduction behavior** — 并行扫描/归约行为
- **Core symbols: pytorch_q8dwconv_ukernel_mp8x27__neon, vld1_dup_u8, vld1q_dup_s16, vld1_u8, vreinterpretq_s16_u16, vmull_s16, vmlal_s16, vst1q_s32** — 核心符号：pytorch_q8dwconv_ukernel_mp8x27__neon、vld1_dup_u8、vld1q_dup_s16、vld1_u8、vreinterpretq_s16_u16、vmull_s16、vmlal_s16、vst1q_s32

## Dependencies / 依赖关系
- **Internal includes / 内部头文件**: 无
- **External includes / 外部头文件**: `arm_neon.h`, `qnnpack/q8dwconv.h`
- **Namespaces / 命名空间**: 无
- **Representative symbols / 代表性符号**: `pytorch_q8dwconv_ukernel_mp8x27__neon`, `vld1_dup_u8`, `vld1q_dup_s16`, `vld1_u8`, `vreinterpretq_s16_u16`, `vmull_s16`, `vmlal_s16`, `vst1q_s32`, `vshl_u64`, `vld1q_f32`, `vmulq_f32`, `vqmovn_high_s32`, `...`
