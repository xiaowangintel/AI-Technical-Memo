# q8dwconv.cc — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `aten/src/ATen/native/quantized/cpu/qnnpack/test/q8dwconv.cc`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements ATen native operators, checks, helper kernels, and backend dispatch behavior. This specific file centers on `q8dwconv.cc`. The file header highlights: "All rights reserved. This source code is licensed under the BSD-style license found in the LICENSE file in the root directory of this source tree.."
- **Purpose (CN)**: 实现 ATen 原生算子、运行时检查、辅助内核以及后端分发行为。 该文件具体围绕 `q8dwconv.cc` 展开。 文件头部注释也概括了其核心职责。

## Line-by-Line Analysis / 逐行分析
### Lines 1-52 / 第 1-52 行

```cpp
0001: /*
0002:  * Copyright (c) Facebook, Inc. and its affiliates.
0003:  * All rights reserved.
0004:  *
0005:  * This source code is licensed under the BSD-style license found in the
0006:  * LICENSE file in the root directory of this source tree.
0007:  */
0008: 
0009: #include <cpuinfo.h>
0010: #include <gtest/gtest.h>
0011: 
0012: #include <qnnpack/isa-checks.h>
0013: #include <qnnpack/q8dwconv.h>
0014: 
0015: #include "dwconv-microkernel-tester.h"
0016: 
0017: #if CPUINFO_ARCH_ARM || CPUINFO_ARCH_ARM64
0018: TEST(Q8DWCONV_UP8x9__NEON, single_output_channels_eq_8) {
0019:   TEST_REQUIRES_ARM_NEON;
0020:   DWConvMicrokernelTester()
0021:       .kernelHeight(3)
0022:       .kernelWidth(3)
0023:       .cr(8)
0024:       .channels(8)
0025:       .width(1)
0026:       .test(pytorch_q8dwconv_ukernel_up8x9__neon);
0027: }
0028: 
0029: TEST(Q8DWCONV_UP8x9__NEON, single_output_channels_eq_8_with_qmin) {
0030:   TEST_REQUIRES_ARM_NEON;
0031:   DWConvMicrokernelTester()
0032:       .kernelHeight(3)
0033:       .kernelWidth(3)
0034:       .cr(8)
0035:       .channels(8)
0036:       .width(1)
0037:       .qmin(128)
0038:       .test(pytorch_q8dwconv_ukernel_up8x9__neon);
0039: }
0040: 
0041: TEST(Q8DWCONV_UP8x9__NEON, single_output_channels_eq_8_with_qmax) {
0042:   TEST_REQUIRES_ARM_NEON;
0043:   DWConvMicrokernelTester()
0044:       .kernelHeight(3)
0045:       .kernelWidth(3)
0046:       .cr(8)
0047:       .channels(8)
0048:       .width(1)
0049:       .qmax(128)
0050:       .test(pytorch_q8dwconv_ukernel_up8x9__neon);
0051: }
0052: 
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends. Key symbols: `DWConvMicrokernelTester`.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端。关键符号：`DWConvMicrokernelTester`。

### Lines 53-105 / 第 53-105 行

```cpp
0053: TEST(
0054:     Q8DWCONV_UP8x9__NEON,
0055:     single_output_channels_eq_8_with_input_zero_point_only) {
0056:   TEST_REQUIRES_ARM_NEON;
0057:   DWConvMicrokernelTester()
0058:       .kernelHeight(3)
0059:       .kernelWidth(3)
0060:       .cr(8)
0061:       .channels(8)
0062:       .width(1)
0063:       .inputZeroPoint(255)
0064:       .kernelZeroPoint(0)
0065:       .test(pytorch_q8dwconv_ukernel_up8x9__neon);
0066: }
0067: 
0068: TEST(
0069:     Q8DWCONV_UP8x9__NEON,
0070:     single_output_channels_eq_8_with_kernel_zero_point_only) {
0071:   TEST_REQUIRES_ARM_NEON;
0072:   DWConvMicrokernelTester()
0073:       .kernelHeight(3)
0074:       .kernelWidth(3)
0075:       .cr(8)
0076:       .channels(8)
0077:       .width(1)
0078:       .inputZeroPoint(0)
0079:       .kernelZeroPoint(255)
0080:       .test(pytorch_q8dwconv_ukernel_up8x9__neon);
0081: }
0082: 
0083: TEST(Q8DWCONV_UP8x9__NEON, multi_output_channels_eq_8) {
0084:   TEST_REQUIRES_ARM_NEON;
0085:   DWConvMicrokernelTester()
0086:       .kernelHeight(3)
0087:       .kernelWidth(3)
0088:       .cr(8)
0089:       .channels(8)
0090:       .width(5)
0091:       .test(pytorch_q8dwconv_ukernel_up8x9__neon);
0092: }
0093: 
0094: TEST(Q8DWCONV_UP8x9__NEON, multi_output_channels_eq_8_with_subsampling) {
0095:   TEST_REQUIRES_ARM_NEON;
0096:   DWConvMicrokernelTester()
0097:       .kernelHeight(3)
0098:       .kernelWidth(3)
0099:       .subsampling(2)
0100:       .cr(8)
0101:       .channels(8)
0102:       .width(5)
0103:       .test(pytorch_q8dwconv_ukernel_up8x9__neon);
0104: }
0105: 
```

- **EN:** This block implements local helper logic for `q8dwconv`. Key symbols: `DWConvMicrokernelTester`.
- **CN:** 该代码块实现与 `q8dwconv` 相关的局部辅助逻辑。关键符号：`DWConvMicrokernelTester`。

### Lines 106-155 / 第 106-155 行

```cpp
0106: TEST(Q8DWCONV_UP8x9__NEON, multi_output_channels_eq_8_with_input_stride) {
0107:   TEST_REQUIRES_ARM_NEON;
0108:   DWConvMicrokernelTester()
0109:       .kernelHeight(3)
0110:       .kernelWidth(3)
0111:       .cr(8)
0112:       .channels(8)
0113:       .width(5)
0114:       .inputStride(17)
0115:       .test(pytorch_q8dwconv_ukernel_up8x9__neon);
0116: }
0117: 
0118: TEST(Q8DWCONV_UP8x9__NEON, multi_output_channels_eq_8_with_output_stride) {
0119:   TEST_REQUIRES_ARM_NEON;
0120:   DWConvMicrokernelTester()
0121:       .kernelHeight(3)
0122:       .kernelWidth(3)
0123:       .cr(8)
0124:       .channels(8)
0125:       .width(5)
0126:       .outputStride(19)
0127:       .test(pytorch_q8dwconv_ukernel_up8x9__neon);
0128: }
0129: 
0130: TEST(Q8DWCONV_UP8x9__NEON, single_output_channels_div_8) {
0131:   TEST_REQUIRES_ARM_NEON;
0132:   for (uint32_t channels = 16; channels < 128; channels += 24) {
0133:     DWConvMicrokernelTester()
0134:         .kernelHeight(3)
0135:         .kernelWidth(3)
0136:         .cr(8)
0137:         .channels(channels)
0138:         .width(1)
0139:         .test(pytorch_q8dwconv_ukernel_up8x9__neon);
0140:   }
0141: }
0142: 
0143: TEST(Q8DWCONV_UP8x9__NEON, multi_output_channels_div_8) {
0144:   TEST_REQUIRES_ARM_NEON;
0145:   for (uint32_t channels = 16; channels < 128; channels += 24) {
0146:     DWConvMicrokernelTester()
0147:         .kernelHeight(3)
0148:         .kernelWidth(3)
0149:         .cr(8)
0150:         .channels(channels)
0151:         .width(5)
0152:         .test(pytorch_q8dwconv_ukernel_up8x9__neon);
0153:   }
0154: }
0155: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `DWConvMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`DWConvMicrokernelTester`。

### Lines 156-210 / 第 156-210 行

```cpp
0156: TEST(Q8DWCONV_UP8x9__NEON, multi_output_channels_div_8_with_output_stride) {
0157:   TEST_REQUIRES_ARM_NEON;
0158:   for (uint32_t channels = 16; channels < 128; channels += 24) {
0159:     DWConvMicrokernelTester()
0160:         .kernelHeight(3)
0161:         .kernelWidth(3)
0162:         .cr(8)
0163:         .channels(channels)
0164:         .width(5)
0165:         .outputStride(171)
0166:         .test(pytorch_q8dwconv_ukernel_up8x9__neon);
0167:   }
0168: }
0169: 
0170: TEST(Q8DWCONV_UP8x9__NEON, single_output_channels_gt_8) {
0171:   TEST_REQUIRES_ARM_NEON;
0172:   for (uint32_t channels = 9; channels < 16; channels++) {
0173:     DWConvMicrokernelTester()
0174:         .kernelHeight(3)
0175:         .kernelWidth(3)
0176:         .cr(8)
0177:         .channels(channels)
0178:         .width(1)
0179:         .test(pytorch_q8dwconv_ukernel_up8x9__neon);
0180:   }
0181: }
0182: 
0183: TEST(Q8DWCONV_UP8x9__NEON, single_output_channels_gt_8_with_qmin) {
0184:   TEST_REQUIRES_ARM_NEON;
0185:   for (uint32_t channels = 9; channels < 16; channels++) {
0186:     DWConvMicrokernelTester()
0187:         .kernelHeight(3)
0188:         .kernelWidth(3)
0189:         .cr(8)
0190:         .channels(channels)
0191:         .width(1)
0192:         .qmin(128)
0193:         .test(pytorch_q8dwconv_ukernel_up8x9__neon);
0194:   }
0195: }
0196: 
0197: TEST(Q8DWCONV_UP8x9__NEON, single_output_channels_gt_8_with_qmax) {
0198:   TEST_REQUIRES_ARM_NEON;
0199:   for (uint32_t channels = 9; channels < 16; channels++) {
0200:     DWConvMicrokernelTester()
0201:         .kernelHeight(3)
0202:         .kernelWidth(3)
0203:         .cr(8)
0204:         .channels(channels)
0205:         .width(1)
0206:         .qmax(128)
0207:         .test(pytorch_q8dwconv_ukernel_up8x9__neon);
0208:   }
0209: }
0210: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `DWConvMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`DWConvMicrokernelTester`。

### Lines 211-260 / 第 211-260 行

```cpp
0211: TEST(
0212:     Q8DWCONV_UP8x9__NEON,
0213:     single_output_channels_gt_8_with_input_zero_point_only) {
0214:   TEST_REQUIRES_ARM_NEON;
0215:   for (uint32_t channels = 9; channels < 16; channels++) {
0216:     DWConvMicrokernelTester()
0217:         .kernelHeight(3)
0218:         .kernelWidth(3)
0219:         .cr(8)
0220:         .channels(channels)
0221:         .width(1)
0222:         .inputZeroPoint(255)
0223:         .kernelZeroPoint(0)
0224:         .test(pytorch_q8dwconv_ukernel_up8x9__neon);
0225:   }
0226: }
0227: 
0228: TEST(
0229:     Q8DWCONV_UP8x9__NEON,
0230:     single_output_channels_gt_8_with_kernel_zero_point_only) {
0231:   TEST_REQUIRES_ARM_NEON;
0232:   for (uint32_t channels = 9; channels < 16; channels++) {
0233:     DWConvMicrokernelTester()
0234:         .kernelHeight(3)
0235:         .kernelWidth(3)
0236:         .cr(8)
0237:         .channels(channels)
0238:         .width(1)
0239:         .inputZeroPoint(0)
0240:         .kernelZeroPoint(255)
0241:         .test(pytorch_q8dwconv_ukernel_up8x9__neon);
0242:   }
0243: }
0244: 
0245: TEST(Q8DWCONV_UP8x9__NEON, multi_output_channels_gt_8) {
0246:   TEST_REQUIRES_ARM_NEON;
0247:   for (uint32_t channels = 9; channels < 16; channels++) {
0248:     DWConvMicrokernelTester()
0249:         .kernelHeight(3)
0250:         .kernelWidth(3)
0251:         .cr(8)
0252:         .channels(channels)
0253:         .width(5)
0254:         .test(pytorch_q8dwconv_ukernel_up8x9__neon);
0255:   }
0256: }
0257: 
0258: TEST(Q8DWCONV_UP8x9__NEON, multi_output_channels_gt_8_with_output_stride) {
0259:   TEST_REQUIRES_ARM_NEON;
0260:   for (uint32_t channels = 9; channels < 16; channels++) {
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `DWConvMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`DWConvMicrokernelTester`。

### Lines 261-318 / 第 261-318 行

```cpp
0261:     DWConvMicrokernelTester()
0262:         .kernelHeight(3)
0263:         .kernelWidth(3)
0264:         .cr(8)
0265:         .channels(channels)
0266:         .width(5)
0267:         .outputStride(17)
0268:         .test(pytorch_q8dwconv_ukernel_up8x9__neon);
0269:   }
0270: }
0271: 
0272: TEST(Q8DWCONV_MP8x25__NEON, single_output_channels_eq_8) {
0273:   TEST_REQUIRES_ARM_NEON;
0274:   DWConvMicrokernelTester()
0275:       .kernelHeight(5)
0276:       .kernelWidth(5)
0277:       .cr(8)
0278:       .channels(8)
0279:       .width(1)
0280:       .test(pytorch_q8dwconv_ukernel_mp8x25__neon);
0281: }
0282: 
0283: TEST(Q8DWCONV_MP8x25__NEON, multi_output_channels_eq_8_with_subsampling) {
0284:   TEST_REQUIRES_ARM_NEON;
0285:   DWConvMicrokernelTester()
0286:       .kernelHeight(5)
0287:       .kernelWidth(5)
0288:       .subsampling(2)
0289:       .cr(8)
0290:       .channels(8)
0291:       .width(5)
0292:       .test(pytorch_q8dwconv_ukernel_mp8x25__neon);
0293: }
0294: 
0295: TEST(Q8DWCONV_MP8x25__NEON, multi_output_channels_eq_8_with_input_stride) {
0296:   TEST_REQUIRES_ARM_NEON;
0297:   DWConvMicrokernelTester()
0298:       .kernelHeight(5)
0299:       .kernelWidth(5)
0300:       .cr(8)
0301:       .channels(8)
0302:       .width(5)
0303:       .inputStride(17)
0304:       .test(pytorch_q8dwconv_ukernel_mp8x25__neon);
0305: }
0306: 
0307: TEST(Q8DWCONV_MP8x25__NEON, multi_output_channels_eq_8_with_output_stride) {
0308:   TEST_REQUIRES_ARM_NEON;
0309:   DWConvMicrokernelTester()
0310:       .kernelHeight(5)
0311:       .kernelWidth(5)
0312:       .cr(8)
0313:       .channels(8)
0314:       .width(5)
0315:       .outputStride(19)
0316:       .test(pytorch_q8dwconv_ukernel_mp8x25__neon);
0317: }
0318: 
```

- **EN:** This block implements local helper logic for `q8dwconv`. Key symbols: `DWConvMicrokernelTester`.
- **CN:** 该代码块实现与 `q8dwconv` 相关的局部辅助逻辑。关键符号：`DWConvMicrokernelTester`。

### Lines 319-372 / 第 319-372 行

```cpp
0319: TEST(Q8DWCONV_MP8x25__NEON, single_output_channels_eq_8_with_qmin) {
0320:   TEST_REQUIRES_ARM_NEON;
0321:   DWConvMicrokernelTester()
0322:       .kernelHeight(5)
0323:       .kernelWidth(5)
0324:       .cr(8)
0325:       .channels(8)
0326:       .width(1)
0327:       .qmin(128)
0328:       .test(pytorch_q8dwconv_ukernel_mp8x25__neon);
0329: }
0330: 
0331: TEST(Q8DWCONV_MP8x25__NEON, single_output_channels_eq_8_with_qmax) {
0332:   TEST_REQUIRES_ARM_NEON;
0333:   DWConvMicrokernelTester()
0334:       .kernelHeight(5)
0335:       .kernelWidth(5)
0336:       .cr(8)
0337:       .channels(8)
0338:       .width(1)
0339:       .qmax(128)
0340:       .test(pytorch_q8dwconv_ukernel_mp8x25__neon);
0341: }
0342: 
0343: TEST(
0344:     Q8DWCONV_MP8x25__NEON,
0345:     single_output_channels_eq_8_with_input_zero_point_only) {
0346:   TEST_REQUIRES_ARM_NEON;
0347:   DWConvMicrokernelTester()
0348:       .kernelHeight(5)
0349:       .kernelWidth(5)
0350:       .cr(8)
0351:       .channels(8)
0352:       .width(1)
0353:       .inputZeroPoint(255)
0354:       .kernelZeroPoint(0)
0355:       .test(pytorch_q8dwconv_ukernel_mp8x25__neon);
0356: }
0357: 
0358: TEST(
0359:     Q8DWCONV_MP8x25__NEON,
0360:     single_output_channels_eq_8_with_kernel_zero_point_only) {
0361:   TEST_REQUIRES_ARM_NEON;
0362:   DWConvMicrokernelTester()
0363:       .kernelHeight(5)
0364:       .kernelWidth(5)
0365:       .cr(8)
0366:       .channels(8)
0367:       .width(1)
0368:       .inputZeroPoint(0)
0369:       .kernelZeroPoint(255)
0370:       .test(pytorch_q8dwconv_ukernel_mp8x25__neon);
0371: }
0372: 
```

- **EN:** This block implements local helper logic for `q8dwconv`. Key symbols: `DWConvMicrokernelTester`.
- **CN:** 该代码块实现与 `q8dwconv` 相关的局部辅助逻辑。关键符号：`DWConvMicrokernelTester`。

### Lines 373-423 / 第 373-423 行

```cpp
0373: TEST(Q8DWCONV_MP8x25__NEON, multi_output_channels_eq_8) {
0374:   TEST_REQUIRES_ARM_NEON;
0375:   DWConvMicrokernelTester()
0376:       .kernelHeight(5)
0377:       .kernelWidth(5)
0378:       .cr(8)
0379:       .channels(8)
0380:       .width(3)
0381:       .test(pytorch_q8dwconv_ukernel_mp8x25__neon);
0382: }
0383: 
0384: TEST(Q8DWCONV_MP8x25__NEON, single_output_channels_div_8) {
0385:   TEST_REQUIRES_ARM_NEON;
0386:   for (uint32_t channels = 16; channels < 128; channels += 24) {
0387:     DWConvMicrokernelTester()
0388:         .kernelHeight(5)
0389:         .kernelWidth(5)
0390:         .cr(8)
0391:         .channels(channels)
0392:         .width(1)
0393:         .test(pytorch_q8dwconv_ukernel_mp8x25__neon);
0394:   }
0395: }
0396: 
0397: TEST(Q8DWCONV_MP8x25__NEON, multi_output_channels_div_8) {
0398:   TEST_REQUIRES_ARM_NEON;
0399:   for (uint32_t channels = 16; channels < 128; channels += 24) {
0400:     DWConvMicrokernelTester()
0401:         .kernelHeight(5)
0402:         .kernelWidth(5)
0403:         .cr(8)
0404:         .channels(channels)
0405:         .width(5)
0406:         .test(pytorch_q8dwconv_ukernel_mp8x25__neon);
0407:   }
0408: }
0409: 
0410: TEST(Q8DWCONV_MP8x25__NEON, multi_output_channels_div_8_with_output_stride) {
0411:   TEST_REQUIRES_ARM_NEON;
0412:   for (uint32_t channels = 16; channels < 128; channels += 24) {
0413:     DWConvMicrokernelTester()
0414:         .kernelHeight(5)
0415:         .kernelWidth(5)
0416:         .cr(8)
0417:         .channels(channels)
0418:         .width(5)
0419:         .outputStride(171)
0420:         .test(pytorch_q8dwconv_ukernel_mp8x25__neon);
0421:   }
0422: }
0423: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `DWConvMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`DWConvMicrokernelTester`。

### Lines 424-477 / 第 424-477 行

```cpp
0424: TEST(Q8DWCONV_MP8x25__NEON, single_output_channels_gt_8) {
0425:   TEST_REQUIRES_ARM_NEON;
0426:   for (uint32_t channels = 9; channels < 16; channels++) {
0427:     DWConvMicrokernelTester()
0428:         .kernelHeight(5)
0429:         .kernelWidth(5)
0430:         .cr(8)
0431:         .channels(channels)
0432:         .width(1)
0433:         .test(pytorch_q8dwconv_ukernel_mp8x25__neon);
0434:   }
0435: }
0436: 
0437: TEST(Q8DWCONV_MP8x25__NEON, single_output_channels_gt_8_with_qmin) {
0438:   TEST_REQUIRES_ARM_NEON;
0439:   for (uint32_t channels = 9; channels < 16; channels++) {
0440:     DWConvMicrokernelTester()
0441:         .kernelHeight(5)
0442:         .kernelWidth(5)
0443:         .cr(8)
0444:         .channels(channels)
0445:         .width(1)
0446:         .qmin(128)
0447:         .test(pytorch_q8dwconv_ukernel_mp8x25__neon);
0448:   }
0449: }
0450: 
0451: TEST(Q8DWCONV_MP8x25__NEON, single_output_channels_gt_8_with_qmax) {
0452:   TEST_REQUIRES_ARM_NEON;
0453:   for (uint32_t channels = 9; channels < 16; channels++) {
0454:     DWConvMicrokernelTester()
0455:         .kernelHeight(5)
0456:         .kernelWidth(5)
0457:         .cr(8)
0458:         .channels(channels)
0459:         .width(1)
0460:         .qmax(128)
0461:         .test(pytorch_q8dwconv_ukernel_mp8x25__neon);
0462:   }
0463: }
0464: 
0465: TEST(Q8DWCONV_MP8x25__NEON, multi_output_channels_gt_8) {
0466:   TEST_REQUIRES_ARM_NEON;
0467:   for (uint32_t channels = 9; channels < 16; channels++) {
0468:     DWConvMicrokernelTester()
0469:         .kernelHeight(5)
0470:         .kernelWidth(5)
0471:         .cr(8)
0472:         .channels(channels)
0473:         .width(5)
0474:         .test(pytorch_q8dwconv_ukernel_mp8x25__neon);
0475:   }
0476: }
0477: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `DWConvMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`DWConvMicrokernelTester`。

### Lines 478-541 / 第 478-541 行

```cpp
0478: TEST(Q8DWCONV_MP8x25__NEON, multi_output_channels_gt_8_with_output_stride) {
0479:   TEST_REQUIRES_ARM_NEON;
0480:   for (uint32_t channels = 9; channels < 16; channels++) {
0481:     DWConvMicrokernelTester()
0482:         .kernelHeight(5)
0483:         .kernelWidth(5)
0484:         .cr(8)
0485:         .channels(channels)
0486:         .width(5)
0487:         .outputStride(17)
0488:         .test(pytorch_q8dwconv_ukernel_mp8x25__neon);
0489:   }
0490: }
0491: 
0492: TEST(Q8DWCONV_UP8x9__NEON, single_output_channels_eq_8_per_channel) {
0493:   TEST_REQUIRES_ARM_NEON;
0494:   DWConvMicrokernelTester()
0495:       .kernelHeight(3)
0496:       .kernelWidth(3)
0497:       .cr(8)
0498:       .channels(8)
0499:       .width(1)
0500:       .test(pytorch_q8dwconv_ukernel_up8x9_per_channel__neon, true);
0501: }
0502: 
0503: TEST(Q8DWCONV_UP8x9__NEON, single_output_channels_eq_8_with_qmin_per_channel) {
0504:   TEST_REQUIRES_ARM_NEON;
0505:   DWConvMicrokernelTester()
0506:       .kernelHeight(3)
0507:       .kernelWidth(3)
0508:       .cr(8)
0509:       .channels(8)
0510:       .width(1)
0511:       .qmin(128)
0512:       .test(pytorch_q8dwconv_ukernel_up8x9_per_channel__neon, true);
0513: }
0514: 
0515: TEST(Q8DWCONV_UP8x9__NEON, single_output_channels_eq_8_with_qmax_per_channel) {
0516:   TEST_REQUIRES_ARM_NEON;
0517:   DWConvMicrokernelTester()
0518:       .kernelHeight(3)
0519:       .kernelWidth(3)
0520:       .cr(8)
0521:       .channels(8)
0522:       .width(1)
0523:       .qmax(128)
0524:       .test(pytorch_q8dwconv_ukernel_up8x9_per_channel__neon, true);
0525: }
0526: 
0527: TEST(
0528:     Q8DWCONV_UP8x9__NEON,
0529:     single_output_channels_eq_8_with_input_zero_point_only_per_channel) {
0530:   TEST_REQUIRES_ARM_NEON;
0531:   DWConvMicrokernelTester()
0532:       .kernelHeight(3)
0533:       .kernelWidth(3)
0534:       .cr(8)
0535:       .channels(8)
0536:       .width(1)
0537:       .inputZeroPoint(255)
0538:       .kernelZeroPoint(0)
0539:       .test(pytorch_q8dwconv_ukernel_up8x9_per_channel__neon, true);
0540: }
0541: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `DWConvMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`DWConvMicrokernelTester`。

### Lines 542-595 / 第 542-595 行

```cpp
0542: TEST(
0543:     Q8DWCONV_UP8x9__NEON,
0544:     single_output_channels_eq_8_with_kernel_zero_point_only_per_channel) {
0545:   TEST_REQUIRES_ARM_NEON;
0546:   DWConvMicrokernelTester()
0547:       .kernelHeight(3)
0548:       .kernelWidth(3)
0549:       .cr(8)
0550:       .channels(8)
0551:       .width(1)
0552:       .inputZeroPoint(0)
0553:       .kernelZeroPoint(255)
0554:       .test(pytorch_q8dwconv_ukernel_up8x9_per_channel__neon, true);
0555: }
0556: 
0557: TEST(Q8DWCONV_UP8x9__NEON, multi_output_channels_eq_8_per_channel) {
0558:   TEST_REQUIRES_ARM_NEON;
0559:   DWConvMicrokernelTester()
0560:       .kernelHeight(3)
0561:       .kernelWidth(3)
0562:       .cr(8)
0563:       .channels(8)
0564:       .width(5)
0565:       .test(pytorch_q8dwconv_ukernel_up8x9_per_channel__neon, true);
0566: }
0567: 
0568: TEST(
0569:     Q8DWCONV_UP8x9__NEON,
0570:     multi_output_channels_eq_8_with_subsampling_per_channel) {
0571:   TEST_REQUIRES_ARM_NEON;
0572:   DWConvMicrokernelTester()
0573:       .kernelHeight(3)
0574:       .kernelWidth(3)
0575:       .subsampling(2)
0576:       .cr(8)
0577:       .channels(8)
0578:       .width(5)
0579:       .test(pytorch_q8dwconv_ukernel_up8x9_per_channel__neon, true);
0580: }
0581: 
0582: TEST(
0583:     Q8DWCONV_UP8x9__NEON,
0584:     multi_output_channels_eq_8_with_input_stride_per_channel) {
0585:   TEST_REQUIRES_ARM_NEON;
0586:   DWConvMicrokernelTester()
0587:       .kernelHeight(3)
0588:       .kernelWidth(3)
0589:       .cr(8)
0590:       .channels(8)
0591:       .width(5)
0592:       .inputStride(17)
0593:       .test(pytorch_q8dwconv_ukernel_up8x9_per_channel__neon, true);
0594: }
0595: 
```

- **EN:** This block implements local helper logic for `q8dwconv`. Key symbols: `DWConvMicrokernelTester`.
- **CN:** 该代码块实现与 `q8dwconv` 相关的局部辅助逻辑。关键符号：`DWConvMicrokernelTester`。

### Lines 596-651 / 第 596-651 行

```cpp
0596: TEST(
0597:     Q8DWCONV_UP8x9__NEON,
0598:     multi_output_channels_eq_8_with_output_stride_per_channel) {
0599:   TEST_REQUIRES_ARM_NEON;
0600:   DWConvMicrokernelTester()
0601:       .kernelHeight(3)
0602:       .kernelWidth(3)
0603:       .cr(8)
0604:       .channels(8)
0605:       .width(5)
0606:       .outputStride(19)
0607:       .test(pytorch_q8dwconv_ukernel_up8x9_per_channel__neon, true);
0608: }
0609: 
0610: TEST(Q8DWCONV_UP8x9__NEON, single_output_channels_div_8_per_channel) {
0611:   TEST_REQUIRES_ARM_NEON;
0612:   for (uint32_t channels = 16; channels < 128; channels += 24) {
0613:     DWConvMicrokernelTester()
0614:         .kernelHeight(3)
0615:         .kernelWidth(3)
0616:         .cr(8)
0617:         .channels(channels)
0618:         .width(1)
0619:         .test(pytorch_q8dwconv_ukernel_up8x9_per_channel__neon, true);
0620:   }
0621: }
0622: 
0623: TEST(Q8DWCONV_UP8x9__NEON, multi_output_channels_div_8_per_channel) {
0624:   TEST_REQUIRES_ARM_NEON;
0625:   for (uint32_t channels = 16; channels < 128; channels += 24) {
0626:     DWConvMicrokernelTester()
0627:         .kernelHeight(3)
0628:         .kernelWidth(3)
0629:         .cr(8)
0630:         .channels(channels)
0631:         .width(5)
0632:         .test(pytorch_q8dwconv_ukernel_up8x9_per_channel__neon, true);
0633:   }
0634: }
0635: 
0636: TEST(
0637:     Q8DWCONV_UP8x9__NEON,
0638:     multi_output_channels_div_8_with_output_stride_per_channel) {
0639:   TEST_REQUIRES_ARM_NEON;
0640:   for (uint32_t channels = 16; channels < 128; channels += 24) {
0641:     DWConvMicrokernelTester()
0642:         .kernelHeight(3)
0643:         .kernelWidth(3)
0644:         .cr(8)
0645:         .channels(channels)
0646:         .width(5)
0647:         .outputStride(171)
0648:         .test(pytorch_q8dwconv_ukernel_up8x9_per_channel__neon, true);
0649:   }
0650: }
0651: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `DWConvMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`DWConvMicrokernelTester`。

### Lines 652-709 / 第 652-709 行

```cpp
0652: TEST(Q8DWCONV_UP8x9__NEON, single_output_channels_gt_8_per_channel) {
0653:   TEST_REQUIRES_ARM_NEON;
0654:   for (uint32_t channels = 9; channels < 16; channels++) {
0655:     DWConvMicrokernelTester()
0656:         .kernelHeight(3)
0657:         .kernelWidth(3)
0658:         .cr(8)
0659:         .channels(channels)
0660:         .width(1)
0661:         .test(pytorch_q8dwconv_ukernel_up8x9_per_channel__neon, true);
0662:   }
0663: }
0664: 
0665: TEST(Q8DWCONV_UP8x9__NEON, single_output_channels_gt_8_with_qmin_per_channel) {
0666:   TEST_REQUIRES_ARM_NEON;
0667:   for (uint32_t channels = 9; channels < 16; channels++) {
0668:     DWConvMicrokernelTester()
0669:         .kernelHeight(3)
0670:         .kernelWidth(3)
0671:         .cr(8)
0672:         .channels(channels)
0673:         .width(1)
0674:         .qmin(128)
0675:         .test(pytorch_q8dwconv_ukernel_up8x9_per_channel__neon, true);
0676:   }
0677: }
0678: 
0679: TEST(Q8DWCONV_UP8x9__NEON, single_output_channels_gt_8_with_qmax_per_channel) {
0680:   TEST_REQUIRES_ARM_NEON;
0681:   for (uint32_t channels = 9; channels < 16; channels++) {
0682:     DWConvMicrokernelTester()
0683:         .kernelHeight(3)
0684:         .kernelWidth(3)
0685:         .cr(8)
0686:         .channels(channels)
0687:         .width(1)
0688:         .qmax(128)
0689:         .test(pytorch_q8dwconv_ukernel_up8x9_per_channel__neon, true);
0690:   }
0691: }
0692: 
0693: TEST(
0694:     Q8DWCONV_UP8x9__NEON,
0695:     single_output_channels_gt_8_with_input_zero_point_only_per_channel) {
0696:   TEST_REQUIRES_ARM_NEON;
0697:   for (uint32_t channels = 9; channels < 16; channels++) {
0698:     DWConvMicrokernelTester()
0699:         .kernelHeight(3)
0700:         .kernelWidth(3)
0701:         .cr(8)
0702:         .channels(channels)
0703:         .width(1)
0704:         .inputZeroPoint(255)
0705:         .kernelZeroPoint(0)
0706:         .test(pytorch_q8dwconv_ukernel_up8x9_per_channel__neon, true);
0707:   }
0708: }
0709: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `DWConvMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`DWConvMicrokernelTester`。

### Lines 710-766 / 第 710-766 行

```cpp
0710: TEST(
0711:     Q8DWCONV_UP8x9__NEON,
0712:     single_output_channels_gt_8_with_kernel_zero_point_only_per_channel) {
0713:   TEST_REQUIRES_ARM_NEON;
0714:   for (uint32_t channels = 9; channels < 16; channels++) {
0715:     DWConvMicrokernelTester()
0716:         .kernelHeight(3)
0717:         .kernelWidth(3)
0718:         .cr(8)
0719:         .channels(channels)
0720:         .width(1)
0721:         .inputZeroPoint(0)
0722:         .kernelZeroPoint(255)
0723:         .test(pytorch_q8dwconv_ukernel_up8x9_per_channel__neon, true);
0724:   }
0725: }
0726: 
0727: TEST(Q8DWCONV_UP8x9__NEON, multi_output_channels_gt_8_per_channel) {
0728:   TEST_REQUIRES_ARM_NEON;
0729:   for (uint32_t channels = 9; channels < 16; channels++) {
0730:     DWConvMicrokernelTester()
0731:         .kernelHeight(3)
0732:         .kernelWidth(3)
0733:         .cr(8)
0734:         .channels(channels)
0735:         .width(5)
0736:         .test(pytorch_q8dwconv_ukernel_up8x9_per_channel__neon, true);
0737:   }
0738: }
0739: 
0740: TEST(
0741:     Q8DWCONV_UP8x9__NEON,
0742:     multi_output_channels_gt_8_with_output_stride_per_channel) {
0743:   TEST_REQUIRES_ARM_NEON;
0744:   for (uint32_t channels = 9; channels < 16; channels++) {
0745:     DWConvMicrokernelTester()
0746:         .kernelHeight(3)
0747:         .kernelWidth(3)
0748:         .cr(8)
0749:         .channels(channels)
0750:         .width(5)
0751:         .outputStride(17)
0752:         .test(pytorch_q8dwconv_ukernel_up8x9_per_channel__neon, true);
0753:   }
0754: }
0755: 
0756: TEST(Q8DWCONV_MP8x25__NEON, single_output_channels_eq_8_per_channel) {
0757:   TEST_REQUIRES_ARM_NEON;
0758:   DWConvMicrokernelTester()
0759:       .kernelHeight(5)
0760:       .kernelWidth(5)
0761:       .cr(8)
0762:       .channels(8)
0763:       .width(1)
0764:       .test(pytorch_q8dwconv_ukernel_mp8x25_per_channel__neon, true);
0765: }
0766: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `DWConvMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`DWConvMicrokernelTester`。

### Lines 767-822 / 第 767-822 行

```cpp
0767: TEST(
0768:     Q8DWCONV_MP8x25__NEON,
0769:     multi_output_channels_eq_8_with_subsampling_per_channel) {
0770:   TEST_REQUIRES_ARM_NEON;
0771:   DWConvMicrokernelTester()
0772:       .kernelHeight(5)
0773:       .kernelWidth(5)
0774:       .subsampling(2)
0775:       .cr(8)
0776:       .channels(8)
0777:       .width(5)
0778:       .test(pytorch_q8dwconv_ukernel_mp8x25_per_channel__neon, true);
0779: }
0780: 
0781: TEST(
0782:     Q8DWCONV_MP8x25__NEON,
0783:     multi_output_channels_eq_8_with_input_stride_per_channel) {
0784:   TEST_REQUIRES_ARM_NEON;
0785:   DWConvMicrokernelTester()
0786:       .kernelHeight(5)
0787:       .kernelWidth(5)
0788:       .cr(8)
0789:       .channels(8)
0790:       .width(5)
0791:       .inputStride(17)
0792:       .test(pytorch_q8dwconv_ukernel_mp8x25_per_channel__neon, true);
0793: }
0794: 
0795: TEST(
0796:     Q8DWCONV_MP8x25__NEON,
0797:     multi_output_channels_eq_8_with_output_stride_per_channel) {
0798:   TEST_REQUIRES_ARM_NEON;
0799:   DWConvMicrokernelTester()
0800:       .kernelHeight(5)
0801:       .kernelWidth(5)
0802:       .cr(8)
0803:       .channels(8)
0804:       .width(5)
0805:       .outputStride(19)
0806:       .test(pytorch_q8dwconv_ukernel_mp8x25_per_channel__neon, true);
0807: }
0808: 
0809: TEST(
0810:     Q8DWCONV_MP8x25__NEON,
0811:     single_output_channels_eq_8_with_qmin_per_channel) {
0812:   TEST_REQUIRES_ARM_NEON;
0813:   DWConvMicrokernelTester()
0814:       .kernelHeight(5)
0815:       .kernelWidth(5)
0816:       .cr(8)
0817:       .channels(8)
0818:       .width(1)
0819:       .qmin(128)
0820:       .test(pytorch_q8dwconv_ukernel_mp8x25_per_channel__neon, true);
0821: }
0822: 
```

- **EN:** This block implements local helper logic for `q8dwconv`. Key symbols: `DWConvMicrokernelTester`.
- **CN:** 该代码块实现与 `q8dwconv` 相关的局部辅助逻辑。关键符号：`DWConvMicrokernelTester`。

### Lines 823-877 / 第 823-877 行

```cpp
0823: TEST(
0824:     Q8DWCONV_MP8x25__NEON,
0825:     single_output_channels_eq_8_with_qmax_per_channel) {
0826:   TEST_REQUIRES_ARM_NEON;
0827:   DWConvMicrokernelTester()
0828:       .kernelHeight(5)
0829:       .kernelWidth(5)
0830:       .cr(8)
0831:       .channels(8)
0832:       .width(1)
0833:       .qmax(128)
0834:       .test(pytorch_q8dwconv_ukernel_mp8x25_per_channel__neon, true);
0835: }
0836: 
0837: TEST(
0838:     Q8DWCONV_MP8x25__NEON,
0839:     single_output_channels_eq_8_with_input_zero_point_only_per_channel) {
0840:   TEST_REQUIRES_ARM_NEON;
0841:   DWConvMicrokernelTester()
0842:       .kernelHeight(5)
0843:       .kernelWidth(5)
0844:       .cr(8)
0845:       .channels(8)
0846:       .width(1)
0847:       .inputZeroPoint(255)
0848:       .kernelZeroPoint(0)
0849:       .test(pytorch_q8dwconv_ukernel_mp8x25_per_channel__neon, true);
0850: }
0851: 
0852: TEST(
0853:     Q8DWCONV_MP8x25__NEON,
0854:     single_output_channels_eq_8_with_kernel_zero_point_only_per_channel) {
0855:   TEST_REQUIRES_ARM_NEON;
0856:   DWConvMicrokernelTester()
0857:       .kernelHeight(5)
0858:       .kernelWidth(5)
0859:       .cr(8)
0860:       .channels(8)
0861:       .width(1)
0862:       .inputZeroPoint(0)
0863:       .kernelZeroPoint(255)
0864:       .test(pytorch_q8dwconv_ukernel_mp8x25_per_channel__neon, true);
0865: }
0866: 
0867: TEST(Q8DWCONV_MP8x25__NEON, multi_output_channels_eq_8_per_channel) {
0868:   TEST_REQUIRES_ARM_NEON;
0869:   DWConvMicrokernelTester()
0870:       .kernelHeight(5)
0871:       .kernelWidth(5)
0872:       .cr(8)
0873:       .channels(8)
0874:       .width(3)
0875:       .test(pytorch_q8dwconv_ukernel_mp8x25_per_channel__neon, true);
0876: }
0877: 
```

- **EN:** This block implements local helper logic for `q8dwconv`. Key symbols: `DWConvMicrokernelTester`.
- **CN:** 该代码块实现与 `q8dwconv` 相关的局部辅助逻辑。关键符号：`DWConvMicrokernelTester`。

### Lines 878-932 / 第 878-932 行

```cpp
0878: TEST(Q8DWCONV_MP8x25__NEON, single_output_channels_div_8_per_channel) {
0879:   TEST_REQUIRES_ARM_NEON;
0880:   for (uint32_t channels = 16; channels < 128; channels += 24) {
0881:     DWConvMicrokernelTester()
0882:         .kernelHeight(5)
0883:         .kernelWidth(5)
0884:         .cr(8)
0885:         .channels(channels)
0886:         .width(1)
0887:         .test(pytorch_q8dwconv_ukernel_mp8x25_per_channel__neon, true);
0888:   }
0889: }
0890: 
0891: TEST(Q8DWCONV_MP8x25__NEON, multi_output_channels_div_8_per_channel) {
0892:   TEST_REQUIRES_ARM_NEON;
0893:   for (uint32_t channels = 16; channels < 128; channels += 24) {
0894:     DWConvMicrokernelTester()
0895:         .kernelHeight(5)
0896:         .kernelWidth(5)
0897:         .cr(8)
0898:         .channels(channels)
0899:         .width(5)
0900:         .test(pytorch_q8dwconv_ukernel_mp8x25_per_channel__neon, true);
0901:   }
0902: }
0903: 
0904: TEST(
0905:     Q8DWCONV_MP8x25__NEON,
0906:     multi_output_channels_div_8_with_output_stride_per_channel) {
0907:   TEST_REQUIRES_ARM_NEON;
0908:   for (uint32_t channels = 16; channels < 128; channels += 24) {
0909:     DWConvMicrokernelTester()
0910:         .kernelHeight(5)
0911:         .kernelWidth(5)
0912:         .cr(8)
0913:         .channels(channels)
0914:         .width(5)
0915:         .outputStride(171)
0916:         .test(pytorch_q8dwconv_ukernel_mp8x25_per_channel__neon, true);
0917:   }
0918: }
0919: 
0920: TEST(Q8DWCONV_MP8x25__NEON, single_output_channels_gt_8_per_channel) {
0921:   TEST_REQUIRES_ARM_NEON;
0922:   for (uint32_t channels = 9; channels < 16; channels++) {
0923:     DWConvMicrokernelTester()
0924:         .kernelHeight(5)
0925:         .kernelWidth(5)
0926:         .cr(8)
0927:         .channels(channels)
0928:         .width(1)
0929:         .test(pytorch_q8dwconv_ukernel_mp8x25_per_channel__neon, true);
0930:   }
0931: }
0932: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `DWConvMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`DWConvMicrokernelTester`。

### Lines 933-982 / 第 933-982 行

```cpp
0933: TEST(
0934:     Q8DWCONV_MP8x25__NEON,
0935:     single_output_channels_gt_8_with_qmin_per_channel) {
0936:   TEST_REQUIRES_ARM_NEON;
0937:   for (uint32_t channels = 9; channels < 16; channels++) {
0938:     DWConvMicrokernelTester()
0939:         .kernelHeight(5)
0940:         .kernelWidth(5)
0941:         .cr(8)
0942:         .channels(channels)
0943:         .width(1)
0944:         .qmin(128)
0945:         .test(pytorch_q8dwconv_ukernel_mp8x25_per_channel__neon, true);
0946:   }
0947: }
0948: 
0949: TEST(
0950:     Q8DWCONV_MP8x25__NEON,
0951:     single_output_channels_gt_8_with_qmax_per_channel) {
0952:   TEST_REQUIRES_ARM_NEON;
0953:   for (uint32_t channels = 9; channels < 16; channels++) {
0954:     DWConvMicrokernelTester()
0955:         .kernelHeight(5)
0956:         .kernelWidth(5)
0957:         .cr(8)
0958:         .channels(channels)
0959:         .width(1)
0960:         .qmax(128)
0961:         .test(pytorch_q8dwconv_ukernel_mp8x25_per_channel__neon, true);
0962:   }
0963: }
0964: 
0965: TEST(Q8DWCONV_MP8x25__NEON, multi_output_channels_gt_8_per_channel) {
0966:   TEST_REQUIRES_ARM_NEON;
0967:   for (uint32_t channels = 9; channels < 16; channels++) {
0968:     DWConvMicrokernelTester()
0969:         .kernelHeight(5)
0970:         .kernelWidth(5)
0971:         .cr(8)
0972:         .channels(channels)
0973:         .width(5)
0974:         .test(pytorch_q8dwconv_ukernel_mp8x25_per_channel__neon, true);
0975:   }
0976: }
0977: 
0978: TEST(
0979:     Q8DWCONV_MP8x25__NEON,
0980:     multi_output_channels_gt_8_with_output_stride_per_channel) {
0981:   TEST_REQUIRES_ARM_NEON;
0982:   for (uint32_t channels = 9; channels < 16; channels++) {
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `DWConvMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`DWConvMicrokernelTester`。

### Lines 983-1045 / 第 983-1045 行

```cpp
0983:     DWConvMicrokernelTester()
0984:         .kernelHeight(5)
0985:         .kernelWidth(5)
0986:         .cr(8)
0987:         .channels(channels)
0988:         .width(5)
0989:         .outputStride(17)
0990:         .test(pytorch_q8dwconv_ukernel_mp8x25_per_channel__neon, true);
0991:   }
0992: }
0993: #endif /* CPUINFO_ARCH_ARM || CPUINFO_ARCH_ARM64 */
0994: 
0995: #if CPUINFO_ARCH_ARM
0996: TEST(Q8DWCONV_UP8x9__AARCH32_NEON, single_output_channels_eq_8) {
0997:   TEST_REQUIRES_ARM_NEON;
0998:   DWConvMicrokernelTester()
0999:       .kernelHeight(3)
1000:       .kernelWidth(3)
1001:       .cr(8)
1002:       .channels(8)
1003:       .width(1)
1004:       .test(pytorch_q8dwconv_ukernel_up8x9__aarch32_neon);
1005: }
1006: 
1007: TEST(Q8DWCONV_UP8x9__AARCH32_NEON, single_output_channels_eq_8_with_qmin) {
1008:   TEST_REQUIRES_ARM_NEON;
1009:   DWConvMicrokernelTester()
1010:       .kernelHeight(3)
1011:       .kernelWidth(3)
1012:       .cr(8)
1013:       .channels(8)
1014:       .width(1)
1015:       .qmin(128)
1016:       .test(pytorch_q8dwconv_ukernel_up8x9__aarch32_neon);
1017: }
1018: 
1019: TEST(Q8DWCONV_UP8x9__AARCH32_NEON, single_output_channels_eq_8_with_qmax) {
1020:   TEST_REQUIRES_ARM_NEON;
1021:   DWConvMicrokernelTester()
1022:       .kernelHeight(3)
1023:       .kernelWidth(3)
1024:       .cr(8)
1025:       .channels(8)
1026:       .width(1)
1027:       .qmax(128)
1028:       .test(pytorch_q8dwconv_ukernel_up8x9__aarch32_neon);
1029: }
1030: 
1031: TEST(
1032:     Q8DWCONV_UP8x9__AARCH32_NEON,
1033:     single_output_channels_eq_8_with_input_zero_point_only) {
1034:   TEST_REQUIRES_ARM_NEON;
1035:   DWConvMicrokernelTester()
1036:       .kernelHeight(3)
1037:       .kernelWidth(3)
1038:       .cr(8)
1039:       .channels(8)
1040:       .width(1)
1041:       .inputZeroPoint(255)
1042:       .kernelZeroPoint(0)
1043:       .test(pytorch_q8dwconv_ukernel_up8x9__aarch32_neon);
1044: }
1045: 
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends. Key symbols: `DWConvMicrokernelTester`.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端。关键符号：`DWConvMicrokernelTester`。

### Lines 1046-1099 / 第 1046-1099 行

```cpp
1046: TEST(
1047:     Q8DWCONV_UP8x9__AARCH32_NEON,
1048:     single_output_channels_eq_8_with_kernel_zero_point_only) {
1049:   TEST_REQUIRES_ARM_NEON;
1050:   DWConvMicrokernelTester()
1051:       .kernelHeight(3)
1052:       .kernelWidth(3)
1053:       .cr(8)
1054:       .channels(8)
1055:       .width(1)
1056:       .inputZeroPoint(0)
1057:       .kernelZeroPoint(255)
1058:       .test(pytorch_q8dwconv_ukernel_up8x9__aarch32_neon);
1059: }
1060: 
1061: TEST(Q8DWCONV_UP8x9__AARCH32_NEON, multi_output_channels_eq_8) {
1062:   TEST_REQUIRES_ARM_NEON;
1063:   DWConvMicrokernelTester()
1064:       .kernelHeight(3)
1065:       .kernelWidth(3)
1066:       .cr(8)
1067:       .channels(8)
1068:       .width(5)
1069:       .test(pytorch_q8dwconv_ukernel_up8x9__aarch32_neon);
1070: }
1071: 
1072: TEST(
1073:     Q8DWCONV_UP8x9__AARCH32_NEON,
1074:     multi_output_channels_eq_8_with_subsampling) {
1075:   TEST_REQUIRES_ARM_NEON;
1076:   DWConvMicrokernelTester()
1077:       .kernelHeight(3)
1078:       .kernelWidth(3)
1079:       .subsampling(2)
1080:       .cr(8)
1081:       .channels(8)
1082:       .width(5)
1083:       .test(pytorch_q8dwconv_ukernel_up8x9__aarch32_neon);
1084: }
1085: 
1086: TEST(
1087:     Q8DWCONV_UP8x9__AARCH32_NEON,
1088:     multi_output_channels_eq_8_with_input_stride) {
1089:   TEST_REQUIRES_ARM_NEON;
1090:   DWConvMicrokernelTester()
1091:       .kernelHeight(3)
1092:       .kernelWidth(3)
1093:       .cr(8)
1094:       .channels(8)
1095:       .width(5)
1096:       .inputStride(17)
1097:       .test(pytorch_q8dwconv_ukernel_up8x9__aarch32_neon);
1098: }
1099: 
```

- **EN:** This block implements local helper logic for `q8dwconv`. Key symbols: `DWConvMicrokernelTester`.
- **CN:** 该代码块实现与 `q8dwconv` 相关的局部辅助逻辑。关键符号：`DWConvMicrokernelTester`。

### Lines 1100-1155 / 第 1100-1155 行

```cpp
1100: TEST(
1101:     Q8DWCONV_UP8x9__AARCH32_NEON,
1102:     multi_output_channels_eq_8_with_output_stride) {
1103:   TEST_REQUIRES_ARM_NEON;
1104:   DWConvMicrokernelTester()
1105:       .kernelHeight(3)
1106:       .kernelWidth(3)
1107:       .cr(8)
1108:       .channels(8)
1109:       .width(5)
1110:       .outputStride(19)
1111:       .test(pytorch_q8dwconv_ukernel_up8x9__aarch32_neon);
1112: }
1113: 
1114: TEST(Q8DWCONV_UP8x9__AARCH32_NEON, single_output_channels_div_8) {
1115:   TEST_REQUIRES_ARM_NEON;
1116:   for (uint32_t channels = 16; channels < 128; channels += 24) {
1117:     DWConvMicrokernelTester()
1118:         .kernelHeight(3)
1119:         .kernelWidth(3)
1120:         .cr(8)
1121:         .channels(channels)
1122:         .width(1)
1123:         .test(pytorch_q8dwconv_ukernel_up8x9__aarch32_neon);
1124:   }
1125: }
1126: 
1127: TEST(Q8DWCONV_UP8x9__AARCH32_NEON, multi_output_channels_div_8) {
1128:   TEST_REQUIRES_ARM_NEON;
1129:   for (uint32_t channels = 16; channels < 128; channels += 24) {
1130:     DWConvMicrokernelTester()
1131:         .kernelHeight(3)
1132:         .kernelWidth(3)
1133:         .cr(8)
1134:         .channels(channels)
1135:         .width(5)
1136:         .test(pytorch_q8dwconv_ukernel_up8x9__aarch32_neon);
1137:   }
1138: }
1139: 
1140: TEST(
1141:     Q8DWCONV_UP8x9__AARCH32_NEON,
1142:     multi_output_channels_div_8_with_output_stride) {
1143:   TEST_REQUIRES_ARM_NEON;
1144:   for (uint32_t channels = 16; channels < 128; channels += 24) {
1145:     DWConvMicrokernelTester()
1146:         .kernelHeight(3)
1147:         .kernelWidth(3)
1148:         .cr(8)
1149:         .channels(channels)
1150:         .width(5)
1151:         .outputStride(171)
1152:         .test(pytorch_q8dwconv_ukernel_up8x9__aarch32_neon);
1153:   }
1154: }
1155: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `DWConvMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`DWConvMicrokernelTester`。

### Lines 1156-1213 / 第 1156-1213 行

```cpp
1156: TEST(Q8DWCONV_UP8x9__AARCH32_NEON, single_output_channels_gt_8) {
1157:   TEST_REQUIRES_ARM_NEON;
1158:   for (uint32_t channels = 9; channels < 16; channels++) {
1159:     DWConvMicrokernelTester()
1160:         .kernelHeight(3)
1161:         .kernelWidth(3)
1162:         .cr(8)
1163:         .channels(channels)
1164:         .width(1)
1165:         .test(pytorch_q8dwconv_ukernel_up8x9__aarch32_neon);
1166:   }
1167: }
1168: 
1169: TEST(Q8DWCONV_UP8x9__AARCH32_NEON, single_output_channels_gt_8_with_qmin) {
1170:   TEST_REQUIRES_ARM_NEON;
1171:   for (uint32_t channels = 9; channels < 16; channels++) {
1172:     DWConvMicrokernelTester()
1173:         .kernelHeight(3)
1174:         .kernelWidth(3)
1175:         .cr(8)
1176:         .channels(channels)
1177:         .width(1)
1178:         .qmin(128)
1179:         .test(pytorch_q8dwconv_ukernel_up8x9__aarch32_neon);
1180:   }
1181: }
1182: 
1183: TEST(Q8DWCONV_UP8x9__AARCH32_NEON, single_output_channels_gt_8_with_qmax) {
1184:   TEST_REQUIRES_ARM_NEON;
1185:   for (uint32_t channels = 9; channels < 16; channels++) {
1186:     DWConvMicrokernelTester()
1187:         .kernelHeight(3)
1188:         .kernelWidth(3)
1189:         .cr(8)
1190:         .channels(channels)
1191:         .width(1)
1192:         .qmax(128)
1193:         .test(pytorch_q8dwconv_ukernel_up8x9__aarch32_neon);
1194:   }
1195: }
1196: 
1197: TEST(
1198:     Q8DWCONV_UP8x9__AARCH32_NEON,
1199:     single_output_channels_gt_8_with_input_zero_point_only) {
1200:   TEST_REQUIRES_ARM_NEON;
1201:   for (uint32_t channels = 9; channels < 16; channels++) {
1202:     DWConvMicrokernelTester()
1203:         .kernelHeight(3)
1204:         .kernelWidth(3)
1205:         .cr(8)
1206:         .channels(channels)
1207:         .width(1)
1208:         .inputZeroPoint(255)
1209:         .kernelZeroPoint(0)
1210:         .test(pytorch_q8dwconv_ukernel_up8x9__aarch32_neon);
1211:   }
1212: }
1213: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `DWConvMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`DWConvMicrokernelTester`。

### Lines 1214-1270 / 第 1214-1270 行

```cpp
1214: TEST(
1215:     Q8DWCONV_UP8x9__AARCH32_NEON,
1216:     single_output_channels_gt_8_with_kernel_zero_point_only) {
1217:   TEST_REQUIRES_ARM_NEON;
1218:   for (uint32_t channels = 9; channels < 16; channels++) {
1219:     DWConvMicrokernelTester()
1220:         .kernelHeight(3)
1221:         .kernelWidth(3)
1222:         .cr(8)
1223:         .channels(channels)
1224:         .width(1)
1225:         .inputZeroPoint(0)
1226:         .kernelZeroPoint(255)
1227:         .test(pytorch_q8dwconv_ukernel_up8x9__aarch32_neon);
1228:   }
1229: }
1230: 
1231: TEST(Q8DWCONV_UP8x9__AARCH32_NEON, multi_output_channels_gt_8) {
1232:   TEST_REQUIRES_ARM_NEON;
1233:   for (uint32_t channels = 9; channels < 16; channels++) {
1234:     DWConvMicrokernelTester()
1235:         .kernelHeight(3)
1236:         .kernelWidth(3)
1237:         .cr(8)
1238:         .channels(channels)
1239:         .width(5)
1240:         .test(pytorch_q8dwconv_ukernel_up8x9__aarch32_neon);
1241:   }
1242: }
1243: 
1244: TEST(
1245:     Q8DWCONV_UP8x9__AARCH32_NEON,
1246:     multi_output_channels_gt_8_with_output_stride) {
1247:   TEST_REQUIRES_ARM_NEON;
1248:   for (uint32_t channels = 9; channels < 16; channels++) {
1249:     DWConvMicrokernelTester()
1250:         .kernelHeight(3)
1251:         .kernelWidth(3)
1252:         .cr(8)
1253:         .channels(channels)
1254:         .width(5)
1255:         .outputStride(17)
1256:         .test(pytorch_q8dwconv_ukernel_up8x9__aarch32_neon);
1257:   }
1258: }
1259: 
1260: TEST(Q8DWCONV_UP8x9__AARCH32_NEON, single_output_channels_eq_8_per_channel) {
1261:   TEST_REQUIRES_ARM_NEON;
1262:   DWConvMicrokernelTester()
1263:       .kernelHeight(3)
1264:       .kernelWidth(3)
1265:       .cr(8)
1266:       .channels(8)
1267:       .width(1)
1268:       .test(pytorch_q8dwconv_ukernel_up8x9_per_channel__aarch32_neon, true);
1269: }
1270: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `DWConvMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`DWConvMicrokernelTester`。

### Lines 1271-1324 / 第 1271-1324 行

```cpp
1271: TEST(Q8DWCONV_UP8x9__AARCH32_NEON, single_output_channels_eq_8_with_qmin_per_channel) {
1272:   TEST_REQUIRES_ARM_NEON;
1273:   DWConvMicrokernelTester()
1274:       .kernelHeight(3)
1275:       .kernelWidth(3)
1276:       .cr(8)
1277:       .channels(8)
1278:       .width(1)
1279:       .qmin(128)
1280:       .test(pytorch_q8dwconv_ukernel_up8x9_per_channel__aarch32_neon, true);
1281: }
1282: 
1283: TEST(Q8DWCONV_UP8x9__AARCH32_NEON, single_output_channels_eq_8_with_qmax_per_channel) {
1284:   TEST_REQUIRES_ARM_NEON;
1285:   DWConvMicrokernelTester()
1286:       .kernelHeight(3)
1287:       .kernelWidth(3)
1288:       .cr(8)
1289:       .channels(8)
1290:       .width(1)
1291:       .qmax(128)
1292:       .test(pytorch_q8dwconv_ukernel_up8x9_per_channel__aarch32_neon, true);
1293: }
1294: 
1295: TEST(
1296:     Q8DWCONV_UP8x9__AARCH32_NEON,
1297:     single_output_channels_eq_8_with_input_zero_point_only_per_channel) {
1298:   TEST_REQUIRES_ARM_NEON;
1299:   DWConvMicrokernelTester()
1300:       .kernelHeight(3)
1301:       .kernelWidth(3)
1302:       .cr(8)
1303:       .channels(8)
1304:       .width(1)
1305:       .inputZeroPoint(255)
1306:       .kernelZeroPoint(0)
1307:       .test(pytorch_q8dwconv_ukernel_up8x9_per_channel__aarch32_neon, true);
1308: }
1309: 
1310: TEST(
1311:     Q8DWCONV_UP8x9__AARCH32_NEON,
1312:     single_output_channels_eq_8_with_kernel_zero_point_only_per_channel) {
1313:   TEST_REQUIRES_ARM_NEON;
1314:   DWConvMicrokernelTester()
1315:       .kernelHeight(3)
1316:       .kernelWidth(3)
1317:       .cr(8)
1318:       .channels(8)
1319:       .width(1)
1320:       .inputZeroPoint(0)
1321:       .kernelZeroPoint(255)
1322:       .test(pytorch_q8dwconv_ukernel_up8x9_per_channel__aarch32_neon, true);
1323: }
1324: 
```

- **EN:** This block implements local helper logic for `q8dwconv`. Key symbols: `DWConvMicrokernelTester`.
- **CN:** 该代码块实现与 `q8dwconv` 相关的局部辅助逻辑。关键符号：`DWConvMicrokernelTester`。

### Lines 1325-1377 / 第 1325-1377 行

```cpp
1325: TEST(Q8DWCONV_UP8x9__AARCH32_NEON, multi_output_channels_eq_8_per_channel) {
1326:   TEST_REQUIRES_ARM_NEON;
1327:   DWConvMicrokernelTester()
1328:       .kernelHeight(3)
1329:       .kernelWidth(3)
1330:       .cr(8)
1331:       .channels(8)
1332:       .width(5)
1333:       .test(pytorch_q8dwconv_ukernel_up8x9_per_channel__aarch32_neon, true);
1334: }
1335: 
1336: TEST(
1337:     Q8DWCONV_UP8x9__AARCH32_NEON,
1338:     multi_output_channels_eq_8_with_subsampling_per_channel) {
1339:   TEST_REQUIRES_ARM_NEON;
1340:   DWConvMicrokernelTester()
1341:       .kernelHeight(3)
1342:       .kernelWidth(3)
1343:       .subsampling(2)
1344:       .cr(8)
1345:       .channels(8)
1346:       .width(5)
1347:       .test(pytorch_q8dwconv_ukernel_up8x9_per_channel__aarch32_neon, true);
1348: }
1349: 
1350: TEST(
1351:     Q8DWCONV_UP8x9__AARCH32_NEON,
1352:     multi_output_channels_eq_8_with_input_stride_per_channel) {
1353:   TEST_REQUIRES_ARM_NEON;
1354:   DWConvMicrokernelTester()
1355:       .kernelHeight(3)
1356:       .kernelWidth(3)
1357:       .cr(8)
1358:       .channels(8)
1359:       .width(5)
1360:       .inputStride(17)
1361:       .test(pytorch_q8dwconv_ukernel_up8x9_per_channel__aarch32_neon, true);
1362: }
1363: 
1364: TEST(
1365:     Q8DWCONV_UP8x9__AARCH32_NEON,
1366:     multi_output_channels_eq_8_with_output_stride_per_channel) {
1367:   TEST_REQUIRES_ARM_NEON;
1368:   DWConvMicrokernelTester()
1369:       .kernelHeight(3)
1370:       .kernelWidth(3)
1371:       .cr(8)
1372:       .channels(8)
1373:       .width(5)
1374:       .outputStride(19)
1375:       .test(pytorch_q8dwconv_ukernel_up8x9_per_channel__aarch32_neon, true);
1376: }
1377: 
```

- **EN:** This block implements local helper logic for `q8dwconv`. Key symbols: `DWConvMicrokernelTester`.
- **CN:** 该代码块实现与 `q8dwconv` 相关的局部辅助逻辑。关键符号：`DWConvMicrokernelTester`。

### Lines 1378-1432 / 第 1378-1432 行

```cpp
1378: TEST(Q8DWCONV_UP8x9__AARCH32_NEON, single_output_channels_div_8_per_channel) {
1379:   TEST_REQUIRES_ARM_NEON;
1380:   for (uint32_t channels = 16; channels < 128; channels += 24) {
1381:     DWConvMicrokernelTester()
1382:         .kernelHeight(3)
1383:         .kernelWidth(3)
1384:         .cr(8)
1385:         .channels(channels)
1386:         .width(1)
1387:         .test(pytorch_q8dwconv_ukernel_up8x9_per_channel__aarch32_neon, true);
1388:   }
1389: }
1390: 
1391: TEST(Q8DWCONV_UP8x9__AARCH32_NEON, multi_output_channels_div_8_per_channel) {
1392:   TEST_REQUIRES_ARM_NEON;
1393:   for (uint32_t channels = 16; channels < 128; channels += 24) {
1394:     DWConvMicrokernelTester()
1395:         .kernelHeight(3)
1396:         .kernelWidth(3)
1397:         .cr(8)
1398:         .channels(channels)
1399:         .width(5)
1400:         .test(pytorch_q8dwconv_ukernel_up8x9_per_channel__aarch32_neon, true);
1401:   }
1402: }
1403: 
1404: TEST(
1405:     Q8DWCONV_UP8x9__AARCH32_NEON,
1406:     multi_output_channels_div_8_with_output_stride_per_channel) {
1407:   TEST_REQUIRES_ARM_NEON;
1408:   for (uint32_t channels = 16; channels < 128; channels += 24) {
1409:     DWConvMicrokernelTester()
1410:         .kernelHeight(3)
1411:         .kernelWidth(3)
1412:         .cr(8)
1413:         .channels(channels)
1414:         .width(5)
1415:         .outputStride(171)
1416:         .test(pytorch_q8dwconv_ukernel_up8x9_per_channel__aarch32_neon, true);
1417:   }
1418: }
1419: 
1420: TEST(Q8DWCONV_UP8x9__AARCH32_NEON, single_output_channels_gt_8_per_channel) {
1421:   TEST_REQUIRES_ARM_NEON;
1422:   for (uint32_t channels = 9; channels < 16; channels++) {
1423:     DWConvMicrokernelTester()
1424:         .kernelHeight(3)
1425:         .kernelWidth(3)
1426:         .cr(8)
1427:         .channels(channels)
1428:         .width(1)
1429:         .test(pytorch_q8dwconv_ukernel_up8x9_per_channel__aarch32_neon, true);
1430:   }
1431: }
1432: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `DWConvMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`DWConvMicrokernelTester`。

### Lines 1433-1482 / 第 1433-1482 行

```cpp
1433: TEST(Q8DWCONV_UP8x9__AARCH32_NEON, single_output_channels_gt_8_with_qmin_per_channel) {
1434:   TEST_REQUIRES_ARM_NEON;
1435:   for (uint32_t channels = 9; channels < 16; channels++) {
1436:     DWConvMicrokernelTester()
1437:         .kernelHeight(3)
1438:         .kernelWidth(3)
1439:         .cr(8)
1440:         .channels(channels)
1441:         .width(1)
1442:         .qmin(128)
1443:         .test(pytorch_q8dwconv_ukernel_up8x9_per_channel__aarch32_neon, true);
1444:   }
1445: }
1446: 
1447: TEST(Q8DWCONV_UP8x9__AARCH32_NEON, single_output_channels_gt_8_with_qmax_per_channel) {
1448:   TEST_REQUIRES_ARM_NEON;
1449:   for (uint32_t channels = 9; channels < 16; channels++) {
1450:     DWConvMicrokernelTester()
1451:         .kernelHeight(3)
1452:         .kernelWidth(3)
1453:         .cr(8)
1454:         .channels(channels)
1455:         .width(1)
1456:         .qmax(128)
1457:         .test(pytorch_q8dwconv_ukernel_up8x9_per_channel__aarch32_neon, true);
1458:   }
1459: }
1460: 
1461: TEST(
1462:     Q8DWCONV_UP8x9__AARCH32_NEON,
1463:     single_output_channels_gt_8_with_input_zero_point_only_per_channel) {
1464:   TEST_REQUIRES_ARM_NEON;
1465:   for (uint32_t channels = 9; channels < 16; channels++) {
1466:     DWConvMicrokernelTester()
1467:         .kernelHeight(3)
1468:         .kernelWidth(3)
1469:         .cr(8)
1470:         .channels(channels)
1471:         .width(1)
1472:         .inputZeroPoint(255)
1473:         .kernelZeroPoint(0)
1474:         .test(pytorch_q8dwconv_ukernel_up8x9_per_channel__aarch32_neon, true);
1475:   }
1476: }
1477: 
1478: TEST(
1479:     Q8DWCONV_UP8x9__AARCH32_NEON,
1480:     single_output_channels_gt_8_with_kernel_zero_point_only_per_channel) {
1481:   TEST_REQUIRES_ARM_NEON;
1482:   for (uint32_t channels = 9; channels < 16; channels++) {
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `DWConvMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`DWConvMicrokernelTester`。

### Lines 1483-1536 / 第 1483-1536 行

```cpp
1483:     DWConvMicrokernelTester()
1484:         .kernelHeight(3)
1485:         .kernelWidth(3)
1486:         .cr(8)
1487:         .channels(channels)
1488:         .width(1)
1489:         .inputZeroPoint(0)
1490:         .kernelZeroPoint(255)
1491:         .test(pytorch_q8dwconv_ukernel_up8x9_per_channel__aarch32_neon, true);
1492:   }
1493: }
1494: 
1495: TEST(Q8DWCONV_UP8x9__AARCH32_NEON, multi_output_channels_gt_8_per_channel) {
1496:   TEST_REQUIRES_ARM_NEON;
1497:   for (uint32_t channels = 9; channels < 16; channels++) {
1498:     DWConvMicrokernelTester()
1499:         .kernelHeight(3)
1500:         .kernelWidth(3)
1501:         .cr(8)
1502:         .channels(channels)
1503:         .width(5)
1504:         .test(pytorch_q8dwconv_ukernel_up8x9_per_channel__aarch32_neon, true);
1505:   }
1506: }
1507: 
1508: TEST(
1509:     Q8DWCONV_UP8x9__AARCH32_NEON,
1510:     multi_output_channels_gt_8_with_output_stride_per_channel) {
1511:   TEST_REQUIRES_ARM_NEON;
1512:   for (uint32_t channels = 9; channels < 16; channels++) {
1513:     DWConvMicrokernelTester()
1514:         .kernelHeight(3)
1515:         .kernelWidth(3)
1516:         .cr(8)
1517:         .channels(channels)
1518:         .width(5)
1519:         .outputStride(17)
1520:         .test(pytorch_q8dwconv_ukernel_up8x9_per_channel__aarch32_neon, true);
1521:   }
1522: }
1523: #endif /* CPUINFO_ARCH_ARM */
1524: 
1525: #if CPUINFO_ARCH_X86 || CPUINFO_ARCH_X86_64
1526: TEST(Q8DWCONV_UP8x9__SSE2, single_output_channels_eq_8) {
1527:   TEST_REQUIRES_X86_SSE2;
1528:   DWConvMicrokernelTester()
1529:       .kernelHeight(3)
1530:       .kernelWidth(3)
1531:       .cr(8)
1532:       .channels(8)
1533:       .width(1)
1534:       .test(pytorch_q8dwconv_ukernel_up8x9__sse2);
1535: }
1536: 
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends; iterates over tensors, descriptors, options, or runtime state. Key symbols: `DWConvMicrokernelTester`.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端；遍历张量、描述符、配置项或运行时状态。关键符号：`DWConvMicrokernelTester`。

### Lines 1537-1590 / 第 1537-1590 行

```cpp
1537: TEST(Q8DWCONV_UP8x9__SSE2, single_output_channels_eq_8_with_qmin) {
1538:   TEST_REQUIRES_X86_SSE2;
1539:   DWConvMicrokernelTester()
1540:       .kernelHeight(3)
1541:       .kernelWidth(3)
1542:       .cr(8)
1543:       .channels(8)
1544:       .width(1)
1545:       .qmin(128)
1546:       .test(pytorch_q8dwconv_ukernel_up8x9__sse2);
1547: }
1548: 
1549: TEST(Q8DWCONV_UP8x9__SSE2, single_output_channels_eq_8_with_qmax) {
1550:   TEST_REQUIRES_X86_SSE2;
1551:   DWConvMicrokernelTester()
1552:       .kernelHeight(3)
1553:       .kernelWidth(3)
1554:       .cr(8)
1555:       .channels(8)
1556:       .width(1)
1557:       .qmax(128)
1558:       .test(pytorch_q8dwconv_ukernel_up8x9__sse2);
1559: }
1560: 
1561: TEST(
1562:     Q8DWCONV_UP8x9__SSE2,
1563:     single_output_channels_eq_8_with_input_zero_point_only) {
1564:   TEST_REQUIRES_X86_SSE2;
1565:   DWConvMicrokernelTester()
1566:       .kernelHeight(3)
1567:       .kernelWidth(3)
1568:       .cr(8)
1569:       .channels(8)
1570:       .width(1)
1571:       .inputZeroPoint(255)
1572:       .kernelZeroPoint(0)
1573:       .test(pytorch_q8dwconv_ukernel_up8x9__sse2);
1574: }
1575: 
1576: TEST(
1577:     Q8DWCONV_UP8x9__SSE2,
1578:     single_output_channels_eq_8_with_kernel_zero_point_only) {
1579:   TEST_REQUIRES_X86_SSE2;
1580:   DWConvMicrokernelTester()
1581:       .kernelHeight(3)
1582:       .kernelWidth(3)
1583:       .cr(8)
1584:       .channels(8)
1585:       .width(1)
1586:       .inputZeroPoint(0)
1587:       .kernelZeroPoint(255)
1588:       .test(pytorch_q8dwconv_ukernel_up8x9__sse2);
1589: }
1590: 
```

- **EN:** This block implements local helper logic for `q8dwconv`. Key symbols: `DWConvMicrokernelTester`.
- **CN:** 该代码块实现与 `q8dwconv` 相关的局部辅助逻辑。关键符号：`DWConvMicrokernelTester`。

### Lines 1591-1640 / 第 1591-1640 行

```cpp
1591: TEST(Q8DWCONV_UP8x9__SSE2, multi_output_channels_eq_8) {
1592:   TEST_REQUIRES_X86_SSE2;
1593:   DWConvMicrokernelTester()
1594:       .kernelHeight(3)
1595:       .kernelWidth(3)
1596:       .cr(8)
1597:       .channels(8)
1598:       .width(5)
1599:       .test(pytorch_q8dwconv_ukernel_up8x9__sse2);
1600: }
1601: 
1602: TEST(Q8DWCONV_UP8x9__SSE2, multi_output_channels_eq_8_with_subsampling) {
1603:   TEST_REQUIRES_X86_SSE2;
1604:   DWConvMicrokernelTester()
1605:       .kernelHeight(3)
1606:       .kernelWidth(3)
1607:       .subsampling(2)
1608:       .cr(8)
1609:       .channels(8)
1610:       .width(5)
1611:       .test(pytorch_q8dwconv_ukernel_up8x9__sse2);
1612: }
1613: 
1614: TEST(Q8DWCONV_UP8x9__SSE2, multi_output_channels_eq_8_with_input_stride) {
1615:   TEST_REQUIRES_X86_SSE2;
1616:   DWConvMicrokernelTester()
1617:       .kernelHeight(3)
1618:       .kernelWidth(3)
1619:       .cr(8)
1620:       .channels(8)
1621:       .width(5)
1622:       .inputStride(17)
1623:       .test(pytorch_q8dwconv_ukernel_up8x9__sse2);
1624: }
1625: 
1626: TEST(Q8DWCONV_UP8x9__SSE2, multi_output_channels_eq_8_with_output_stride) {
1627:   TEST_REQUIRES_X86_SSE2;
1628:   DWConvMicrokernelTester()
1629:       .kernelHeight(3)
1630:       .kernelWidth(3)
1631:       .cr(8)
1632:       .channels(8)
1633:       .width(5)
1634:       .outputStride(19)
1635:       .test(pytorch_q8dwconv_ukernel_up8x9__sse2);
1636: }
1637: 
1638: TEST(Q8DWCONV_UP8x9__SSE2, single_output_channels_div_8) {
1639:   TEST_REQUIRES_X86_SSE2;
1640:   for (uint32_t channels = 16; channels < 128; channels += 24) {
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `DWConvMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`DWConvMicrokernelTester`。

### Lines 1641-1690 / 第 1641-1690 行

```cpp
1641:     DWConvMicrokernelTester()
1642:         .kernelHeight(3)
1643:         .kernelWidth(3)
1644:         .cr(8)
1645:         .channels(channels)
1646:         .width(1)
1647:         .test(pytorch_q8dwconv_ukernel_up8x9__sse2);
1648:   }
1649: }
1650: 
1651: TEST(Q8DWCONV_UP8x9__SSE2, multi_output_channels_div_8) {
1652:   TEST_REQUIRES_X86_SSE2;
1653:   for (uint32_t channels = 16; channels < 128; channels += 24) {
1654:     DWConvMicrokernelTester()
1655:         .kernelHeight(3)
1656:         .kernelWidth(3)
1657:         .cr(8)
1658:         .channels(channels)
1659:         .width(5)
1660:         .test(pytorch_q8dwconv_ukernel_up8x9__sse2);
1661:   }
1662: }
1663: 
1664: TEST(Q8DWCONV_UP8x9__SSE2, multi_output_channels_div_8_with_output_stride) {
1665:   TEST_REQUIRES_X86_SSE2;
1666:   for (uint32_t channels = 16; channels < 128; channels += 24) {
1667:     DWConvMicrokernelTester()
1668:         .kernelHeight(3)
1669:         .kernelWidth(3)
1670:         .cr(8)
1671:         .channels(channels)
1672:         .width(5)
1673:         .outputStride(171)
1674:         .test(pytorch_q8dwconv_ukernel_up8x9__sse2);
1675:   }
1676: }
1677: 
1678: TEST(Q8DWCONV_UP8x9__SSE2, single_output_channels_gt_8) {
1679:   TEST_REQUIRES_X86_SSE2;
1680:   for (uint32_t channels = 9; channels < 16; channels++) {
1681:     DWConvMicrokernelTester()
1682:         .kernelHeight(3)
1683:         .kernelWidth(3)
1684:         .cr(8)
1685:         .channels(channels)
1686:         .width(1)
1687:         .test(pytorch_q8dwconv_ukernel_up8x9__sse2);
1688:   }
1689: }
1690: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `DWConvMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`DWConvMicrokernelTester`。

### Lines 1691-1740 / 第 1691-1740 行

```cpp
1691: TEST(Q8DWCONV_UP8x9__SSE2, single_output_channels_gt_8_with_qmin) {
1692:   TEST_REQUIRES_X86_SSE2;
1693:   for (uint32_t channels = 9; channels < 16; channels++) {
1694:     DWConvMicrokernelTester()
1695:         .kernelHeight(3)
1696:         .kernelWidth(3)
1697:         .cr(8)
1698:         .channels(channels)
1699:         .width(1)
1700:         .qmin(128)
1701:         .test(pytorch_q8dwconv_ukernel_up8x9__sse2);
1702:   }
1703: }
1704: 
1705: TEST(Q8DWCONV_UP8x9__SSE2, single_output_channels_gt_8_with_qmax) {
1706:   TEST_REQUIRES_X86_SSE2;
1707:   for (uint32_t channels = 9; channels < 16; channels++) {
1708:     DWConvMicrokernelTester()
1709:         .kernelHeight(3)
1710:         .kernelWidth(3)
1711:         .cr(8)
1712:         .channels(channels)
1713:         .width(1)
1714:         .qmax(128)
1715:         .test(pytorch_q8dwconv_ukernel_up8x9__sse2);
1716:   }
1717: }
1718: 
1719: TEST(
1720:     Q8DWCONV_UP8x9__SSE2,
1721:     single_output_channels_gt_8_with_input_zero_point_only) {
1722:   TEST_REQUIRES_X86_SSE2;
1723:   for (uint32_t channels = 9; channels < 16; channels++) {
1724:     DWConvMicrokernelTester()
1725:         .kernelHeight(3)
1726:         .kernelWidth(3)
1727:         .cr(8)
1728:         .channels(channels)
1729:         .width(1)
1730:         .inputZeroPoint(255)
1731:         .kernelZeroPoint(0)
1732:         .test(pytorch_q8dwconv_ukernel_up8x9__sse2);
1733:   }
1734: }
1735: 
1736: TEST(
1737:     Q8DWCONV_UP8x9__SSE2,
1738:     single_output_channels_gt_8_with_kernel_zero_point_only) {
1739:   TEST_REQUIRES_X86_SSE2;
1740:   for (uint32_t channels = 9; channels < 16; channels++) {
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `DWConvMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`DWConvMicrokernelTester`。

### Lines 1741-1790 / 第 1741-1790 行

```cpp
1741:     DWConvMicrokernelTester()
1742:         .kernelHeight(3)
1743:         .kernelWidth(3)
1744:         .cr(8)
1745:         .channels(channels)
1746:         .width(1)
1747:         .inputZeroPoint(0)
1748:         .kernelZeroPoint(255)
1749:         .test(pytorch_q8dwconv_ukernel_up8x9__sse2);
1750:   }
1751: }
1752: 
1753: TEST(Q8DWCONV_UP8x9__SSE2, multi_output_channels_gt_8) {
1754:   TEST_REQUIRES_X86_SSE2;
1755:   for (uint32_t channels = 9; channels < 16; channels++) {
1756:     DWConvMicrokernelTester()
1757:         .kernelHeight(3)
1758:         .kernelWidth(3)
1759:         .cr(8)
1760:         .channels(channels)
1761:         .width(5)
1762:         .test(pytorch_q8dwconv_ukernel_up8x9__sse2);
1763:   }
1764: }
1765: 
1766: TEST(Q8DWCONV_UP8x9__SSE2, multi_output_channels_gt_8_with_output_stride) {
1767:   TEST_REQUIRES_X86_SSE2;
1768:   for (uint32_t channels = 9; channels < 16; channels++) {
1769:     DWConvMicrokernelTester()
1770:         .kernelHeight(3)
1771:         .kernelWidth(3)
1772:         .cr(8)
1773:         .channels(channels)
1774:         .width(5)
1775:         .outputStride(17)
1776:         .test(pytorch_q8dwconv_ukernel_up8x9__sse2);
1777:   }
1778: }
1779: 
1780: TEST(Q8DWCONV_MP8x25__SSE2, single_output_channels_eq_8) {
1781:   TEST_REQUIRES_X86_SSE2;
1782:   DWConvMicrokernelTester()
1783:       .kernelHeight(5)
1784:       .kernelWidth(5)
1785:       .cr(8)
1786:       .channels(8)
1787:       .width(1)
1788:       .test(pytorch_q8dwconv_ukernel_mp8x25__sse2);
1789: }
1790: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `DWConvMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`DWConvMicrokernelTester`。

### Lines 1791-1844 / 第 1791-1844 行

```cpp
1791: TEST(Q8DWCONV_MP8x25__SSE2, single_output_channels_eq_8_with_qmin) {
1792:   TEST_REQUIRES_X86_SSE2;
1793:   DWConvMicrokernelTester()
1794:       .kernelHeight(5)
1795:       .kernelWidth(5)
1796:       .cr(8)
1797:       .channels(8)
1798:       .width(1)
1799:       .qmin(128)
1800:       .test(pytorch_q8dwconv_ukernel_mp8x25__sse2);
1801: }
1802: 
1803: TEST(Q8DWCONV_MP8x25__SSE2, single_output_channels_eq_8_with_qmax) {
1804:   TEST_REQUIRES_X86_SSE2;
1805:   DWConvMicrokernelTester()
1806:       .kernelHeight(5)
1807:       .kernelWidth(5)
1808:       .cr(8)
1809:       .channels(8)
1810:       .width(1)
1811:       .qmax(128)
1812:       .test(pytorch_q8dwconv_ukernel_mp8x25__sse2);
1813: }
1814: 
1815: TEST(
1816:     Q8DWCONV_MP8x25__SSE2,
1817:     single_output_channels_eq_8_with_input_zero_point_only) {
1818:   TEST_REQUIRES_X86_SSE2;
1819:   DWConvMicrokernelTester()
1820:       .kernelHeight(5)
1821:       .kernelWidth(5)
1822:       .cr(8)
1823:       .channels(8)
1824:       .width(1)
1825:       .inputZeroPoint(255)
1826:       .kernelZeroPoint(0)
1827:       .test(pytorch_q8dwconv_ukernel_mp8x25__sse2);
1828: }
1829: 
1830: TEST(
1831:     Q8DWCONV_MP8x25__SSE2,
1832:     single_output_channels_eq_8_with_kernel_zero_point_only) {
1833:   TEST_REQUIRES_X86_SSE2;
1834:   DWConvMicrokernelTester()
1835:       .kernelHeight(5)
1836:       .kernelWidth(5)
1837:       .cr(8)
1838:       .channels(8)
1839:       .width(1)
1840:       .inputZeroPoint(0)
1841:       .kernelZeroPoint(255)
1842:       .test(pytorch_q8dwconv_ukernel_mp8x25__sse2);
1843: }
1844: 
```

- **EN:** This block implements local helper logic for `q8dwconv`. Key symbols: `DWConvMicrokernelTester`.
- **CN:** 该代码块实现与 `q8dwconv` 相关的局部辅助逻辑。关键符号：`DWConvMicrokernelTester`。

### Lines 1845-1894 / 第 1845-1894 行

```cpp
1845: TEST(Q8DWCONV_MP8x25__SSE2, multi_output_channels_eq_8) {
1846:   TEST_REQUIRES_X86_SSE2;
1847:   DWConvMicrokernelTester()
1848:       .kernelHeight(5)
1849:       .kernelWidth(5)
1850:       .cr(8)
1851:       .channels(8)
1852:       .width(5)
1853:       .test(pytorch_q8dwconv_ukernel_mp8x25__sse2);
1854: }
1855: 
1856: TEST(Q8DWCONV_MP8x25__SSE2, multi_output_channels_eq_8_with_subsampling) {
1857:   TEST_REQUIRES_X86_SSE2;
1858:   DWConvMicrokernelTester()
1859:       .kernelHeight(5)
1860:       .kernelWidth(5)
1861:       .subsampling(2)
1862:       .cr(8)
1863:       .channels(8)
1864:       .width(5)
1865:       .test(pytorch_q8dwconv_ukernel_mp8x25__sse2);
1866: }
1867: 
1868: TEST(Q8DWCONV_MP8x25__SSE2, multi_output_channels_eq_8_with_input_stride) {
1869:   TEST_REQUIRES_X86_SSE2;
1870:   DWConvMicrokernelTester()
1871:       .kernelHeight(5)
1872:       .kernelWidth(5)
1873:       .cr(8)
1874:       .channels(8)
1875:       .width(5)
1876:       .inputStride(17)
1877:       .test(pytorch_q8dwconv_ukernel_mp8x25__sse2);
1878: }
1879: 
1880: TEST(Q8DWCONV_MP8x25__SSE2, multi_output_channels_eq_8_with_output_stride) {
1881:   TEST_REQUIRES_X86_SSE2;
1882:   DWConvMicrokernelTester()
1883:       .kernelHeight(5)
1884:       .kernelWidth(5)
1885:       .cr(8)
1886:       .channels(8)
1887:       .width(5)
1888:       .outputStride(19)
1889:       .test(pytorch_q8dwconv_ukernel_mp8x25__sse2);
1890: }
1891: 
1892: TEST(Q8DWCONV_MP8x25__SSE2, single_output_channels_div_8) {
1893:   TEST_REQUIRES_X86_SSE2;
1894:   for (uint32_t channels = 16; channels < 128; channels += 24) {
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `DWConvMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`DWConvMicrokernelTester`。

### Lines 1895-1944 / 第 1895-1944 行

```cpp
1895:     DWConvMicrokernelTester()
1896:         .kernelHeight(5)
1897:         .kernelWidth(5)
1898:         .cr(8)
1899:         .channels(channels)
1900:         .width(1)
1901:         .test(pytorch_q8dwconv_ukernel_mp8x25__sse2);
1902:   }
1903: }
1904: 
1905: TEST(Q8DWCONV_MP8x25__SSE2, multi_output_channels_div_8) {
1906:   TEST_REQUIRES_X86_SSE2;
1907:   for (uint32_t channels = 16; channels < 128; channels += 24) {
1908:     DWConvMicrokernelTester()
1909:         .kernelHeight(5)
1910:         .kernelWidth(5)
1911:         .cr(8)
1912:         .channels(channels)
1913:         .width(5)
1914:         .test(pytorch_q8dwconv_ukernel_mp8x25__sse2);
1915:   }
1916: }
1917: 
1918: TEST(Q8DWCONV_MP8x25__SSE2, multi_output_channels_div_8_with_output_stride) {
1919:   TEST_REQUIRES_X86_SSE2;
1920:   for (uint32_t channels = 16; channels < 128; channels += 24) {
1921:     DWConvMicrokernelTester()
1922:         .kernelHeight(5)
1923:         .kernelWidth(5)
1924:         .cr(8)
1925:         .channels(channels)
1926:         .width(5)
1927:         .outputStride(171)
1928:         .test(pytorch_q8dwconv_ukernel_mp8x25__sse2);
1929:   }
1930: }
1931: 
1932: TEST(Q8DWCONV_MP8x25__SSE2, single_output_channels_gt_8) {
1933:   TEST_REQUIRES_X86_SSE2;
1934:   for (uint32_t channels = 9; channels < 16; channels++) {
1935:     DWConvMicrokernelTester()
1936:         .kernelHeight(5)
1937:         .kernelWidth(5)
1938:         .cr(8)
1939:         .channels(channels)
1940:         .width(1)
1941:         .test(pytorch_q8dwconv_ukernel_mp8x25__sse2);
1942:   }
1943: }
1944: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `DWConvMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`DWConvMicrokernelTester`。

### Lines 1945-1994 / 第 1945-1994 行

```cpp
1945: TEST(Q8DWCONV_MP8x25__SSE2, single_output_channels_gt_8_with_qmin) {
1946:   TEST_REQUIRES_X86_SSE2;
1947:   for (uint32_t channels = 9; channels < 16; channels++) {
1948:     DWConvMicrokernelTester()
1949:         .kernelHeight(5)
1950:         .kernelWidth(5)
1951:         .cr(8)
1952:         .channels(channels)
1953:         .width(1)
1954:         .qmin(128)
1955:         .test(pytorch_q8dwconv_ukernel_mp8x25__sse2);
1956:   }
1957: }
1958: 
1959: TEST(Q8DWCONV_MP8x25__SSE2, single_output_channels_gt_8_with_qmax) {
1960:   TEST_REQUIRES_X86_SSE2;
1961:   for (uint32_t channels = 9; channels < 16; channels++) {
1962:     DWConvMicrokernelTester()
1963:         .kernelHeight(5)
1964:         .kernelWidth(5)
1965:         .cr(8)
1966:         .channels(channels)
1967:         .width(1)
1968:         .qmax(128)
1969:         .test(pytorch_q8dwconv_ukernel_mp8x25__sse2);
1970:   }
1971: }
1972: 
1973: TEST(
1974:     Q8DWCONV_MP8x25__SSE2,
1975:     single_output_channels_gt_8_with_input_zero_point_only) {
1976:   TEST_REQUIRES_X86_SSE2;
1977:   for (uint32_t channels = 9; channels < 16; channels++) {
1978:     DWConvMicrokernelTester()
1979:         .kernelHeight(5)
1980:         .kernelWidth(5)
1981:         .cr(8)
1982:         .channels(channels)
1983:         .width(1)
1984:         .inputZeroPoint(255)
1985:         .kernelZeroPoint(0)
1986:         .test(pytorch_q8dwconv_ukernel_mp8x25__sse2);
1987:   }
1988: }
1989: 
1990: TEST(
1991:     Q8DWCONV_MP8x25__SSE2,
1992:     single_output_channels_gt_8_with_kernel_zero_point_only) {
1993:   TEST_REQUIRES_X86_SSE2;
1994:   for (uint32_t channels = 9; channels < 16; channels++) {
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `DWConvMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`DWConvMicrokernelTester`。

### Lines 1995-2044 / 第 1995-2044 行

```cpp
1995:     DWConvMicrokernelTester()
1996:         .kernelHeight(5)
1997:         .kernelWidth(5)
1998:         .cr(8)
1999:         .channels(channels)
2000:         .width(1)
2001:         .inputZeroPoint(0)
2002:         .kernelZeroPoint(255)
2003:         .test(pytorch_q8dwconv_ukernel_mp8x25__sse2);
2004:   }
2005: }
2006: 
2007: TEST(Q8DWCONV_MP8x25__SSE2, multi_output_channels_gt_8) {
2008:   TEST_REQUIRES_X86_SSE2;
2009:   for (uint32_t channels = 9; channels < 16; channels++) {
2010:     DWConvMicrokernelTester()
2011:         .kernelHeight(5)
2012:         .kernelWidth(5)
2013:         .cr(8)
2014:         .channels(channels)
2015:         .width(5)
2016:         .test(pytorch_q8dwconv_ukernel_mp8x25__sse2);
2017:   }
2018: }
2019: 
2020: TEST(Q8DWCONV_MP8x25__SSE2, multi_output_channels_gt_8_with_output_stride) {
2021:   TEST_REQUIRES_X86_SSE2;
2022:   for (uint32_t channels = 9; channels < 16; channels++) {
2023:     DWConvMicrokernelTester()
2024:         .kernelHeight(5)
2025:         .kernelWidth(5)
2026:         .cr(8)
2027:         .channels(channels)
2028:         .width(5)
2029:         .outputStride(17)
2030:         .test(pytorch_q8dwconv_ukernel_mp8x25__sse2);
2031:   }
2032: }
2033: 
2034: TEST(Q8DWCONV_UP8x9__SSE2, single_output_channels_eq_8_per_channel) {
2035:   TEST_REQUIRES_X86_SSE2;
2036:   DWConvMicrokernelTester()
2037:       .kernelHeight(3)
2038:       .kernelWidth(3)
2039:       .cr(8)
2040:       .channels(8)
2041:       .width(1)
2042:       .test(pytorch_q8dwconv_ukernel_up8x9_per_channel__sse2, true);
2043: }
2044: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `DWConvMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`DWConvMicrokernelTester`。

### Lines 2045-2098 / 第 2045-2098 行

```cpp
2045: TEST(Q8DWCONV_UP8x9__SSE2, single_output_channels_eq_8_with_qmin_per_channel) {
2046:   TEST_REQUIRES_X86_SSE2;
2047:   DWConvMicrokernelTester()
2048:       .kernelHeight(3)
2049:       .kernelWidth(3)
2050:       .cr(8)
2051:       .channels(8)
2052:       .width(1)
2053:       .qmin(128)
2054:       .test(pytorch_q8dwconv_ukernel_up8x9_per_channel__sse2, true);
2055: }
2056: 
2057: TEST(Q8DWCONV_UP8x9__SSE2, single_output_channels_eq_8_with_qmax_per_channel) {
2058:   TEST_REQUIRES_X86_SSE2;
2059:   DWConvMicrokernelTester()
2060:       .kernelHeight(3)
2061:       .kernelWidth(3)
2062:       .cr(8)
2063:       .channels(8)
2064:       .width(1)
2065:       .qmax(128)
2066:       .test(pytorch_q8dwconv_ukernel_up8x9_per_channel__sse2, true);
2067: }
2068: 
2069: TEST(
2070:     Q8DWCONV_UP8x9__SSE2,
2071:     single_output_channels_eq_8_with_input_zero_point_only_per_channel) {
2072:   TEST_REQUIRES_X86_SSE2;
2073:   DWConvMicrokernelTester()
2074:       .kernelHeight(3)
2075:       .kernelWidth(3)
2076:       .cr(8)
2077:       .channels(8)
2078:       .width(1)
2079:       .inputZeroPoint(255)
2080:       .kernelZeroPoint(0)
2081:       .test(pytorch_q8dwconv_ukernel_up8x9_per_channel__sse2, true);
2082: }
2083: 
2084: TEST(
2085:     Q8DWCONV_UP8x9__SSE2,
2086:     single_output_channels_eq_8_with_kernel_zero_point_only_per_channel) {
2087:   TEST_REQUIRES_X86_SSE2;
2088:   DWConvMicrokernelTester()
2089:       .kernelHeight(3)
2090:       .kernelWidth(3)
2091:       .cr(8)
2092:       .channels(8)
2093:       .width(1)
2094:       .inputZeroPoint(0)
2095:       .kernelZeroPoint(255)
2096:       .test(pytorch_q8dwconv_ukernel_up8x9_per_channel__sse2, true);
2097: }
2098: 
```

- **EN:** This block implements local helper logic for `q8dwconv`. Key symbols: `DWConvMicrokernelTester`.
- **CN:** 该代码块实现与 `q8dwconv` 相关的局部辅助逻辑。关键符号：`DWConvMicrokernelTester`。

### Lines 2099-2151 / 第 2099-2151 行

```cpp
2099: TEST(Q8DWCONV_UP8x9__SSE2, multi_output_channels_eq_8_per_channel) {
2100:   TEST_REQUIRES_X86_SSE2;
2101:   DWConvMicrokernelTester()
2102:       .kernelHeight(3)
2103:       .kernelWidth(3)
2104:       .cr(8)
2105:       .channels(8)
2106:       .width(5)
2107:       .test(pytorch_q8dwconv_ukernel_up8x9_per_channel__sse2, true);
2108: }
2109: 
2110: TEST(
2111:     Q8DWCONV_UP8x9__SSE2,
2112:     multi_output_channels_eq_8_with_subsampling_per_channel) {
2113:   TEST_REQUIRES_X86_SSE2;
2114:   DWConvMicrokernelTester()
2115:       .kernelHeight(3)
2116:       .kernelWidth(3)
2117:       .subsampling(2)
2118:       .cr(8)
2119:       .channels(8)
2120:       .width(5)
2121:       .test(pytorch_q8dwconv_ukernel_up8x9_per_channel__sse2, true);
2122: }
2123: 
2124: TEST(
2125:     Q8DWCONV_UP8x9__SSE2,
2126:     multi_output_channels_eq_8_with_input_stride_per_channel) {
2127:   TEST_REQUIRES_X86_SSE2;
2128:   DWConvMicrokernelTester()
2129:       .kernelHeight(3)
2130:       .kernelWidth(3)
2131:       .cr(8)
2132:       .channels(8)
2133:       .width(5)
2134:       .inputStride(17)
2135:       .test(pytorch_q8dwconv_ukernel_up8x9_per_channel__sse2, true);
2136: }
2137: 
2138: TEST(
2139:     Q8DWCONV_UP8x9__SSE2,
2140:     multi_output_channels_eq_8_with_output_stride_per_channel) {
2141:   TEST_REQUIRES_X86_SSE2;
2142:   DWConvMicrokernelTester()
2143:       .kernelHeight(3)
2144:       .kernelWidth(3)
2145:       .cr(8)
2146:       .channels(8)
2147:       .width(5)
2148:       .outputStride(19)
2149:       .test(pytorch_q8dwconv_ukernel_up8x9_per_channel__sse2, true);
2150: }
2151: 
```

- **EN:** This block implements local helper logic for `q8dwconv`. Key symbols: `DWConvMicrokernelTester`.
- **CN:** 该代码块实现与 `q8dwconv` 相关的局部辅助逻辑。关键符号：`DWConvMicrokernelTester`。

### Lines 2152-2206 / 第 2152-2206 行

```cpp
2152: TEST(Q8DWCONV_UP8x9__SSE2, single_output_channels_div_8_per_channel) {
2153:   TEST_REQUIRES_X86_SSE2;
2154:   for (uint32_t channels = 16; channels < 128; channels += 24) {
2155:     DWConvMicrokernelTester()
2156:         .kernelHeight(3)
2157:         .kernelWidth(3)
2158:         .cr(8)
2159:         .channels(channels)
2160:         .width(1)
2161:         .test(pytorch_q8dwconv_ukernel_up8x9_per_channel__sse2, true);
2162:   }
2163: }
2164: 
2165: TEST(Q8DWCONV_UP8x9__SSE2, multi_output_channels_div_8_per_channel) {
2166:   TEST_REQUIRES_X86_SSE2;
2167:   for (uint32_t channels = 16; channels < 128; channels += 24) {
2168:     DWConvMicrokernelTester()
2169:         .kernelHeight(3)
2170:         .kernelWidth(3)
2171:         .cr(8)
2172:         .channels(channels)
2173:         .width(5)
2174:         .test(pytorch_q8dwconv_ukernel_up8x9_per_channel__sse2, true);
2175:   }
2176: }
2177: 
2178: TEST(
2179:     Q8DWCONV_UP8x9__SSE2,
2180:     multi_output_channels_div_8_with_output_stride_per_channel) {
2181:   TEST_REQUIRES_X86_SSE2;
2182:   for (uint32_t channels = 16; channels < 128; channels += 24) {
2183:     DWConvMicrokernelTester()
2184:         .kernelHeight(3)
2185:         .kernelWidth(3)
2186:         .cr(8)
2187:         .channels(channels)
2188:         .width(5)
2189:         .outputStride(171)
2190:         .test(pytorch_q8dwconv_ukernel_up8x9_per_channel__sse2, true);
2191:   }
2192: }
2193: 
2194: TEST(Q8DWCONV_UP8x9__SSE2, single_output_channels_gt_8_per_channel) {
2195:   TEST_REQUIRES_X86_SSE2;
2196:   for (uint32_t channels = 9; channels < 16; channels++) {
2197:     DWConvMicrokernelTester()
2198:         .kernelHeight(3)
2199:         .kernelWidth(3)
2200:         .cr(8)
2201:         .channels(channels)
2202:         .width(1)
2203:         .test(pytorch_q8dwconv_ukernel_up8x9_per_channel__sse2, true);
2204:   }
2205: }
2206: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `DWConvMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`DWConvMicrokernelTester`。

### Lines 2207-2256 / 第 2207-2256 行

```cpp
2207: TEST(Q8DWCONV_UP8x9__SSE2, single_output_channels_gt_8_with_qmin_per_channel) {
2208:   TEST_REQUIRES_X86_SSE2;
2209:   for (uint32_t channels = 9; channels < 16; channels++) {
2210:     DWConvMicrokernelTester()
2211:         .kernelHeight(3)
2212:         .kernelWidth(3)
2213:         .cr(8)
2214:         .channels(channels)
2215:         .width(1)
2216:         .qmin(128)
2217:         .test(pytorch_q8dwconv_ukernel_up8x9_per_channel__sse2, true);
2218:   }
2219: }
2220: 
2221: TEST(Q8DWCONV_UP8x9__SSE2, single_output_channels_gt_8_with_qmax_per_channel) {
2222:   TEST_REQUIRES_X86_SSE2;
2223:   for (uint32_t channels = 9; channels < 16; channels++) {
2224:     DWConvMicrokernelTester()
2225:         .kernelHeight(3)
2226:         .kernelWidth(3)
2227:         .cr(8)
2228:         .channels(channels)
2229:         .width(1)
2230:         .qmax(128)
2231:         .test(pytorch_q8dwconv_ukernel_up8x9_per_channel__sse2, true);
2232:   }
2233: }
2234: 
2235: TEST(
2236:     Q8DWCONV_UP8x9__SSE2,
2237:     single_output_channels_gt_8_with_input_zero_point_only_per_channel) {
2238:   TEST_REQUIRES_X86_SSE2;
2239:   for (uint32_t channels = 9; channels < 16; channels++) {
2240:     DWConvMicrokernelTester()
2241:         .kernelHeight(3)
2242:         .kernelWidth(3)
2243:         .cr(8)
2244:         .channels(channels)
2245:         .width(1)
2246:         .inputZeroPoint(255)
2247:         .kernelZeroPoint(0)
2248:         .test(pytorch_q8dwconv_ukernel_up8x9_per_channel__sse2, true);
2249:   }
2250: }
2251: 
2252: TEST(
2253:     Q8DWCONV_UP8x9__SSE2,
2254:     single_output_channels_gt_8_with_kernel_zero_point_only_per_channel) {
2255:   TEST_REQUIRES_X86_SSE2;
2256:   for (uint32_t channels = 9; channels < 16; channels++) {
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `DWConvMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`DWConvMicrokernelTester`。

### Lines 2257-2308 / 第 2257-2308 行

```cpp
2257:     DWConvMicrokernelTester()
2258:         .kernelHeight(3)
2259:         .kernelWidth(3)
2260:         .cr(8)
2261:         .channels(channels)
2262:         .width(1)
2263:         .inputZeroPoint(0)
2264:         .kernelZeroPoint(255)
2265:         .test(pytorch_q8dwconv_ukernel_up8x9_per_channel__sse2, true);
2266:   }
2267: }
2268: 
2269: TEST(Q8DWCONV_UP8x9__SSE2, multi_output_channels_gt_8_per_channel) {
2270:   TEST_REQUIRES_X86_SSE2;
2271:   for (uint32_t channels = 9; channels < 16; channels++) {
2272:     DWConvMicrokernelTester()
2273:         .kernelHeight(3)
2274:         .kernelWidth(3)
2275:         .cr(8)
2276:         .channels(channels)
2277:         .width(5)
2278:         .test(pytorch_q8dwconv_ukernel_up8x9_per_channel__sse2, true);
2279:   }
2280: }
2281: 
2282: TEST(
2283:     Q8DWCONV_UP8x9__SSE2,
2284:     multi_output_channels_gt_8_with_output_stride_per_channel) {
2285:   TEST_REQUIRES_X86_SSE2;
2286:   for (uint32_t channels = 9; channels < 16; channels++) {
2287:     DWConvMicrokernelTester()
2288:         .kernelHeight(3)
2289:         .kernelWidth(3)
2290:         .cr(8)
2291:         .channels(channels)
2292:         .width(5)
2293:         .outputStride(17)
2294:         .test(pytorch_q8dwconv_ukernel_up8x9_per_channel__sse2, true);
2295:   }
2296: }
2297: 
2298: TEST(Q8DWCONV_MP8x25__SSE2, single_output_channels_eq_8_per_channel) {
2299:   TEST_REQUIRES_X86_SSE2;
2300:   DWConvMicrokernelTester()
2301:       .kernelHeight(5)
2302:       .kernelWidth(5)
2303:       .cr(8)
2304:       .channels(8)
2305:       .width(1)
2306:       .test(pytorch_q8dwconv_ukernel_mp8x25_per_channel__sse2, true);
2307: }
2308: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `DWConvMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`DWConvMicrokernelTester`。

### Lines 2309-2362 / 第 2309-2362 行

```cpp
2309: TEST(Q8DWCONV_MP8x25__SSE2, single_output_channels_eq_8_with_qmin_per_channel) {
2310:   TEST_REQUIRES_X86_SSE2;
2311:   DWConvMicrokernelTester()
2312:       .kernelHeight(5)
2313:       .kernelWidth(5)
2314:       .cr(8)
2315:       .channels(8)
2316:       .width(1)
2317:       .qmin(128)
2318:       .test(pytorch_q8dwconv_ukernel_mp8x25_per_channel__sse2, true);
2319: }
2320: 
2321: TEST(Q8DWCONV_MP8x25__SSE2, single_output_channels_eq_8_with_qmax_per_channel) {
2322:   TEST_REQUIRES_X86_SSE2;
2323:   DWConvMicrokernelTester()
2324:       .kernelHeight(5)
2325:       .kernelWidth(5)
2326:       .cr(8)
2327:       .channels(8)
2328:       .width(1)
2329:       .qmax(128)
2330:       .test(pytorch_q8dwconv_ukernel_mp8x25_per_channel__sse2, true);
2331: }
2332: 
2333: TEST(
2334:     Q8DWCONV_MP8x25__SSE2,
2335:     single_output_channels_eq_8_with_input_zero_point_only_per_channel) {
2336:   TEST_REQUIRES_X86_SSE2;
2337:   DWConvMicrokernelTester()
2338:       .kernelHeight(5)
2339:       .kernelWidth(5)
2340:       .cr(8)
2341:       .channels(8)
2342:       .width(1)
2343:       .inputZeroPoint(255)
2344:       .kernelZeroPoint(0)
2345:       .test(pytorch_q8dwconv_ukernel_mp8x25_per_channel__sse2, true);
2346: }
2347: 
2348: TEST(
2349:     Q8DWCONV_MP8x25__SSE2,
2350:     single_output_channels_eq_8_with_kernel_zero_point_only_per_channel) {
2351:   TEST_REQUIRES_X86_SSE2;
2352:   DWConvMicrokernelTester()
2353:       .kernelHeight(5)
2354:       .kernelWidth(5)
2355:       .cr(8)
2356:       .channels(8)
2357:       .width(1)
2358:       .inputZeroPoint(0)
2359:       .kernelZeroPoint(255)
2360:       .test(pytorch_q8dwconv_ukernel_mp8x25_per_channel__sse2, true);
2361: }
2362: 
```

- **EN:** This block implements local helper logic for `q8dwconv`. Key symbols: `DWConvMicrokernelTester`.
- **CN:** 该代码块实现与 `q8dwconv` 相关的局部辅助逻辑。关键符号：`DWConvMicrokernelTester`。

### Lines 2363-2415 / 第 2363-2415 行

```cpp
2363: TEST(Q8DWCONV_MP8x25__SSE2, multi_output_channels_eq_8_per_channel) {
2364:   TEST_REQUIRES_X86_SSE2;
2365:   DWConvMicrokernelTester()
2366:       .kernelHeight(5)
2367:       .kernelWidth(5)
2368:       .cr(8)
2369:       .channels(8)
2370:       .width(5)
2371:       .test(pytorch_q8dwconv_ukernel_mp8x25_per_channel__sse2, true);
2372: }
2373: 
2374: TEST(
2375:     Q8DWCONV_MP8x25__SSE2,
2376:     multi_output_channels_eq_8_with_subsampling_per_channel) {
2377:   TEST_REQUIRES_X86_SSE2;
2378:   DWConvMicrokernelTester()
2379:       .kernelHeight(5)
2380:       .kernelWidth(5)
2381:       .subsampling(2)
2382:       .cr(8)
2383:       .channels(8)
2384:       .width(5)
2385:       .test(pytorch_q8dwconv_ukernel_mp8x25_per_channel__sse2, true);
2386: }
2387: 
2388: TEST(
2389:     Q8DWCONV_MP8x25__SSE2,
2390:     multi_output_channels_eq_8_with_input_stride_per_channel) {
2391:   TEST_REQUIRES_X86_SSE2;
2392:   DWConvMicrokernelTester()
2393:       .kernelHeight(5)
2394:       .kernelWidth(5)
2395:       .cr(8)
2396:       .channels(8)
2397:       .width(5)
2398:       .inputStride(17)
2399:       .test(pytorch_q8dwconv_ukernel_mp8x25_per_channel__sse2, true);
2400: }
2401: 
2402: TEST(
2403:     Q8DWCONV_MP8x25__SSE2,
2404:     multi_output_channels_eq_8_with_output_stride_per_channel) {
2405:   TEST_REQUIRES_X86_SSE2;
2406:   DWConvMicrokernelTester()
2407:       .kernelHeight(5)
2408:       .kernelWidth(5)
2409:       .cr(8)
2410:       .channels(8)
2411:       .width(5)
2412:       .outputStride(19)
2413:       .test(pytorch_q8dwconv_ukernel_mp8x25_per_channel__sse2, true);
2414: }
2415: 
```

- **EN:** This block implements local helper logic for `q8dwconv`. Key symbols: `DWConvMicrokernelTester`.
- **CN:** 该代码块实现与 `q8dwconv` 相关的局部辅助逻辑。关键符号：`DWConvMicrokernelTester`。

### Lines 2416-2470 / 第 2416-2470 行

```cpp
2416: TEST(Q8DWCONV_MP8x25__SSE2, single_output_channels_div_8_per_channel) {
2417:   TEST_REQUIRES_X86_SSE2;
2418:   for (uint32_t channels = 16; channels < 128; channels += 24) {
2419:     DWConvMicrokernelTester()
2420:         .kernelHeight(5)
2421:         .kernelWidth(5)
2422:         .cr(8)
2423:         .channels(channels)
2424:         .width(1)
2425:         .test(pytorch_q8dwconv_ukernel_mp8x25_per_channel__sse2, true);
2426:   }
2427: }
2428: 
2429: TEST(Q8DWCONV_MP8x25__SSE2, multi_output_channels_div_8_per_channel) {
2430:   TEST_REQUIRES_X86_SSE2;
2431:   for (uint32_t channels = 16; channels < 128; channels += 24) {
2432:     DWConvMicrokernelTester()
2433:         .kernelHeight(5)
2434:         .kernelWidth(5)
2435:         .cr(8)
2436:         .channels(channels)
2437:         .width(5)
2438:         .test(pytorch_q8dwconv_ukernel_mp8x25_per_channel__sse2, true);
2439:   }
2440: }
2441: 
2442: TEST(
2443:     Q8DWCONV_MP8x25__SSE2,
2444:     multi_output_channels_div_8_with_output_stride_per_channel) {
2445:   TEST_REQUIRES_X86_SSE2;
2446:   for (uint32_t channels = 16; channels < 128; channels += 24) {
2447:     DWConvMicrokernelTester()
2448:         .kernelHeight(5)
2449:         .kernelWidth(5)
2450:         .cr(8)
2451:         .channels(channels)
2452:         .width(5)
2453:         .outputStride(171)
2454:         .test(pytorch_q8dwconv_ukernel_mp8x25_per_channel__sse2, true);
2455:   }
2456: }
2457: 
2458: TEST(Q8DWCONV_MP8x25__SSE2, single_output_channels_gt_8_per_channel) {
2459:   TEST_REQUIRES_X86_SSE2;
2460:   for (uint32_t channels = 9; channels < 16; channels++) {
2461:     DWConvMicrokernelTester()
2462:         .kernelHeight(5)
2463:         .kernelWidth(5)
2464:         .cr(8)
2465:         .channels(channels)
2466:         .width(1)
2467:         .test(pytorch_q8dwconv_ukernel_mp8x25_per_channel__sse2, true);
2468:   }
2469: }
2470: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `DWConvMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`DWConvMicrokernelTester`。

### Lines 2471-2520 / 第 2471-2520 行

```cpp
2471: TEST(Q8DWCONV_MP8x25__SSE2, single_output_channels_gt_8_with_qmin_per_channel) {
2472:   TEST_REQUIRES_X86_SSE2;
2473:   for (uint32_t channels = 9; channels < 16; channels++) {
2474:     DWConvMicrokernelTester()
2475:         .kernelHeight(5)
2476:         .kernelWidth(5)
2477:         .cr(8)
2478:         .channels(channels)
2479:         .width(1)
2480:         .qmin(128)
2481:         .test(pytorch_q8dwconv_ukernel_mp8x25_per_channel__sse2, true);
2482:   }
2483: }
2484: 
2485: TEST(Q8DWCONV_MP8x25__SSE2, single_output_channels_gt_8_with_qmax_per_channel) {
2486:   TEST_REQUIRES_X86_SSE2;
2487:   for (uint32_t channels = 9; channels < 16; channels++) {
2488:     DWConvMicrokernelTester()
2489:         .kernelHeight(5)
2490:         .kernelWidth(5)
2491:         .cr(8)
2492:         .channels(channels)
2493:         .width(1)
2494:         .qmax(128)
2495:         .test(pytorch_q8dwconv_ukernel_mp8x25_per_channel__sse2, true);
2496:   }
2497: }
2498: 
2499: TEST(
2500:     Q8DWCONV_MP8x25__SSE2,
2501:     single_output_channels_gt_8_with_input_zero_point_only_per_channel) {
2502:   TEST_REQUIRES_X86_SSE2;
2503:   for (uint32_t channels = 9; channels < 16; channels++) {
2504:     DWConvMicrokernelTester()
2505:         .kernelHeight(5)
2506:         .kernelWidth(5)
2507:         .cr(8)
2508:         .channels(channels)
2509:         .width(1)
2510:         .inputZeroPoint(255)
2511:         .kernelZeroPoint(0)
2512:         .test(pytorch_q8dwconv_ukernel_mp8x25_per_channel__sse2, true);
2513:   }
2514: }
2515: 
2516: TEST(
2517:     Q8DWCONV_MP8x25__SSE2,
2518:     single_output_channels_gt_8_with_kernel_zero_point_only_per_channel) {
2519:   TEST_REQUIRES_X86_SSE2;
2520:   for (uint32_t channels = 9; channels < 16; channels++) {
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `DWConvMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`DWConvMicrokernelTester`。

### Lines 2521-2561 / 第 2521-2561 行

```cpp
2521:     DWConvMicrokernelTester()
2522:         .kernelHeight(5)
2523:         .kernelWidth(5)
2524:         .cr(8)
2525:         .channels(channels)
2526:         .width(1)
2527:         .inputZeroPoint(0)
2528:         .kernelZeroPoint(255)
2529:         .test(pytorch_q8dwconv_ukernel_mp8x25_per_channel__sse2, true);
2530:   }
2531: }
2532: 
2533: TEST(Q8DWCONV_MP8x25__SSE2, multi_output_channels_gt_8_per_channel) {
2534:   TEST_REQUIRES_X86_SSE2;
2535:   for (uint32_t channels = 9; channels < 16; channels++) {
2536:     DWConvMicrokernelTester()
2537:         .kernelHeight(5)
2538:         .kernelWidth(5)
2539:         .cr(8)
2540:         .channels(channels)
2541:         .width(5)
2542:         .test(pytorch_q8dwconv_ukernel_mp8x25_per_channel__sse2, true);
2543:   }
2544: }
2545: 
2546: TEST(
2547:     Q8DWCONV_MP8x25__SSE2,
2548:     multi_output_channels_gt_8_with_output_stride_per_channel) {
2549:   TEST_REQUIRES_X86_SSE2;
2550:   for (uint32_t channels = 9; channels < 16; channels++) {
2551:     DWConvMicrokernelTester()
2552:         .kernelHeight(5)
2553:         .kernelWidth(5)
2554:         .cr(8)
2555:         .channels(channels)
2556:         .width(5)
2557:         .outputStride(17)
2558:         .test(pytorch_q8dwconv_ukernel_mp8x25_per_channel__sse2, true);
2559:   }
2560: }
2561: #endif /* CPUINFO_ARCH_X86 || CPUINFO_ARCH_X86_64 */
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends; iterates over tensors, descriptors, options, or runtime state. Key symbols: `DWConvMicrokernelTester`.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端；遍历张量、描述符、配置项或运行时状态。关键符号：`DWConvMicrokernelTester`。


## Key Concepts / 关键概念
- **ATen native operator logic** — ATen 原生算子逻辑
- **GPU kernel/runtime interaction** — GPU 内核/运行时交互
- **Mixed-precision/autocast logic** — 混合精度/自动混精逻辑
- **Runtime validation and invariants** — 运行时校验与不变量
- **Core symbols: DWConvMicrokernelTester** — 核心符号：DWConvMicrokernelTester

## Dependencies / 依赖关系
- **Internal includes / 内部头文件**: 无
- **External includes / 外部头文件**: `cpuinfo.h`, `gtest/gtest.h`, `qnnpack/isa-checks.h`, `qnnpack/q8dwconv.h`, `dwconv-microkernel-tester.h`
- **Namespaces / 命名空间**: 无
- **Representative symbols / 代表性符号**: `DWConvMicrokernelTester`
