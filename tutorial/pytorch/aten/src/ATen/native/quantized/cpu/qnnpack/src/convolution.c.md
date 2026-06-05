# convolution.c — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `aten/src/ATen/native/quantized/cpu/qnnpack/src/convolution.c`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements ATen native operators, checks, helper kernels, and backend dispatch behavior. This specific file centers on `convolution.c`. The file header highlights: "All rights reserved. This source code is licensed under the BSD-style license found in the LICENSE file in the root directory of this source tree.."
- **Purpose (CN)**: 实现 ATen 原生算子、运行时检查、辅助内核以及后端分发行为。 该文件具体围绕 `convolution.c` 展开。 文件头部注释也概括了其核心职责。

## Line-by-Line Analysis / 逐行分析
### Lines 1-35 / 第 1-35 行

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
0010: #include <math.h>
0011: #include <stdbool.h>
0012: #include <stddef.h>
0013: #include <stdint.h>
0014: #include <stdlib.h>
0015: #include <string.h>
0016: 
0017: #include <fxdiv.h>
0018: 
0019: #include <pytorch_qnnpack.h>
0020: #include <qnnpack/common.h>
0021: #include <qnnpack/indirection.h>
0022: #include <qnnpack/log.h>
0023: #include <qnnpack/math.h>
0024: #include <qnnpack/operator.h>
0025: #include <qnnpack/pack.h>
0026: #include <qnnpack/params.h>
0027: 
0028: static inline size_t compute_output_dimension(
0029:     size_t padded_input_dimension,
0030:     size_t kernel_dimension,
0031:     size_t dilation_dimension,
0032:     size_t subsampling_dimension) {
0033:   const size_t effective_kernel_dimension =
0034:       (kernel_dimension - 1) * dilation_dimension + 1;
0035:   return (padded_input_dimension - effective_kernel_dimension) /
```

- **EN:** This block produces a result or forwards a computed value. Key symbols: `compute_output_dimension`.
- **CN:** 该代码块返回结果或转发已计算的值。关键符号：`compute_output_dimension`。

### Lines 36-72 / 第 36-72 行

```c
0036:       subsampling_dimension +
0037:       1;
0038: }
0039: 
0040: /**
0041:  * Not exposed in header file
0042:  */
0043: static enum pytorch_qnnp_status pytorch_qnnp_create_convolution_ndhwc_q8(
0044:     uint32_t input_padding_depth,
0045:     uint32_t input_padding_height,
0046:     uint32_t input_padding_width,
0047:     uint32_t kernel_depth,
0048:     uint32_t kernel_height,
0049:     uint32_t kernel_width,
0050:     uint32_t subsampling_depth,
0051:     uint32_t subsampling_height,
0052:     uint32_t subsampling_width,
0053:     uint32_t dilation_depth,
0054:     uint32_t dilation_height,
0055:     uint32_t dilation_width,
0056:     uint32_t groups,
0057:     size_t group_input_channels,
0058:     size_t group_output_channels,
0059:     uint8_t input_zero_point,
0060:     const uint8_t* kernel_zero_points,
0061:     const uint8_t* kernel,
0062:     const int32_t* bias,
0063:     uint8_t output_zero_point,
0064:     uint8_t output_min,
0065:     uint8_t output_max,
0066:     uint32_t flags,
0067:     const float* requantization_scales,
0068:     bool per_channel,
0069:     pytorch_qnnp_operator_t* convolution_out,
0070:     bool is_2d /* true: 2d, false: 3d */) {
0071:   pytorch_qnnp_operator_t convolution = NULL;
0072:   enum pytorch_qnnp_status status = pytorch_qnnp_status_uninitialized;
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `pytorch_qnnp_status`, `pytorch_qnnp_create_convolution_ndhwc_q8`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`pytorch_qnnp_status`, `pytorch_qnnp_create_convolution_ndhwc_q8`。

### Lines 73-110 / 第 73-110 行

```c
0073: 
0074:   if (!pytorch_qnnp_params.initialized) {
0075:     pytorch_qnnp_log_error(
0076:         "pytorch_qnnp_create_convolution2d_nhwc_q8 failed because QNNPACK is not properly initialized");
0077:     goto error;
0078:   }
0079: 
0080:   status = pytorch_qnnp_status_invalid_parameter;
0081: 
0082:   if (kernel_width == 0 || kernel_height == 0) {
0083:     pytorch_qnnp_log_error(
0084:         "failed to create convolution with %" PRIu32 "x%" PRIu32
0085:         " kernel: kernel dimensions must be non-zero",
0086:         kernel_width,
0087:         kernel_height);
0088:     goto error;
0089:   }
0090: 
0091:   if (subsampling_width == 0 || subsampling_height == 0) {
0092:     pytorch_qnnp_log_error(
0093:         "failed to create convolution with %" PRIu32 "x%" PRIu32
0094:         " subsampling: "
0095:         "subsampling dimensions must be non-zero",
0096:         subsampling_width,
0097:         subsampling_height);
0098:     goto error;
0099:   }
0100: 
0101:   if (dilation_width == 0 || dilation_height == 0) {
0102:     pytorch_qnnp_log_error(
0103:         "failed to create convolution with %" PRIu32 "x%" PRIu32
0104:         " dilation: "
0105:         "dilation dimensions must be non-zero",
0106:         dilation_width,
0107:         dilation_height);
0108:     goto error;
0109:   }
0110: 
```

- **EN:** This block handles conditional branches and special cases. Key symbols: `pytorch_qnnp_log_error`.
- **CN:** 该代码块处理条件分支与特殊情况。关键符号：`pytorch_qnnp_log_error`。

### Lines 111-149 / 第 111-149 行

```c
0111:   status = pytorch_qnnp_status_unsupported_parameter;
0112: 
0113:   if (subsampling_height > kernel_height) {
0114:     pytorch_qnnp_log_info(
0115:         "inefficiency in convolution with %" PRIu32 "x%" PRIu32
0116:         " kernel and %" PRIu32 "x%" PRIu32
0117:         " subsampling: "
0118:         "height subsampling is greater than kernel height; subsampling should be performed before the convolution",
0119:         kernel_width,
0120:         kernel_height,
0121:         subsampling_width,
0122:         subsampling_height);
0123:   }
0124: 
0125:   if (subsampling_width > kernel_width) {
0126:     pytorch_qnnp_log_info(
0127:         "inefficiency in convolution with %" PRIu32 "x%" PRIu32
0128:         " kernel and %" PRIu32 "x%" PRIu32
0129:         " subsampling: "
0130:         "width subsampling is greater than kernel width; subsampling should be performed before the convolution",
0131:         kernel_width,
0132:         kernel_height,
0133:         subsampling_width,
0134:         subsampling_height);
0135:   }
0136: 
0137:   if (input_padding_depth >= kernel_depth) {
0138:     pytorch_qnnp_log_info(
0139:         "inefficiency in convolution with %" PRIu32 "x%" PRIu32 "x%" PRIu32
0140:         " kernel and %" PRIu32 "+%" PRIu32
0141:         " depth padding: "
0142:         "input depth padding is greater or equal to kernel depth",
0143:         kernel_depth,
0144:         kernel_height,
0145:         kernel_width,
0146:         input_padding_depth,
0147:         input_padding_depth);
0148:   }
0149: 
```

- **EN:** This block handles conditional branches and special cases. Key symbols: `pytorch_qnnp_log_info`.
- **CN:** 该代码块处理条件分支与特殊情况。关键符号：`pytorch_qnnp_log_info`。

### Lines 150-185 / 第 150-185 行

```c
0150:   if (input_padding_height >= kernel_height) {
0151:     pytorch_qnnp_log_info(
0152:         "inefficiency in convolution with %" PRIu32 "x%" PRIu32 "x%" PRIu32
0153:         " kernel and %" PRIu32 "+%" PRIu32
0154:         " height padding: "
0155:         "input height padding is greater or equal to kernel height",
0156:         kernel_depth,
0157:         kernel_height,
0158:         kernel_width,
0159:         input_padding_height,
0160:         input_padding_height);
0161:   }
0162: 
0163:   if (input_padding_width >= kernel_width) {
0164:     pytorch_qnnp_log_info(
0165:         "inefficiency in convolution with %" PRIu32 "x%" PRIu32 "x%" PRIu32
0166:         " kernel and %" PRIu32 "+%" PRIu32
0167:         " width padding: "
0168:         "input width padding is greater or equal to kernel width",
0169:         kernel_depth,
0170:         kernel_height,
0171:         kernel_width,
0172:         input_padding_width,
0173:         input_padding_width);
0174:   }
0175: 
0176:   for (int i = 0; i < groups * group_output_channels; ++i) {
0177:     if (requantization_scales[i] <= 0.0f ||
0178:         !isnormal(requantization_scales[i])) {
0179:       pytorch_qnnp_log_error(
0180:           "failed to create fully connected operator with %.7g requantization scale: scale must be finite and positive",
0181:           requantization_scales[i]);
0182:       goto error;
0183:     }
0184:   }
0185: 
```

- **EN:** This block handles conditional branches and special cases; iterates over tensors, descriptors, options, or runtime state. Key symbols: `pytorch_qnnp_log_info`, `pytorch_qnnp_log_error`.
- **CN:** 该代码块处理条件分支与特殊情况；遍历张量、描述符、配置项或运行时状态。关键符号：`pytorch_qnnp_log_info`, `pytorch_qnnp_log_error`。

### Lines 186-222 / 第 186-222 行

```c
0186:   status = pytorch_qnnp_status_out_of_memory;
0187: 
0188:   convolution = calloc(1, sizeof(struct pytorch_qnnp_operator));
0189:   if (convolution == NULL) {
0190:     pytorch_qnnp_log_error(
0191:         "failed to allocate %zu bytes for pytorch_qnnp_operator structure",
0192:         sizeof(struct pytorch_qnnp_operator));
0193:     goto error;
0194:   }
0195: 
0196:   const size_t kernel_size = kernel_height * kernel_width * kernel_depth;
0197: 
0198:   enum pytorch_qnnp_ukernel_type ukernel_type = pytorch_qnnp_ukernel_type_none;
0199:   const bool any_padding =
0200:       (input_padding_depth | input_padding_height | input_padding_width) != 0;
0201: 
0202:   const bool has_depthwise_dimensions =
0203:       (is_2d &&
0204:        ((kernel_height == 3 && kernel_width == 3) ||
0205:         (kernel_height == 5 && kernel_width == 5))) ||
0206:       (!is_2d && kernel_height == 3 && kernel_width == 3 && kernel_depth == 3);
0207:   const bool has_depthwise_grouping =
0208:       group_input_channels == 1 && group_output_channels == 1 && groups > 1;
0209:   if (has_depthwise_dimensions && has_depthwise_grouping) {
0210:     ukernel_type = pytorch_qnnp_ukernel_type_dwconv;
0211:   } else if (
0212:       kernel_size == 1 && subsampling_height == 1 && subsampling_width == 1 &&
0213:       !any_padding) {
0214:     ukernel_type =
0215:         group_input_channels >= pytorch_qnnp_params.q8conv_xzp.kthreshold
0216:         ? pytorch_qnnp_ukernel_type_xzp_gemm
0217:         : pytorch_qnnp_ukernel_type_gemm;
0218:   } else {
0219:     ukernel_type = pytorch_qnnp_ukernel_type_conv;
0220:   }
0221:   size_t zero_size = 0, zero_offset = 0;
0222: 
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `pytorch_qnnp_operator`, `pytorch_qnnp_ukernel_type`, `pytorch_qnnp_log_error`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`pytorch_qnnp_operator`, `pytorch_qnnp_ukernel_type`, `pytorch_qnnp_log_error`。

### Lines 223-282 / 第 223-282 行

```c
0223:   switch (ukernel_type) {
0224:     // This also covers the case of dwconv_per_channel
0225:     // since the weight packing is shared between the two.
0226:     case pytorch_qnnp_ukernel_type_dwconv: {
0227:       const uint32_t cr = pytorch_qnnp_params.q8dw9.cr;
0228:       const uint32_t c_stride = (groups + (cr - 1)) & -cr;
0229:       convolution->group_stride = c_stride;
0230:       const size_t packed_weights_size =
0231:           (sizeof(uint8_t) * kernel_size + sizeof(int32_t)) * c_stride;
0232:       convolution->packed_weights = malloc(packed_weights_size);
0233:       if (convolution->packed_weights == NULL) {
0234:         pytorch_qnnp_log_error(
0235:             "failed to allocate %zu bytes for packed weights",
0236:             packed_weights_size);
0237:         goto error;
0238:       }
0239: 
0240:       switch (kernel_size) {
0241:         case 9:
0242:           pytorch_pack_q8dw_w(
0243:               kernel_height,
0244:               kernel_width,
0245:               groups,
0246:               cr,
0247: #if !PYTORCH_QNNPACK_RUNTIME_QUANTIZATION
0248:               input_zero_point,
0249:               kernel_zero_points[0],
0250: #endif
0251:               kernel,
0252:               bias,
0253:               convolution->packed_weights);
0254:           break;
0255:         case 25:
0256:           /* change this later */
0257:           pytorch_pack_q8dw_2d_w_dilation(
0258:               kernel_height,
0259:               kernel_width,
0260:               groups,
0261:               cr,
0262:               0,
0263:               kernel_height,
0264:               0,
0265:               2,
0266:               kernel,
0267:               bias,
0268:               convolution->packed_weights,
0269:               true);
0270:           pytorch_pack_q8dw_2d_w_dilation(
0271:               kernel_height,
0272:               kernel_width,
0273:               groups,
0274:               cr,
0275:               0,
0276:               kernel_height,
0277:               2,
0278:               4,
0279:               kernel,
0280:               bias,
0281:               (char*)convolution->packed_weights +
0282:                   (10 + sizeof(int32_t) / sizeof(uint8_t)) * c_stride,
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends; handles conditional branches and special cases; manages memory allocation, buffers, or ownership boundaries. Key symbols: `pytorch_qnnp_log_error`, `pytorch_pack_q8dw_w`, `pytorch_pack_q8dw_2d_w_dilation`.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端；处理条件分支与特殊情况；管理内存分配、缓冲区或所有权边界。关键符号：`pytorch_qnnp_log_error`, `pytorch_pack_q8dw_w`, `pytorch_pack_q8dw_2d_w_dilation`。

### Lines 283-342 / 第 283-342 行

```c
0283:               false);
0284:           pytorch_pack_q8dw_2d_w_dilation(
0285:               kernel_height,
0286:               kernel_width,
0287:               groups,
0288:               cr,
0289:               0,
0290:               kernel_height,
0291:               4,
0292:               5,
0293:               kernel,
0294:               bias,
0295:               (char*)convolution->packed_weights +
0296:                   (20 + sizeof(int32_t) / sizeof(uint8_t)) * c_stride,
0297:               false);
0298:           break;
0299:         case 27:
0300:           pytorch_pack_q8dw_3d_w_dilation(
0301:               kernel_depth,
0302:               kernel_height,
0303:               kernel_width,
0304:               groups,
0305:               cr,
0306:               0,
0307:               kernel_depth,
0308:               0,
0309:               kernel_height,
0310:               0,
0311:               1,
0312:               kernel,
0313:               bias,
0314:               convolution->packed_weights,
0315:               true);
0316:           pytorch_pack_q8dw_3d_w_dilation(
0317:               kernel_depth,
0318:               kernel_height,
0319:               kernel_width,
0320:               groups,
0321:               cr,
0322:               0,
0323:               kernel_depth,
0324:               0,
0325:               kernel_height,
0326:               1,
0327:               2,
0328:               kernel,
0329:               bias,
0330:               (char*)convolution->packed_weights +
0331:                   (kernel_depth * kernel_height +
0332:                    sizeof(int32_t) / sizeof(uint8_t)) *
0333:                       c_stride,
0334:               false);
0335:           pytorch_pack_q8dw_3d_w_dilation(
0336:               kernel_depth,
0337:               kernel_height,
0338:               kernel_width,
0339:               groups,
0340:               cr,
0341:               0,
0342:               kernel_depth,
```

- **EN:** This block implements local helper logic for `convolution`. Key symbols: `pytorch_pack_q8dw_2d_w_dilation`, `pytorch_pack_q8dw_3d_w_dilation`.
- **CN:** 该代码块实现与 `convolution` 相关的局部辅助逻辑。关键符号：`pytorch_pack_q8dw_2d_w_dilation`, `pytorch_pack_q8dw_3d_w_dilation`。

### Lines 343-377 / 第 343-377 行

```c
0343:               0,
0344:               kernel_height,
0345:               2,
0346:               3,
0347:               kernel,
0348:               bias,
0349:               (char*)convolution->packed_weights +
0350:                   (2 * kernel_depth * kernel_height +
0351:                    sizeof(int32_t) / sizeof(uint8_t)) *
0352:                       c_stride,
0353:               false);
0354:           break;
0355:         default:
0356:           PYTORCH_QNNP_UNREACHABLE;
0357:       }
0358: 
0359:       if (groups >= 8) {
0360:         zero_size = sizeof(uint8_t) * c_stride;
0361:         zero_offset = 0;
0362:       } else {
0363:         zero_size = sizeof(uint8_t) * c_stride + 8;
0364:         zero_offset = sizeof(uint8_t) * 8;
0365:       }
0366:       break;
0367:     }
0368:     case pytorch_qnnp_ukernel_type_xzp_gemm: {
0369:       // TODO: XZP kernels won't be supporting per channel quantization.
0370:       // For now we dont use XZP kernels anywhere. Probably deprecate it for now
0371:       // and resurrect later if needed.
0372:       const uint32_t nr = pytorch_qnnp_params.q8conv_xzp.nr;
0373:       const uint32_t kr = pytorch_qnnp_params.q8conv_xzp.kr;
0374:       const uint32_t sr = pytorch_qnnp_params.q8conv_xzp.kc;
0375:       const uint32_t n_stride = (group_output_channels + (nr - 1)) & -nr;
0376:       const uint32_t k_stride = (group_input_channels + (kr - 1)) & -kr;
0377: 
```

- **EN:** This block handles conditional branches and special cases. Key symbols: no prominent local symbols.
- **CN:** 该代码块处理条件分支与特殊情况。关键符号：无明显局部符号。

### Lines 378-415 / 第 378-415 行

```c
0378:       const size_t packed_group_weights_size =
0379:           (sizeof(uint8_t) * kernel_size * k_stride + sizeof(int32_t)) *
0380:           n_stride;
0381:       convolution->packed_weights = malloc(packed_group_weights_size * groups);
0382:       if (convolution->packed_weights == NULL) {
0383:         pytorch_qnnp_log_error(
0384:             "failed to allocate %zu bytes for packed weights",
0385:             packed_group_weights_size * groups);
0386:         goto error;
0387:       }
0388:       /* The XZP ukernel needs the padding to be 0 */
0389:       memset(
0390:           convolution->packed_weights, 0, packed_group_weights_size * groups);
0391: 
0392:       for (uint32_t group = 0; group < groups; group++) {
0393:         pytorch_pack_swizzle_q8gemm_b(
0394:             group_output_channels,
0395:             group_input_channels,
0396:             nr,
0397:             kr,
0398:             sr,
0399: #if !PYTORCH_QNNPACK_RUNTIME_QUANTIZATION
0400:             input_zero_point,
0401:             kernel_zero_points[0],
0402: #endif
0403:             kernel + group * group_output_channels * group_input_channels,
0404:             bias + group * group_output_channels,
0405:             (void*)((uintptr_t)convolution->packed_weights + group * packed_group_weights_size));
0406:       }
0407:       break;
0408:     }
0409:     case pytorch_qnnp_ukernel_type_gemm:
0410:     case pytorch_qnnp_ukernel_type_conv: {
0411:       const uint32_t nr = pytorch_qnnp_params.q8conv.nr;
0412:       const uint32_t kr = pytorch_qnnp_params.q8conv.kr;
0413:       const uint32_t n_stride = (group_output_channels + (nr - 1)) & -nr;
0414:       const uint32_t k_stride = (group_input_channels + (kr - 1)) & -kr;
0415: 
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends; handles conditional branches and special cases; iterates over tensors, descriptors, options, or runtime state; manages memory allocation, buffers, or ownership boundaries. Key symbols: `pytorch_qnnp_log_error`, `memset`, `pytorch_pack_swizzle_q8gemm_b`.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端；处理条件分支与特殊情况；遍历张量、描述符、配置项或运行时状态；管理内存分配、缓冲区或所有权边界。关键符号：`pytorch_qnnp_log_error`, `memset`, `pytorch_pack_swizzle_q8gemm_b`。

### Lines 416-446 / 第 416-446 行

```c
0416:       const size_t packed_group_weights_size =
0417:           (sizeof(uint8_t) * kernel_size * k_stride + sizeof(int32_t)) *
0418:           n_stride;
0419:       convolution->packed_weights = malloc(packed_group_weights_size * groups);
0420:       if (convolution->packed_weights == NULL) {
0421:         pytorch_qnnp_log_error(
0422:             "failed to allocate %zu bytes for packed weights",
0423:             packed_group_weights_size * groups);
0424:         goto error;
0425:       }
0426:       memset(
0427:           convolution->packed_weights,
0428:           kernel_zero_points[0],
0429:           packed_group_weights_size * groups);
0430: 
0431:       switch (ukernel_type) {
0432:         case pytorch_qnnp_ukernel_type_gemm:
0433:           for (uint32_t group = 0; group < groups; group++) {
0434:             pytorch_pack_q8gemm_w(
0435:                 group_output_channels,
0436:                 group_input_channels,
0437:                 nr,
0438:                 nr,
0439:                 kr,
0440: #if !PYTORCH_QNNPACK_RUNTIME_QUANTIZATION
0441:                 input_zero_point,
0442:                 kernel_zero_points[0],
0443: #endif
0444:                 kernel + group * group_output_channels * group_input_channels,
0445:                 bias + group * group_output_channels,
0446: #if PYTORCH_QNNPACK_RUNTIME_QUANTIZATION
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends; handles conditional branches and special cases; iterates over tensors, descriptors, options, or runtime state; manages memory allocation, buffers, or ownership boundaries. Key symbols: `pytorch_qnnp_log_error`, `memset`.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端；处理条件分支与特殊情况；遍历张量、描述符、配置项或运行时状态；管理内存分配、缓冲区或所有权边界。关键符号：`pytorch_qnnp_log_error`, `memset`。

### Lines 447-477 / 第 447-477 行

```c
0447:                 kernel_zero_points + group * group_output_channels,
0448: #endif
0449:                 (void*)((uintptr_t)convolution->packed_weights + group * packed_group_weights_size));
0450:           }
0451:           break;
0452:         case pytorch_qnnp_ukernel_type_conv:
0453:           for (uint32_t group = 0; group < groups; group++) {
0454:             pytorch_pack_q8conv_w(
0455:                 group_output_channels,
0456:                 kernel_size,
0457:                 group_input_channels,
0458:                 nr,
0459:                 kr,
0460: #if !PYTORCH_QNNPACK_RUNTIME_QUANTIZATION
0461:                 input_zero_point,
0462:                 kernel_zero_points[0],
0463: #endif
0464:                 kernel +
0465:                     group * group_output_channels * kernel_size *
0466:                         group_input_channels,
0467:                 bias + group * group_output_channels,
0468: #if PYTORCH_QNNPACK_RUNTIME_QUANTIZATION
0469:                 kernel_zero_points + group * group_output_channels,
0470: #endif
0471:                 (void*)((uintptr_t)convolution->packed_weights + group * packed_group_weights_size));
0472:           }
0473:           break;
0474:         default:
0475:           PYTORCH_QNNP_UNREACHABLE;
0476:       }
0477: 
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends; iterates over tensors, descriptors, options, or runtime state. Key symbols: `pytorch_pack_q8conv_w`.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端；遍历张量、描述符、配置项或运行时状态。关键符号：`pytorch_pack_q8conv_w`。

### Lines 478-518 / 第 478-518 行

```c
0478:       if (group_input_channels >= 8) {
0479:         zero_size = sizeof(uint8_t) * k_stride;
0480:         zero_offset = 0;
0481:       } else {
0482:         zero_size = sizeof(uint8_t) * k_stride + 8;
0483:         zero_offset = 8;
0484:       }
0485:       break;
0486:     }
0487:     default:
0488:       PYTORCH_QNNP_UNREACHABLE;
0489:   }
0490: 
0491:   if (any_padding) {
0492:     void* zero_buffer = malloc(zero_size);
0493:     if (zero_buffer == NULL) {
0494:       pytorch_qnnp_log_error(
0495:           "failed to allocate %zu bytes for zero padding", zero_size);
0496:       goto error;
0497:     }
0498:     memset(zero_buffer, input_zero_point, zero_size);
0499:     convolution->zero_buffer = zero_buffer;
0500:     convolution->zero_pointer = (void*)((uintptr_t)zero_buffer + zero_offset);
0501:   }
0502: 
0503:   convolution->input_padding_depth = input_padding_depth;
0504:   convolution->input_padding_height = input_padding_height;
0505:   convolution->input_padding_width = input_padding_width;
0506:   convolution->kernel_depth = kernel_depth;
0507:   convolution->kernel_height = kernel_height;
0508:   convolution->kernel_width = kernel_width;
0509:   convolution->stride_depth = subsampling_depth;
0510:   convolution->stride_height = subsampling_height;
0511:   convolution->stride_width = subsampling_width;
0512:   convolution->dilation_depth = dilation_depth;
0513:   convolution->dilation_height = dilation_height;
0514:   convolution->dilation_width = dilation_width;
0515:   convolution->groups = groups;
0516:   convolution->group_input_channels = group_input_channels;
0517:   convolution->group_output_channels = group_output_channels;
0518: 
```

- **EN:** This block handles conditional branches and special cases; manages memory allocation, buffers, or ownership boundaries. Key symbols: `pytorch_qnnp_log_error`, `memset`.
- **CN:** 该代码块处理条件分支与特殊情况；管理内存分配、缓冲区或所有权边界。关键符号：`pytorch_qnnp_log_error`, `memset`。

### Lines 519-548 / 第 519-548 行

```c
0519:   convolution->kernel_zero_point = kernel_zero_points[0];
0520: 
0521:   if (ukernel_type == pytorch_qnnp_ukernel_type_xzp_gemm) {
0522:     convolution->requantization_params =
0523:         pytorch_qnnp_compute_requantization_params(
0524:             requantization_scales[0], output_zero_point, output_min, output_max);
0525:   } else {
0526:     convolution->conv_quantization_params =
0527:         pytorch_qnnp_compute_conv_quantization_params(
0528:             input_zero_point,
0529:             kernel_zero_points,
0530:             requantization_scales,
0531:             output_zero_point,
0532:             output_min,
0533:             output_max);
0534:   }
0535: 
0536:   convolution->ukernel_type = ukernel_type;
0537:   convolution->format = pytorch_qnnp_format_quint8;
0538: 
0539:   convolution->per_channel = per_channel;
0540: 
0541:   *convolution_out = convolution;
0542:   return pytorch_qnnp_status_success;
0543: 
0544: error:
0545:   pytorch_qnnp_delete_operator(convolution);
0546:   return status;
0547: }
0548: 
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value. Key symbols: `pytorch_qnnp_compute_requantization_params`, `pytorch_qnnp_compute_conv_quantization_params`, `pytorch_qnnp_delete_operator`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值。关键符号：`pytorch_qnnp_compute_requantization_params`, `pytorch_qnnp_compute_conv_quantization_params`, `pytorch_qnnp_delete_operator`。

### Lines 549-601 / 第 549-601 行

```c
0549: enum pytorch_qnnp_status pytorch_qnnp_create_convolution2d_nhwc_q8(
0550:     uint32_t input_padding_height,
0551:     uint32_t input_padding_width,
0552:     uint32_t kernel_height,
0553:     uint32_t kernel_width,
0554:     uint32_t subsampling_height,
0555:     uint32_t subsampling_width,
0556:     uint32_t dilation_height,
0557:     uint32_t dilation_width,
0558:     uint32_t groups,
0559:     size_t group_input_channels,
0560:     size_t group_output_channels,
0561:     uint8_t input_zero_point,
0562:     const uint8_t* kernel_zero_points,
0563:     const uint8_t* kernel,
0564:     const int32_t* bias,
0565:     uint8_t output_zero_point,
0566:     uint8_t output_min,
0567:     uint8_t output_max,
0568:     uint32_t flags,
0569:     const float* requantization_scales,
0570:     bool per_channel,
0571:     pytorch_qnnp_operator_t* convolution_out) {
0572:   return pytorch_qnnp_create_convolution_ndhwc_q8(
0573:       0,
0574:       input_padding_height,
0575:       input_padding_width,
0576:       1,
0577:       kernel_height,
0578:       kernel_width,
0579:       1,
0580:       subsampling_height,
0581:       subsampling_width,
0582:       1,
0583:       dilation_height,
0584:       dilation_width,
0585:       groups,
0586:       group_input_channels,
0587:       group_output_channels,
0588:       input_zero_point,
0589:       kernel_zero_points,
0590:       kernel,
0591:       bias,
0592:       output_zero_point,
0593:       output_min,
0594:       output_max,
0595:       flags,
0596:       requantization_scales,
0597:       per_channel,
0598:       convolution_out,
0599:       true /* is_2d? */);
0600: }
0601: 
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `pytorch_qnnp_status`, `pytorch_qnnp_create_convolution2d_nhwc_q8`, `pytorch_qnnp_create_convolution_ndhwc_q8`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`pytorch_qnnp_status`, `pytorch_qnnp_create_convolution2d_nhwc_q8`, `pytorch_qnnp_create_convolution_ndhwc_q8`。

### Lines 602-658 / 第 602-658 行

```c
0602: enum pytorch_qnnp_status pytorch_qnnp_create_convolution3d_ndhwc_q8(
0603:     uint32_t input_padding_depth,
0604:     uint32_t input_padding_height,
0605:     uint32_t input_padding_width,
0606:     uint32_t kernel_depth,
0607:     uint32_t kernel_height,
0608:     uint32_t kernel_width,
0609:     uint32_t subsampling_depth,
0610:     uint32_t subsampling_height,
0611:     uint32_t subsampling_width,
0612:     uint32_t dilation_depth,
0613:     uint32_t dilation_height,
0614:     uint32_t dilation_width,
0615:     uint32_t groups,
0616:     size_t group_input_channels,
0617:     size_t group_output_channels,
0618:     uint8_t input_zero_point,
0619:     const uint8_t* kernel_zero_points,
0620:     const uint8_t* kernel,
0621:     const int32_t* bias,
0622:     uint8_t output_zero_point,
0623:     uint8_t output_min,
0624:     uint8_t output_max,
0625:     uint32_t flags,
0626:     const float* requantization_scales,
0627:     bool per_channel,
0628:     pytorch_qnnp_operator_t* convolution_out) {
0629:   return pytorch_qnnp_create_convolution_ndhwc_q8(
0630:       input_padding_depth,
0631:       input_padding_height,
0632:       input_padding_width,
0633:       kernel_depth,
0634:       kernel_height,
0635:       kernel_width,
0636:       subsampling_depth,
0637:       subsampling_height,
0638:       subsampling_width,
0639:       dilation_depth,
0640:       dilation_height,
0641:       dilation_width,
0642:       groups,
0643:       group_input_channels,
0644:       group_output_channels,
0645:       input_zero_point,
0646:       kernel_zero_points,
0647:       kernel,
0648:       bias,
0649:       output_zero_point,
0650:       output_min,
0651:       output_max,
0652:       flags,
0653:       requantization_scales,
0654:       per_channel,
0655:       convolution_out,
0656:       false /* is_2d? */);
0657: }
0658: 
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `pytorch_qnnp_status`, `pytorch_qnnp_create_convolution3d_ndhwc_q8`, `pytorch_qnnp_create_convolution_ndhwc_q8`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`pytorch_qnnp_status`, `pytorch_qnnp_create_convolution3d_ndhwc_q8`, `pytorch_qnnp_create_convolution_ndhwc_q8`。

### Lines 659-693 / 第 659-693 行

```c
0659: enum pytorch_qnnp_status pytorch_qnnp_setup_convolution2d_nhwc_q8(
0660:     pytorch_qnnp_operator_t convolution,
0661:     size_t batch_size,
0662:     size_t input_height,
0663:     size_t input_width,
0664:     const uint8_t* input,
0665:     size_t input_pixel_stride,
0666:     uint8_t* output,
0667:     size_t output_pixel_stride,
0668:     pthreadpool_t threadpool) {
0669:   return pytorch_qnnp_setup_convolution_ndhwc_q8(
0670:       convolution,
0671:       batch_size,
0672:       1,
0673:       input_height,
0674:       input_width,
0675:       input,
0676:       input_pixel_stride,
0677:       output,
0678:       output_pixel_stride,
0679:       threadpool);
0680: }
0681: 
0682: enum pytorch_qnnp_status pytorch_qnnp_setup_convolution_ndhwc_q8(
0683:     pytorch_qnnp_operator_t convolution,
0684:     size_t batch_size,
0685:     size_t input_depth,
0686:     size_t input_height,
0687:     size_t input_width,
0688:     const uint8_t* input,
0689:     size_t input_pixel_stride,
0690:     uint8_t* output,
0691:     size_t output_pixel_stride,
0692:     pthreadpool_t threadpool) {
0693:   if (!pytorch_qnnp_params.initialized) {
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `pytorch_qnnp_status`, `pytorch_qnnp_setup_convolution2d_nhwc_q8`, `pytorch_qnnp_setup_convolution_ndhwc_q8`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`pytorch_qnnp_status`, `pytorch_qnnp_setup_convolution2d_nhwc_q8`, `pytorch_qnnp_setup_convolution_ndhwc_q8`。

### Lines 694-737 / 第 694-737 行

```c
0694:     pytorch_qnnp_log_error(
0695:         "pytorch_qnnp_setup_convolution_ndhwc_q8 failed because QNNPACK is not properly initialized");
0696:     return pytorch_qnnp_status_uninitialized;
0697:   }
0698: 
0699:   if (batch_size == 0) {
0700:     convolution->batch_size = 0;
0701:     return pytorch_qnnp_status_success;
0702:   }
0703: 
0704:   if (input_width == 0 || input_height == 0 || input_depth == 0) {
0705:     pytorch_qnnp_log_error(
0706:         "failed to setup convolution with %zux%zux%zu input: input dimensions must be non-zero",
0707:         input_width,
0708:         input_height,
0709:         input_depth);
0710:     return pytorch_qnnp_status_invalid_parameter;
0711:   }
0712: 
0713:   convolution->batch_size = batch_size;
0714:   convolution->input_depth = input_depth;
0715:   convolution->input_height = input_height;
0716:   convolution->input_width = input_width;
0717:   convolution->input = input;
0718:   convolution->input_pixel_stride = input_pixel_stride;
0719: 
0720:   convolution->output_depth = compute_output_dimension(
0721:       input_depth + convolution->input_padding_depth * 2,
0722:       convolution->kernel_depth,
0723:       convolution->dilation_depth,
0724:       convolution->stride_depth);
0725:   convolution->output_height = compute_output_dimension(
0726:       input_height + convolution->input_padding_height * 2,
0727:       convolution->kernel_height,
0728:       convolution->dilation_height,
0729:       convolution->stride_height);
0730:   convolution->output_width = compute_output_dimension(
0731:       input_width + convolution->input_padding_width * 2,
0732:       convolution->kernel_width,
0733:       convolution->dilation_width,
0734:       convolution->stride_width);
0735:   convolution->output = output;
0736:   convolution->output_pixel_stride = output_pixel_stride;
0737: 
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value. Key symbols: `pytorch_qnnp_log_error`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值。关键符号：`pytorch_qnnp_log_error`。

### Lines 738-771 / 第 738-771 行

```c
0738:   switch (convolution->ukernel_type) {
0739:     case pytorch_qnnp_ukernel_type_gemm:
0740:       /* Convolution maps directly to GEMM and doesn't use indirection buffer */
0741:       return pytorch_qnnp_status_success;
0742:     case pytorch_qnnp_ukernel_type_xzp_gemm: {
0743:       const size_t groups = convolution->groups;
0744:       const size_t input_size = input_depth * input_height * input_width;
0745:       void* a_sum = (void*)realloc(
0746:           convolution->a_sum,
0747:           sizeof(int32_t) * batch_size * groups * input_size);
0748:       if (a_sum == NULL) {
0749:         pytorch_qnnp_log_error(
0750:             "failed to allocate %zu bytes for row sum data",
0751:             sizeof(int32_t) * batch_size * groups * input_size);
0752:         return pytorch_qnnp_status_out_of_memory;
0753:       }
0754:       convolution->a_sum = a_sum;
0755:       return pytorch_qnnp_status_success;
0756:     }
0757:     case pytorch_qnnp_ukernel_type_conv: {
0758:       const size_t groups = convolution->groups;
0759:       const size_t kernel_depth = convolution->kernel_depth;
0760:       const size_t kernel_height = convolution->kernel_height;
0761:       const size_t kernel_width = convolution->kernel_width;
0762:       const size_t kernel_size = kernel_depth * kernel_height * kernel_width;
0763:       const size_t output_depth = convolution->output_depth;
0764:       const size_t output_height = convolution->output_height;
0765:       const size_t output_width = convolution->output_width;
0766:       const size_t output_size = output_depth * output_height * output_width;
0767:       const size_t output_tile_size = pytorch_qnnp_params.q8conv.mr;
0768:       const size_t tiled_output_size = round_up(output_size, output_tile_size);
0769:       const size_t indirection_buffer_size =
0770:           sizeof(void*) * batch_size * groups * tiled_output_size * kernel_size;
0771: 
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; manages memory allocation, buffers, or ownership boundaries. Key symbols: `pytorch_qnnp_log_error`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；管理内存分配、缓冲区或所有权边界。关键符号：`pytorch_qnnp_log_error`。

### Lines 772-802 / 第 772-802 行

```c
0772:       const void** indirection_buffer = (const void**)realloc(
0773:           convolution->indirection_buffer, indirection_buffer_size);
0774:       if (indirection_buffer == NULL) {
0775:         pytorch_qnnp_log_error(
0776:             "failed to allocate %zu bytes for indirection buffer",
0777:             indirection_buffer_size);
0778:         return pytorch_qnnp_status_out_of_memory;
0779:       }
0780:       convolution->indirection_buffer = indirection_buffer;
0781:       pytorch_qnnp_indirection_init_conv3d(
0782:           convolution, output_tile_size, tiled_output_size);
0783:       return pytorch_qnnp_status_success;
0784:     }
0785:     case pytorch_qnnp_ukernel_type_dwconv: {
0786:       pytorch_qnnp_indirection_set_step_dimensions(convolution);
0787: 
0788:       const size_t indirection_buffer_size = sizeof(void*) * batch_size *
0789:           convolution->output_depth * convolution->step_depth;
0790: 
0791:       const void** indirection_buffer = (const void**)realloc(
0792:           convolution->indirection_buffer, indirection_buffer_size);
0793:       if (indirection_buffer == NULL) {
0794:         pytorch_qnnp_log_error(
0795:             "failed to allocate %zu bytes for indirection buffer",
0796:             indirection_buffer_size);
0797:         return pytorch_qnnp_status_out_of_memory;
0798:       }
0799:       convolution->indirection_buffer = indirection_buffer;
0800: 
0801:       pytorch_qnnp_indirection_init_dwconv(convolution, 0);
0802:       return pytorch_qnnp_status_success;
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; manages memory allocation, buffers, or ownership boundaries. Key symbols: `pytorch_qnnp_log_error`, `pytorch_qnnp_indirection_init_conv3d`, `pytorch_qnnp_indirection_set_step_dimensions`, `pytorch_qnnp_indirection_init_dwconv`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；管理内存分配、缓冲区或所有权边界。关键符号：`pytorch_qnnp_log_error`, `pytorch_qnnp_indirection_init_conv3d`, `pytorch_qnnp_indirection_set_step_dimensions`, `pytorch_qnnp_indirection_init_dwconv`。

### Lines 803-807 / 第 803-807 行

```c
0803:     }
0804:     default:
0805:       PYTORCH_QNNP_UNREACHABLE;
0806:   }
0807: }
```

- **EN:** This block implements local helper logic for `convolution`. Key symbols: no prominent local symbols.
- **CN:** 该代码块实现与 `convolution` 相关的局部辅助逻辑。关键符号：无明显局部符号。


## Key Concepts / 关键概念
- **ATen native operator logic** — ATen 原生算子逻辑
- **GPU kernel/runtime interaction** — GPU 内核/运行时交互
- **Linear algebra backend integration** — 线性代数后端集成
- **Memory allocation strategy** — 内存分配策略
- **Mixed-precision/autocast logic** — 混合精度/自动混精逻辑
- **Runtime validation and invariants** — 运行时校验与不变量
- **Parallel scan/reduction behavior** — 并行扫描/归约行为
- **Core symbols: pytorch_qnnp_operator, pytorch_qnnp_status, pytorch_qnnp_ukernel_type, compute_output_dimension, pytorch_qnnp_create_convolution_ndhwc_q8, pytorch_qnnp_log_error, pytorch_qnnp_log_info, pytorch_pack_q8dw_w** — 核心符号：pytorch_qnnp_operator、pytorch_qnnp_status、pytorch_qnnp_ukernel_type、compute_output_dimension、pytorch_qnnp_create_convolution_ndhwc_q8、pytorch_qnnp_log_error、pytorch_qnnp_log_info、pytorch_pack_q8dw_w

## Dependencies / 依赖关系
- **Internal includes / 内部头文件**: 无
- **External includes / 外部头文件**: `assert.h`, `math.h`, `stdbool.h`, `stddef.h`, `stdint.h`, `stdlib.h`, `string.h`, `fxdiv.h`, `pytorch_qnnpack.h`, `qnnpack/common.h`, `qnnpack/indirection.h`, `qnnpack/log.h`, `...`
- **Namespaces / 命名空间**: 无
- **Representative symbols / 代表性符号**: `pytorch_qnnp_operator`, `pytorch_qnnp_status`, `pytorch_qnnp_ukernel_type`, `compute_output_dimension`, `pytorch_qnnp_create_convolution_ndhwc_q8`, `pytorch_qnnp_log_error`, `pytorch_qnnp_log_info`, `pytorch_pack_q8dw_w`, `pytorch_pack_q8dw_2d_w_dilation`, `pytorch_pack_q8dw_3d_w_dilation`, `memset`, `pytorch_pack_swizzle_q8gemm_b`, `...`
