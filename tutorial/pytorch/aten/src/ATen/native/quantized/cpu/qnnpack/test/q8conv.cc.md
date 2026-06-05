# q8conv.cc — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `aten/src/ATen/native/quantized/cpu/qnnpack/test/q8conv.cc`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements ATen native operators, checks, helper kernels, and backend dispatch behavior. This specific file centers on `q8conv.cc`. The file header highlights: "All rights reserved. This source code is licensed under the BSD-style license found in the LICENSE file in the root directory of this source tree.."
- **Purpose (CN)**: 实现 ATen 原生算子、运行时检查、辅助内核以及后端分发行为。 该文件具体围绕 `q8conv.cc` 展开。 文件头部注释也概括了其核心职责。

## Line-by-Line Analysis / 逐行分析
### Lines 1-31 / 第 1-31 行

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
0013: #include <qnnpack/q8conv.h>
0014: 
0015: #include "gemm-microkernel-tester.h"
0016: 
0017: #if CPUINFO_ARCH_ARM
0018: TEST(Q8CONV_4x8__AARCH32_NEON, k_eq_8) {
0019:   TEST_REQUIRES_ARM_NEON;
0020:   GemmMicrokernelTester()
0021:       .mr(4)
0022:       .nr(8)
0023:       .np(8)
0024:       .kr(1)
0025:       .m(4)
0026:       .n(8)
0027:       .k(8)
0028:       .aStride(37)
0029:       .test(pytorch_q8conv_ukernel_4x8__aarch32_neon);
0030: }
0031: 
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends. Key symbols: `GemmMicrokernelTester`.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端。关键符号：`GemmMicrokernelTester`。

### Lines 32-73 / 第 32-73 行

```cpp
0032: TEST(Q8CONV_4x8__AARCH32_NEON, k_eq_8_strided_c) {
0033:   TEST_REQUIRES_ARM_NEON;
0034:   GemmMicrokernelTester()
0035:       .mr(4)
0036:       .nr(8)
0037:       .np(8)
0038:       .kr(1)
0039:       .m(4)
0040:       .n(8)
0041:       .k(8)
0042:       .aStride(37)
0043:       .cStride(17)
0044:       .test(pytorch_q8conv_ukernel_4x8__aarch32_neon);
0045: }
0046: 
0047: TEST(Q8CONV_4x8__AARCH32_NEON, k_eq_8_qmin128) {
0048:   TEST_REQUIRES_ARM_NEON;
0049:   GemmMicrokernelTester().mr(4).nr(8).np(8).kr(1).m(4).n(8).k(8).qmin(128).test(
0050:       pytorch_q8conv_ukernel_4x8__aarch32_neon);
0051: }
0052: 
0053: TEST(Q8CONV_4x8__AARCH32_NEON, k_eq_8_qmax128) {
0054:   TEST_REQUIRES_ARM_NEON;
0055:   GemmMicrokernelTester().mr(4).nr(8).np(8).kr(1).m(4).n(8).k(8).qmax(128).test(
0056:       pytorch_q8conv_ukernel_4x8__aarch32_neon);
0057: }
0058: 
0059: TEST(Q8CONV_4x8__AARCH32_NEON, k_eq_8_azp_only) {
0060:   TEST_REQUIRES_ARM_NEON;
0061:   GemmMicrokernelTester()
0062:       .mr(4)
0063:       .nr(8)
0064:       .np(8)
0065:       .kr(1)
0066:       .m(4)
0067:       .n(8)
0068:       .k(8)
0069:       .aZeroPoint(255)
0070:       .bZeroPoint(0)
0071:       .test(pytorch_q8conv_ukernel_4x8__aarch32_neon);
0072: }
0073: 
```

- **EN:** This block implements local helper logic for `q8conv`. Key symbols: `GemmMicrokernelTester`.
- **CN:** 该代码块实现与 `q8conv` 相关的局部辅助逻辑。关键符号：`GemmMicrokernelTester`。

### Lines 74-104 / 第 74-104 行

```cpp
0074: TEST(Q8CONV_4x8__AARCH32_NEON, k_eq_8_bzp_only) {
0075:   TEST_REQUIRES_ARM_NEON;
0076:   GemmMicrokernelTester()
0077:       .mr(4)
0078:       .nr(8)
0079:       .np(8)
0080:       .kr(1)
0081:       .m(4)
0082:       .n(8)
0083:       .k(8)
0084:       .aZeroPoint(0)
0085:       .bZeroPoint(255)
0086:       .test(pytorch_q8conv_ukernel_4x8__aarch32_neon);
0087: }
0088: 
0089: TEST(Q8CONV_4x8__AARCH32_NEON, k_gt_8) {
0090:   TEST_REQUIRES_ARM_NEON;
0091:   for (size_t k = 9; k < 16; k++) {
0092:     GemmMicrokernelTester()
0093:         .mr(4)
0094:         .nr(8)
0095:         .np(8)
0096:         .kr(1)
0097:         .m(4)
0098:         .n(8)
0099:         .k(k)
0100:         .aStride(37)
0101:         .test(pytorch_q8conv_ukernel_4x8__aarch32_neon);
0102:   }
0103: }
0104: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `GemmMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`GemmMicrokernelTester`。

### Lines 105-139 / 第 105-139 行

```cpp
0105: TEST(Q8CONV_4x8__AARCH32_NEON, k_gt_8_strided_c) {
0106:   TEST_REQUIRES_ARM_NEON;
0107:   for (size_t k = 9; k < 16; k++) {
0108:     GemmMicrokernelTester()
0109:         .mr(4)
0110:         .nr(8)
0111:         .np(8)
0112:         .kr(1)
0113:         .m(4)
0114:         .n(8)
0115:         .k(k)
0116:         .aStride(37)
0117:         .cStride(17)
0118:         .test(pytorch_q8conv_ukernel_4x8__aarch32_neon);
0119:   }
0120: }
0121: 
0122: TEST(Q8CONV_4x8__AARCH32_NEON, k_gt_8_azp_only) {
0123:   TEST_REQUIRES_ARM_NEON;
0124:   for (size_t k = 9; k < 16; k++) {
0125:     GemmMicrokernelTester()
0126:         .mr(4)
0127:         .nr(8)
0128:         .np(8)
0129:         .kr(1)
0130:         .m(4)
0131:         .n(8)
0132:         .k(k)
0133:         .aStride(37)
0134:         .aZeroPoint(255)
0135:         .bZeroPoint(0)
0136:         .test(pytorch_q8conv_ukernel_4x8__aarch32_neon);
0137:   }
0138: }
0139: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `GemmMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`GemmMicrokernelTester`。

### Lines 140-178 / 第 140-178 行

```cpp
0140: TEST(Q8CONV_4x8__AARCH32_NEON, k_gt_8_bzp_only) {
0141:   TEST_REQUIRES_ARM_NEON;
0142:   for (size_t k = 9; k < 16; k++) {
0143:     GemmMicrokernelTester()
0144:         .mr(4)
0145:         .nr(8)
0146:         .np(8)
0147:         .kr(1)
0148:         .m(4)
0149:         .n(8)
0150:         .k(k)
0151:         .aStride(37)
0152:         .aZeroPoint(0)
0153:         .bZeroPoint(255)
0154:         .test(pytorch_q8conv_ukernel_4x8__aarch32_neon);
0155:   }
0156: }
0157: 
0158: TEST(Q8CONV_4x8__AARCH32_NEON, k_gt_8_subtile) {
0159:   TEST_REQUIRES_ARM_NEON;
0160:   for (size_t k = 9; k < 16; k++) {
0161:     for (uint32_t m = 1; m <= 4; m++) {
0162:       for (uint32_t n = 1; n <= 8; n++) {
0163:         GemmMicrokernelTester()
0164:             .mr(4)
0165:             .nr(8)
0166:             .np(8)
0167:             .kr(1)
0168:             .m(m)
0169:             .n(n)
0170:             .k(k)
0171:             .aStride(37)
0172:             .iterations(3)
0173:             .test(pytorch_q8conv_ukernel_4x8__aarch32_neon);
0174:       }
0175:     }
0176:   }
0177: }
0178: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `GemmMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`GemmMicrokernelTester`。

### Lines 179-211 / 第 179-211 行

```cpp
0179: TEST(Q8CONV_4x8__AARCH32_NEON, k_div_8) {
0180:   TEST_REQUIRES_ARM_NEON;
0181:   for (size_t k = 16; k < 128; k += 8) {
0182:     GemmMicrokernelTester()
0183:         .mr(4)
0184:         .nr(8)
0185:         .np(8)
0186:         .kr(1)
0187:         .m(4)
0188:         .n(8)
0189:         .k(k)
0190:         .aStride(171)
0191:         .test(pytorch_q8conv_ukernel_4x8__aarch32_neon);
0192:   }
0193: }
0194: 
0195: TEST(Q8CONV_4x8__AARCH32_NEON, k_div_8_strided_c) {
0196:   TEST_REQUIRES_ARM_NEON;
0197:   for (size_t k = 16; k < 128; k += 8) {
0198:     GemmMicrokernelTester()
0199:         .mr(4)
0200:         .nr(8)
0201:         .np(8)
0202:         .kr(1)
0203:         .m(4)
0204:         .n(8)
0205:         .k(k)
0206:         .aStride(171)
0207:         .cStride(17)
0208:         .test(pytorch_q8conv_ukernel_4x8__aarch32_neon);
0209:   }
0210: }
0211: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `GemmMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`GemmMicrokernelTester`。

### Lines 212-247 / 第 212-247 行

```cpp
0212: TEST(Q8CONV_4x8__AARCH32_NEON, k_div_8_subtile) {
0213:   TEST_REQUIRES_ARM_NEON;
0214:   for (size_t k = 16; k < 128; k += 24) {
0215:     for (uint32_t m = 1; m <= 4; m++) {
0216:       for (uint32_t n = 1; n <= 8; n++) {
0217:         GemmMicrokernelTester()
0218:             .mr(4)
0219:             .nr(8)
0220:             .np(8)
0221:             .kr(1)
0222:             .m(m)
0223:             .n(n)
0224:             .k(k)
0225:             .aStride(171)
0226:             .iterations(3)
0227:             .test(pytorch_q8conv_ukernel_4x8__aarch32_neon);
0228:       }
0229:     }
0230:   }
0231: }
0232: #endif
0233: 
0234: #if CPUINFO_ARCH_ARM64
0235: TEST(Q8CONV_8x8__AARCH64_NEON, k_eq_8) {
0236:   GemmMicrokernelTester()
0237:       .mr(8)
0238:       .nr(8)
0239:       .np(8)
0240:       .kr(1)
0241:       .m(8)
0242:       .n(8)
0243:       .k(8)
0244:       .aStride(37)
0245:       .test(pytorch_q8conv_ukernel_8x8__aarch64_neon);
0246: }
0247: 
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends; iterates over tensors, descriptors, options, or runtime state. Key symbols: `GemmMicrokernelTester`.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端；遍历张量、描述符、配置项或运行时状态。关键符号：`GemmMicrokernelTester`。

### Lines 248-285 / 第 248-285 行

```cpp
0248: TEST(Q8CONV_8x8__AARCH64_NEON, k_eq_8_strided_c) {
0249:   GemmMicrokernelTester()
0250:       .mr(8)
0251:       .nr(8)
0252:       .np(8)
0253:       .kr(1)
0254:       .m(8)
0255:       .n(8)
0256:       .k(8)
0257:       .aStride(37)
0258:       .cStride(17)
0259:       .test(pytorch_q8conv_ukernel_8x8__aarch64_neon);
0260: }
0261: 
0262: TEST(Q8CONV_8x8__AARCH64_NEON, k_eq_8_qmin128) {
0263:   GemmMicrokernelTester().mr(8).nr(8).np(8).kr(1).m(8).n(8).k(8).qmin(128).test(
0264:       pytorch_q8conv_ukernel_8x8__aarch64_neon);
0265: }
0266: 
0267: TEST(Q8CONV_8x8__AARCH64_NEON, k_eq_8_qmax128) {
0268:   GemmMicrokernelTester().mr(8).nr(8).np(8).kr(1).m(8).n(8).k(8).qmax(128).test(
0269:       pytorch_q8conv_ukernel_8x8__aarch64_neon);
0270: }
0271: 
0272: TEST(Q8CONV_8x8__AARCH64_NEON, k_eq_8_azp_only) {
0273:   GemmMicrokernelTester()
0274:       .mr(8)
0275:       .nr(8)
0276:       .np(8)
0277:       .kr(1)
0278:       .m(8)
0279:       .n(8)
0280:       .k(8)
0281:       .aZeroPoint(255)
0282:       .bZeroPoint(0)
0283:       .test(pytorch_q8conv_ukernel_8x8__aarch64_neon);
0284: }
0285: 
```

- **EN:** This block implements local helper logic for `q8conv`. Key symbols: `GemmMicrokernelTester`.
- **CN:** 该代码块实现与 `q8conv` 相关的局部辅助逻辑。关键符号：`GemmMicrokernelTester`。

### Lines 286-316 / 第 286-316 行

```cpp
0286: TEST(Q8CONV_8x8__AARCH64_NEON, k_eq_8_bzp_only) {
0287:   GemmMicrokernelTester()
0288:       .mr(8)
0289:       .nr(8)
0290:       .np(8)
0291:       .kr(1)
0292:       .m(8)
0293:       .n(8)
0294:       .k(8)
0295:       .aZeroPoint(0)
0296:       .bZeroPoint(255)
0297:       .test(pytorch_q8conv_ukernel_8x8__aarch64_neon);
0298: }
0299: 
0300: TEST(Q8CONV_8x8__AARCH64_NEON, k_gt_8) {
0301:   for (size_t k = 9; k < 16; k++) {
0302:     GemmMicrokernelTester()
0303:         .mr(8)
0304:         .nr(8)
0305:         .np(8)
0306:         .kr(1)
0307:         .m(8)
0308:         .n(8)
0309:         .k(k)
0310:         .aStride(37)
0311:         .test(pytorch_q8conv_ukernel_8x8__aarch64_neon);
0312:   }
0313: }
0314: 
0315: TEST(Q8CONV_8x8__AARCH64_NEON, k_gt_8_strided_c) {
0316:   for (size_t k = 9; k < 16; k++) {
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `GemmMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`GemmMicrokernelTester`。

### Lines 317-347 / 第 317-347 行

```cpp
0317:     GemmMicrokernelTester()
0318:         .mr(8)
0319:         .nr(8)
0320:         .np(8)
0321:         .kr(1)
0322:         .m(8)
0323:         .n(8)
0324:         .k(k)
0325:         .aStride(37)
0326:         .cStride(17)
0327:         .test(pytorch_q8conv_ukernel_8x8__aarch64_neon);
0328:   }
0329: }
0330: 
0331: TEST(Q8CONV_8x8__AARCH64_NEON, k_gt_8_azp_only) {
0332:   for (size_t k = 9; k < 16; k++) {
0333:     GemmMicrokernelTester()
0334:         .mr(8)
0335:         .nr(8)
0336:         .np(8)
0337:         .kr(1)
0338:         .m(8)
0339:         .n(8)
0340:         .k(k)
0341:         .aStride(37)
0342:         .aZeroPoint(255)
0343:         .bZeroPoint(0)
0344:         .test(pytorch_q8conv_ukernel_8x8__aarch64_neon);
0345:   }
0346: }
0347: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `GemmMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`GemmMicrokernelTester`。

### Lines 348-384 / 第 348-384 行

```cpp
0348: TEST(Q8CONV_8x8__AARCH64_NEON, k_gt_8_bzp_only) {
0349:   for (size_t k = 9; k < 16; k++) {
0350:     GemmMicrokernelTester()
0351:         .mr(8)
0352:         .nr(8)
0353:         .np(8)
0354:         .kr(1)
0355:         .m(8)
0356:         .n(8)
0357:         .k(k)
0358:         .aStride(37)
0359:         .aZeroPoint(0)
0360:         .bZeroPoint(255)
0361:         .test(pytorch_q8conv_ukernel_8x8__aarch64_neon);
0362:   }
0363: }
0364: 
0365: TEST(Q8CONV_8x8__AARCH64_NEON, k_gt_8_subtile) {
0366:   for (size_t k = 9; k < 16; k++) {
0367:     for (uint32_t m = 1; m <= 8; m++) {
0368:       for (uint32_t n = 1; n <= 8; n++) {
0369:         GemmMicrokernelTester()
0370:             .mr(8)
0371:             .nr(8)
0372:             .np(8)
0373:             .kr(1)
0374:             .m(m)
0375:             .n(n)
0376:             .k(k)
0377:             .aStride(37)
0378:             .iterations(3)
0379:             .test(pytorch_q8conv_ukernel_8x8__aarch64_neon);
0380:       }
0381:     }
0382:   }
0383: }
0384: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `GemmMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`GemmMicrokernelTester`。

### Lines 385-415 / 第 385-415 行

```cpp
0385: TEST(Q8CONV_8x8__AARCH64_NEON, k_div_8) {
0386:   for (size_t k = 16; k < 128; k += 8) {
0387:     GemmMicrokernelTester()
0388:         .mr(8)
0389:         .nr(8)
0390:         .np(8)
0391:         .kr(1)
0392:         .m(8)
0393:         .n(8)
0394:         .k(k)
0395:         .aStride(171)
0396:         .test(pytorch_q8conv_ukernel_8x8__aarch64_neon);
0397:   }
0398: }
0399: 
0400: TEST(Q8CONV_8x8__AARCH64_NEON, k_div_8_strided_c) {
0401:   for (size_t k = 16; k < 128; k += 8) {
0402:     GemmMicrokernelTester()
0403:         .mr(8)
0404:         .nr(8)
0405:         .np(8)
0406:         .kr(1)
0407:         .m(8)
0408:         .n(8)
0409:         .k(k)
0410:         .aStride(171)
0411:         .cStride(17)
0412:         .test(pytorch_q8conv_ukernel_8x8__aarch64_neon);
0413:   }
0414: }
0415: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `GemmMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`GemmMicrokernelTester`。

### Lines 416-451 / 第 416-451 行

```cpp
0416: TEST(Q8CONV_8x8__AARCH64_NEON, k_div_8_subtile) {
0417:   for (size_t k = 16; k < 128; k += 24) {
0418:     for (uint32_t m = 1; m <= 8; m++) {
0419:       for (uint32_t n = 1; n <= 8; n++) {
0420:         GemmMicrokernelTester()
0421:             .mr(8)
0422:             .nr(8)
0423:             .np(8)
0424:             .kr(1)
0425:             .m(m)
0426:             .n(n)
0427:             .k(k)
0428:             .aStride(171)
0429:             .iterations(3)
0430:             .test(pytorch_q8conv_ukernel_8x8__aarch64_neon);
0431:       }
0432:     }
0433:   }
0434: }
0435: #endif
0436: 
0437: #if CPUINFO_ARCH_ARM || CPUINFO_ARCH_ARM64
0438: TEST(Q8CONV_4x8__NEON, k_eq_8) {
0439:   TEST_REQUIRES_ARM_NEON;
0440:   GemmMicrokernelTester()
0441:       .mr(4)
0442:       .nr(8)
0443:       .np(8)
0444:       .kr(1)
0445:       .m(4)
0446:       .n(8)
0447:       .k(8)
0448:       .aStride(37)
0449:       .test(pytorch_q8conv_ukernel_4x8__neon);
0450: }
0451: 
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends; iterates over tensors, descriptors, options, or runtime state. Key symbols: `GemmMicrokernelTester`.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端；遍历张量、描述符、配置项或运行时状态。关键符号：`GemmMicrokernelTester`。

### Lines 452-493 / 第 452-493 行

```cpp
0452: TEST(Q8CONV_4x8__NEON, k_eq_8_strided_c) {
0453:   TEST_REQUIRES_ARM_NEON;
0454:   GemmMicrokernelTester()
0455:       .mr(4)
0456:       .nr(8)
0457:       .np(8)
0458:       .kr(1)
0459:       .m(4)
0460:       .n(8)
0461:       .k(8)
0462:       .aStride(37)
0463:       .cStride(17)
0464:       .test(pytorch_q8conv_ukernel_4x8__neon);
0465: }
0466: 
0467: TEST(Q8CONV_4x8__NEON, k_eq_8_qmin128) {
0468:   TEST_REQUIRES_ARM_NEON;
0469:   GemmMicrokernelTester().mr(4).nr(8).np(8).kr(1).m(4).n(8).k(8).qmin(128).test(
0470:       pytorch_q8conv_ukernel_4x8__neon);
0471: }
0472: 
0473: TEST(Q8CONV_4x8__NEON, k_eq_8_qmax128) {
0474:   TEST_REQUIRES_ARM_NEON;
0475:   GemmMicrokernelTester().mr(4).nr(8).np(8).kr(1).m(4).n(8).k(8).qmax(128).test(
0476:       pytorch_q8conv_ukernel_4x8__neon);
0477: }
0478: 
0479: TEST(Q8CONV_4x8__NEON, k_eq_8_azp_only) {
0480:   TEST_REQUIRES_ARM_NEON;
0481:   GemmMicrokernelTester()
0482:       .mr(4)
0483:       .nr(8)
0484:       .np(8)
0485:       .kr(1)
0486:       .m(4)
0487:       .n(8)
0488:       .k(8)
0489:       .aZeroPoint(255)
0490:       .bZeroPoint(0)
0491:       .test(pytorch_q8conv_ukernel_4x8__neon);
0492: }
0493: 
```

- **EN:** This block implements local helper logic for `q8conv`. Key symbols: `GemmMicrokernelTester`.
- **CN:** 该代码块实现与 `q8conv` 相关的局部辅助逻辑。关键符号：`GemmMicrokernelTester`。

### Lines 494-524 / 第 494-524 行

```cpp
0494: TEST(Q8CONV_4x8__NEON, k_eq_8_bzp_only) {
0495:   TEST_REQUIRES_ARM_NEON;
0496:   GemmMicrokernelTester()
0497:       .mr(4)
0498:       .nr(8)
0499:       .np(8)
0500:       .kr(1)
0501:       .m(4)
0502:       .n(8)
0503:       .k(8)
0504:       .aZeroPoint(0)
0505:       .bZeroPoint(255)
0506:       .test(pytorch_q8conv_ukernel_4x8__neon);
0507: }
0508: 
0509: TEST(Q8CONV_4x8__NEON, k_gt_8) {
0510:   TEST_REQUIRES_ARM_NEON;
0511:   for (size_t k = 9; k < 16; k++) {
0512:     GemmMicrokernelTester()
0513:         .mr(4)
0514:         .nr(8)
0515:         .np(8)
0516:         .kr(1)
0517:         .m(4)
0518:         .n(8)
0519:         .k(k)
0520:         .aStride(37)
0521:         .test(pytorch_q8conv_ukernel_4x8__neon);
0522:   }
0523: }
0524: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `GemmMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`GemmMicrokernelTester`。

### Lines 525-559 / 第 525-559 行

```cpp
0525: TEST(Q8CONV_4x8__NEON, k_gt_8_strided_c) {
0526:   TEST_REQUIRES_ARM_NEON;
0527:   for (size_t k = 9; k < 16; k++) {
0528:     GemmMicrokernelTester()
0529:         .mr(4)
0530:         .nr(8)
0531:         .np(8)
0532:         .kr(1)
0533:         .m(4)
0534:         .n(8)
0535:         .k(k)
0536:         .aStride(37)
0537:         .cStride(17)
0538:         .test(pytorch_q8conv_ukernel_4x8__neon);
0539:   }
0540: }
0541: 
0542: TEST(Q8CONV_4x8__NEON, k_gt_8_azp_only) {
0543:   TEST_REQUIRES_ARM_NEON;
0544:   for (size_t k = 9; k < 16; k++) {
0545:     GemmMicrokernelTester()
0546:         .mr(4)
0547:         .nr(8)
0548:         .np(8)
0549:         .kr(1)
0550:         .m(4)
0551:         .n(8)
0552:         .k(k)
0553:         .aStride(37)
0554:         .aZeroPoint(255)
0555:         .bZeroPoint(0)
0556:         .test(pytorch_q8conv_ukernel_4x8__neon);
0557:   }
0558: }
0559: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `GemmMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`GemmMicrokernelTester`。

### Lines 560-598 / 第 560-598 行

```cpp
0560: TEST(Q8CONV_4x8__NEON, k_gt_8_bzp_only) {
0561:   TEST_REQUIRES_ARM_NEON;
0562:   for (size_t k = 9; k < 16; k++) {
0563:     GemmMicrokernelTester()
0564:         .mr(4)
0565:         .nr(8)
0566:         .np(8)
0567:         .kr(1)
0568:         .m(4)
0569:         .n(8)
0570:         .k(k)
0571:         .aStride(37)
0572:         .aZeroPoint(0)
0573:         .bZeroPoint(255)
0574:         .test(pytorch_q8conv_ukernel_4x8__neon);
0575:   }
0576: }
0577: 
0578: TEST(Q8CONV_4x8__NEON, k_gt_8_subtile) {
0579:   TEST_REQUIRES_ARM_NEON;
0580:   for (size_t k = 9; k < 16; k++) {
0581:     for (uint32_t m = 1; m <= 4; m++) {
0582:       for (uint32_t n = 1; n <= 8; n++) {
0583:         GemmMicrokernelTester()
0584:             .mr(4)
0585:             .nr(8)
0586:             .np(8)
0587:             .kr(1)
0588:             .m(m)
0589:             .n(n)
0590:             .k(k)
0591:             .aStride(37)
0592:             .iterations(3)
0593:             .test(pytorch_q8conv_ukernel_4x8__neon);
0594:       }
0595:     }
0596:   }
0597: }
0598: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `GemmMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`GemmMicrokernelTester`。

### Lines 599-631 / 第 599-631 行

```cpp
0599: TEST(Q8CONV_4x8__NEON, k_div_8) {
0600:   TEST_REQUIRES_ARM_NEON;
0601:   for (size_t k = 16; k < 128; k += 8) {
0602:     GemmMicrokernelTester()
0603:         .mr(4)
0604:         .nr(8)
0605:         .np(8)
0606:         .kr(1)
0607:         .m(4)
0608:         .n(8)
0609:         .k(k)
0610:         .aStride(171)
0611:         .test(pytorch_q8conv_ukernel_4x8__neon);
0612:   }
0613: }
0614: 
0615: TEST(Q8CONV_4x8__NEON, k_div_8_strided_c) {
0616:   TEST_REQUIRES_ARM_NEON;
0617:   for (size_t k = 16; k < 128; k += 8) {
0618:     GemmMicrokernelTester()
0619:         .mr(4)
0620:         .nr(8)
0621:         .np(8)
0622:         .kr(1)
0623:         .m(4)
0624:         .n(8)
0625:         .k(k)
0626:         .aStride(171)
0627:         .cStride(17)
0628:         .test(pytorch_q8conv_ukernel_4x8__neon);
0629:   }
0630: }
0631: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `GemmMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`GemmMicrokernelTester`。

### Lines 632-666 / 第 632-666 行

```cpp
0632: TEST(Q8CONV_4x8__NEON, k_div_8_subtile) {
0633:   TEST_REQUIRES_ARM_NEON;
0634:   for (size_t k = 16; k < 128; k += 24) {
0635:     for (uint32_t m = 1; m <= 4; m++) {
0636:       for (uint32_t n = 1; n <= 8; n++) {
0637:         GemmMicrokernelTester()
0638:             .mr(4)
0639:             .nr(8)
0640:             .np(8)
0641:             .kr(1)
0642:             .m(m)
0643:             .n(n)
0644:             .k(k)
0645:             .aStride(171)
0646:             .iterations(3)
0647:             .test(pytorch_q8conv_ukernel_4x8__neon);
0648:       }
0649:     }
0650:   }
0651: }
0652: 
0653: TEST(Q8CONV_8x8__NEON, k_eq_8) {
0654:   TEST_REQUIRES_ARM_NEON;
0655:   GemmMicrokernelTester()
0656:       .mr(8)
0657:       .nr(8)
0658:       .np(8)
0659:       .kr(1)
0660:       .m(8)
0661:       .n(8)
0662:       .k(8)
0663:       .aStride(37)
0664:       .test(pytorch_q8conv_ukernel_8x8__neon);
0665: }
0666: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `GemmMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`GemmMicrokernelTester`。

### Lines 667-708 / 第 667-708 行

```cpp
0667: TEST(Q8CONV_8x8__NEON, k_eq_8_strided_c) {
0668:   TEST_REQUIRES_ARM_NEON;
0669:   GemmMicrokernelTester()
0670:       .mr(8)
0671:       .nr(8)
0672:       .np(8)
0673:       .kr(1)
0674:       .m(8)
0675:       .n(8)
0676:       .k(8)
0677:       .aStride(37)
0678:       .cStride(17)
0679:       .test(pytorch_q8conv_ukernel_8x8__neon);
0680: }
0681: 
0682: TEST(Q8CONV_8x8__NEON, k_eq_8_qmin128) {
0683:   TEST_REQUIRES_ARM_NEON;
0684:   GemmMicrokernelTester().mr(8).nr(8).np(8).kr(1).m(8).n(8).k(8).qmin(128).test(
0685:       pytorch_q8conv_ukernel_8x8__neon);
0686: }
0687: 
0688: TEST(Q8CONV_8x8__NEON, k_eq_8_qmax128) {
0689:   TEST_REQUIRES_ARM_NEON;
0690:   GemmMicrokernelTester().mr(8).nr(8).np(8).kr(1).m(8).n(8).k(8).qmax(128).test(
0691:       pytorch_q8conv_ukernel_8x8__neon);
0692: }
0693: 
0694: TEST(Q8CONV_8x8__NEON, k_eq_8_azp_only) {
0695:   TEST_REQUIRES_ARM_NEON;
0696:   GemmMicrokernelTester()
0697:       .mr(8)
0698:       .nr(8)
0699:       .np(8)
0700:       .kr(1)
0701:       .m(8)
0702:       .n(8)
0703:       .k(8)
0704:       .aZeroPoint(255)
0705:       .bZeroPoint(0)
0706:       .test(pytorch_q8conv_ukernel_8x8__neon);
0707: }
0708: 
```

- **EN:** This block implements local helper logic for `q8conv`. Key symbols: `GemmMicrokernelTester`.
- **CN:** 该代码块实现与 `q8conv` 相关的局部辅助逻辑。关键符号：`GemmMicrokernelTester`。

### Lines 709-739 / 第 709-739 行

```cpp
0709: TEST(Q8CONV_8x8__NEON, k_eq_8_bzp_only) {
0710:   TEST_REQUIRES_ARM_NEON;
0711:   GemmMicrokernelTester()
0712:       .mr(8)
0713:       .nr(8)
0714:       .np(8)
0715:       .kr(1)
0716:       .m(8)
0717:       .n(8)
0718:       .k(8)
0719:       .aZeroPoint(0)
0720:       .bZeroPoint(255)
0721:       .test(pytorch_q8conv_ukernel_8x8__neon);
0722: }
0723: 
0724: TEST(Q8CONV_8x8__NEON, k_gt_8) {
0725:   TEST_REQUIRES_ARM_NEON;
0726:   for (size_t k = 9; k < 16; k++) {
0727:     GemmMicrokernelTester()
0728:         .mr(8)
0729:         .nr(8)
0730:         .np(8)
0731:         .kr(1)
0732:         .m(8)
0733:         .n(8)
0734:         .k(k)
0735:         .aStride(37)
0736:         .test(pytorch_q8conv_ukernel_8x8__neon);
0737:   }
0738: }
0739: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `GemmMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`GemmMicrokernelTester`。

### Lines 740-774 / 第 740-774 行

```cpp
0740: TEST(Q8CONV_8x8__NEON, k_gt_8_strided_c) {
0741:   TEST_REQUIRES_ARM_NEON;
0742:   for (size_t k = 9; k < 16; k++) {
0743:     GemmMicrokernelTester()
0744:         .mr(8)
0745:         .nr(8)
0746:         .np(8)
0747:         .kr(1)
0748:         .m(8)
0749:         .n(8)
0750:         .k(k)
0751:         .aStride(37)
0752:         .cStride(17)
0753:         .test(pytorch_q8conv_ukernel_8x8__neon);
0754:   }
0755: }
0756: 
0757: TEST(Q8CONV_8x8__NEON, k_gt_8_azp_only) {
0758:   TEST_REQUIRES_ARM_NEON;
0759:   for (size_t k = 9; k < 16; k++) {
0760:     GemmMicrokernelTester()
0761:         .mr(8)
0762:         .nr(8)
0763:         .np(8)
0764:         .kr(1)
0765:         .m(8)
0766:         .n(8)
0767:         .k(k)
0768:         .aStride(37)
0769:         .aZeroPoint(255)
0770:         .bZeroPoint(0)
0771:         .test(pytorch_q8conv_ukernel_8x8__neon);
0772:   }
0773: }
0774: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `GemmMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`GemmMicrokernelTester`。

### Lines 775-813 / 第 775-813 行

```cpp
0775: TEST(Q8CONV_8x8__NEON, k_gt_8_bzp_only) {
0776:   TEST_REQUIRES_ARM_NEON;
0777:   for (size_t k = 9; k < 16; k++) {
0778:     GemmMicrokernelTester()
0779:         .mr(8)
0780:         .nr(8)
0781:         .np(8)
0782:         .kr(1)
0783:         .m(8)
0784:         .n(8)
0785:         .k(k)
0786:         .aStride(37)
0787:         .aZeroPoint(0)
0788:         .bZeroPoint(255)
0789:         .test(pytorch_q8conv_ukernel_8x8__neon);
0790:   }
0791: }
0792: 
0793: TEST(Q8CONV_8x8__NEON, k_gt_8_subtile) {
0794:   TEST_REQUIRES_ARM_NEON;
0795:   for (size_t k = 9; k < 16; k++) {
0796:     for (uint32_t m = 1; m <= 8; m++) {
0797:       for (uint32_t n = 1; n <= 8; n++) {
0798:         GemmMicrokernelTester()
0799:             .mr(8)
0800:             .nr(8)
0801:             .np(8)
0802:             .kr(1)
0803:             .m(m)
0804:             .n(n)
0805:             .k(k)
0806:             .aStride(37)
0807:             .iterations(3)
0808:             .test(pytorch_q8conv_ukernel_8x8__neon);
0809:       }
0810:     }
0811:   }
0812: }
0813: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `GemmMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`GemmMicrokernelTester`。

### Lines 814-846 / 第 814-846 行

```cpp
0814: TEST(Q8CONV_8x8__NEON, k_div_8) {
0815:   TEST_REQUIRES_ARM_NEON;
0816:   for (size_t k = 16; k < 128; k += 8) {
0817:     GemmMicrokernelTester()
0818:         .mr(8)
0819:         .nr(8)
0820:         .np(8)
0821:         .kr(1)
0822:         .m(8)
0823:         .n(8)
0824:         .k(k)
0825:         .aStride(171)
0826:         .test(pytorch_q8conv_ukernel_8x8__neon);
0827:   }
0828: }
0829: 
0830: TEST(Q8CONV_8x8__NEON, k_div_8_strided_c) {
0831:   TEST_REQUIRES_ARM_NEON;
0832:   for (size_t k = 16; k < 128; k += 8) {
0833:     GemmMicrokernelTester()
0834:         .mr(8)
0835:         .nr(8)
0836:         .np(8)
0837:         .kr(1)
0838:         .m(8)
0839:         .n(8)
0840:         .k(k)
0841:         .aStride(171)
0842:         .cStride(17)
0843:         .test(pytorch_q8conv_ukernel_8x8__neon);
0844:   }
0845: }
0846: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `GemmMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`GemmMicrokernelTester`。

### Lines 847-883 / 第 847-883 行

```cpp
0847: TEST(Q8CONV_8x8__NEON, k_div_8_subtile) {
0848:   TEST_REQUIRES_ARM_NEON;
0849:   for (size_t k = 16; k < 128; k += 24) {
0850:     for (uint32_t m = 1; m <= 8; m++) {
0851:       for (uint32_t n = 1; n <= 8; n++) {
0852:         GemmMicrokernelTester()
0853:             .mr(8)
0854:             .nr(8)
0855:             .np(8)
0856:             .kr(1)
0857:             .m(m)
0858:             .n(n)
0859:             .k(k)
0860:             .aStride(171)
0861:             .iterations(3)
0862:             .test(pytorch_q8conv_ukernel_8x8__neon);
0863:       }
0864:     }
0865:   }
0866: }
0867: #endif
0868: 
0869: #if CPUINFO_ARCH_X86 || CPUINFO_ARCH_X86_64
0870: TEST(Q8CONV_4x4c2__SSE2, k_eq_8) {
0871:   TEST_REQUIRES_X86_SSE2;
0872:   GemmMicrokernelTester()
0873:       .mr(4)
0874:       .nr(4)
0875:       .np(4)
0876:       .kr(2)
0877:       .m(4)
0878:       .n(4)
0879:       .k(8)
0880:       .aStride(37)
0881:       .test(pytorch_q8conv_ukernel_4x4c2__sse2);
0882: }
0883: 
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends; iterates over tensors, descriptors, options, or runtime state. Key symbols: `GemmMicrokernelTester`.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端；遍历张量、描述符、配置项或运行时状态。关键符号：`GemmMicrokernelTester`。

### Lines 884-925 / 第 884-925 行

```cpp
0884: TEST(Q8CONV_4x4c2__SSE2, k_eq_8_strided_c) {
0885:   TEST_REQUIRES_X86_SSE2;
0886:   GemmMicrokernelTester()
0887:       .mr(4)
0888:       .nr(4)
0889:       .np(4)
0890:       .kr(2)
0891:       .m(4)
0892:       .n(4)
0893:       .k(8)
0894:       .aStride(37)
0895:       .cStride(17)
0896:       .test(pytorch_q8conv_ukernel_4x4c2__sse2);
0897: }
0898: 
0899: TEST(Q8CONV_4x4c2__SSE2, k_eq_8_qmin128) {
0900:   TEST_REQUIRES_X86_SSE2;
0901:   GemmMicrokernelTester().mr(4).nr(4).np(4).kr(2).m(4).n(4).k(8).qmin(128).test(
0902:       pytorch_q8conv_ukernel_4x4c2__sse2);
0903: }
0904: 
0905: TEST(Q8CONV_4x4c2__SSE2, k_eq_8_qmax128) {
0906:   TEST_REQUIRES_X86_SSE2;
0907:   GemmMicrokernelTester().mr(4).nr(4).np(4).kr(2).m(4).n(4).k(8).qmax(128).test(
0908:       pytorch_q8conv_ukernel_4x4c2__sse2);
0909: }
0910: 
0911: TEST(Q8CONV_4x4c2__SSE2, k_eq_8_azp_only) {
0912:   TEST_REQUIRES_X86_SSE2;
0913:   GemmMicrokernelTester()
0914:       .mr(4)
0915:       .nr(4)
0916:       .np(4)
0917:       .kr(2)
0918:       .m(4)
0919:       .n(4)
0920:       .k(8)
0921:       .aZeroPoint(255)
0922:       .bZeroPoint(0)
0923:       .test(pytorch_q8conv_ukernel_4x4c2__sse2);
0924: }
0925: 
```

- **EN:** This block implements local helper logic for `q8conv`. Key symbols: `GemmMicrokernelTester`.
- **CN:** 该代码块实现与 `q8conv` 相关的局部辅助逻辑。关键符号：`GemmMicrokernelTester`。

### Lines 926-956 / 第 926-956 行

```cpp
0926: TEST(Q8CONV_4x4c2__SSE2, k_eq_8_bzp_only) {
0927:   TEST_REQUIRES_X86_SSE2;
0928:   GemmMicrokernelTester()
0929:       .mr(4)
0930:       .nr(4)
0931:       .np(4)
0932:       .kr(2)
0933:       .m(4)
0934:       .n(4)
0935:       .k(8)
0936:       .aZeroPoint(0)
0937:       .bZeroPoint(255)
0938:       .test(pytorch_q8conv_ukernel_4x4c2__sse2);
0939: }
0940: 
0941: TEST(Q8CONV_4x4c2__SSE2, k_gt_8) {
0942:   TEST_REQUIRES_X86_SSE2;
0943:   for (size_t k = 9; k < 16; k++) {
0944:     GemmMicrokernelTester()
0945:         .mr(4)
0946:         .nr(4)
0947:         .np(4)
0948:         .kr(2)
0949:         .m(4)
0950:         .n(4)
0951:         .k(k)
0952:         .aStride(37)
0953:         .test(pytorch_q8conv_ukernel_4x4c2__sse2);
0954:   }
0955: }
0956: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `GemmMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`GemmMicrokernelTester`。

### Lines 957-991 / 第 957-991 行

```cpp
0957: TEST(Q8CONV_4x4c2__SSE2, k_gt_8_strided_c) {
0958:   TEST_REQUIRES_X86_SSE2;
0959:   for (size_t k = 9; k < 16; k++) {
0960:     GemmMicrokernelTester()
0961:         .mr(4)
0962:         .nr(4)
0963:         .np(4)
0964:         .kr(2)
0965:         .m(4)
0966:         .n(4)
0967:         .k(k)
0968:         .aStride(37)
0969:         .cStride(17)
0970:         .test(pytorch_q8conv_ukernel_4x4c2__sse2);
0971:   }
0972: }
0973: 
0974: TEST(Q8CONV_4x4c2__SSE2, k_gt_8_azp_only) {
0975:   TEST_REQUIRES_X86_SSE2;
0976:   for (size_t k = 9; k < 16; k++) {
0977:     GemmMicrokernelTester()
0978:         .mr(4)
0979:         .nr(4)
0980:         .np(4)
0981:         .kr(2)
0982:         .m(4)
0983:         .n(4)
0984:         .k(k)
0985:         .aStride(37)
0986:         .aZeroPoint(255)
0987:         .bZeroPoint(0)
0988:         .test(pytorch_q8conv_ukernel_4x4c2__sse2);
0989:   }
0990: }
0991: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `GemmMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`GemmMicrokernelTester`。

### Lines 992-1030 / 第 992-1030 行

```cpp
0992: TEST(Q8CONV_4x4c2__SSE2, k_gt_8_bzp_only) {
0993:   TEST_REQUIRES_X86_SSE2;
0994:   for (size_t k = 9; k < 16; k++) {
0995:     GemmMicrokernelTester()
0996:         .mr(4)
0997:         .nr(4)
0998:         .np(4)
0999:         .kr(2)
1000:         .m(4)
1001:         .n(4)
1002:         .k(k)
1003:         .aStride(37)
1004:         .aZeroPoint(0)
1005:         .bZeroPoint(255)
1006:         .test(pytorch_q8conv_ukernel_4x4c2__sse2);
1007:   }
1008: }
1009: 
1010: TEST(Q8CONV_4x4c2__SSE2, k_gt_8_subtile) {
1011:   TEST_REQUIRES_X86_SSE2;
1012:   for (size_t k = 9; k < 16; k++) {
1013:     for (uint32_t m = 1; m <= 4; m++) {
1014:       for (uint32_t n = 1; n <= 4; n++) {
1015:         GemmMicrokernelTester()
1016:             .mr(4)
1017:             .nr(4)
1018:             .np(4)
1019:             .kr(2)
1020:             .m(m)
1021:             .n(n)
1022:             .k(k)
1023:             .aStride(37)
1024:             .iterations(3)
1025:             .test(pytorch_q8conv_ukernel_4x4c2__sse2);
1026:       }
1027:     }
1028:   }
1029: }
1030: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `GemmMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`GemmMicrokernelTester`。

### Lines 1031-1063 / 第 1031-1063 行

```cpp
1031: TEST(Q8CONV_4x4c2__SSE2, k_div_8) {
1032:   TEST_REQUIRES_X86_SSE2;
1033:   for (size_t k = 16; k < 128; k += 8) {
1034:     GemmMicrokernelTester()
1035:         .mr(4)
1036:         .nr(4)
1037:         .np(4)
1038:         .kr(2)
1039:         .m(4)
1040:         .n(4)
1041:         .k(k)
1042:         .aStride(171)
1043:         .test(pytorch_q8conv_ukernel_4x4c2__sse2);
1044:   }
1045: }
1046: 
1047: TEST(Q8CONV_4x4c2__SSE2, k_div_8_strided_c) {
1048:   TEST_REQUIRES_X86_SSE2;
1049:   for (size_t k = 16; k < 128; k += 8) {
1050:     GemmMicrokernelTester()
1051:         .mr(4)
1052:         .nr(4)
1053:         .np(4)
1054:         .kr(2)
1055:         .m(4)
1056:         .n(4)
1057:         .k(k)
1058:         .aStride(171)
1059:         .cStride(17)
1060:         .test(pytorch_q8conv_ukernel_4x4c2__sse2);
1061:   }
1062: }
1063: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `GemmMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`GemmMicrokernelTester`。

### Lines 1064-1084 / 第 1064-1084 行

```cpp
1064: TEST(Q8CONV_4x4c2__SSE2, k_div_8_subtile) {
1065:   TEST_REQUIRES_X86_SSE2;
1066:   for (size_t k = 16; k < 128; k += 24) {
1067:     for (uint32_t m = 1; m <= 4; m++) {
1068:       for (uint32_t n = 1; n <= 4; n++) {
1069:         GemmMicrokernelTester()
1070:             .mr(4)
1071:             .nr(4)
1072:             .np(4)
1073:             .kr(2)
1074:             .m(m)
1075:             .n(n)
1076:             .k(k)
1077:             .aStride(171)
1078:             .iterations(3)
1079:             .test(pytorch_q8conv_ukernel_4x4c2__sse2);
1080:       }
1081:     }
1082:   }
1083: }
1084: #endif
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends; iterates over tensors, descriptors, options, or runtime state. Key symbols: `GemmMicrokernelTester`.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端；遍历张量、描述符、配置项或运行时状态。关键符号：`GemmMicrokernelTester`。


## Key Concepts / 关键概念
- **ATen native operator logic** — ATen 原生算子逻辑
- **GPU kernel/runtime interaction** — GPU 内核/运行时交互
- **Linear algebra backend integration** — 线性代数后端集成
- **Runtime validation and invariants** — 运行时校验与不变量
- **Core symbols: GemmMicrokernelTester** — 核心符号：GemmMicrokernelTester

## Dependencies / 依赖关系
- **Internal includes / 内部头文件**: 无
- **External includes / 外部头文件**: `cpuinfo.h`, `gtest/gtest.h`, `qnnpack/isa-checks.h`, `qnnpack/q8conv.h`, `gemm-microkernel-tester.h`
- **Namespaces / 命名空间**: 无
- **Representative symbols / 代表性符号**: `GemmMicrokernelTester`
