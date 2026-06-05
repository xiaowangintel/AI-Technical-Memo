# q8gemm.cc — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `aten/src/ATen/native/quantized/cpu/qnnpack/test/q8gemm.cc`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements ATen native operators, checks, helper kernels, and backend dispatch behavior. This specific file centers on `q8gemm.cc`. The file header highlights: "All rights reserved. This source code is licensed under the BSD-style license found in the LICENSE file in the root directory of this source tree.." Matrix multiplication and GEMM-style kernels are a central concern.
- **Purpose (CN)**: 实现 ATen 原生算子、运行时检查、辅助内核以及后端分发行为。 该文件具体围绕 `q8gemm.cc` 展开。 文件头部注释也概括了其核心职责。 矩阵乘法与 GEMM 风格内核是该文件的核心关注点。

## Line-by-Line Analysis / 逐行分析
### Lines 1-51 / 第 1-51 行

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
0013: #include <qnnpack/q8gemm.h>
0014: 
0015: #include "gemm-microkernel-tester.h"
0016: 
0017: #if CPUINFO_ARCH_ARM
0018: TEST(Q8GEMM_4x8__AARCH32_NEON, k_eq_8) {
0019:   TEST_REQUIRES_ARM_NEON;
0020:   GemmMicrokernelTester().mr(4).nr(8).np(8).kr(1).m(4).n(8).k(8).test(
0021:       pytorch_q8gemm_ukernel_4x8__aarch32_neon);
0022: }
0023: 
0024: TEST(Q8GEMM_4x8__AARCH32_NEON, k_eq_8_strided_a) {
0025:   TEST_REQUIRES_ARM_NEON;
0026:   GemmMicrokernelTester()
0027:       .mr(4)
0028:       .nr(8)
0029:       .np(8)
0030:       .kr(1)
0031:       .m(4)
0032:       .n(8)
0033:       .k(8)
0034:       .aStride(37)
0035:       .test(pytorch_q8gemm_ukernel_4x8__aarch32_neon);
0036: }
0037: 
0038: TEST(Q8GEMM_4x8__AARCH32_NEON, k_eq_8_strided_c) {
0039:   TEST_REQUIRES_ARM_NEON;
0040:   GemmMicrokernelTester()
0041:       .mr(4)
0042:       .nr(8)
0043:       .np(8)
0044:       .kr(1)
0045:       .m(4)
0046:       .n(8)
0047:       .k(8)
0048:       .cStride(17)
0049:       .test(pytorch_q8gemm_ukernel_4x8__aarch32_neon);
0050: }
0051: 
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends. Key symbols: `GemmMicrokernelTester`.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端。关键符号：`GemmMicrokernelTester`。

### Lines 52-106 / 第 52-106 行

```cpp
0052: TEST(Q8GEMM_4x8__AARCH32_NEON, k_eq_8_qmin128) {
0053:   TEST_REQUIRES_ARM_NEON;
0054:   GemmMicrokernelTester().mr(4).nr(8).np(8).kr(1).m(4).n(8).k(8).qmin(128).test(
0055:       pytorch_q8gemm_ukernel_4x8__aarch32_neon);
0056: }
0057: 
0058: TEST(Q8GEMM_4x8__AARCH32_NEON, k_eq_8_qmax128) {
0059:   TEST_REQUIRES_ARM_NEON;
0060:   GemmMicrokernelTester().mr(4).nr(8).np(8).kr(1).m(4).n(8).k(8).qmax(128).test(
0061:       pytorch_q8gemm_ukernel_4x8__aarch32_neon);
0062: }
0063: 
0064: TEST(Q8GEMM_4x8__AARCH32_NEON, k_eq_8_azp0) {
0065:   TEST_REQUIRES_ARM_NEON;
0066:   GemmMicrokernelTester()
0067:       .mr(4)
0068:       .nr(8)
0069:       .np(8)
0070:       .kr(1)
0071:       .m(4)
0072:       .n(8)
0073:       .k(8)
0074:       .aZeroPoint(0)
0075:       .test(pytorch_q8gemm_ukernel_4x8__aarch32_neon);
0076: }
0077: 
0078: TEST(Q8GEMM_4x8__AARCH32_NEON, k_eq_8_bzp0) {
0079:   TEST_REQUIRES_ARM_NEON;
0080:   GemmMicrokernelTester()
0081:       .mr(4)
0082:       .nr(8)
0083:       .np(8)
0084:       .kr(1)
0085:       .m(4)
0086:       .n(8)
0087:       .k(8)
0088:       .bZeroPoint(0)
0089:       .test(pytorch_q8gemm_ukernel_4x8__aarch32_neon);
0090: }
0091: 
0092: TEST(Q8GEMM_4x8__AARCH32_NEON, k_eq_8_nozp) {
0093:   TEST_REQUIRES_ARM_NEON;
0094:   GemmMicrokernelTester()
0095:       .mr(4)
0096:       .nr(8)
0097:       .np(8)
0098:       .kr(1)
0099:       .m(4)
0100:       .n(8)
0101:       .k(8)
0102:       .aZeroPoint(0)
0103:       .bZeroPoint(0)
0104:       .test(pytorch_q8gemm_ukernel_4x8__aarch32_neon);
0105: }
0106: 
```

- **EN:** This block implements local helper logic for `q8gemm`. Key symbols: `GemmMicrokernelTester`.
- **CN:** 该代码块实现与 `q8gemm` 相关的局部辅助逻辑。关键符号：`GemmMicrokernelTester`。

### Lines 107-162 / 第 107-162 行

```cpp
0107: TEST(Q8GEMM_4x8__AARCH32_NEON, k_gt_8) {
0108:   TEST_REQUIRES_ARM_NEON;
0109:   for (size_t k = 9; k < 16; k++) {
0110:     GemmMicrokernelTester().mr(4).nr(8).np(8).kr(1).m(4).n(8).k(k).test(
0111:         pytorch_q8gemm_ukernel_4x8__aarch32_neon);
0112:   }
0113: }
0114: 
0115: TEST(Q8GEMM_4x8__AARCH32_NEON, k_gt_8_strided_a) {
0116:   TEST_REQUIRES_ARM_NEON;
0117:   for (size_t k = 9; k < 16; k++) {
0118:     GemmMicrokernelTester()
0119:         .mr(4)
0120:         .nr(8)
0121:         .np(8)
0122:         .kr(1)
0123:         .m(4)
0124:         .n(8)
0125:         .k(k)
0126:         .aStride(37)
0127:         .test(pytorch_q8gemm_ukernel_4x8__aarch32_neon);
0128:   }
0129: }
0130: 
0131: TEST(Q8GEMM_4x8__AARCH32_NEON, k_gt_8_strided_c) {
0132:   TEST_REQUIRES_ARM_NEON;
0133:   for (size_t k = 9; k < 16; k++) {
0134:     GemmMicrokernelTester()
0135:         .mr(4)
0136:         .nr(8)
0137:         .np(8)
0138:         .kr(1)
0139:         .m(4)
0140:         .n(8)
0141:         .k(k)
0142:         .cStride(17)
0143:         .test(pytorch_q8gemm_ukernel_4x8__aarch32_neon);
0144:   }
0145: }
0146: 
0147: TEST(Q8GEMM_4x8__AARCH32_NEON, k_gt_8_azp0) {
0148:   TEST_REQUIRES_ARM_NEON;
0149:   for (size_t k = 9; k < 16; k++) {
0150:     GemmMicrokernelTester()
0151:         .mr(4)
0152:         .nr(8)
0153:         .np(8)
0154:         .kr(1)
0155:         .m(4)
0156:         .n(8)
0157:         .k(k)
0158:         .aZeroPoint(0)
0159:         .test(pytorch_q8gemm_ukernel_4x8__aarch32_neon);
0160:   }
0161: }
0162: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `GemmMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`GemmMicrokernelTester`。

### Lines 163-215 / 第 163-215 行

```cpp
0163: TEST(Q8GEMM_4x8__AARCH32_NEON, k_gt_8_bzp0) {
0164:   TEST_REQUIRES_ARM_NEON;
0165:   for (size_t k = 9; k < 16; k++) {
0166:     GemmMicrokernelTester()
0167:         .mr(4)
0168:         .nr(8)
0169:         .np(8)
0170:         .kr(1)
0171:         .m(4)
0172:         .n(8)
0173:         .k(k)
0174:         .bZeroPoint(0)
0175:         .test(pytorch_q8gemm_ukernel_4x8__aarch32_neon);
0176:   }
0177: }
0178: 
0179: TEST(Q8GEMM_4x8__AARCH32_NEON, k_gt_8_nozp) {
0180:   TEST_REQUIRES_ARM_NEON;
0181:   for (size_t k = 9; k < 16; k++) {
0182:     GemmMicrokernelTester()
0183:         .mr(4)
0184:         .nr(8)
0185:         .np(8)
0186:         .kr(1)
0187:         .m(4)
0188:         .n(8)
0189:         .k(k)
0190:         .aZeroPoint(0)
0191:         .bZeroPoint(0)
0192:         .test(pytorch_q8gemm_ukernel_4x8__aarch32_neon);
0193:   }
0194: }
0195: 
0196: TEST(Q8GEMM_4x8__AARCH32_NEON, k_gt_8_subtile) {
0197:   TEST_REQUIRES_ARM_NEON;
0198:   for (size_t k = 9; k < 16; k++) {
0199:     for (uint32_t m = 1; m <= 4; m++) {
0200:       for (uint32_t n = 1; n <= 8; n++) {
0201:         GemmMicrokernelTester()
0202:             .mr(4)
0203:             .nr(8)
0204:             .np(8)
0205:             .kr(1)
0206:             .m(m)
0207:             .n(n)
0208:             .k(k)
0209:             .iterations(3)
0210:             .test(pytorch_q8gemm_ukernel_4x8__aarch32_neon);
0211:       }
0212:     }
0213:   }
0214: }
0215: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `GemmMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`GemmMicrokernelTester`。

### Lines 216-275 / 第 216-275 行

```cpp
0216: TEST(Q8GEMM_4x8__AARCH32_NEON, k_div_8) {
0217:   TEST_REQUIRES_ARM_NEON;
0218:   for (size_t k = 16; k < 128; k += 8) {
0219:     GemmMicrokernelTester().mr(4).nr(8).np(8).kr(1).m(4).n(8).k(k).test(
0220:         pytorch_q8gemm_ukernel_4x8__aarch32_neon);
0221:   }
0222: }
0223: 
0224: TEST(Q8GEMM_4x8__AARCH32_NEON, k_div_8_strided_a) {
0225:   TEST_REQUIRES_ARM_NEON;
0226:   for (size_t k = 16; k < 128; k += 8) {
0227:     GemmMicrokernelTester()
0228:         .mr(4)
0229:         .nr(8)
0230:         .np(8)
0231:         .kr(1)
0232:         .m(4)
0233:         .n(8)
0234:         .k(k)
0235:         .aStride(171)
0236:         .test(pytorch_q8gemm_ukernel_4x8__aarch32_neon);
0237:   }
0238: }
0239: 
0240: TEST(Q8GEMM_4x8__AARCH32_NEON, k_div_8_strided_c) {
0241:   TEST_REQUIRES_ARM_NEON;
0242:   for (size_t k = 16; k < 128; k += 8) {
0243:     GemmMicrokernelTester()
0244:         .mr(4)
0245:         .nr(8)
0246:         .np(8)
0247:         .kr(1)
0248:         .m(4)
0249:         .n(8)
0250:         .k(k)
0251:         .cStride(17)
0252:         .test(pytorch_q8gemm_ukernel_4x8__aarch32_neon);
0253:   }
0254: }
0255: 
0256: TEST(Q8GEMM_4x8__AARCH32_NEON, k_div_8_subtile) {
0257:   TEST_REQUIRES_ARM_NEON;
0258:   for (size_t k = 16; k < 128; k += 24) {
0259:     for (uint32_t m = 1; m <= 4; m++) {
0260:       for (uint32_t n = 1; n <= 8; n++) {
0261:         GemmMicrokernelTester()
0262:             .mr(4)
0263:             .nr(8)
0264:             .np(8)
0265:             .kr(1)
0266:             .m(m)
0267:             .n(n)
0268:             .k(k)
0269:             .iterations(3)
0270:             .test(pytorch_q8gemm_ukernel_4x8__aarch32_neon);
0271:       }
0272:     }
0273:   }
0274: }
0275: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `GemmMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`GemmMicrokernelTester`。

### Lines 276-325 / 第 276-325 行

```cpp
0276: //
0277: // Dynamic Quantization
0278: //
0279: 
0280: TEST(Q8GEMM_DQ_4x8__AARCH32_NEON, k_eq_8) {
0281:   TEST_REQUIRES_ARM_NEON;
0282:   GemmMicrokernelTester().mr(4).nr(8).np(8).kr(1).m(4).n(8).k(8).test(
0283:       pytorch_q8gemm_dq_ukernel_4x8__aarch32_neon);
0284: }
0285: 
0286: TEST(Q8GEMM_DQ_4x8__AARCH32_NEON, k_eq_8_strided_a) {
0287:   TEST_REQUIRES_ARM_NEON;
0288:   GemmMicrokernelTester()
0289:       .mr(4)
0290:       .nr(8)
0291:       .np(8)
0292:       .kr(1)
0293:       .m(4)
0294:       .n(8)
0295:       .k(8)
0296:       .aStride(37)
0297:       .test(pytorch_q8gemm_dq_ukernel_4x8__aarch32_neon);
0298: }
0299: 
0300: TEST(Q8GEMM_DQ_4x8__AARCH32_NEON, k_eq_8_strided_c) {
0301:   TEST_REQUIRES_ARM_NEON;
0302:   GemmMicrokernelTester()
0303:       .mr(4)
0304:       .nr(8)
0305:       .np(8)
0306:       .kr(1)
0307:       .m(4)
0308:       .n(8)
0309:       .k(8)
0310:       .cStride(17)
0311:       .test(pytorch_q8gemm_dq_ukernel_4x8__aarch32_neon);
0312: }
0313: 
0314: TEST(Q8GEMM_DQ_4x8__AARCH32_NEON, k_eq_8_qmin128) {
0315:   TEST_REQUIRES_ARM_NEON;
0316:   GemmMicrokernelTester().mr(4).nr(8).np(8).kr(1).m(4).n(8).k(8).qmin(128).test(
0317:       pytorch_q8gemm_dq_ukernel_4x8__aarch32_neon);
0318: }
0319: 
0320: TEST(Q8GEMM_DQ_4x8__AARCH32_NEON, k_eq_8_qmax128) {
0321:   TEST_REQUIRES_ARM_NEON;
0322:   GemmMicrokernelTester().mr(4).nr(8).np(8).kr(1).m(4).n(8).k(8).qmax(128).test(
0323:       pytorch_q8gemm_dq_ukernel_4x8__aarch32_neon);
0324: }
0325: 
```

- **EN:** This block implements local helper logic for `q8gemm`. Key symbols: `GemmMicrokernelTester`.
- **CN:** 该代码块实现与 `q8gemm` 相关的局部辅助逻辑。关键符号：`GemmMicrokernelTester`。

### Lines 326-376 / 第 326-376 行

```cpp
0326: TEST(Q8GEMM_DQ_4x8__AARCH32_NEON, k_eq_8_azp0) {
0327:   TEST_REQUIRES_ARM_NEON;
0328:   GemmMicrokernelTester()
0329:       .mr(4)
0330:       .nr(8)
0331:       .np(8)
0332:       .kr(1)
0333:       .m(4)
0334:       .n(8)
0335:       .k(8)
0336:       .aZeroPoint(0)
0337:       .test(pytorch_q8gemm_dq_ukernel_4x8__aarch32_neon);
0338: }
0339: 
0340: TEST(Q8GEMM_DQ_4x8__AARCH32_NEON, k_eq_8_bzp0) {
0341:   TEST_REQUIRES_ARM_NEON;
0342:   GemmMicrokernelTester()
0343:       .mr(4)
0344:       .nr(8)
0345:       .np(8)
0346:       .kr(1)
0347:       .m(4)
0348:       .n(8)
0349:       .k(8)
0350:       .bZeroPoint(0)
0351:       .test(pytorch_q8gemm_dq_ukernel_4x8__aarch32_neon);
0352: }
0353: 
0354: TEST(Q8GEMM_DQ_4x8__AARCH32_NEON, k_eq_8_nozp) {
0355:   TEST_REQUIRES_ARM_NEON;
0356:   GemmMicrokernelTester()
0357:       .mr(4)
0358:       .nr(8)
0359:       .np(8)
0360:       .kr(1)
0361:       .m(4)
0362:       .n(8)
0363:       .k(8)
0364:       .aZeroPoint(0)
0365:       .bZeroPoint(0)
0366:       .test(pytorch_q8gemm_dq_ukernel_4x8__aarch32_neon);
0367: }
0368: 
0369: TEST(Q8GEMM_DQ_4x8__AARCH32_NEON, k_gt_8) {
0370:   TEST_REQUIRES_ARM_NEON;
0371:   for (size_t k = 9; k < 16; k++) {
0372:     GemmMicrokernelTester().mr(4).nr(8).np(8).kr(1).m(4).n(8).k(k).test(
0373:         pytorch_q8gemm_dq_ukernel_4x8__aarch32_neon);
0374:   }
0375: }
0376: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `GemmMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`GemmMicrokernelTester`。

### Lines 377-427 / 第 377-427 行

```cpp
0377: TEST(Q8GEMM_DQ_4x8__AARCH32_NEON, k_gt_8_strided_a) {
0378:   TEST_REQUIRES_ARM_NEON;
0379:   for (size_t k = 9; k < 16; k++) {
0380:     GemmMicrokernelTester()
0381:         .mr(4)
0382:         .nr(8)
0383:         .np(8)
0384:         .kr(1)
0385:         .m(4)
0386:         .n(8)
0387:         .k(k)
0388:         .aStride(37)
0389:         .test(pytorch_q8gemm_dq_ukernel_4x8__aarch32_neon);
0390:   }
0391: }
0392: 
0393: TEST(Q8GEMM_DQ_4x8__AARCH32_NEON, k_gt_8_strided_c) {
0394:   TEST_REQUIRES_ARM_NEON;
0395:   for (size_t k = 9; k < 16; k++) {
0396:     GemmMicrokernelTester()
0397:         .mr(4)
0398:         .nr(8)
0399:         .np(8)
0400:         .kr(1)
0401:         .m(4)
0402:         .n(8)
0403:         .k(k)
0404:         .cStride(17)
0405:         .test(pytorch_q8gemm_dq_ukernel_4x8__aarch32_neon);
0406:   }
0407: }
0408: 
0409: TEST(Q8GEMM_DQ_4x8__AARCH32_NEON, k_gt_8_azp0) {
0410:   TEST_REQUIRES_ARM_NEON;
0411:   for (size_t k = 9; k < 16; k++) {
0412:     GemmMicrokernelTester()
0413:         .mr(4)
0414:         .nr(8)
0415:         .np(8)
0416:         .kr(1)
0417:         .m(4)
0418:         .n(8)
0419:         .k(k)
0420:         .aZeroPoint(0)
0421:         .test(pytorch_q8gemm_dq_ukernel_4x8__aarch32_neon);
0422:   }
0423: }
0424: 
0425: TEST(Q8GEMM_DQ_4x8__AARCH32_NEON, k_gt_8_bzp0) {
0426:   TEST_REQUIRES_ARM_NEON;
0427:   for (size_t k = 9; k < 16; k++) {
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `GemmMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`GemmMicrokernelTester`。

### Lines 428-477 / 第 428-477 行

```cpp
0428:     GemmMicrokernelTester()
0429:         .mr(4)
0430:         .nr(8)
0431:         .np(8)
0432:         .kr(1)
0433:         .m(4)
0434:         .n(8)
0435:         .k(k)
0436:         .bZeroPoint(0)
0437:         .test(pytorch_q8gemm_dq_ukernel_4x8__aarch32_neon);
0438:   }
0439: }
0440: 
0441: TEST(Q8GEMM_DQ_4x8__AARCH32_NEON, k_gt_8_nozp) {
0442:   TEST_REQUIRES_ARM_NEON;
0443:   for (size_t k = 9; k < 16; k++) {
0444:     GemmMicrokernelTester()
0445:         .mr(4)
0446:         .nr(8)
0447:         .np(8)
0448:         .kr(1)
0449:         .m(4)
0450:         .n(8)
0451:         .k(k)
0452:         .aZeroPoint(0)
0453:         .bZeroPoint(0)
0454:         .test(pytorch_q8gemm_dq_ukernel_4x8__aarch32_neon);
0455:   }
0456: }
0457: 
0458: TEST(Q8GEMM_DQ_4x8__AARCH32_NEON, k_gt_8_subtile) {
0459:   TEST_REQUIRES_ARM_NEON;
0460:   for (size_t k = 9; k < 16; k++) {
0461:     for (uint32_t m = 1; m <= 4; m++) {
0462:       for (uint32_t n = 1; n <= 8; n++) {
0463:         GemmMicrokernelTester()
0464:             .mr(4)
0465:             .nr(8)
0466:             .np(8)
0467:             .kr(1)
0468:             .m(m)
0469:             .n(n)
0470:             .k(k)
0471:             .iterations(3)
0472:             .test(pytorch_q8gemm_dq_ukernel_4x8__aarch32_neon);
0473:       }
0474:     }
0475:   }
0476: }
0477: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `GemmMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`GemmMicrokernelTester`。

### Lines 478-537 / 第 478-537 行

```cpp
0478: TEST(Q8GEMM_DQ_4x8__AARCH32_NEON, k_div_8) {
0479:   TEST_REQUIRES_ARM_NEON;
0480:   for (size_t k = 16; k < 128; k += 8) {
0481:     GemmMicrokernelTester().mr(4).nr(8).np(8).kr(1).m(4).n(8).k(k).test(
0482:         pytorch_q8gemm_dq_ukernel_4x8__aarch32_neon);
0483:   }
0484: }
0485: 
0486: TEST(Q8GEMM_DQ_4x8__AARCH32_NEON, k_div_8_strided_a) {
0487:   TEST_REQUIRES_ARM_NEON;
0488:   for (size_t k = 16; k < 128; k += 8) {
0489:     GemmMicrokernelTester()
0490:         .mr(4)
0491:         .nr(8)
0492:         .np(8)
0493:         .kr(1)
0494:         .m(4)
0495:         .n(8)
0496:         .k(k)
0497:         .aStride(171)
0498:         .test(pytorch_q8gemm_dq_ukernel_4x8__aarch32_neon);
0499:   }
0500: }
0501: 
0502: TEST(Q8GEMM_DQ_4x8__AARCH32_NEON, k_div_8_strided_c) {
0503:   TEST_REQUIRES_ARM_NEON;
0504:   for (size_t k = 16; k < 128; k += 8) {
0505:     GemmMicrokernelTester()
0506:         .mr(4)
0507:         .nr(8)
0508:         .np(8)
0509:         .kr(1)
0510:         .m(4)
0511:         .n(8)
0512:         .k(k)
0513:         .cStride(17)
0514:         .test(pytorch_q8gemm_dq_ukernel_4x8__aarch32_neon);
0515:   }
0516: }
0517: 
0518: TEST(Q8GEMM_DQ_4x8__AARCH32_NEON, k_div_8_subtile) {
0519:   TEST_REQUIRES_ARM_NEON;
0520:   for (size_t k = 16; k < 128; k += 24) {
0521:     for (uint32_t m = 1; m <= 4; m++) {
0522:       for (uint32_t n = 1; n <= 8; n++) {
0523:         GemmMicrokernelTester()
0524:             .mr(4)
0525:             .nr(8)
0526:             .np(8)
0527:             .kr(1)
0528:             .m(m)
0529:             .n(n)
0530:             .k(k)
0531:             .iterations(3)
0532:             .test(pytorch_q8gemm_dq_ukernel_4x8__aarch32_neon);
0533:       }
0534:     }
0535:   }
0536: }
0537: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `GemmMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`GemmMicrokernelTester`。

### Lines 538-597 / 第 538-597 行

```cpp
0538: TEST(Q8GEMM_4x8c2_XZP__AARCH32_NEON, k_eq_8) {
0539:   TEST_REQUIRES_ARM_NEON;
0540:   GemmMicrokernelTester().mr(4).nr(8).np(8).kr(2).m(4).n(8).k(8).test(
0541:       pytorch_q8gemm_xzp_ukernel_4x8c2__aarch32_neon);
0542: }
0543: 
0544: TEST(Q8GEMM_4x8c2_XZP__AARCH32_NEON, k_eq_8_strided_a) {
0545:   TEST_REQUIRES_ARM_NEON;
0546:   GemmMicrokernelTester()
0547:       .mr(4)
0548:       .nr(8)
0549:       .np(8)
0550:       .kr(2)
0551:       .m(4)
0552:       .n(8)
0553:       .k(8)
0554:       .aStride(37)
0555:       .test(pytorch_q8gemm_xzp_ukernel_4x8c2__aarch32_neon);
0556: }
0557: 
0558: TEST(Q8GEMM_4x8c2_XZP__AARCH32_NEON, k_eq_8_strided_c) {
0559:   TEST_REQUIRES_ARM_NEON;
0560:   GemmMicrokernelTester()
0561:       .mr(4)
0562:       .nr(8)
0563:       .np(8)
0564:       .kr(2)
0565:       .m(4)
0566:       .n(8)
0567:       .k(8)
0568:       .cStride(17)
0569:       .test(pytorch_q8gemm_xzp_ukernel_4x8c2__aarch32_neon);
0570: }
0571: 
0572: TEST(Q8GEMM_4x8c2_XZP__AARCH32_NEON, k_eq_8_qmin128) {
0573:   TEST_REQUIRES_ARM_NEON;
0574:   GemmMicrokernelTester().mr(4).nr(8).np(8).kr(2).m(4).n(8).k(8).qmin(128).test(
0575:       pytorch_q8gemm_xzp_ukernel_4x8c2__aarch32_neon);
0576: }
0577: 
0578: TEST(Q8GEMM_4x8c2_XZP__AARCH32_NEON, k_eq_8_qmax128) {
0579:   TEST_REQUIRES_ARM_NEON;
0580:   GemmMicrokernelTester().mr(4).nr(8).np(8).kr(2).m(4).n(8).k(8).qmax(128).test(
0581:       pytorch_q8gemm_xzp_ukernel_4x8c2__aarch32_neon);
0582: }
0583: 
0584: TEST(Q8GEMM_4x8c2_XZP__AARCH32_NEON, k_eq_8_azp0) {
0585:   TEST_REQUIRES_ARM_NEON;
0586:   GemmMicrokernelTester()
0587:       .mr(4)
0588:       .nr(8)
0589:       .np(8)
0590:       .kr(2)
0591:       .m(4)
0592:       .n(8)
0593:       .k(8)
0594:       .aZeroPoint(0)
0595:       .test(pytorch_q8gemm_xzp_ukernel_4x8c2__aarch32_neon);
0596: }
0597: 
```

- **EN:** This block implements local helper logic for `q8gemm`. Key symbols: `GemmMicrokernelTester`.
- **CN:** 该代码块实现与 `q8gemm` 相关的局部辅助逻辑。关键符号：`GemmMicrokernelTester`。

### Lines 598-650 / 第 598-650 行

```cpp
0598: TEST(Q8GEMM_4x8c2_XZP__AARCH32_NEON, k_eq_8_bzp0) {
0599:   TEST_REQUIRES_ARM_NEON;
0600:   GemmMicrokernelTester()
0601:       .mr(4)
0602:       .nr(8)
0603:       .np(8)
0604:       .kr(2)
0605:       .m(4)
0606:       .n(8)
0607:       .k(8)
0608:       .bZeroPoint(0)
0609:       .test(pytorch_q8gemm_xzp_ukernel_4x8c2__aarch32_neon);
0610: }
0611: 
0612: TEST(Q8GEMM_4x8c2_XZP__AARCH32_NEON, k_eq_8_nozp) {
0613:   TEST_REQUIRES_ARM_NEON;
0614:   GemmMicrokernelTester()
0615:       .mr(4)
0616:       .nr(8)
0617:       .np(8)
0618:       .kr(2)
0619:       .m(4)
0620:       .n(8)
0621:       .k(8)
0622:       .aZeroPoint(0)
0623:       .bZeroPoint(0)
0624:       .test(pytorch_q8gemm_xzp_ukernel_4x8c2__aarch32_neon);
0625: }
0626: 
0627: TEST(Q8GEMM_4x8c2_XZP__AARCH32_NEON, k_gt_8) {
0628:   TEST_REQUIRES_ARM_NEON;
0629:   for (size_t k = 9; k < 16; k++) {
0630:     GemmMicrokernelTester().mr(4).nr(8).np(8).kr(2).m(4).n(8).k(k).test(
0631:         pytorch_q8gemm_xzp_ukernel_4x8c2__aarch32_neon);
0632:   }
0633: }
0634: 
0635: TEST(Q8GEMM_4x8c2_XZP__AARCH32_NEON, k_gt_8_strided_a) {
0636:   TEST_REQUIRES_ARM_NEON;
0637:   for (size_t k = 9; k < 16; k++) {
0638:     GemmMicrokernelTester()
0639:         .mr(4)
0640:         .nr(8)
0641:         .np(8)
0642:         .kr(2)
0643:         .m(4)
0644:         .n(8)
0645:         .k(k)
0646:         .aStride(37)
0647:         .test(pytorch_q8gemm_xzp_ukernel_4x8c2__aarch32_neon);
0648:   }
0649: }
0650: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `GemmMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`GemmMicrokernelTester`。

### Lines 651-701 / 第 651-701 行

```cpp
0651: TEST(Q8GEMM_4x8c2_XZP__AARCH32_NEON, k_gt_8_strided_c) {
0652:   TEST_REQUIRES_ARM_NEON;
0653:   for (size_t k = 9; k < 16; k++) {
0654:     GemmMicrokernelTester()
0655:         .mr(4)
0656:         .nr(8)
0657:         .np(8)
0658:         .kr(2)
0659:         .m(4)
0660:         .n(8)
0661:         .k(k)
0662:         .cStride(17)
0663:         .test(pytorch_q8gemm_xzp_ukernel_4x8c2__aarch32_neon);
0664:   }
0665: }
0666: 
0667: TEST(Q8GEMM_4x8c2_XZP__AARCH32_NEON, k_gt_8_azp0) {
0668:   TEST_REQUIRES_ARM_NEON;
0669:   for (size_t k = 9; k < 16; k++) {
0670:     GemmMicrokernelTester()
0671:         .mr(4)
0672:         .nr(8)
0673:         .np(8)
0674:         .kr(2)
0675:         .m(4)
0676:         .n(8)
0677:         .k(k)
0678:         .aZeroPoint(0)
0679:         .test(pytorch_q8gemm_xzp_ukernel_4x8c2__aarch32_neon);
0680:   }
0681: }
0682: 
0683: TEST(Q8GEMM_4x8c2_XZP__AARCH32_NEON, k_gt_8_bzp0) {
0684:   TEST_REQUIRES_ARM_NEON;
0685:   for (size_t k = 9; k < 16; k++) {
0686:     GemmMicrokernelTester()
0687:         .mr(4)
0688:         .nr(8)
0689:         .np(8)
0690:         .kr(2)
0691:         .m(4)
0692:         .n(8)
0693:         .k(k)
0694:         .bZeroPoint(0)
0695:         .test(pytorch_q8gemm_xzp_ukernel_4x8c2__aarch32_neon);
0696:   }
0697: }
0698: 
0699: TEST(Q8GEMM_4x8c2_XZP__AARCH32_NEON, k_gt_8_nozp) {
0700:   TEST_REQUIRES_ARM_NEON;
0701:   for (size_t k = 9; k < 16; k++) {
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `GemmMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`GemmMicrokernelTester`。

### Lines 702-759 / 第 702-759 行

```cpp
0702:     GemmMicrokernelTester()
0703:         .mr(4)
0704:         .nr(8)
0705:         .np(8)
0706:         .kr(2)
0707:         .m(4)
0708:         .n(8)
0709:         .k(k)
0710:         .aZeroPoint(0)
0711:         .bZeroPoint(0)
0712:         .test(pytorch_q8gemm_xzp_ukernel_4x8c2__aarch32_neon);
0713:   }
0714: }
0715: 
0716: TEST(Q8GEMM_4x8c2_XZP__AARCH32_NEON, k_gt_8_subtile) {
0717:   TEST_REQUIRES_ARM_NEON;
0718:   for (size_t k = 9; k < 16; k++) {
0719:     for (uint32_t m = 1; m <= 4; m++) {
0720:       for (uint32_t n = 1; n <= 8; n++) {
0721:         GemmMicrokernelTester()
0722:             .mr(4)
0723:             .nr(8)
0724:             .np(8)
0725:             .kr(2)
0726:             .m(m)
0727:             .n(n)
0728:             .k(k)
0729:             .iterations(3)
0730:             .test(pytorch_q8gemm_xzp_ukernel_4x8c2__aarch32_neon);
0731:       }
0732:     }
0733:   }
0734: }
0735: 
0736: TEST(Q8GEMM_4x8c2_XZP__AARCH32_NEON, k_div_8) {
0737:   TEST_REQUIRES_ARM_NEON;
0738:   for (size_t k = 16; k < 128; k += 8) {
0739:     GemmMicrokernelTester().mr(4).nr(8).np(8).kr(2).m(4).n(8).k(k).test(
0740:         pytorch_q8gemm_xzp_ukernel_4x8c2__aarch32_neon);
0741:   }
0742: }
0743: 
0744: TEST(Q8GEMM_4x8c2_XZP__AARCH32_NEON, k_div_8_strided_a) {
0745:   TEST_REQUIRES_ARM_NEON;
0746:   for (size_t k = 16; k < 128; k += 8) {
0747:     GemmMicrokernelTester()
0748:         .mr(4)
0749:         .nr(8)
0750:         .np(8)
0751:         .kr(2)
0752:         .m(4)
0753:         .n(8)
0754:         .k(k)
0755:         .aStride(171)
0756:         .test(pytorch_q8gemm_xzp_ukernel_4x8c2__aarch32_neon);
0757:   }
0758: }
0759: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `GemmMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`GemmMicrokernelTester`。

### Lines 760-815 / 第 760-815 行

```cpp
0760: TEST(Q8GEMM_4x8c2_XZP__AARCH32_NEON, k_div_8_strided_c) {
0761:   TEST_REQUIRES_ARM_NEON;
0762:   for (size_t k = 16; k < 128; k += 8) {
0763:     GemmMicrokernelTester()
0764:         .mr(4)
0765:         .nr(8)
0766:         .np(8)
0767:         .kr(2)
0768:         .m(4)
0769:         .n(8)
0770:         .k(k)
0771:         .cStride(17)
0772:         .test(pytorch_q8gemm_xzp_ukernel_4x8c2__aarch32_neon);
0773:   }
0774: }
0775: 
0776: TEST(Q8GEMM_4x8c2_XZP__AARCH32_NEON, k_div_8_subtile) {
0777:   TEST_REQUIRES_ARM_NEON;
0778:   for (size_t k = 16; k < 128; k += 24) {
0779:     for (uint32_t m = 1; m <= 4; m++) {
0780:       for (uint32_t n = 1; n <= 8; n++) {
0781:         GemmMicrokernelTester()
0782:             .mr(4)
0783:             .nr(8)
0784:             .np(8)
0785:             .kr(2)
0786:             .m(m)
0787:             .n(n)
0788:             .k(k)
0789:             .iterations(3)
0790:             .test(pytorch_q8gemm_xzp_ukernel_4x8c2__aarch32_neon);
0791:       }
0792:     }
0793:   }
0794: }
0795: #endif
0796: 
0797: #if CPUINFO_ARCH_ARM64
0798: TEST(Q8GEMM_8x8__AARCH64_NEON, k_eq_8) {
0799:   GemmMicrokernelTester().mr(8).nr(8).np(8).kr(1).m(8).n(8).k(8).test(
0800:       pytorch_q8gemm_ukernel_8x8__aarch64_neon);
0801: }
0802: 
0803: TEST(Q8GEMM_8x8__AARCH64_NEON, k_eq_8_strided_a) {
0804:   GemmMicrokernelTester()
0805:       .mr(8)
0806:       .nr(8)
0807:       .np(8)
0808:       .kr(1)
0809:       .m(8)
0810:       .n(8)
0811:       .k(8)
0812:       .aStride(37)
0813:       .test(pytorch_q8gemm_ukernel_8x8__aarch64_neon);
0814: }
0815: 
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends; iterates over tensors, descriptors, options, or runtime state. Key symbols: `GemmMicrokernelTester`.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端；遍历张量、描述符、配置项或运行时状态。关键符号：`GemmMicrokernelTester`。

### Lines 816-878 / 第 816-878 行

```cpp
0816: TEST(Q8GEMM_8x8__AARCH64_NEON, k_eq_8_strided_c) {
0817:   GemmMicrokernelTester()
0818:       .mr(8)
0819:       .nr(8)
0820:       .np(8)
0821:       .kr(1)
0822:       .m(8)
0823:       .n(8)
0824:       .k(8)
0825:       .cStride(17)
0826:       .test(pytorch_q8gemm_ukernel_8x8__aarch64_neon);
0827: }
0828: 
0829: TEST(Q8GEMM_8x8__AARCH64_NEON, k_eq_8_qmin128) {
0830:   GemmMicrokernelTester().mr(8).nr(8).np(8).kr(1).m(8).n(8).k(8).qmin(128).test(
0831:       pytorch_q8gemm_ukernel_8x8__aarch64_neon);
0832: }
0833: 
0834: TEST(Q8GEMM_8x8__AARCH64_NEON, k_eq_8_qmax128) {
0835:   GemmMicrokernelTester().mr(8).nr(8).np(8).kr(1).m(8).n(8).k(8).qmax(128).test(
0836:       pytorch_q8gemm_ukernel_8x8__aarch64_neon);
0837: }
0838: 
0839: TEST(Q8GEMM_8x8__AARCH64_NEON, k_eq_8_azp0) {
0840:   GemmMicrokernelTester()
0841:       .mr(8)
0842:       .nr(8)
0843:       .np(8)
0844:       .kr(1)
0845:       .m(8)
0846:       .n(8)
0847:       .k(8)
0848:       .aZeroPoint(0)
0849:       .test(pytorch_q8gemm_ukernel_8x8__aarch64_neon);
0850: }
0851: 
0852: TEST(Q8GEMM_8x8__AARCH64_NEON, k_eq_8_bzp0) {
0853:   GemmMicrokernelTester()
0854:       .mr(8)
0855:       .nr(8)
0856:       .np(8)
0857:       .kr(1)
0858:       .m(8)
0859:       .n(8)
0860:       .k(8)
0861:       .bZeroPoint(0)
0862:       .test(pytorch_q8gemm_ukernel_8x8__aarch64_neon);
0863: }
0864: 
0865: TEST(Q8GEMM_8x8__AARCH64_NEON, k_eq_8_nozp) {
0866:   GemmMicrokernelTester()
0867:       .mr(8)
0868:       .nr(8)
0869:       .np(8)
0870:       .kr(1)
0871:       .m(8)
0872:       .n(8)
0873:       .k(8)
0874:       .aZeroPoint(0)
0875:       .bZeroPoint(0)
0876:       .test(pytorch_q8gemm_ukernel_8x8__aarch64_neon);
0877: }
0878: 
```

- **EN:** This block implements local helper logic for `q8gemm`. Key symbols: `GemmMicrokernelTester`.
- **CN:** 该代码块实现与 `q8gemm` 相关的局部辅助逻辑。关键符号：`GemmMicrokernelTester`。

### Lines 879-930 / 第 879-930 行

```cpp
0879: TEST(Q8GEMM_8x8__AARCH64_NEON, k_gt_8) {
0880:   for (size_t k = 9; k < 16; k++) {
0881:     GemmMicrokernelTester().mr(8).nr(8).np(8).kr(1).m(8).n(8).k(k).test(
0882:         pytorch_q8gemm_ukernel_8x8__aarch64_neon);
0883:   }
0884: }
0885: 
0886: TEST(Q8GEMM_8x8__AARCH64_NEON, k_gt_8_strided_a) {
0887:   for (size_t k = 9; k < 16; k++) {
0888:     GemmMicrokernelTester()
0889:         .mr(8)
0890:         .nr(8)
0891:         .np(8)
0892:         .kr(1)
0893:         .m(8)
0894:         .n(8)
0895:         .k(k)
0896:         .aStride(37)
0897:         .test(pytorch_q8gemm_ukernel_8x8__aarch64_neon);
0898:   }
0899: }
0900: 
0901: TEST(Q8GEMM_8x8__AARCH64_NEON, k_gt_8_strided_c) {
0902:   for (size_t k = 9; k < 16; k++) {
0903:     GemmMicrokernelTester()
0904:         .mr(8)
0905:         .nr(8)
0906:         .np(8)
0907:         .kr(1)
0908:         .m(8)
0909:         .n(8)
0910:         .k(k)
0911:         .cStride(17)
0912:         .test(pytorch_q8gemm_ukernel_8x8__aarch64_neon);
0913:   }
0914: }
0915: 
0916: TEST(Q8GEMM_8x8__AARCH64_NEON, k_gt_8_azp0) {
0917:   for (size_t k = 9; k < 16; k++) {
0918:     GemmMicrokernelTester()
0919:         .mr(8)
0920:         .nr(8)
0921:         .np(8)
0922:         .kr(1)
0923:         .m(8)
0924:         .n(8)
0925:         .k(k)
0926:         .aZeroPoint(0)
0927:         .test(pytorch_q8gemm_ukernel_8x8__aarch64_neon);
0928:   }
0929: }
0930: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `GemmMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`GemmMicrokernelTester`。

### Lines 931-980 / 第 931-980 行

```cpp
0931: TEST(Q8GEMM_8x8__AARCH64_NEON, k_gt_8_bzp0) {
0932:   for (size_t k = 9; k < 16; k++) {
0933:     GemmMicrokernelTester()
0934:         .mr(8)
0935:         .nr(8)
0936:         .np(8)
0937:         .kr(1)
0938:         .m(8)
0939:         .n(8)
0940:         .k(k)
0941:         .bZeroPoint(0)
0942:         .test(pytorch_q8gemm_ukernel_8x8__aarch64_neon);
0943:   }
0944: }
0945: 
0946: TEST(Q8GEMM_8x8__AARCH64_NEON, k_gt_8_nozp) {
0947:   for (size_t k = 9; k < 16; k++) {
0948:     GemmMicrokernelTester()
0949:         .mr(8)
0950:         .nr(8)
0951:         .np(8)
0952:         .kr(1)
0953:         .m(8)
0954:         .n(8)
0955:         .k(k)
0956:         .aZeroPoint(0)
0957:         .bZeroPoint(0)
0958:         .test(pytorch_q8gemm_ukernel_8x8__aarch64_neon);
0959:   }
0960: }
0961: 
0962: TEST(Q8GEMM_8x8__AARCH64_NEON, k_gt_8_subtile) {
0963:   for (size_t k = 9; k < 16; k++) {
0964:     for (uint32_t m = 1; m <= 8; m++) {
0965:       for (uint32_t n = 1; n <= 8; n++) {
0966:         GemmMicrokernelTester()
0967:             .mr(8)
0968:             .nr(8)
0969:             .np(8)
0970:             .kr(1)
0971:             .m(m)
0972:             .n(n)
0973:             .k(k)
0974:             .iterations(3)
0975:             .test(pytorch_q8gemm_ukernel_8x8__aarch64_neon);
0976:       }
0977:     }
0978:   }
0979: }
0980: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `GemmMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`GemmMicrokernelTester`。

### Lines 981-1036 / 第 981-1036 行

```cpp
0981: TEST(Q8GEMM_8x8__AARCH64_NEON, k_div_8) {
0982:   for (size_t k = 16; k < 128; k += 8) {
0983:     GemmMicrokernelTester().mr(8).nr(8).np(8).kr(1).m(8).n(8).k(k).test(
0984:         pytorch_q8gemm_ukernel_8x8__aarch64_neon);
0985:   }
0986: }
0987: 
0988: TEST(Q8GEMM_8x8__AARCH64_NEON, k_div_8_strided_a) {
0989:   for (size_t k = 16; k < 128; k += 8) {
0990:     GemmMicrokernelTester()
0991:         .mr(8)
0992:         .nr(8)
0993:         .np(8)
0994:         .kr(1)
0995:         .m(8)
0996:         .n(8)
0997:         .k(k)
0998:         .aStride(171)
0999:         .test(pytorch_q8gemm_ukernel_8x8__aarch64_neon);
1000:   }
1001: }
1002: 
1003: TEST(Q8GEMM_8x8__AARCH64_NEON, k_div_8_strided_c) {
1004:   for (size_t k = 16; k < 128; k += 8) {
1005:     GemmMicrokernelTester()
1006:         .mr(8)
1007:         .nr(8)
1008:         .np(8)
1009:         .kr(1)
1010:         .m(8)
1011:         .n(8)
1012:         .k(k)
1013:         .cStride(17)
1014:         .test(pytorch_q8gemm_ukernel_8x8__aarch64_neon);
1015:   }
1016: }
1017: 
1018: TEST(Q8GEMM_8x8__AARCH64_NEON, k_div_8_subtile) {
1019:   for (size_t k = 16; k < 128; k += 24) {
1020:     for (uint32_t m = 1; m <= 8; m++) {
1021:       for (uint32_t n = 1; n <= 8; n++) {
1022:         GemmMicrokernelTester()
1023:             .mr(8)
1024:             .nr(8)
1025:             .np(8)
1026:             .kr(1)
1027:             .m(m)
1028:             .n(n)
1029:             .k(k)
1030:             .iterations(3)
1031:             .test(pytorch_q8gemm_ukernel_8x8__aarch64_neon);
1032:       }
1033:     }
1034:   }
1035: }
1036: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `GemmMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`GemmMicrokernelTester`。

### Lines 1037-1094 / 第 1037-1094 行

```cpp
1037: //
1038: // Dynamic Quantization
1039: //
1040: 
1041: TEST(Q8GEMM_DQ_8x8__AARCH64_NEON, k_eq_8) {
1042:   GemmMicrokernelTester().mr(8).nr(8).np(8).kr(1).m(8).n(8).k(8).test(
1043:       pytorch_q8gemm_dq_ukernel_8x8__aarch64_neon);
1044: }
1045: 
1046: TEST(Q8GEMM_DQ_8x8__AARCH64_NEON, k_eq_8_strided_a) {
1047:   GemmMicrokernelTester()
1048:       .mr(8)
1049:       .nr(8)
1050:       .np(8)
1051:       .kr(1)
1052:       .m(8)
1053:       .n(8)
1054:       .k(8)
1055:       .aStride(37)
1056:       .test(pytorch_q8gemm_dq_ukernel_8x8__aarch64_neon);
1057: }
1058: 
1059: TEST(Q8GEMM_DQ_8x8__AARCH64_NEON, k_eq_8_strided_c) {
1060:   GemmMicrokernelTester()
1061:       .mr(8)
1062:       .nr(8)
1063:       .np(8)
1064:       .kr(1)
1065:       .m(8)
1066:       .n(8)
1067:       .k(8)
1068:       .cStride(17)
1069:       .test(pytorch_q8gemm_dq_ukernel_8x8__aarch64_neon);
1070: }
1071: 
1072: TEST(Q8GEMM_DQ_8x8__AARCH64_NEON, k_eq_8_qmin128) {
1073:   GemmMicrokernelTester().mr(8).nr(8).np(8).kr(1).m(8).n(8).k(8).qmin(128).test(
1074:       pytorch_q8gemm_dq_ukernel_8x8__aarch64_neon);
1075: }
1076: 
1077: TEST(Q8GEMM_DQ_8x8__AARCH64_NEON, k_eq_8_qmax128) {
1078:   GemmMicrokernelTester().mr(8).nr(8).np(8).kr(1).m(8).n(8).k(8).qmax(128).test(
1079:       pytorch_q8gemm_dq_ukernel_8x8__aarch64_neon);
1080: }
1081: 
1082: TEST(Q8GEMM_DQ_8x8__AARCH64_NEON, k_eq_8_azp0) {
1083:   GemmMicrokernelTester()
1084:       .mr(8)
1085:       .nr(8)
1086:       .np(8)
1087:       .kr(1)
1088:       .m(8)
1089:       .n(8)
1090:       .k(8)
1091:       .aZeroPoint(0)
1092:       .test(pytorch_q8gemm_dq_ukernel_8x8__aarch64_neon);
1093: }
1094: 
```

- **EN:** This block implements local helper logic for `q8gemm`. Key symbols: `GemmMicrokernelTester`.
- **CN:** 该代码块实现与 `q8gemm` 相关的局部辅助逻辑。关键符号：`GemmMicrokernelTester`。

### Lines 1095-1145 / 第 1095-1145 行

```cpp
1095: TEST(Q8GEMM_DQ_8x8__AARCH64_NEON, k_eq_8_bzp0) {
1096:   GemmMicrokernelTester()
1097:       .mr(8)
1098:       .nr(8)
1099:       .np(8)
1100:       .kr(1)
1101:       .m(8)
1102:       .n(8)
1103:       .k(8)
1104:       .bZeroPoint(0)
1105:       .test(pytorch_q8gemm_dq_ukernel_8x8__aarch64_neon);
1106: }
1107: 
1108: TEST(Q8GEMM_DQ_8x8__AARCH64_NEON, k_eq_8_nozp) {
1109:   GemmMicrokernelTester()
1110:       .mr(8)
1111:       .nr(8)
1112:       .np(8)
1113:       .kr(1)
1114:       .m(8)
1115:       .n(8)
1116:       .k(8)
1117:       .aZeroPoint(0)
1118:       .bZeroPoint(0)
1119:       .test(pytorch_q8gemm_dq_ukernel_8x8__aarch64_neon);
1120: }
1121: 
1122: TEST(Q8GEMM_DQ_8x8__AARCH64_NEON, k_gt_8) {
1123:   for (size_t k = 9; k < 16; k++) {
1124:     GemmMicrokernelTester().mr(8).nr(8).np(8).kr(1).m(8).n(8).k(k).test(
1125:         pytorch_q8gemm_dq_ukernel_8x8__aarch64_neon);
1126:   }
1127: }
1128: 
1129: TEST(Q8GEMM_DQ_8x8__AARCH64_NEON, k_gt_8_strided_a) {
1130:   for (size_t k = 9; k < 16; k++) {
1131:     GemmMicrokernelTester()
1132:         .mr(8)
1133:         .nr(8)
1134:         .np(8)
1135:         .kr(1)
1136:         .m(8)
1137:         .n(8)
1138:         .k(k)
1139:         .aStride(37)
1140:         .test(pytorch_q8gemm_dq_ukernel_8x8__aarch64_neon);
1141:   }
1142: }
1143: 
1144: TEST(Q8GEMM_DQ_8x8__AARCH64_NEON, k_gt_8_strided_c) {
1145:   for (size_t k = 9; k < 16; k++) {
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `GemmMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`GemmMicrokernelTester`。

### Lines 1146-1204 / 第 1146-1204 行

```cpp
1146:     GemmMicrokernelTester()
1147:         .mr(8)
1148:         .nr(8)
1149:         .np(8)
1150:         .kr(1)
1151:         .m(8)
1152:         .n(8)
1153:         .k(k)
1154:         .cStride(17)
1155:         .test(pytorch_q8gemm_dq_ukernel_8x8__aarch64_neon);
1156:   }
1157: }
1158: 
1159: TEST(Q8GEMM_DQ_8x8__AARCH64_NEON, k_gt_8_azp0) {
1160:   for (size_t k = 9; k < 16; k++) {
1161:     GemmMicrokernelTester()
1162:         .mr(8)
1163:         .nr(8)
1164:         .np(8)
1165:         .kr(1)
1166:         .m(8)
1167:         .n(8)
1168:         .k(k)
1169:         .aZeroPoint(0)
1170:         .test(pytorch_q8gemm_dq_ukernel_8x8__aarch64_neon);
1171:   }
1172: }
1173: 
1174: TEST(Q8GEMM_DQ_8x8__AARCH64_NEON, k_gt_8_bzp0) {
1175:   for (size_t k = 9; k < 16; k++) {
1176:     GemmMicrokernelTester()
1177:         .mr(8)
1178:         .nr(8)
1179:         .np(8)
1180:         .kr(1)
1181:         .m(8)
1182:         .n(8)
1183:         .k(k)
1184:         .bZeroPoint(0)
1185:         .test(pytorch_q8gemm_dq_ukernel_8x8__aarch64_neon);
1186:   }
1187: }
1188: 
1189: TEST(Q8GEMM_DQ_8x8__AARCH64_NEON, k_gt_8_nozp) {
1190:   for (size_t k = 9; k < 16; k++) {
1191:     GemmMicrokernelTester()
1192:         .mr(8)
1193:         .nr(8)
1194:         .np(8)
1195:         .kr(1)
1196:         .m(8)
1197:         .n(8)
1198:         .k(k)
1199:         .aZeroPoint(0)
1200:         .bZeroPoint(0)
1201:         .test(pytorch_q8gemm_dq_ukernel_8x8__aarch64_neon);
1202:   }
1203: }
1204: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `GemmMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`GemmMicrokernelTester`。

### Lines 1205-1260 / 第 1205-1260 行

```cpp
1205: TEST(Q8GEMM_DQ_8x8__AARCH64_NEON, k_gt_8_subtile) {
1206:   for (size_t k = 9; k < 16; k++) {
1207:     for (uint32_t m = 1; m <= 8; m++) {
1208:       for (uint32_t n = 1; n <= 8; n++) {
1209:         GemmMicrokernelTester()
1210:             .mr(8)
1211:             .nr(8)
1212:             .np(8)
1213:             .kr(1)
1214:             .m(m)
1215:             .n(n)
1216:             .k(k)
1217:             .iterations(3)
1218:             .test(pytorch_q8gemm_dq_ukernel_8x8__aarch64_neon);
1219:       }
1220:     }
1221:   }
1222: }
1223: 
1224: TEST(Q8GEMM_DQ_8x8__AARCH64_NEON, k_div_8) {
1225:   for (size_t k = 16; k < 128; k += 8) {
1226:     GemmMicrokernelTester().mr(8).nr(8).np(8).kr(1).m(8).n(8).k(k).test(
1227:         pytorch_q8gemm_dq_ukernel_8x8__aarch64_neon);
1228:   }
1229: }
1230: 
1231: TEST(Q8GEMM_DQ_8x8__AARCH64_NEON, k_div_8_strided_a) {
1232:   for (size_t k = 16; k < 128; k += 8) {
1233:     GemmMicrokernelTester()
1234:         .mr(8)
1235:         .nr(8)
1236:         .np(8)
1237:         .kr(1)
1238:         .m(8)
1239:         .n(8)
1240:         .k(k)
1241:         .aStride(171)
1242:         .test(pytorch_q8gemm_dq_ukernel_8x8__aarch64_neon);
1243:   }
1244: }
1245: 
1246: TEST(Q8GEMM_DQ_8x8__AARCH64_NEON, k_div_8_strided_c) {
1247:   for (size_t k = 16; k < 128; k += 8) {
1248:     GemmMicrokernelTester()
1249:         .mr(8)
1250:         .nr(8)
1251:         .np(8)
1252:         .kr(1)
1253:         .m(8)
1254:         .n(8)
1255:         .k(k)
1256:         .cStride(17)
1257:         .test(pytorch_q8gemm_dq_ukernel_8x8__aarch64_neon);
1258:   }
1259: }
1260: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `GemmMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`GemmMicrokernelTester`。

### Lines 1261-1315 / 第 1261-1315 行

```cpp
1261: TEST(Q8GEMM_DQ_8x8__AARCH64_NEON, k_div_8_subtile) {
1262:   for (size_t k = 16; k < 128; k += 24) {
1263:     for (uint32_t m = 1; m <= 8; m++) {
1264:       for (uint32_t n = 1; n <= 8; n++) {
1265:         GemmMicrokernelTester()
1266:             .mr(8)
1267:             .nr(8)
1268:             .np(8)
1269:             .kr(1)
1270:             .m(m)
1271:             .n(n)
1272:             .k(k)
1273:             .iterations(3)
1274:             .test(pytorch_q8gemm_dq_ukernel_8x8__aarch64_neon);
1275:       }
1276:     }
1277:   }
1278: }
1279: #endif
1280: 
1281: #if CPUINFO_ARCH_ARM || CPUINFO_ARCH_ARM64
1282: TEST(Q8GEMM_4x8__NEON, k_eq_8) {
1283:   TEST_REQUIRES_ARM_NEON;
1284:   GemmMicrokernelTester().mr(4).nr(8).np(8).kr(1).m(4).n(8).k(8).test(
1285:       pytorch_q8gemm_ukernel_4x8__neon);
1286: }
1287: 
1288: TEST(Q8GEMM_4x8__NEON, k_eq_8_strided_a) {
1289:   TEST_REQUIRES_ARM_NEON;
1290:   GemmMicrokernelTester()
1291:       .mr(4)
1292:       .nr(8)
1293:       .np(8)
1294:       .kr(1)
1295:       .m(4)
1296:       .n(8)
1297:       .k(8)
1298:       .aStride(37)
1299:       .test(pytorch_q8gemm_ukernel_4x8__neon);
1300: }
1301: 
1302: TEST(Q8GEMM_4x8__NEON, k_eq_8_strided_c) {
1303:   TEST_REQUIRES_ARM_NEON;
1304:   GemmMicrokernelTester()
1305:       .mr(4)
1306:       .nr(8)
1307:       .np(8)
1308:       .kr(1)
1309:       .m(4)
1310:       .n(8)
1311:       .k(8)
1312:       .cStride(17)
1313:       .test(pytorch_q8gemm_ukernel_4x8__neon);
1314: }
1315: 
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends; iterates over tensors, descriptors, options, or runtime state. Key symbols: `GemmMicrokernelTester`.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端；遍历张量、描述符、配置项或运行时状态。关键符号：`GemmMicrokernelTester`。

### Lines 1316-1370 / 第 1316-1370 行

```cpp
1316: TEST(Q8GEMM_4x8__NEON, k_eq_8_qmin128) {
1317:   TEST_REQUIRES_ARM_NEON;
1318:   GemmMicrokernelTester().mr(4).nr(8).np(8).kr(1).m(4).n(8).k(8).qmin(128).test(
1319:       pytorch_q8gemm_ukernel_4x8__neon);
1320: }
1321: 
1322: TEST(Q8GEMM_4x8__NEON, k_eq_8_qmax128) {
1323:   TEST_REQUIRES_ARM_NEON;
1324:   GemmMicrokernelTester().mr(4).nr(8).np(8).kr(1).m(4).n(8).k(8).qmax(128).test(
1325:       pytorch_q8gemm_ukernel_4x8__neon);
1326: }
1327: 
1328: TEST(Q8GEMM_4x8__NEON, k_eq_8_azp0) {
1329:   TEST_REQUIRES_ARM_NEON;
1330:   GemmMicrokernelTester()
1331:       .mr(4)
1332:       .nr(8)
1333:       .np(8)
1334:       .kr(1)
1335:       .m(4)
1336:       .n(8)
1337:       .k(8)
1338:       .aZeroPoint(0)
1339:       .test(pytorch_q8gemm_ukernel_4x8__neon);
1340: }
1341: 
1342: TEST(Q8GEMM_4x8__NEON, k_eq_8_bzp0) {
1343:   TEST_REQUIRES_ARM_NEON;
1344:   GemmMicrokernelTester()
1345:       .mr(4)
1346:       .nr(8)
1347:       .np(8)
1348:       .kr(1)
1349:       .m(4)
1350:       .n(8)
1351:       .k(8)
1352:       .bZeroPoint(0)
1353:       .test(pytorch_q8gemm_ukernel_4x8__neon);
1354: }
1355: 
1356: TEST(Q8GEMM_4x8__NEON, k_eq_8_nozp) {
1357:   TEST_REQUIRES_ARM_NEON;
1358:   GemmMicrokernelTester()
1359:       .mr(4)
1360:       .nr(8)
1361:       .np(8)
1362:       .kr(1)
1363:       .m(4)
1364:       .n(8)
1365:       .k(8)
1366:       .aZeroPoint(0)
1367:       .bZeroPoint(0)
1368:       .test(pytorch_q8gemm_ukernel_4x8__neon);
1369: }
1370: 
```

- **EN:** This block implements local helper logic for `q8gemm`. Key symbols: `GemmMicrokernelTester`.
- **CN:** 该代码块实现与 `q8gemm` 相关的局部辅助逻辑。关键符号：`GemmMicrokernelTester`。

### Lines 1371-1426 / 第 1371-1426 行

```cpp
1371: TEST(Q8GEMM_4x8__NEON, k_gt_8) {
1372:   TEST_REQUIRES_ARM_NEON;
1373:   for (size_t k = 9; k < 16; k++) {
1374:     GemmMicrokernelTester().mr(4).nr(8).np(8).kr(1).m(4).n(8).k(k).test(
1375:         pytorch_q8gemm_ukernel_4x8__neon);
1376:   }
1377: }
1378: 
1379: TEST(Q8GEMM_4x8__NEON, k_gt_8_strided_a) {
1380:   TEST_REQUIRES_ARM_NEON;
1381:   for (size_t k = 9; k < 16; k++) {
1382:     GemmMicrokernelTester()
1383:         .mr(4)
1384:         .nr(8)
1385:         .np(8)
1386:         .kr(1)
1387:         .m(4)
1388:         .n(8)
1389:         .k(k)
1390:         .aStride(37)
1391:         .test(pytorch_q8gemm_ukernel_4x8__neon);
1392:   }
1393: }
1394: 
1395: TEST(Q8GEMM_4x8__NEON, k_gt_8_strided_c) {
1396:   TEST_REQUIRES_ARM_NEON;
1397:   for (size_t k = 9; k < 16; k++) {
1398:     GemmMicrokernelTester()
1399:         .mr(4)
1400:         .nr(8)
1401:         .np(8)
1402:         .kr(1)
1403:         .m(4)
1404:         .n(8)
1405:         .k(k)
1406:         .cStride(17)
1407:         .test(pytorch_q8gemm_ukernel_4x8__neon);
1408:   }
1409: }
1410: 
1411: TEST(Q8GEMM_4x8__NEON, k_gt_8_azp0) {
1412:   TEST_REQUIRES_ARM_NEON;
1413:   for (size_t k = 9; k < 16; k++) {
1414:     GemmMicrokernelTester()
1415:         .mr(4)
1416:         .nr(8)
1417:         .np(8)
1418:         .kr(1)
1419:         .m(4)
1420:         .n(8)
1421:         .k(k)
1422:         .aZeroPoint(0)
1423:         .test(pytorch_q8gemm_ukernel_4x8__neon);
1424:   }
1425: }
1426: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `GemmMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`GemmMicrokernelTester`。

### Lines 1427-1479 / 第 1427-1479 行

```cpp
1427: TEST(Q8GEMM_4x8__NEON, k_gt_8_bzp0) {
1428:   TEST_REQUIRES_ARM_NEON;
1429:   for (size_t k = 9; k < 16; k++) {
1430:     GemmMicrokernelTester()
1431:         .mr(4)
1432:         .nr(8)
1433:         .np(8)
1434:         .kr(1)
1435:         .m(4)
1436:         .n(8)
1437:         .k(k)
1438:         .bZeroPoint(0)
1439:         .test(pytorch_q8gemm_ukernel_4x8__neon);
1440:   }
1441: }
1442: 
1443: TEST(Q8GEMM_4x8__NEON, k_gt_8_nozp) {
1444:   TEST_REQUIRES_ARM_NEON;
1445:   for (size_t k = 9; k < 16; k++) {
1446:     GemmMicrokernelTester()
1447:         .mr(4)
1448:         .nr(8)
1449:         .np(8)
1450:         .kr(1)
1451:         .m(4)
1452:         .n(8)
1453:         .k(k)
1454:         .aZeroPoint(0)
1455:         .bZeroPoint(0)
1456:         .test(pytorch_q8gemm_ukernel_4x8__neon);
1457:   }
1458: }
1459: 
1460: TEST(Q8GEMM_4x8__NEON, k_gt_8_subtile) {
1461:   TEST_REQUIRES_ARM_NEON;
1462:   for (size_t k = 9; k < 16; k++) {
1463:     for (uint32_t m = 1; m <= 4; m++) {
1464:       for (uint32_t n = 1; n <= 8; n++) {
1465:         GemmMicrokernelTester()
1466:             .mr(4)
1467:             .nr(8)
1468:             .np(8)
1469:             .kr(1)
1470:             .m(m)
1471:             .n(n)
1472:             .k(k)
1473:             .iterations(3)
1474:             .test(pytorch_q8gemm_ukernel_4x8__neon);
1475:       }
1476:     }
1477:   }
1478: }
1479: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `GemmMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`GemmMicrokernelTester`。

### Lines 1480-1539 / 第 1480-1539 行

```cpp
1480: TEST(Q8GEMM_4x8__NEON, k_div_8) {
1481:   TEST_REQUIRES_ARM_NEON;
1482:   for (size_t k = 16; k < 128; k += 8) {
1483:     GemmMicrokernelTester().mr(4).nr(8).np(8).kr(1).m(4).n(8).k(k).test(
1484:         pytorch_q8gemm_ukernel_4x8__neon);
1485:   }
1486: }
1487: 
1488: TEST(Q8GEMM_4x8__NEON, k_div_8_strided_a) {
1489:   TEST_REQUIRES_ARM_NEON;
1490:   for (size_t k = 16; k < 128; k += 8) {
1491:     GemmMicrokernelTester()
1492:         .mr(4)
1493:         .nr(8)
1494:         .np(8)
1495:         .kr(1)
1496:         .m(4)
1497:         .n(8)
1498:         .k(k)
1499:         .aStride(171)
1500:         .test(pytorch_q8gemm_ukernel_4x8__neon);
1501:   }
1502: }
1503: 
1504: TEST(Q8GEMM_4x8__NEON, k_div_8_strided_c) {
1505:   TEST_REQUIRES_ARM_NEON;
1506:   for (size_t k = 16; k < 128; k += 8) {
1507:     GemmMicrokernelTester()
1508:         .mr(4)
1509:         .nr(8)
1510:         .np(8)
1511:         .kr(1)
1512:         .m(4)
1513:         .n(8)
1514:         .k(k)
1515:         .cStride(17)
1516:         .test(pytorch_q8gemm_ukernel_4x8__neon);
1517:   }
1518: }
1519: 
1520: TEST(Q8GEMM_4x8__NEON, k_div_8_subtile) {
1521:   TEST_REQUIRES_ARM_NEON;
1522:   for (size_t k = 16; k < 128; k += 24) {
1523:     for (uint32_t m = 1; m <= 4; m++) {
1524:       for (uint32_t n = 1; n <= 8; n++) {
1525:         GemmMicrokernelTester()
1526:             .mr(4)
1527:             .nr(8)
1528:             .np(8)
1529:             .kr(1)
1530:             .m(m)
1531:             .n(n)
1532:             .k(k)
1533:             .iterations(3)
1534:             .test(pytorch_q8gemm_ukernel_4x8__neon);
1535:       }
1536:     }
1537:   }
1538: }
1539: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `GemmMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`GemmMicrokernelTester`。

### Lines 1540-1589 / 第 1540-1589 行

```cpp
1540: //
1541: // Dynamic Quantization
1542: //
1543: 
1544: TEST(Q8GEMM_DQ_4x8__NEON, k_eq_8) {
1545:   TEST_REQUIRES_ARM_NEON;
1546:   GemmMicrokernelTester().mr(4).nr(8).np(8).kr(1).m(4).n(8).k(8).test(
1547:       pytorch_q8gemm_dq_ukernel_4x8__neon);
1548: }
1549: 
1550: TEST(Q8GEMM_DQ_4x8__NEON, k_eq_8_strided_a) {
1551:   TEST_REQUIRES_ARM_NEON;
1552:   GemmMicrokernelTester()
1553:       .mr(4)
1554:       .nr(8)
1555:       .np(8)
1556:       .kr(1)
1557:       .m(4)
1558:       .n(8)
1559:       .k(8)
1560:       .aStride(37)
1561:       .test(pytorch_q8gemm_dq_ukernel_4x8__neon);
1562: }
1563: 
1564: TEST(Q8GEMM_DQ_4x8__NEON, k_eq_8_strided_c) {
1565:   TEST_REQUIRES_ARM_NEON;
1566:   GemmMicrokernelTester()
1567:       .mr(4)
1568:       .nr(8)
1569:       .np(8)
1570:       .kr(1)
1571:       .m(4)
1572:       .n(8)
1573:       .k(8)
1574:       .cStride(17)
1575:       .test(pytorch_q8gemm_dq_ukernel_4x8__neon);
1576: }
1577: 
1578: TEST(Q8GEMM_DQ_4x8__NEON, k_eq_8_qmin128) {
1579:   TEST_REQUIRES_ARM_NEON;
1580:   GemmMicrokernelTester().mr(4).nr(8).np(8).kr(1).m(4).n(8).k(8).qmin(128).test(
1581:       pytorch_q8gemm_dq_ukernel_4x8__neon);
1582: }
1583: 
1584: TEST(Q8GEMM_DQ_4x8__NEON, k_eq_8_qmax128) {
1585:   TEST_REQUIRES_ARM_NEON;
1586:   GemmMicrokernelTester().mr(4).nr(8).np(8).kr(1).m(4).n(8).k(8).qmax(128).test(
1587:       pytorch_q8gemm_dq_ukernel_4x8__neon);
1588: }
1589: 
```

- **EN:** This block implements local helper logic for `q8gemm`. Key symbols: `GemmMicrokernelTester`.
- **CN:** 该代码块实现与 `q8gemm` 相关的局部辅助逻辑。关键符号：`GemmMicrokernelTester`。

### Lines 1590-1640 / 第 1590-1640 行

```cpp
1590: TEST(Q8GEMM_DQ_4x8__NEON, k_eq_8_azp0) {
1591:   TEST_REQUIRES_ARM_NEON;
1592:   GemmMicrokernelTester()
1593:       .mr(4)
1594:       .nr(8)
1595:       .np(8)
1596:       .kr(1)
1597:       .m(4)
1598:       .n(8)
1599:       .k(8)
1600:       .aZeroPoint(0)
1601:       .test(pytorch_q8gemm_dq_ukernel_4x8__neon);
1602: }
1603: 
1604: TEST(Q8GEMM_DQ_4x8__NEON, k_eq_8_bzp0) {
1605:   TEST_REQUIRES_ARM_NEON;
1606:   GemmMicrokernelTester()
1607:       .mr(4)
1608:       .nr(8)
1609:       .np(8)
1610:       .kr(1)
1611:       .m(4)
1612:       .n(8)
1613:       .k(8)
1614:       .bZeroPoint(0)
1615:       .test(pytorch_q8gemm_dq_ukernel_4x8__neon);
1616: }
1617: 
1618: TEST(Q8GEMM_DQ_4x8__NEON, k_eq_8_nozp) {
1619:   TEST_REQUIRES_ARM_NEON;
1620:   GemmMicrokernelTester()
1621:       .mr(4)
1622:       .nr(8)
1623:       .np(8)
1624:       .kr(1)
1625:       .m(4)
1626:       .n(8)
1627:       .k(8)
1628:       .aZeroPoint(0)
1629:       .bZeroPoint(0)
1630:       .test(pytorch_q8gemm_dq_ukernel_4x8__neon);
1631: }
1632: 
1633: TEST(Q8GEMM_DQ_4x8__NEON, k_gt_8) {
1634:   TEST_REQUIRES_ARM_NEON;
1635:   for (size_t k = 9; k < 16; k++) {
1636:     GemmMicrokernelTester().mr(4).nr(8).np(8).kr(1).m(4).n(8).k(k).test(
1637:         pytorch_q8gemm_dq_ukernel_4x8__neon);
1638:   }
1639: }
1640: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `GemmMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`GemmMicrokernelTester`。

### Lines 1641-1691 / 第 1641-1691 行

```cpp
1641: TEST(Q8GEMM_DQ_4x8__NEON, k_gt_8_strided_a) {
1642:   TEST_REQUIRES_ARM_NEON;
1643:   for (size_t k = 9; k < 16; k++) {
1644:     GemmMicrokernelTester()
1645:         .mr(4)
1646:         .nr(8)
1647:         .np(8)
1648:         .kr(1)
1649:         .m(4)
1650:         .n(8)
1651:         .k(k)
1652:         .aStride(37)
1653:         .test(pytorch_q8gemm_dq_ukernel_4x8__neon);
1654:   }
1655: }
1656: 
1657: TEST(Q8GEMM_DQ_4x8__NEON, k_gt_8_strided_c) {
1658:   TEST_REQUIRES_ARM_NEON;
1659:   for (size_t k = 9; k < 16; k++) {
1660:     GemmMicrokernelTester()
1661:         .mr(4)
1662:         .nr(8)
1663:         .np(8)
1664:         .kr(1)
1665:         .m(4)
1666:         .n(8)
1667:         .k(k)
1668:         .cStride(17)
1669:         .test(pytorch_q8gemm_dq_ukernel_4x8__neon);
1670:   }
1671: }
1672: 
1673: TEST(Q8GEMM_DQ_4x8__NEON, k_gt_8_azp0) {
1674:   TEST_REQUIRES_ARM_NEON;
1675:   for (size_t k = 9; k < 16; k++) {
1676:     GemmMicrokernelTester()
1677:         .mr(4)
1678:         .nr(8)
1679:         .np(8)
1680:         .kr(1)
1681:         .m(4)
1682:         .n(8)
1683:         .k(k)
1684:         .aZeroPoint(0)
1685:         .test(pytorch_q8gemm_dq_ukernel_4x8__neon);
1686:   }
1687: }
1688: 
1689: TEST(Q8GEMM_DQ_4x8__NEON, k_gt_8_bzp0) {
1690:   TEST_REQUIRES_ARM_NEON;
1691:   for (size_t k = 9; k < 16; k++) {
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `GemmMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`GemmMicrokernelTester`。

### Lines 1692-1741 / 第 1692-1741 行

```cpp
1692:     GemmMicrokernelTester()
1693:         .mr(4)
1694:         .nr(8)
1695:         .np(8)
1696:         .kr(1)
1697:         .m(4)
1698:         .n(8)
1699:         .k(k)
1700:         .bZeroPoint(0)
1701:         .test(pytorch_q8gemm_dq_ukernel_4x8__neon);
1702:   }
1703: }
1704: 
1705: TEST(Q8GEMM_DQ_4x8__NEON, k_gt_8_nozp) {
1706:   TEST_REQUIRES_ARM_NEON;
1707:   for (size_t k = 9; k < 16; k++) {
1708:     GemmMicrokernelTester()
1709:         .mr(4)
1710:         .nr(8)
1711:         .np(8)
1712:         .kr(1)
1713:         .m(4)
1714:         .n(8)
1715:         .k(k)
1716:         .aZeroPoint(0)
1717:         .bZeroPoint(0)
1718:         .test(pytorch_q8gemm_dq_ukernel_4x8__neon);
1719:   }
1720: }
1721: 
1722: TEST(Q8GEMM_DQ_4x8__NEON, k_gt_8_subtile) {
1723:   TEST_REQUIRES_ARM_NEON;
1724:   for (size_t k = 9; k < 16; k++) {
1725:     for (uint32_t m = 1; m <= 4; m++) {
1726:       for (uint32_t n = 1; n <= 8; n++) {
1727:         GemmMicrokernelTester()
1728:             .mr(4)
1729:             .nr(8)
1730:             .np(8)
1731:             .kr(1)
1732:             .m(m)
1733:             .n(n)
1734:             .k(k)
1735:             .iterations(3)
1736:             .test(pytorch_q8gemm_dq_ukernel_4x8__neon);
1737:       }
1738:     }
1739:   }
1740: }
1741: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `GemmMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`GemmMicrokernelTester`。

### Lines 1742-1801 / 第 1742-1801 行

```cpp
1742: TEST(Q8GEMM_DQ_4x8__NEON, k_div_8) {
1743:   TEST_REQUIRES_ARM_NEON;
1744:   for (size_t k = 16; k < 128; k += 8) {
1745:     GemmMicrokernelTester().mr(4).nr(8).np(8).kr(1).m(4).n(8).k(k).test(
1746:         pytorch_q8gemm_dq_ukernel_4x8__neon);
1747:   }
1748: }
1749: 
1750: TEST(Q8GEMM_DQ_4x8__NEON, k_div_8_strided_a) {
1751:   TEST_REQUIRES_ARM_NEON;
1752:   for (size_t k = 16; k < 128; k += 8) {
1753:     GemmMicrokernelTester()
1754:         .mr(4)
1755:         .nr(8)
1756:         .np(8)
1757:         .kr(1)
1758:         .m(4)
1759:         .n(8)
1760:         .k(k)
1761:         .aStride(171)
1762:         .test(pytorch_q8gemm_dq_ukernel_4x8__neon);
1763:   }
1764: }
1765: 
1766: TEST(Q8GEMM_DQ_4x8__NEON, k_div_8_strided_c) {
1767:   TEST_REQUIRES_ARM_NEON;
1768:   for (size_t k = 16; k < 128; k += 8) {
1769:     GemmMicrokernelTester()
1770:         .mr(4)
1771:         .nr(8)
1772:         .np(8)
1773:         .kr(1)
1774:         .m(4)
1775:         .n(8)
1776:         .k(k)
1777:         .cStride(17)
1778:         .test(pytorch_q8gemm_dq_ukernel_4x8__neon);
1779:   }
1780: }
1781: 
1782: TEST(Q8GEMM_DQ_4x8__NEON, k_div_8_subtile) {
1783:   TEST_REQUIRES_ARM_NEON;
1784:   for (size_t k = 16; k < 128; k += 24) {
1785:     for (uint32_t m = 1; m <= 4; m++) {
1786:       for (uint32_t n = 1; n <= 8; n++) {
1787:         GemmMicrokernelTester()
1788:             .mr(4)
1789:             .nr(8)
1790:             .np(8)
1791:             .kr(1)
1792:             .m(m)
1793:             .n(n)
1794:             .k(k)
1795:             .iterations(3)
1796:             .test(pytorch_q8gemm_dq_ukernel_4x8__neon);
1797:       }
1798:     }
1799:   }
1800: }
1801: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `GemmMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`GemmMicrokernelTester`。

### Lines 1802-1861 / 第 1802-1861 行

```cpp
1802: TEST(Q8GEMM_8x8__NEON, k_eq_8) {
1803:   TEST_REQUIRES_ARM_NEON;
1804:   GemmMicrokernelTester().mr(8).nr(8).np(8).kr(1).m(8).n(8).k(8).test(
1805:       pytorch_q8gemm_ukernel_8x8__neon);
1806: }
1807: 
1808: TEST(Q8GEMM_8x8__NEON, k_eq_8_strided_a) {
1809:   TEST_REQUIRES_ARM_NEON;
1810:   GemmMicrokernelTester()
1811:       .mr(8)
1812:       .nr(8)
1813:       .np(8)
1814:       .kr(1)
1815:       .m(8)
1816:       .n(8)
1817:       .k(8)
1818:       .aStride(37)
1819:       .test(pytorch_q8gemm_ukernel_8x8__neon);
1820: }
1821: 
1822: TEST(Q8GEMM_8x8__NEON, k_eq_8_strided_c) {
1823:   TEST_REQUIRES_ARM_NEON;
1824:   GemmMicrokernelTester()
1825:       .mr(8)
1826:       .nr(8)
1827:       .np(8)
1828:       .kr(1)
1829:       .m(8)
1830:       .n(8)
1831:       .k(8)
1832:       .cStride(17)
1833:       .test(pytorch_q8gemm_ukernel_8x8__neon);
1834: }
1835: 
1836: TEST(Q8GEMM_8x8__NEON, k_eq_8_qmin128) {
1837:   TEST_REQUIRES_ARM_NEON;
1838:   GemmMicrokernelTester().mr(8).nr(8).np(8).kr(1).m(8).n(8).k(8).qmin(128).test(
1839:       pytorch_q8gemm_ukernel_8x8__neon);
1840: }
1841: 
1842: TEST(Q8GEMM_8x8__NEON, k_eq_8_qmax128) {
1843:   TEST_REQUIRES_ARM_NEON;
1844:   GemmMicrokernelTester().mr(8).nr(8).np(8).kr(1).m(8).n(8).k(8).qmax(128).test(
1845:       pytorch_q8gemm_ukernel_8x8__neon);
1846: }
1847: 
1848: TEST(Q8GEMM_8x8__NEON, k_eq_8_azp0) {
1849:   TEST_REQUIRES_ARM_NEON;
1850:   GemmMicrokernelTester()
1851:       .mr(8)
1852:       .nr(8)
1853:       .np(8)
1854:       .kr(1)
1855:       .m(8)
1856:       .n(8)
1857:       .k(8)
1858:       .aZeroPoint(0)
1859:       .test(pytorch_q8gemm_ukernel_8x8__neon);
1860: }
1861: 
```

- **EN:** This block implements local helper logic for `q8gemm`. Key symbols: `GemmMicrokernelTester`.
- **CN:** 该代码块实现与 `q8gemm` 相关的局部辅助逻辑。关键符号：`GemmMicrokernelTester`。

### Lines 1862-1914 / 第 1862-1914 行

```cpp
1862: TEST(Q8GEMM_8x8__NEON, k_eq_8_bzp0) {
1863:   TEST_REQUIRES_ARM_NEON;
1864:   GemmMicrokernelTester()
1865:       .mr(8)
1866:       .nr(8)
1867:       .np(8)
1868:       .kr(1)
1869:       .m(8)
1870:       .n(8)
1871:       .k(8)
1872:       .bZeroPoint(0)
1873:       .test(pytorch_q8gemm_ukernel_8x8__neon);
1874: }
1875: 
1876: TEST(Q8GEMM_8x8__NEON, k_eq_8_nozp) {
1877:   TEST_REQUIRES_ARM_NEON;
1878:   GemmMicrokernelTester()
1879:       .mr(8)
1880:       .nr(8)
1881:       .np(8)
1882:       .kr(1)
1883:       .m(8)
1884:       .n(8)
1885:       .k(8)
1886:       .aZeroPoint(0)
1887:       .bZeroPoint(0)
1888:       .test(pytorch_q8gemm_ukernel_8x8__neon);
1889: }
1890: 
1891: TEST(Q8GEMM_8x8__NEON, k_gt_8) {
1892:   TEST_REQUIRES_ARM_NEON;
1893:   for (size_t k = 9; k < 16; k++) {
1894:     GemmMicrokernelTester().mr(8).nr(8).np(8).kr(1).m(8).n(8).k(k).test(
1895:         pytorch_q8gemm_ukernel_8x8__neon);
1896:   }
1897: }
1898: 
1899: TEST(Q8GEMM_8x8__NEON, k_gt_8_strided_a) {
1900:   TEST_REQUIRES_ARM_NEON;
1901:   for (size_t k = 9; k < 16; k++) {
1902:     GemmMicrokernelTester()
1903:         .mr(8)
1904:         .nr(8)
1905:         .np(8)
1906:         .kr(1)
1907:         .m(8)
1908:         .n(8)
1909:         .k(k)
1910:         .aStride(37)
1911:         .test(pytorch_q8gemm_ukernel_8x8__neon);
1912:   }
1913: }
1914: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `GemmMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`GemmMicrokernelTester`。

### Lines 1915-1965 / 第 1915-1965 行

```cpp
1915: TEST(Q8GEMM_8x8__NEON, k_gt_8_strided_c) {
1916:   TEST_REQUIRES_ARM_NEON;
1917:   for (size_t k = 9; k < 16; k++) {
1918:     GemmMicrokernelTester()
1919:         .mr(8)
1920:         .nr(8)
1921:         .np(8)
1922:         .kr(1)
1923:         .m(8)
1924:         .n(8)
1925:         .k(k)
1926:         .cStride(17)
1927:         .test(pytorch_q8gemm_ukernel_8x8__neon);
1928:   }
1929: }
1930: 
1931: TEST(Q8GEMM_8x8__NEON, k_gt_8_azp0) {
1932:   TEST_REQUIRES_ARM_NEON;
1933:   for (size_t k = 9; k < 16; k++) {
1934:     GemmMicrokernelTester()
1935:         .mr(8)
1936:         .nr(8)
1937:         .np(8)
1938:         .kr(1)
1939:         .m(8)
1940:         .n(8)
1941:         .k(k)
1942:         .aZeroPoint(0)
1943:         .test(pytorch_q8gemm_ukernel_8x8__neon);
1944:   }
1945: }
1946: 
1947: TEST(Q8GEMM_8x8__NEON, k_gt_8_bzp0) {
1948:   TEST_REQUIRES_ARM_NEON;
1949:   for (size_t k = 9; k < 16; k++) {
1950:     GemmMicrokernelTester()
1951:         .mr(8)
1952:         .nr(8)
1953:         .np(8)
1954:         .kr(1)
1955:         .m(8)
1956:         .n(8)
1957:         .k(k)
1958:         .bZeroPoint(0)
1959:         .test(pytorch_q8gemm_ukernel_8x8__neon);
1960:   }
1961: }
1962: 
1963: TEST(Q8GEMM_8x8__NEON, k_gt_8_nozp) {
1964:   TEST_REQUIRES_ARM_NEON;
1965:   for (size_t k = 9; k < 16; k++) {
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `GemmMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`GemmMicrokernelTester`。

### Lines 1966-2023 / 第 1966-2023 行

```cpp
1966:     GemmMicrokernelTester()
1967:         .mr(8)
1968:         .nr(8)
1969:         .np(8)
1970:         .kr(1)
1971:         .m(8)
1972:         .n(8)
1973:         .k(k)
1974:         .aZeroPoint(0)
1975:         .bZeroPoint(0)
1976:         .test(pytorch_q8gemm_ukernel_8x8__neon);
1977:   }
1978: }
1979: 
1980: TEST(Q8GEMM_8x8__NEON, k_gt_8_subtile) {
1981:   TEST_REQUIRES_ARM_NEON;
1982:   for (size_t k = 9; k < 16; k++) {
1983:     for (uint32_t m = 1; m <= 8; m++) {
1984:       for (uint32_t n = 1; n <= 8; n++) {
1985:         GemmMicrokernelTester()
1986:             .mr(8)
1987:             .nr(8)
1988:             .np(8)
1989:             .kr(1)
1990:             .m(m)
1991:             .n(n)
1992:             .k(k)
1993:             .iterations(3)
1994:             .test(pytorch_q8gemm_ukernel_8x8__neon);
1995:       }
1996:     }
1997:   }
1998: }
1999: 
2000: TEST(Q8GEMM_8x8__NEON, k_div_8) {
2001:   TEST_REQUIRES_ARM_NEON;
2002:   for (size_t k = 16; k < 128; k += 8) {
2003:     GemmMicrokernelTester().mr(8).nr(8).np(8).kr(1).m(8).n(8).k(k).test(
2004:         pytorch_q8gemm_ukernel_8x8__neon);
2005:   }
2006: }
2007: 
2008: TEST(Q8GEMM_8x8__NEON, k_div_8_strided_a) {
2009:   TEST_REQUIRES_ARM_NEON;
2010:   for (size_t k = 16; k < 128; k += 8) {
2011:     GemmMicrokernelTester()
2012:         .mr(8)
2013:         .nr(8)
2014:         .np(8)
2015:         .kr(1)
2016:         .m(8)
2017:         .n(8)
2018:         .k(k)
2019:         .aStride(171)
2020:         .test(pytorch_q8gemm_ukernel_8x8__neon);
2021:   }
2022: }
2023: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `GemmMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`GemmMicrokernelTester`。

### Lines 2024-2079 / 第 2024-2079 行

```cpp
2024: TEST(Q8GEMM_8x8__NEON, k_div_8_strided_c) {
2025:   TEST_REQUIRES_ARM_NEON;
2026:   for (size_t k = 16; k < 128; k += 8) {
2027:     GemmMicrokernelTester()
2028:         .mr(8)
2029:         .nr(8)
2030:         .np(8)
2031:         .kr(1)
2032:         .m(8)
2033:         .n(8)
2034:         .k(k)
2035:         .cStride(17)
2036:         .test(pytorch_q8gemm_ukernel_8x8__neon);
2037:   }
2038: }
2039: 
2040: TEST(Q8GEMM_8x8__NEON, k_div_8_subtile) {
2041:   TEST_REQUIRES_ARM_NEON;
2042:   for (size_t k = 16; k < 128; k += 24) {
2043:     for (uint32_t m = 1; m <= 8; m++) {
2044:       for (uint32_t n = 1; n <= 8; n++) {
2045:         GemmMicrokernelTester()
2046:             .mr(8)
2047:             .nr(8)
2048:             .np(8)
2049:             .kr(1)
2050:             .m(m)
2051:             .n(n)
2052:             .k(k)
2053:             .iterations(3)
2054:             .test(pytorch_q8gemm_ukernel_8x8__neon);
2055:       }
2056:     }
2057:   }
2058: }
2059: 
2060: TEST(Q8GEMM_6x4__NEON, k_eq_8) {
2061:   TEST_REQUIRES_ARM_NEON;
2062:   GemmMicrokernelTester().mr(6).nr(4).np(4).kr(1).m(6).n(4).k(8).test(
2063:       pytorch_q8gemm_ukernel_6x4__neon);
2064: }
2065: 
2066: TEST(Q8GEMM_6x4__NEON, k_eq_8_strided_a) {
2067:   TEST_REQUIRES_ARM_NEON;
2068:   GemmMicrokernelTester()
2069:       .mr(6)
2070:       .nr(4)
2071:       .np(4)
2072:       .kr(1)
2073:       .m(6)
2074:       .n(4)
2075:       .k(8)
2076:       .aStride(37)
2077:       .test(pytorch_q8gemm_ukernel_6x4__neon);
2078: }
2079: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `GemmMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`GemmMicrokernelTester`。

### Lines 2080-2133 / 第 2080-2133 行

```cpp
2080: TEST(Q8GEMM_6x4__NEON, k_eq_8_strided_c) {
2081:   TEST_REQUIRES_ARM_NEON;
2082:   GemmMicrokernelTester()
2083:       .mr(6)
2084:       .nr(4)
2085:       .np(4)
2086:       .kr(1)
2087:       .m(6)
2088:       .n(4)
2089:       .k(8)
2090:       .cStride(17)
2091:       .test(pytorch_q8gemm_ukernel_6x4__neon);
2092: }
2093: 
2094: TEST(Q8GEMM_6x4__NEON, k_eq_8_qmin128) {
2095:   TEST_REQUIRES_ARM_NEON;
2096:   GemmMicrokernelTester().mr(6).nr(4).np(4).kr(1).m(6).n(4).k(8).qmin(128).test(
2097:       pytorch_q8gemm_ukernel_6x4__neon);
2098: }
2099: 
2100: TEST(Q8GEMM_6x4__NEON, k_eq_8_qmax128) {
2101:   TEST_REQUIRES_ARM_NEON;
2102:   GemmMicrokernelTester().mr(6).nr(4).np(4).kr(1).m(6).n(4).k(8).qmax(128).test(
2103:       pytorch_q8gemm_ukernel_6x4__neon);
2104: }
2105: 
2106: TEST(Q8GEMM_6x4__NEON, k_eq_8_azp0) {
2107:   TEST_REQUIRES_ARM_NEON;
2108:   GemmMicrokernelTester()
2109:       .mr(6)
2110:       .nr(4)
2111:       .np(4)
2112:       .kr(1)
2113:       .m(6)
2114:       .n(4)
2115:       .k(8)
2116:       .aZeroPoint(0)
2117:       .test(pytorch_q8gemm_ukernel_6x4__neon);
2118: }
2119: 
2120: TEST(Q8GEMM_6x4__NEON, k_eq_8_bzp0) {
2121:   TEST_REQUIRES_ARM_NEON;
2122:   GemmMicrokernelTester()
2123:       .mr(6)
2124:       .nr(4)
2125:       .np(4)
2126:       .kr(1)
2127:       .m(6)
2128:       .n(4)
2129:       .k(8)
2130:       .bZeroPoint(0)
2131:       .test(pytorch_q8gemm_ukernel_6x4__neon);
2132: }
2133: 
```

- **EN:** This block implements local helper logic for `q8gemm`. Key symbols: `GemmMicrokernelTester`.
- **CN:** 该代码块实现与 `q8gemm` 相关的局部辅助逻辑。关键符号：`GemmMicrokernelTester`。

### Lines 2134-2188 / 第 2134-2188 行

```cpp
2134: TEST(Q8GEMM_6x4__NEON, k_eq_8_nozp) {
2135:   TEST_REQUIRES_ARM_NEON;
2136:   GemmMicrokernelTester()
2137:       .mr(6)
2138:       .nr(4)
2139:       .np(4)
2140:       .kr(1)
2141:       .m(6)
2142:       .n(4)
2143:       .k(8)
2144:       .aZeroPoint(0)
2145:       .bZeroPoint(0)
2146:       .test(pytorch_q8gemm_ukernel_6x4__neon);
2147: }
2148: 
2149: TEST(Q8GEMM_6x4__NEON, k_gt_8) {
2150:   TEST_REQUIRES_ARM_NEON;
2151:   for (size_t k = 9; k < 16; k++) {
2152:     GemmMicrokernelTester().mr(6).nr(4).np(4).kr(1).m(6).n(4).k(k).test(
2153:         pytorch_q8gemm_ukernel_6x4__neon);
2154:   }
2155: }
2156: 
2157: TEST(Q8GEMM_6x4__NEON, k_gt_8_strided_a) {
2158:   TEST_REQUIRES_ARM_NEON;
2159:   for (size_t k = 9; k < 16; k++) {
2160:     GemmMicrokernelTester()
2161:         .mr(6)
2162:         .nr(4)
2163:         .np(4)
2164:         .kr(1)
2165:         .m(6)
2166:         .n(4)
2167:         .k(k)
2168:         .aStride(37)
2169:         .test(pytorch_q8gemm_ukernel_6x4__neon);
2170:   }
2171: }
2172: 
2173: TEST(Q8GEMM_6x4__NEON, k_gt_8_strided_c) {
2174:   TEST_REQUIRES_ARM_NEON;
2175:   for (size_t k = 9; k < 16; k++) {
2176:     GemmMicrokernelTester()
2177:         .mr(6)
2178:         .nr(4)
2179:         .np(4)
2180:         .kr(1)
2181:         .m(6)
2182:         .n(4)
2183:         .k(k)
2184:         .cStride(17)
2185:         .test(pytorch_q8gemm_ukernel_6x4__neon);
2186:   }
2187: }
2188: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `GemmMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`GemmMicrokernelTester`。

### Lines 2189-2240 / 第 2189-2240 行

```cpp
2189: TEST(Q8GEMM_6x4__NEON, k_gt_8_azp0) {
2190:   TEST_REQUIRES_ARM_NEON;
2191:   for (size_t k = 9; k < 16; k++) {
2192:     GemmMicrokernelTester()
2193:         .mr(6)
2194:         .nr(4)
2195:         .np(4)
2196:         .kr(1)
2197:         .m(6)
2198:         .n(4)
2199:         .k(k)
2200:         .aZeroPoint(0)
2201:         .test(pytorch_q8gemm_ukernel_6x4__neon);
2202:   }
2203: }
2204: 
2205: TEST(Q8GEMM_6x4__NEON, k_gt_8_bzp0) {
2206:   TEST_REQUIRES_ARM_NEON;
2207:   for (size_t k = 9; k < 16; k++) {
2208:     GemmMicrokernelTester()
2209:         .mr(6)
2210:         .nr(4)
2211:         .np(4)
2212:         .kr(1)
2213:         .m(6)
2214:         .n(4)
2215:         .k(k)
2216:         .bZeroPoint(0)
2217:         .test(pytorch_q8gemm_ukernel_6x4__neon);
2218:   }
2219: }
2220: 
2221: TEST(Q8GEMM_6x4__NEON, k_gt_8_nozp) {
2222:   TEST_REQUIRES_ARM_NEON;
2223:   for (size_t k = 9; k < 16; k++) {
2224:     GemmMicrokernelTester()
2225:         .mr(6)
2226:         .nr(4)
2227:         .np(4)
2228:         .kr(1)
2229:         .m(6)
2230:         .n(4)
2231:         .k(k)
2232:         .aZeroPoint(0)
2233:         .bZeroPoint(0)
2234:         .test(pytorch_q8gemm_ukernel_6x4__neon);
2235:   }
2236: }
2237: 
2238: TEST(Q8GEMM_6x4__NEON, k_gt_8_subtile) {
2239:   TEST_REQUIRES_ARM_NEON;
2240:   for (size_t k = 9; k < 16; k++) {
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `GemmMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`GemmMicrokernelTester`。

### Lines 2241-2297 / 第 2241-2297 行

```cpp
2241:     for (uint32_t m = 1; m <= 6; m++) {
2242:       for (uint32_t n = 1; n <= 4; n++) {
2243:         GemmMicrokernelTester()
2244:             .mr(6)
2245:             .nr(4)
2246:             .np(4)
2247:             .kr(1)
2248:             .m(m)
2249:             .n(n)
2250:             .k(k)
2251:             .iterations(3)
2252:             .test(pytorch_q8gemm_ukernel_6x4__neon);
2253:       }
2254:     }
2255:   }
2256: }
2257: 
2258: TEST(Q8GEMM_6x4__NEON, k_div_8) {
2259:   TEST_REQUIRES_ARM_NEON;
2260:   for (size_t k = 16; k < 128; k += 8) {
2261:     GemmMicrokernelTester().mr(6).nr(4).np(4).kr(1).m(6).n(4).k(k).test(
2262:         pytorch_q8gemm_ukernel_6x4__neon);
2263:   }
2264: }
2265: 
2266: TEST(Q8GEMM_6x4__NEON, k_div_8_strided_a) {
2267:   TEST_REQUIRES_ARM_NEON;
2268:   for (size_t k = 16; k < 128; k += 8) {
2269:     GemmMicrokernelTester()
2270:         .mr(6)
2271:         .nr(4)
2272:         .np(4)
2273:         .kr(1)
2274:         .m(6)
2275:         .n(4)
2276:         .k(k)
2277:         .aStride(171)
2278:         .test(pytorch_q8gemm_ukernel_6x4__neon);
2279:   }
2280: }
2281: 
2282: TEST(Q8GEMM_6x4__NEON, k_div_8_strided_c) {
2283:   TEST_REQUIRES_ARM_NEON;
2284:   for (size_t k = 16; k < 128; k += 8) {
2285:     GemmMicrokernelTester()
2286:         .mr(6)
2287:         .nr(4)
2288:         .np(4)
2289:         .kr(1)
2290:         .m(6)
2291:         .n(4)
2292:         .k(k)
2293:         .cStride(17)
2294:         .test(pytorch_q8gemm_ukernel_6x4__neon);
2295:   }
2296: }
2297: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `GemmMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`GemmMicrokernelTester`。

### Lines 2298-2349 / 第 2298-2349 行

```cpp
2298: TEST(Q8GEMM_6x4__NEON, k_div_8_subtile) {
2299:   TEST_REQUIRES_ARM_NEON;
2300:   for (size_t k = 16; k < 128; k += 24) {
2301:     for (uint32_t m = 1; m <= 6; m++) {
2302:       for (uint32_t n = 1; n <= 4; n++) {
2303:         GemmMicrokernelTester().mr(6).nr(4).np(4).kr(1).m(m).n(n).k(k).test(
2304:             pytorch_q8gemm_ukernel_6x4__neon);
2305:       }
2306:     }
2307:   }
2308: }
2309: 
2310: TEST(Q8GEMM_4x8c2_XZP__NEON, k_eq_8) {
2311:   TEST_REQUIRES_ARM_NEON;
2312:   GemmMicrokernelTester().mr(4).nr(8).np(8).kr(2).m(4).n(8).k(8).test(
2313:       pytorch_q8gemm_xzp_ukernel_4x8c2__neon);
2314: }
2315: 
2316: TEST(Q8GEMM_4x8c2_XZP__NEON, k_eq_8_strided_a) {
2317:   TEST_REQUIRES_ARM_NEON;
2318:   GemmMicrokernelTester()
2319:       .mr(4)
2320:       .nr(8)
2321:       .np(8)
2322:       .kr(2)
2323:       .m(4)
2324:       .n(8)
2325:       .k(8)
2326:       .aStride(37)
2327:       .test(pytorch_q8gemm_xzp_ukernel_4x8c2__neon);
2328: }
2329: 
2330: TEST(Q8GEMM_4x8c2_XZP__NEON, k_eq_8_strided_c) {
2331:   TEST_REQUIRES_ARM_NEON;
2332:   GemmMicrokernelTester()
2333:       .mr(4)
2334:       .nr(8)
2335:       .np(8)
2336:       .kr(2)
2337:       .m(4)
2338:       .n(8)
2339:       .k(8)
2340:       .cStride(17)
2341:       .test(pytorch_q8gemm_xzp_ukernel_4x8c2__neon);
2342: }
2343: 
2344: TEST(Q8GEMM_4x8c2_XZP__NEON, k_eq_8_qmin128) {
2345:   TEST_REQUIRES_ARM_NEON;
2346:   GemmMicrokernelTester().mr(4).nr(8).np(8).kr(2).m(4).n(8).k(8).qmin(128).test(
2347:       pytorch_q8gemm_xzp_ukernel_4x8c2__neon);
2348: }
2349: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `GemmMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`GemmMicrokernelTester`。

### Lines 2350-2401 / 第 2350-2401 行

```cpp
2350: TEST(Q8GEMM_4x8c2_XZP__NEON, k_eq_8_qmax128) {
2351:   TEST_REQUIRES_ARM_NEON;
2352:   GemmMicrokernelTester().mr(4).nr(8).np(8).kr(2).m(4).n(8).k(8).qmax(128).test(
2353:       pytorch_q8gemm_xzp_ukernel_4x8c2__neon);
2354: }
2355: 
2356: TEST(Q8GEMM_4x8c2_XZP__NEON, k_eq_8_azp0) {
2357:   TEST_REQUIRES_ARM_NEON;
2358:   GemmMicrokernelTester()
2359:       .mr(4)
2360:       .nr(8)
2361:       .np(8)
2362:       .kr(2)
2363:       .m(4)
2364:       .n(8)
2365:       .k(8)
2366:       .aZeroPoint(0)
2367:       .test(pytorch_q8gemm_xzp_ukernel_4x8c2__neon);
2368: }
2369: 
2370: TEST(Q8GEMM_4x8c2_XZP__NEON, k_eq_8_bzp0) {
2371:   TEST_REQUIRES_ARM_NEON;
2372:   GemmMicrokernelTester()
2373:       .mr(4)
2374:       .nr(8)
2375:       .np(8)
2376:       .kr(2)
2377:       .m(4)
2378:       .n(8)
2379:       .k(8)
2380:       .bZeroPoint(0)
2381:       .test(pytorch_q8gemm_xzp_ukernel_4x8c2__neon);
2382: }
2383: 
2384: TEST(Q8GEMM_4x8c2_XZP__NEON, k_eq_8_nozp) {
2385:   TEST_REQUIRES_ARM_NEON;
2386:   GemmMicrokernelTester()
2387:       .mr(4)
2388:       .nr(8)
2389:       .np(8)
2390:       .kr(2)
2391:       .m(4)
2392:       .n(8)
2393:       .k(8)
2394:       .aZeroPoint(0)
2395:       .bZeroPoint(0)
2396:       .test(pytorch_q8gemm_xzp_ukernel_4x8c2__neon);
2397: }
2398: 
2399: TEST(Q8GEMM_4x8c2_XZP__NEON, k_gt_8) {
2400:   TEST_REQUIRES_ARM_NEON;
2401:   for (size_t k = 9; k < 16; k++) {
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `GemmMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`GemmMicrokernelTester`。

### Lines 2402-2454 / 第 2402-2454 行

```cpp
2402:     GemmMicrokernelTester().mr(4).nr(8).np(8).kr(2).m(4).n(8).k(k).test(
2403:         pytorch_q8gemm_xzp_ukernel_4x8c2__neon);
2404:   }
2405: }
2406: 
2407: TEST(Q8GEMM_4x8c2_XZP__NEON, k_gt_8_strided_a) {
2408:   TEST_REQUIRES_ARM_NEON;
2409:   for (size_t k = 9; k < 16; k++) {
2410:     GemmMicrokernelTester()
2411:         .mr(4)
2412:         .nr(8)
2413:         .np(8)
2414:         .kr(2)
2415:         .m(4)
2416:         .n(8)
2417:         .k(k)
2418:         .aStride(37)
2419:         .test(pytorch_q8gemm_xzp_ukernel_4x8c2__neon);
2420:   }
2421: }
2422: 
2423: TEST(Q8GEMM_4x8c2_XZP__NEON, k_gt_8_strided_c) {
2424:   TEST_REQUIRES_ARM_NEON;
2425:   for (size_t k = 9; k < 16; k++) {
2426:     GemmMicrokernelTester()
2427:         .mr(4)
2428:         .nr(8)
2429:         .np(8)
2430:         .kr(2)
2431:         .m(4)
2432:         .n(8)
2433:         .k(k)
2434:         .cStride(17)
2435:         .test(pytorch_q8gemm_xzp_ukernel_4x8c2__neon);
2436:   }
2437: }
2438: 
2439: TEST(Q8GEMM_4x8c2_XZP__NEON, k_gt_8_azp0) {
2440:   TEST_REQUIRES_ARM_NEON;
2441:   for (size_t k = 9; k < 16; k++) {
2442:     GemmMicrokernelTester()
2443:         .mr(4)
2444:         .nr(8)
2445:         .np(8)
2446:         .kr(2)
2447:         .m(4)
2448:         .n(8)
2449:         .k(k)
2450:         .aZeroPoint(0)
2451:         .test(pytorch_q8gemm_xzp_ukernel_4x8c2__neon);
2452:   }
2453: }
2454: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `GemmMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`GemmMicrokernelTester`。

### Lines 2455-2507 / 第 2455-2507 行

```cpp
2455: TEST(Q8GEMM_4x8c2_XZP__NEON, k_gt_8_bzp0) {
2456:   TEST_REQUIRES_ARM_NEON;
2457:   for (size_t k = 9; k < 16; k++) {
2458:     GemmMicrokernelTester()
2459:         .mr(4)
2460:         .nr(8)
2461:         .np(8)
2462:         .kr(2)
2463:         .m(4)
2464:         .n(8)
2465:         .k(k)
2466:         .bZeroPoint(0)
2467:         .test(pytorch_q8gemm_xzp_ukernel_4x8c2__neon);
2468:   }
2469: }
2470: 
2471: TEST(Q8GEMM_4x8c2_XZP__NEON, k_gt_8_nozp) {
2472:   TEST_REQUIRES_ARM_NEON;
2473:   for (size_t k = 9; k < 16; k++) {
2474:     GemmMicrokernelTester()
2475:         .mr(4)
2476:         .nr(8)
2477:         .np(8)
2478:         .kr(2)
2479:         .m(4)
2480:         .n(8)
2481:         .k(k)
2482:         .aZeroPoint(0)
2483:         .bZeroPoint(0)
2484:         .test(pytorch_q8gemm_xzp_ukernel_4x8c2__neon);
2485:   }
2486: }
2487: 
2488: TEST(Q8GEMM_4x8c2_XZP__NEON, k_gt_8_subtile) {
2489:   TEST_REQUIRES_ARM_NEON;
2490:   for (size_t k = 9; k < 16; k++) {
2491:     for (uint32_t m = 1; m <= 4; m++) {
2492:       for (uint32_t n = 1; n <= 8; n++) {
2493:         GemmMicrokernelTester()
2494:             .mr(4)
2495:             .nr(8)
2496:             .np(8)
2497:             .kr(2)
2498:             .m(m)
2499:             .n(n)
2500:             .k(k)
2501:             .iterations(3)
2502:             .test(pytorch_q8gemm_xzp_ukernel_4x8c2__neon);
2503:       }
2504:     }
2505:   }
2506: }
2507: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `GemmMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`GemmMicrokernelTester`。

### Lines 2508-2567 / 第 2508-2567 行

```cpp
2508: TEST(Q8GEMM_4x8c2_XZP__NEON, k_div_8) {
2509:   TEST_REQUIRES_ARM_NEON;
2510:   for (size_t k = 16; k < 128; k += 8) {
2511:     GemmMicrokernelTester().mr(4).nr(8).np(8).kr(2).m(4).n(8).k(k).test(
2512:         pytorch_q8gemm_xzp_ukernel_4x8c2__neon);
2513:   }
2514: }
2515: 
2516: TEST(Q8GEMM_4x8c2_XZP__NEON, k_div_8_strided_a) {
2517:   TEST_REQUIRES_ARM_NEON;
2518:   for (size_t k = 16; k < 128; k += 8) {
2519:     GemmMicrokernelTester()
2520:         .mr(4)
2521:         .nr(8)
2522:         .np(8)
2523:         .kr(2)
2524:         .m(4)
2525:         .n(8)
2526:         .k(k)
2527:         .aStride(171)
2528:         .test(pytorch_q8gemm_xzp_ukernel_4x8c2__neon);
2529:   }
2530: }
2531: 
2532: TEST(Q8GEMM_4x8c2_XZP__NEON, k_div_8_strided_c) {
2533:   TEST_REQUIRES_ARM_NEON;
2534:   for (size_t k = 16; k < 128; k += 8) {
2535:     GemmMicrokernelTester()
2536:         .mr(4)
2537:         .nr(8)
2538:         .np(8)
2539:         .kr(2)
2540:         .m(4)
2541:         .n(8)
2542:         .k(k)
2543:         .cStride(17)
2544:         .test(pytorch_q8gemm_xzp_ukernel_4x8c2__neon);
2545:   }
2546: }
2547: 
2548: TEST(Q8GEMM_4x8c2_XZP__NEON, k_div_8_subtile) {
2549:   TEST_REQUIRES_ARM_NEON;
2550:   for (size_t k = 16; k < 128; k += 24) {
2551:     for (uint32_t m = 1; m <= 4; m++) {
2552:       for (uint32_t n = 1; n <= 8; n++) {
2553:         GemmMicrokernelTester()
2554:             .mr(4)
2555:             .nr(8)
2556:             .np(8)
2557:             .kr(2)
2558:             .m(m)
2559:             .n(n)
2560:             .k(k)
2561:             .iterations(3)
2562:             .test(pytorch_q8gemm_xzp_ukernel_4x8c2__neon);
2563:       }
2564:     }
2565:   }
2566: }
2567: #endif
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends; iterates over tensors, descriptors, options, or runtime state. Key symbols: `GemmMicrokernelTester`.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端；遍历张量、描述符、配置项或运行时状态。关键符号：`GemmMicrokernelTester`。

### Lines 2568-2629 / 第 2568-2629 行

```cpp
2568: 
2569: #if CPUINFO_ARCH_X86 || CPUINFO_ARCH_X86_64
2570: TEST(Q8GEMM_2x4c8__SSE2, k_eq_8) {
2571:   TEST_REQUIRES_X86_SSE2;
2572:   GemmMicrokernelTester().mr(2).nr(4).np(1).kr(8).m(2).n(4).k(8).test(
2573:       pytorch_q8gemm_ukernel_2x4c8__sse2);
2574: }
2575: 
2576: TEST(Q8GEMM_2x4c8__SSE2, k_eq_8_strided_a) {
2577:   TEST_REQUIRES_X86_SSE2;
2578:   GemmMicrokernelTester()
2579:       .mr(2)
2580:       .nr(4)
2581:       .np(1)
2582:       .kr(8)
2583:       .m(2)
2584:       .n(4)
2585:       .k(8)
2586:       .aStride(37)
2587:       .test(pytorch_q8gemm_ukernel_2x4c8__sse2);
2588: }
2589: 
2590: TEST(Q8GEMM_2x4c8__SSE2, k_eq_8_strided_c) {
2591:   TEST_REQUIRES_X86_SSE2;
2592:   GemmMicrokernelTester()
2593:       .mr(2)
2594:       .nr(4)
2595:       .np(1)
2596:       .kr(8)
2597:       .m(2)
2598:       .n(4)
2599:       .k(8)
2600:       .cStride(17)
2601:       .test(pytorch_q8gemm_ukernel_2x4c8__sse2);
2602: }
2603: 
2604: TEST(Q8GEMM_2x4c8__SSE2, k_eq_8_qmin128) {
2605:   TEST_REQUIRES_X86_SSE2;
2606:   GemmMicrokernelTester().mr(2).nr(4).np(1).kr(8).m(2).n(4).k(8).qmin(128).test(
2607:       pytorch_q8gemm_ukernel_2x4c8__sse2);
2608: }
2609: 
2610: TEST(Q8GEMM_2x4c8__SSE2, k_eq_8_qmax128) {
2611:   TEST_REQUIRES_X86_SSE2;
2612:   GemmMicrokernelTester().mr(2).nr(4).np(1).kr(8).m(2).n(4).k(8).qmax(128).test(
2613:       pytorch_q8gemm_ukernel_2x4c8__sse2);
2614: }
2615: 
2616: TEST(Q8GEMM_2x4c8__SSE2, k_eq_8_azp0) {
2617:   TEST_REQUIRES_X86_SSE2;
2618:   GemmMicrokernelTester()
2619:       .mr(2)
2620:       .nr(4)
2621:       .np(1)
2622:       .kr(8)
2623:       .m(2)
2624:       .n(4)
2625:       .k(8)
2626:       .aZeroPoint(0)
2627:       .test(pytorch_q8gemm_ukernel_2x4c8__sse2);
2628: }
2629: 
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends. Key symbols: `GemmMicrokernelTester`.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端。关键符号：`GemmMicrokernelTester`。

### Lines 2630-2682 / 第 2630-2682 行

```cpp
2630: TEST(Q8GEMM_2x4c8__SSE2, k_eq_8_bzp0) {
2631:   TEST_REQUIRES_X86_SSE2;
2632:   GemmMicrokernelTester()
2633:       .mr(2)
2634:       .nr(4)
2635:       .np(1)
2636:       .kr(8)
2637:       .m(2)
2638:       .n(4)
2639:       .k(8)
2640:       .bZeroPoint(0)
2641:       .test(pytorch_q8gemm_ukernel_2x4c8__sse2);
2642: }
2643: 
2644: TEST(Q8GEMM_2x4c8__SSE2, k_eq_8_nozp) {
2645:   TEST_REQUIRES_X86_SSE2;
2646:   GemmMicrokernelTester()
2647:       .mr(2)
2648:       .nr(4)
2649:       .np(1)
2650:       .kr(8)
2651:       .m(2)
2652:       .n(4)
2653:       .k(8)
2654:       .aZeroPoint(0)
2655:       .bZeroPoint(0)
2656:       .test(pytorch_q8gemm_ukernel_2x4c8__sse2);
2657: }
2658: 
2659: TEST(Q8GEMM_2x4c8__SSE2, k_gt_8) {
2660:   TEST_REQUIRES_X86_SSE2;
2661:   for (size_t k = 9; k < 16; k++) {
2662:     GemmMicrokernelTester().mr(2).nr(4).np(1).kr(8).m(2).n(4).k(k).test(
2663:         pytorch_q8gemm_ukernel_2x4c8__sse2);
2664:   }
2665: }
2666: 
2667: TEST(Q8GEMM_2x4c8__SSE2, k_gt_8_strided_a) {
2668:   TEST_REQUIRES_X86_SSE2;
2669:   for (size_t k = 9; k < 16; k++) {
2670:     GemmMicrokernelTester()
2671:         .mr(2)
2672:         .nr(4)
2673:         .np(1)
2674:         .kr(8)
2675:         .m(2)
2676:         .n(4)
2677:         .k(k)
2678:         .aStride(37)
2679:         .test(pytorch_q8gemm_ukernel_2x4c8__sse2);
2680:   }
2681: }
2682: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `GemmMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`GemmMicrokernelTester`。

### Lines 2683-2733 / 第 2683-2733 行

```cpp
2683: TEST(Q8GEMM_2x4c8__SSE2, k_gt_8_strided_c) {
2684:   TEST_REQUIRES_X86_SSE2;
2685:   for (size_t k = 9; k < 16; k++) {
2686:     GemmMicrokernelTester()
2687:         .mr(2)
2688:         .nr(4)
2689:         .np(1)
2690:         .kr(8)
2691:         .m(2)
2692:         .n(4)
2693:         .k(k)
2694:         .cStride(17)
2695:         .test(pytorch_q8gemm_ukernel_2x4c8__sse2);
2696:   }
2697: }
2698: 
2699: TEST(Q8GEMM_2x4c8__SSE2, k_gt_8_azp0) {
2700:   TEST_REQUIRES_X86_SSE2;
2701:   for (size_t k = 9; k < 16; k++) {
2702:     GemmMicrokernelTester()
2703:         .mr(2)
2704:         .nr(4)
2705:         .np(1)
2706:         .kr(8)
2707:         .m(2)
2708:         .n(4)
2709:         .k(k)
2710:         .aZeroPoint(0)
2711:         .test(pytorch_q8gemm_ukernel_2x4c8__sse2);
2712:   }
2713: }
2714: 
2715: TEST(Q8GEMM_2x4c8__SSE2, k_gt_8_bzp0) {
2716:   TEST_REQUIRES_X86_SSE2;
2717:   for (size_t k = 9; k < 16; k++) {
2718:     GemmMicrokernelTester()
2719:         .mr(2)
2720:         .nr(4)
2721:         .np(1)
2722:         .kr(8)
2723:         .m(2)
2724:         .n(4)
2725:         .k(k)
2726:         .bZeroPoint(0)
2727:         .test(pytorch_q8gemm_ukernel_2x4c8__sse2);
2728:   }
2729: }
2730: 
2731: TEST(Q8GEMM_2x4c8__SSE2, k_gt_8_nozp) {
2732:   TEST_REQUIRES_X86_SSE2;
2733:   for (size_t k = 9; k < 16; k++) {
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `GemmMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`GemmMicrokernelTester`。

### Lines 2734-2791 / 第 2734-2791 行

```cpp
2734:     GemmMicrokernelTester()
2735:         .mr(2)
2736:         .nr(4)
2737:         .np(1)
2738:         .kr(8)
2739:         .m(2)
2740:         .n(4)
2741:         .k(k)
2742:         .aZeroPoint(0)
2743:         .bZeroPoint(0)
2744:         .test(pytorch_q8gemm_ukernel_2x4c8__sse2);
2745:   }
2746: }
2747: 
2748: TEST(Q8GEMM_2x4c8__SSE2, k_gt_8_subtile) {
2749:   TEST_REQUIRES_X86_SSE2;
2750:   for (size_t k = 9; k < 16; k++) {
2751:     for (uint32_t m = 1; m <= 2; m++) {
2752:       for (uint32_t n = 1; n <= 4; n++) {
2753:         GemmMicrokernelTester()
2754:             .mr(2)
2755:             .nr(4)
2756:             .np(1)
2757:             .kr(8)
2758:             .m(m)
2759:             .n(n)
2760:             .k(k)
2761:             .iterations(3)
2762:             .test(pytorch_q8gemm_ukernel_2x4c8__sse2);
2763:       }
2764:     }
2765:   }
2766: }
2767: 
2768: TEST(Q8GEMM_2x4c8__SSE2, k_div_8) {
2769:   TEST_REQUIRES_X86_SSE2;
2770:   for (size_t k = 16; k < 128; k += 8) {
2771:     GemmMicrokernelTester().mr(2).nr(4).np(1).kr(8).m(2).n(4).k(k).test(
2772:         pytorch_q8gemm_ukernel_2x4c8__sse2);
2773:   }
2774: }
2775: 
2776: TEST(Q8GEMM_2x4c8__SSE2, k_div_8_strided_a) {
2777:   TEST_REQUIRES_X86_SSE2;
2778:   for (size_t k = 16; k < 128; k += 8) {
2779:     GemmMicrokernelTester()
2780:         .mr(2)
2781:         .nr(4)
2782:         .np(1)
2783:         .kr(8)
2784:         .m(2)
2785:         .n(4)
2786:         .k(k)
2787:         .aStride(171)
2788:         .test(pytorch_q8gemm_ukernel_2x4c8__sse2);
2789:   }
2790: }
2791: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `GemmMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`GemmMicrokernelTester`。

### Lines 2792-2844 / 第 2792-2844 行

```cpp
2792: TEST(Q8GEMM_2x4c8__SSE2, k_div_8_strided_c) {
2793:   TEST_REQUIRES_X86_SSE2;
2794:   for (size_t k = 16; k < 128; k += 8) {
2795:     GemmMicrokernelTester()
2796:         .mr(2)
2797:         .nr(4)
2798:         .np(1)
2799:         .kr(8)
2800:         .m(2)
2801:         .n(4)
2802:         .k(k)
2803:         .cStride(17)
2804:         .test(pytorch_q8gemm_ukernel_2x4c8__sse2);
2805:   }
2806: }
2807: 
2808: TEST(Q8GEMM_2x4c8__SSE2, k_div_8_subtile) {
2809:   TEST_REQUIRES_X86_SSE2;
2810:   for (size_t k = 16; k < 128; k += 24) {
2811:     for (uint32_t m = 1; m <= 2; m++) {
2812:       for (uint32_t n = 1; n <= 4; n++) {
2813:         GemmMicrokernelTester()
2814:             .mr(2)
2815:             .nr(4)
2816:             .np(1)
2817:             .kr(8)
2818:             .m(m)
2819:             .n(n)
2820:             .k(k)
2821:             .iterations(3)
2822:             .test(pytorch_q8gemm_ukernel_2x4c8__sse2);
2823:       }
2824:     }
2825:   }
2826: }
2827: 
2828: // Following tests fail both on original QNNPack and the version
2829: // with runtime requantization.
2830: 
2831: #if 0
2832:   TEST(Q8GEMM_4x4c2__SSE2, k_eq_1) {
2833:     TEST_REQUIRES_X86_SSE2;
2834:     GemmMicrokernelTester()
2835:       .mr(4)
2836:       .nr(4)
2837:       .np(4)
2838:       .kr(2)
2839:       .m(4)
2840:       .n(4)
2841:       .k(1)
2842:       .test(pytorch_q8gemm_ukernel_4x4c2__sse2);
2843:   }
2844: 
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends; iterates over tensors, descriptors, options, or runtime state. Key symbols: `GemmMicrokernelTester`.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端；遍历张量、描述符、配置项或运行时状态。关键符号：`GemmMicrokernelTester`。

### Lines 2845-2900 / 第 2845-2900 行

```cpp
2845:   TEST(Q8GEMM_4x4c2__SSE2, k_eq_1_strided_a) {
2846:     TEST_REQUIRES_X86_SSE2;
2847:     GemmMicrokernelTester()
2848:       .mr(4)
2849:       .nr(4)
2850:       .np(4)
2851:       .kr(2)
2852:       .m(4)
2853:       .n(4)
2854:       .k(1)
2855:       .aStride(37)
2856:       .test(pytorch_q8gemm_ukernel_4x4c2__sse2);
2857:   }
2858: 
2859:   TEST(Q8GEMM_4x4c2__SSE2, k_eq_1_strided_c) {
2860:     TEST_REQUIRES_X86_SSE2;
2861:     GemmMicrokernelTester()
2862:       .mr(4)
2863:       .nr(4)
2864:       .np(4)
2865:       .kr(2)
2866:       .m(4)
2867:       .n(4)
2868:       .k(1)
2869:       .cStride(17)
2870:       .test(pytorch_q8gemm_ukernel_4x4c2__sse2);
2871:   }
2872: 
2873:   TEST(Q8GEMM_4x4c2__SSE2, k_eq_1_qmin128) {
2874:     TEST_REQUIRES_X86_SSE2;
2875:     GemmMicrokernelTester()
2876:       .mr(4)
2877:       .nr(4)
2878:       .np(4)
2879:       .kr(2)
2880:       .m(4)
2881:       .n(4)
2882:       .k(1)
2883:       .qmin(128)
2884:       .test(pytorch_q8gemm_ukernel_4x4c2__sse2);
2885:   }
2886: 
2887:   TEST(Q8GEMM_4x4c2__SSE2, k_eq_1_qmax128) {
2888:     TEST_REQUIRES_X86_SSE2;
2889:     GemmMicrokernelTester()
2890:       .mr(4)
2891:       .nr(4)
2892:       .np(4)
2893:       .kr(2)
2894:       .m(4)
2895:       .n(4)
2896:       .k(1)
2897:       .qmax(128)
2898:       .test(pytorch_q8gemm_ukernel_4x4c2__sse2);
2899:   }
2900: 
```

- **EN:** This block implements local helper logic for `q8gemm`. Key symbols: `GemmMicrokernelTester`.
- **CN:** 该代码块实现与 `q8gemm` 相关的局部辅助逻辑。关键符号：`GemmMicrokernelTester`。

### Lines 2901-2950 / 第 2901-2950 行

```cpp
2901:   TEST(Q8GEMM_4x4c2__SSE2, k_eq_1_azp0) {
2902:     TEST_REQUIRES_X86_SSE2;
2903:     GemmMicrokernelTester()
2904:       .mr(4)
2905:       .nr(4)
2906:       .np(4)
2907:       .kr(2)
2908:       .m(4)
2909:       .n(4)
2910:       .k(1)
2911:       .aZeroPoint(0)
2912:       .test(pytorch_q8gemm_ukernel_4x4c2__sse2);
2913:   }
2914: 
2915:   TEST(Q8GEMM_4x4c2__SSE2, k_eq_1_bzp0) {
2916:     TEST_REQUIRES_X86_SSE2;
2917:     GemmMicrokernelTester()
2918:       .mr(4)
2919:       .nr(4)
2920:       .np(4)
2921:       .kr(2)
2922:       .m(4)
2923:       .n(4)
2924:       .k(1)
2925:       .bZeroPoint(0)
2926:       .test(pytorch_q8gemm_ukernel_4x4c2__sse2);
2927:   }
2928: 
2929:   TEST(Q8GEMM_4x4c2__SSE2, k_eq_1_nozp) {
2930:     TEST_REQUIRES_X86_SSE2;
2931:     GemmMicrokernelTester()
2932:       .mr(4)
2933:       .nr(4)
2934:       .np(4)
2935:       .kr(2)
2936:       .m(4)
2937:       .n(4)
2938:       .k(1)
2939:       .aZeroPoint(0)
2940:       .bZeroPoint(0)
2941:       .test(pytorch_q8gemm_ukernel_4x4c2__sse2);
2942:   }
2943: #endif
2944: 
2945: TEST(Q8GEMM_4x4c2__SSE2, k_lt_4) {
2946:   TEST_REQUIRES_X86_SSE2;
2947:   GemmMicrokernelTester().mr(4).nr(4).np(4).kr(2).m(4).n(4).k(3).test(
2948:       pytorch_q8gemm_ukernel_4x4c2__sse2);
2949: }
2950: 
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends. Key symbols: `GemmMicrokernelTester`.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端。关键符号：`GemmMicrokernelTester`。

### Lines 2951-3004 / 第 2951-3004 行

```cpp
2951: TEST(Q8GEMM_4x4c2__SSE2, k_lt_4_strided_a) {
2952:   TEST_REQUIRES_X86_SSE2;
2953:   GemmMicrokernelTester()
2954:       .mr(4)
2955:       .nr(4)
2956:       .np(4)
2957:       .kr(2)
2958:       .m(4)
2959:       .n(4)
2960:       .k(3)
2961:       .aStride(37)
2962:       .test(pytorch_q8gemm_ukernel_4x4c2__sse2);
2963: }
2964: 
2965: TEST(Q8GEMM_4x4c2__SSE2, k_lt_4_strided_c) {
2966:   TEST_REQUIRES_X86_SSE2;
2967:   GemmMicrokernelTester()
2968:       .mr(4)
2969:       .nr(4)
2970:       .np(4)
2971:       .kr(2)
2972:       .m(4)
2973:       .n(4)
2974:       .k(3)
2975:       .cStride(17)
2976:       .test(pytorch_q8gemm_ukernel_4x4c2__sse2);
2977: }
2978: 
2979: TEST(Q8GEMM_4x4c2__SSE2, k_lt_4_qmin128) {
2980:   TEST_REQUIRES_X86_SSE2;
2981:   GemmMicrokernelTester().mr(4).nr(4).np(4).kr(2).m(4).n(4).k(3).qmin(128).test(
2982:       pytorch_q8gemm_ukernel_4x4c2__sse2);
2983: }
2984: 
2985: TEST(Q8GEMM_4x4c2__SSE2, k_lt_4_qmax128) {
2986:   TEST_REQUIRES_X86_SSE2;
2987:   GemmMicrokernelTester().mr(4).nr(4).np(4).kr(2).m(4).n(4).k(3).qmax(128).test(
2988:       pytorch_q8gemm_ukernel_4x4c2__sse2);
2989: }
2990: 
2991: TEST(Q8GEMM_4x4c2__SSE2, k_lt_4_azp0) {
2992:   TEST_REQUIRES_X86_SSE2;
2993:   GemmMicrokernelTester()
2994:       .mr(4)
2995:       .nr(4)
2996:       .np(4)
2997:       .kr(2)
2998:       .m(4)
2999:       .n(4)
3000:       .k(3)
3001:       .aZeroPoint(0)
3002:       .test(pytorch_q8gemm_ukernel_4x4c2__sse2);
3003: }
3004: 
```

- **EN:** This block implements local helper logic for `q8gemm`. Key symbols: `GemmMicrokernelTester`.
- **CN:** 该代码块实现与 `q8gemm` 相关的局部辅助逻辑。关键符号：`GemmMicrokernelTester`。

### Lines 3005-3067 / 第 3005-3067 行

```cpp
3005: TEST(Q8GEMM_4x4c2__SSE2, k_lt_4_bzp0) {
3006:   TEST_REQUIRES_X86_SSE2;
3007:   GemmMicrokernelTester()
3008:       .mr(4)
3009:       .nr(4)
3010:       .np(4)
3011:       .kr(2)
3012:       .m(4)
3013:       .n(4)
3014:       .k(3)
3015:       .bZeroPoint(0)
3016:       .test(pytorch_q8gemm_ukernel_4x4c2__sse2);
3017: }
3018: 
3019: TEST(Q8GEMM_4x4c2__SSE2, k_lt_4_nozp) {
3020:   TEST_REQUIRES_X86_SSE2;
3021:   GemmMicrokernelTester()
3022:       .mr(4)
3023:       .nr(4)
3024:       .np(4)
3025:       .kr(2)
3026:       .m(4)
3027:       .n(4)
3028:       .k(3)
3029:       .aZeroPoint(0)
3030:       .bZeroPoint(0)
3031:       .test(pytorch_q8gemm_ukernel_4x4c2__sse2);
3032: }
3033: 
3034: TEST(Q8GEMM_4x4c2__SSE2, k_lt_8) {
3035:   TEST_REQUIRES_X86_SSE2;
3036:   GemmMicrokernelTester().mr(4).nr(4).np(4).kr(2).m(4).n(4).k(5).test(
3037:       pytorch_q8gemm_ukernel_4x4c2__sse2);
3038: }
3039: 
3040: TEST(Q8GEMM_4x4c2__SSE2, k_lt_8_strided_a) {
3041:   TEST_REQUIRES_X86_SSE2;
3042:   GemmMicrokernelTester()
3043:       .mr(4)
3044:       .nr(4)
3045:       .np(4)
3046:       .kr(2)
3047:       .m(4)
3048:       .n(4)
3049:       .k(5)
3050:       .aStride(37)
3051:       .test(pytorch_q8gemm_ukernel_4x4c2__sse2);
3052: }
3053: 
3054: TEST(Q8GEMM_4x4c2__SSE2, k_lt_8_strided_c) {
3055:   TEST_REQUIRES_X86_SSE2;
3056:   GemmMicrokernelTester()
3057:       .mr(4)
3058:       .nr(4)
3059:       .np(4)
3060:       .kr(2)
3061:       .m(4)
3062:       .n(4)
3063:       .k(5)
3064:       .cStride(17)
3065:       .test(pytorch_q8gemm_ukernel_4x4c2__sse2);
3066: }
3067: 
```

- **EN:** This block implements local helper logic for `q8gemm`. Key symbols: `GemmMicrokernelTester`.
- **CN:** 该代码块实现与 `q8gemm` 相关的局部辅助逻辑。关键符号：`GemmMicrokernelTester`。

### Lines 3068-3122 / 第 3068-3122 行

```cpp
3068: TEST(Q8GEMM_4x4c2__SSE2, k_lt_8_qmin128) {
3069:   TEST_REQUIRES_X86_SSE2;
3070:   GemmMicrokernelTester().mr(4).nr(4).np(4).kr(2).m(4).n(4).k(5).qmin(128).test(
3071:       pytorch_q8gemm_ukernel_4x4c2__sse2);
3072: }
3073: 
3074: TEST(Q8GEMM_4x4c2__SSE2, k_lt_8_qmax128) {
3075:   TEST_REQUIRES_X86_SSE2;
3076:   GemmMicrokernelTester().mr(4).nr(4).np(4).kr(2).m(4).n(4).k(5).qmax(128).test(
3077:       pytorch_q8gemm_ukernel_4x4c2__sse2);
3078: }
3079: 
3080: TEST(Q8GEMM_4x4c2__SSE2, k_lt_8_azp0) {
3081:   TEST_REQUIRES_X86_SSE2;
3082:   GemmMicrokernelTester()
3083:       .mr(4)
3084:       .nr(4)
3085:       .np(4)
3086:       .kr(2)
3087:       .m(4)
3088:       .n(4)
3089:       .k(5)
3090:       .aZeroPoint(0)
3091:       .test(pytorch_q8gemm_ukernel_4x4c2__sse2);
3092: }
3093: 
3094: TEST(Q8GEMM_4x4c2__SSE2, k_lt_8_bzp0) {
3095:   TEST_REQUIRES_X86_SSE2;
3096:   GemmMicrokernelTester()
3097:       .mr(4)
3098:       .nr(4)
3099:       .np(4)
3100:       .kr(2)
3101:       .m(4)
3102:       .n(4)
3103:       .k(5)
3104:       .bZeroPoint(0)
3105:       .test(pytorch_q8gemm_ukernel_4x4c2__sse2);
3106: }
3107: 
3108: TEST(Q8GEMM_4x4c2__SSE2, k_lt_8_nozp) {
3109:   TEST_REQUIRES_X86_SSE2;
3110:   GemmMicrokernelTester()
3111:       .mr(4)
3112:       .nr(4)
3113:       .np(4)
3114:       .kr(2)
3115:       .m(4)
3116:       .n(4)
3117:       .k(5)
3118:       .aZeroPoint(0)
3119:       .bZeroPoint(0)
3120:       .test(pytorch_q8gemm_ukernel_4x4c2__sse2);
3121: }
3122: 
```

- **EN:** This block implements local helper logic for `q8gemm`. Key symbols: `GemmMicrokernelTester`.
- **CN:** 该代码块实现与 `q8gemm` 相关的局部辅助逻辑。关键符号：`GemmMicrokernelTester`。

### Lines 3123-3182 / 第 3123-3182 行

```cpp
3123: TEST(Q8GEMM_4x4c2__SSE2, k_eq_8) {
3124:   TEST_REQUIRES_X86_SSE2;
3125:   GemmMicrokernelTester().mr(4).nr(4).np(4).kr(2).m(4).n(4).k(8).test(
3126:       pytorch_q8gemm_ukernel_4x4c2__sse2);
3127: }
3128: 
3129: TEST(Q8GEMM_4x4c2__SSE2, k_eq_8_strided_a) {
3130:   TEST_REQUIRES_X86_SSE2;
3131:   GemmMicrokernelTester()
3132:       .mr(4)
3133:       .nr(4)
3134:       .np(4)
3135:       .kr(2)
3136:       .m(4)
3137:       .n(4)
3138:       .k(8)
3139:       .aStride(37)
3140:       .test(pytorch_q8gemm_ukernel_4x4c2__sse2);
3141: }
3142: 
3143: TEST(Q8GEMM_4x4c2__SSE2, k_eq_8_strided_c) {
3144:   TEST_REQUIRES_X86_SSE2;
3145:   GemmMicrokernelTester()
3146:       .mr(4)
3147:       .nr(4)
3148:       .np(4)
3149:       .kr(2)
3150:       .m(4)
3151:       .n(4)
3152:       .k(8)
3153:       .cStride(17)
3154:       .test(pytorch_q8gemm_ukernel_4x4c2__sse2);
3155: }
3156: 
3157: TEST(Q8GEMM_4x4c2__SSE2, k_eq_8_qmin128) {
3158:   TEST_REQUIRES_X86_SSE2;
3159:   GemmMicrokernelTester().mr(4).nr(4).np(4).kr(2).m(4).n(4).k(8).qmin(128).test(
3160:       pytorch_q8gemm_ukernel_4x4c2__sse2);
3161: }
3162: 
3163: TEST(Q8GEMM_4x4c2__SSE2, k_eq_8_qmax128) {
3164:   TEST_REQUIRES_X86_SSE2;
3165:   GemmMicrokernelTester().mr(4).nr(4).np(4).kr(2).m(4).n(4).k(8).qmax(128).test(
3166:       pytorch_q8gemm_ukernel_4x4c2__sse2);
3167: }
3168: 
3169: TEST(Q8GEMM_4x4c2__SSE2, k_eq_8_azp0) {
3170:   TEST_REQUIRES_X86_SSE2;
3171:   GemmMicrokernelTester()
3172:       .mr(4)
3173:       .nr(4)
3174:       .np(4)
3175:       .kr(2)
3176:       .m(4)
3177:       .n(4)
3178:       .k(8)
3179:       .aZeroPoint(0)
3180:       .test(pytorch_q8gemm_ukernel_4x4c2__sse2);
3181: }
3182: 
```

- **EN:** This block implements local helper logic for `q8gemm`. Key symbols: `GemmMicrokernelTester`.
- **CN:** 该代码块实现与 `q8gemm` 相关的局部辅助逻辑。关键符号：`GemmMicrokernelTester`。

### Lines 3183-3235 / 第 3183-3235 行

```cpp
3183: TEST(Q8GEMM_4x4c2__SSE2, k_eq_8_bzp0) {
3184:   TEST_REQUIRES_X86_SSE2;
3185:   GemmMicrokernelTester()
3186:       .mr(4)
3187:       .nr(4)
3188:       .np(4)
3189:       .kr(2)
3190:       .m(4)
3191:       .n(4)
3192:       .k(8)
3193:       .bZeroPoint(0)
3194:       .test(pytorch_q8gemm_ukernel_4x4c2__sse2);
3195: }
3196: 
3197: TEST(Q8GEMM_4x4c2__SSE2, k_eq_8_nozp) {
3198:   TEST_REQUIRES_X86_SSE2;
3199:   GemmMicrokernelTester()
3200:       .mr(4)
3201:       .nr(4)
3202:       .np(4)
3203:       .kr(2)
3204:       .m(4)
3205:       .n(4)
3206:       .k(8)
3207:       .aZeroPoint(0)
3208:       .bZeroPoint(0)
3209:       .test(pytorch_q8gemm_ukernel_4x4c2__sse2);
3210: }
3211: 
3212: TEST(Q8GEMM_4x4c2__SSE2, k_gt_8) {
3213:   TEST_REQUIRES_X86_SSE2;
3214:   for (size_t k = 9; k < 16; k++) {
3215:     GemmMicrokernelTester().mr(4).nr(4).np(4).kr(2).m(4).n(4).k(k).test(
3216:         pytorch_q8gemm_ukernel_4x4c2__sse2);
3217:   }
3218: }
3219: 
3220: TEST(Q8GEMM_4x4c2__SSE2, k_gt_8_strided_a) {
3221:   TEST_REQUIRES_X86_SSE2;
3222:   for (size_t k = 9; k < 16; k++) {
3223:     GemmMicrokernelTester()
3224:         .mr(4)
3225:         .nr(4)
3226:         .np(4)
3227:         .kr(2)
3228:         .m(4)
3229:         .n(4)
3230:         .k(k)
3231:         .aStride(37)
3232:         .test(pytorch_q8gemm_ukernel_4x4c2__sse2);
3233:   }
3234: }
3235: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `GemmMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`GemmMicrokernelTester`。

### Lines 3236-3286 / 第 3236-3286 行

```cpp
3236: TEST(Q8GEMM_4x4c2__SSE2, k_gt_8_strided_c) {
3237:   TEST_REQUIRES_X86_SSE2;
3238:   for (size_t k = 9; k < 16; k++) {
3239:     GemmMicrokernelTester()
3240:         .mr(4)
3241:         .nr(4)
3242:         .np(4)
3243:         .kr(2)
3244:         .m(4)
3245:         .n(4)
3246:         .k(k)
3247:         .cStride(17)
3248:         .test(pytorch_q8gemm_ukernel_4x4c2__sse2);
3249:   }
3250: }
3251: 
3252: TEST(Q8GEMM_4x4c2__SSE2, k_gt_8_azp0) {
3253:   TEST_REQUIRES_X86_SSE2;
3254:   for (size_t k = 9; k < 16; k++) {
3255:     GemmMicrokernelTester()
3256:         .mr(4)
3257:         .nr(4)
3258:         .np(4)
3259:         .kr(2)
3260:         .m(4)
3261:         .n(4)
3262:         .k(k)
3263:         .aZeroPoint(0)
3264:         .test(pytorch_q8gemm_ukernel_4x4c2__sse2);
3265:   }
3266: }
3267: 
3268: TEST(Q8GEMM_4x4c2__SSE2, k_gt_8_bzp0) {
3269:   TEST_REQUIRES_X86_SSE2;
3270:   for (size_t k = 9; k < 16; k++) {
3271:     GemmMicrokernelTester()
3272:         .mr(4)
3273:         .nr(4)
3274:         .np(4)
3275:         .kr(2)
3276:         .m(4)
3277:         .n(4)
3278:         .k(k)
3279:         .bZeroPoint(0)
3280:         .test(pytorch_q8gemm_ukernel_4x4c2__sse2);
3281:   }
3282: }
3283: 
3284: TEST(Q8GEMM_4x4c2__SSE2, k_gt_8_nozp) {
3285:   TEST_REQUIRES_X86_SSE2;
3286:   for (size_t k = 9; k < 16; k++) {
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `GemmMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`GemmMicrokernelTester`。

### Lines 3287-3344 / 第 3287-3344 行

```cpp
3287:     GemmMicrokernelTester()
3288:         .mr(4)
3289:         .nr(4)
3290:         .np(4)
3291:         .kr(2)
3292:         .m(4)
3293:         .n(4)
3294:         .k(k)
3295:         .aZeroPoint(0)
3296:         .bZeroPoint(0)
3297:         .test(pytorch_q8gemm_ukernel_4x4c2__sse2);
3298:   }
3299: }
3300: 
3301: TEST(Q8GEMM_4x4c2__SSE2, k_gt_8_subtile) {
3302:   TEST_REQUIRES_X86_SSE2;
3303:   for (size_t k = 9; k < 16; k++) {
3304:     for (uint32_t m = 1; m <= 4; m++) {
3305:       for (uint32_t n = 1; n <= 4; n++) {
3306:         GemmMicrokernelTester()
3307:             .mr(4)
3308:             .nr(4)
3309:             .np(4)
3310:             .kr(2)
3311:             .m(m)
3312:             .n(n)
3313:             .k(k)
3314:             .iterations(3)
3315:             .test(pytorch_q8gemm_ukernel_4x4c2__sse2);
3316:       }
3317:     }
3318:   }
3319: }
3320: 
3321: TEST(Q8GEMM_4x4c2__SSE2, k_div_8) {
3322:   TEST_REQUIRES_X86_SSE2;
3323:   for (size_t k = 16; k < 128; k += 8) {
3324:     GemmMicrokernelTester().mr(4).nr(4).np(4).kr(2).m(4).n(4).k(k).test(
3325:         pytorch_q8gemm_ukernel_4x4c2__sse2);
3326:   }
3327: }
3328: 
3329: TEST(Q8GEMM_4x4c2__SSE2, k_div_8_strided_a) {
3330:   TEST_REQUIRES_X86_SSE2;
3331:   for (size_t k = 16; k < 128; k += 8) {
3332:     GemmMicrokernelTester()
3333:         .mr(4)
3334:         .nr(4)
3335:         .np(4)
3336:         .kr(2)
3337:         .m(4)
3338:         .n(4)
3339:         .k(k)
3340:         .aStride(171)
3341:         .test(pytorch_q8gemm_ukernel_4x4c2__sse2);
3342:   }
3343: }
3344: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `GemmMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`GemmMicrokernelTester`。

### Lines 3345-3404 / 第 3345-3404 行

```cpp
3345: TEST(Q8GEMM_4x4c2__SSE2, k_div_8_strided_c) {
3346:   TEST_REQUIRES_X86_SSE2;
3347:   for (size_t k = 16; k < 128; k += 8) {
3348:     GemmMicrokernelTester()
3349:         .mr(4)
3350:         .nr(4)
3351:         .np(4)
3352:         .kr(2)
3353:         .m(4)
3354:         .n(4)
3355:         .k(k)
3356:         .cStride(17)
3357:         .test(pytorch_q8gemm_ukernel_4x4c2__sse2);
3358:   }
3359: }
3360: 
3361: TEST(Q8GEMM_4x4c2__SSE2, k_div_8_subtile) {
3362:   TEST_REQUIRES_X86_SSE2;
3363:   for (size_t k = 16; k < 128; k += 24) {
3364:     for (uint32_t m = 1; m <= 4; m++) {
3365:       for (uint32_t n = 1; n <= 4; n++) {
3366:         GemmMicrokernelTester()
3367:             .mr(4)
3368:             .nr(4)
3369:             .np(4)
3370:             .kr(2)
3371:             .m(m)
3372:             .n(n)
3373:             .k(k)
3374:             .iterations(3)
3375:             .test(pytorch_q8gemm_ukernel_4x4c2__sse2);
3376:       }
3377:     }
3378:   }
3379: }
3380: 
3381: //
3382: // Dynamic Quantization
3383: //
3384: 
3385: TEST(Q8GEMM_DQ_4x4c2__SSE2, k_lt_4) {
3386:   TEST_REQUIRES_X86_SSE2;
3387:   GemmMicrokernelTester().mr(4).nr(4).np(4).kr(2).m(4).n(4).k(3).test(
3388:       pytorch_q8gemm_dq_ukernel_4x4c2__sse2);
3389: }
3390: 
3391: TEST(Q8GEMM_DQ_4x4c2__SSE2, k_lt_4_strided_a) {
3392:   TEST_REQUIRES_X86_SSE2;
3393:   GemmMicrokernelTester()
3394:       .mr(4)
3395:       .nr(4)
3396:       .np(4)
3397:       .kr(2)
3398:       .m(4)
3399:       .n(4)
3400:       .k(3)
3401:       .aStride(37)
3402:       .test(pytorch_q8gemm_dq_ukernel_4x4c2__sse2);
3403: }
3404: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `GemmMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`GemmMicrokernelTester`。

### Lines 3405-3458 / 第 3405-3458 行

```cpp
3405: TEST(Q8GEMM_DQ_4x4c2__SSE2, k_lt_4_strided_c) {
3406:   TEST_REQUIRES_X86_SSE2;
3407:   GemmMicrokernelTester()
3408:       .mr(4)
3409:       .nr(4)
3410:       .np(4)
3411:       .kr(2)
3412:       .m(4)
3413:       .n(4)
3414:       .k(3)
3415:       .cStride(17)
3416:       .test(pytorch_q8gemm_dq_ukernel_4x4c2__sse2);
3417: }
3418: 
3419: TEST(Q8GEMM_DQ_4x4c2__SSE2, k_lt_4_qmin128) {
3420:   TEST_REQUIRES_X86_SSE2;
3421:   GemmMicrokernelTester().mr(4).nr(4).np(4).kr(2).m(4).n(4).k(3).qmin(128).test(
3422:       pytorch_q8gemm_dq_ukernel_4x4c2__sse2);
3423: }
3424: 
3425: TEST(Q8GEMM_DQ_4x4c2__SSE2, k_lt_4_qmax128) {
3426:   TEST_REQUIRES_X86_SSE2;
3427:   GemmMicrokernelTester().mr(4).nr(4).np(4).kr(2).m(4).n(4).k(3).qmax(128).test(
3428:       pytorch_q8gemm_dq_ukernel_4x4c2__sse2);
3429: }
3430: 
3431: TEST(Q8GEMM_DQ_4x4c2__SSE2, k_lt_4_azp0) {
3432:   TEST_REQUIRES_X86_SSE2;
3433:   GemmMicrokernelTester()
3434:       .mr(4)
3435:       .nr(4)
3436:       .np(4)
3437:       .kr(2)
3438:       .m(4)
3439:       .n(4)
3440:       .k(3)
3441:       .aZeroPoint(0)
3442:       .test(pytorch_q8gemm_dq_ukernel_4x4c2__sse2);
3443: }
3444: 
3445: TEST(Q8GEMM_DQ_4x4c2__SSE2, k_lt_4_bzp0) {
3446:   TEST_REQUIRES_X86_SSE2;
3447:   GemmMicrokernelTester()
3448:       .mr(4)
3449:       .nr(4)
3450:       .np(4)
3451:       .kr(2)
3452:       .m(4)
3453:       .n(4)
3454:       .k(3)
3455:       .bZeroPoint(0)
3456:       .test(pytorch_q8gemm_dq_ukernel_4x4c2__sse2);
3457: }
3458: 
```

- **EN:** This block implements local helper logic for `q8gemm`. Key symbols: `GemmMicrokernelTester`.
- **CN:** 该代码块实现与 `q8gemm` 相关的局部辅助逻辑。关键符号：`GemmMicrokernelTester`。

### Lines 3459-3513 / 第 3459-3513 行

```cpp
3459: TEST(Q8GEMM_DQ_4x4c2__SSE2, k_lt_4_nozp) {
3460:   TEST_REQUIRES_X86_SSE2;
3461:   GemmMicrokernelTester()
3462:       .mr(4)
3463:       .nr(4)
3464:       .np(4)
3465:       .kr(2)
3466:       .m(4)
3467:       .n(4)
3468:       .k(3)
3469:       .aZeroPoint(0)
3470:       .bZeroPoint(0)
3471:       .test(pytorch_q8gemm_dq_ukernel_4x4c2__sse2);
3472: }
3473: 
3474: TEST(Q8GEMM_DQ_4x4c2__SSE2, k_lt_8) {
3475:   TEST_REQUIRES_X86_SSE2;
3476:   GemmMicrokernelTester().mr(4).nr(4).np(4).kr(2).m(4).n(4).k(5).test(
3477:       pytorch_q8gemm_dq_ukernel_4x4c2__sse2);
3478: }
3479: 
3480: TEST(Q8GEMM_DQ_4x4c2__SSE2, k_lt_8_strided_a) {
3481:   TEST_REQUIRES_X86_SSE2;
3482:   GemmMicrokernelTester()
3483:       .mr(4)
3484:       .nr(4)
3485:       .np(4)
3486:       .kr(2)
3487:       .m(4)
3488:       .n(4)
3489:       .k(5)
3490:       .aStride(37)
3491:       .test(pytorch_q8gemm_dq_ukernel_4x4c2__sse2);
3492: }
3493: 
3494: TEST(Q8GEMM_DQ_4x4c2__SSE2, k_lt_8_strided_c) {
3495:   TEST_REQUIRES_X86_SSE2;
3496:   GemmMicrokernelTester()
3497:       .mr(4)
3498:       .nr(4)
3499:       .np(4)
3500:       .kr(2)
3501:       .m(4)
3502:       .n(4)
3503:       .k(5)
3504:       .cStride(17)
3505:       .test(pytorch_q8gemm_dq_ukernel_4x4c2__sse2);
3506: }
3507: 
3508: TEST(Q8GEMM_DQ_4x4c2__SSE2, k_lt_8_qmin128) {
3509:   TEST_REQUIRES_X86_SSE2;
3510:   GemmMicrokernelTester().mr(4).nr(4).np(4).kr(2).m(4).n(4).k(5).qmin(128).test(
3511:       pytorch_q8gemm_dq_ukernel_4x4c2__sse2);
3512: }
3513: 
```

- **EN:** This block implements local helper logic for `q8gemm`. Key symbols: `GemmMicrokernelTester`.
- **CN:** 该代码块实现与 `q8gemm` 相关的局部辅助逻辑。关键符号：`GemmMicrokernelTester`。

### Lines 3514-3568 / 第 3514-3568 行

```cpp
3514: TEST(Q8GEMM_DQ_4x4c2__SSE2, k_lt_8_qmax128) {
3515:   TEST_REQUIRES_X86_SSE2;
3516:   GemmMicrokernelTester().mr(4).nr(4).np(4).kr(2).m(4).n(4).k(5).qmax(128).test(
3517:       pytorch_q8gemm_dq_ukernel_4x4c2__sse2);
3518: }
3519: 
3520: TEST(Q8GEMM_DQ_4x4c2__SSE2, k_lt_8_azp0) {
3521:   TEST_REQUIRES_X86_SSE2;
3522:   GemmMicrokernelTester()
3523:       .mr(4)
3524:       .nr(4)
3525:       .np(4)
3526:       .kr(2)
3527:       .m(4)
3528:       .n(4)
3529:       .k(5)
3530:       .aZeroPoint(0)
3531:       .test(pytorch_q8gemm_dq_ukernel_4x4c2__sse2);
3532: }
3533: 
3534: TEST(Q8GEMM_DQ_4x4c2__SSE2, k_lt_8_bzp0) {
3535:   TEST_REQUIRES_X86_SSE2;
3536:   GemmMicrokernelTester()
3537:       .mr(4)
3538:       .nr(4)
3539:       .np(4)
3540:       .kr(2)
3541:       .m(4)
3542:       .n(4)
3543:       .k(5)
3544:       .bZeroPoint(0)
3545:       .test(pytorch_q8gemm_dq_ukernel_4x4c2__sse2);
3546: }
3547: 
3548: TEST(Q8GEMM_DQ_4x4c2__SSE2, k_lt_8_nozp) {
3549:   TEST_REQUIRES_X86_SSE2;
3550:   GemmMicrokernelTester()
3551:       .mr(4)
3552:       .nr(4)
3553:       .np(4)
3554:       .kr(2)
3555:       .m(4)
3556:       .n(4)
3557:       .k(5)
3558:       .aZeroPoint(0)
3559:       .bZeroPoint(0)
3560:       .test(pytorch_q8gemm_dq_ukernel_4x4c2__sse2);
3561: }
3562: 
3563: TEST(Q8GEMM_DQ_4x4c2__SSE2, k_eq_8) {
3564:   TEST_REQUIRES_X86_SSE2;
3565:   GemmMicrokernelTester().mr(4).nr(4).np(4).kr(2).m(4).n(4).k(8).test(
3566:       pytorch_q8gemm_dq_ukernel_4x4c2__sse2);
3567: }
3568: 
```

- **EN:** This block implements local helper logic for `q8gemm`. Key symbols: `GemmMicrokernelTester`.
- **CN:** 该代码块实现与 `q8gemm` 相关的局部辅助逻辑。关键符号：`GemmMicrokernelTester`。

### Lines 3569-3622 / 第 3569-3622 行

```cpp
3569: TEST(Q8GEMM_DQ_4x4c2__SSE2, k_eq_8_strided_a) {
3570:   TEST_REQUIRES_X86_SSE2;
3571:   GemmMicrokernelTester()
3572:       .mr(4)
3573:       .nr(4)
3574:       .np(4)
3575:       .kr(2)
3576:       .m(4)
3577:       .n(4)
3578:       .k(8)
3579:       .aStride(37)
3580:       .test(pytorch_q8gemm_dq_ukernel_4x4c2__sse2);
3581: }
3582: 
3583: TEST(Q8GEMM_DQ_4x4c2__SSE2, k_eq_8_strided_c) {
3584:   TEST_REQUIRES_X86_SSE2;
3585:   GemmMicrokernelTester()
3586:       .mr(4)
3587:       .nr(4)
3588:       .np(4)
3589:       .kr(2)
3590:       .m(4)
3591:       .n(4)
3592:       .k(8)
3593:       .cStride(17)
3594:       .test(pytorch_q8gemm_dq_ukernel_4x4c2__sse2);
3595: }
3596: 
3597: TEST(Q8GEMM_DQ_4x4c2__SSE2, k_eq_8_qmin128) {
3598:   TEST_REQUIRES_X86_SSE2;
3599:   GemmMicrokernelTester().mr(4).nr(4).np(4).kr(2).m(4).n(4).k(8).qmin(128).test(
3600:       pytorch_q8gemm_dq_ukernel_4x4c2__sse2);
3601: }
3602: 
3603: TEST(Q8GEMM_DQ_4x4c2__SSE2, k_eq_8_qmax128) {
3604:   TEST_REQUIRES_X86_SSE2;
3605:   GemmMicrokernelTester().mr(4).nr(4).np(4).kr(2).m(4).n(4).k(8).qmax(128).test(
3606:       pytorch_q8gemm_dq_ukernel_4x4c2__sse2);
3607: }
3608: 
3609: TEST(Q8GEMM_DQ_4x4c2__SSE2, k_eq_8_azp0) {
3610:   TEST_REQUIRES_X86_SSE2;
3611:   GemmMicrokernelTester()
3612:       .mr(4)
3613:       .nr(4)
3614:       .np(4)
3615:       .kr(2)
3616:       .m(4)
3617:       .n(4)
3618:       .k(8)
3619:       .aZeroPoint(0)
3620:       .test(pytorch_q8gemm_dq_ukernel_4x4c2__sse2);
3621: }
3622: 
```

- **EN:** This block implements local helper logic for `q8gemm`. Key symbols: `GemmMicrokernelTester`.
- **CN:** 该代码块实现与 `q8gemm` 相关的局部辅助逻辑。关键符号：`GemmMicrokernelTester`。

### Lines 3623-3675 / 第 3623-3675 行

```cpp
3623: TEST(Q8GEMM_DQ_4x4c2__SSE2, k_eq_8_bzp0) {
3624:   TEST_REQUIRES_X86_SSE2;
3625:   GemmMicrokernelTester()
3626:       .mr(4)
3627:       .nr(4)
3628:       .np(4)
3629:       .kr(2)
3630:       .m(4)
3631:       .n(4)
3632:       .k(8)
3633:       .bZeroPoint(0)
3634:       .test(pytorch_q8gemm_dq_ukernel_4x4c2__sse2);
3635: }
3636: 
3637: TEST(Q8GEMM_DQ_4x4c2__SSE2, k_eq_8_nozp) {
3638:   TEST_REQUIRES_X86_SSE2;
3639:   GemmMicrokernelTester()
3640:       .mr(4)
3641:       .nr(4)
3642:       .np(4)
3643:       .kr(2)
3644:       .m(4)
3645:       .n(4)
3646:       .k(8)
3647:       .aZeroPoint(0)
3648:       .bZeroPoint(0)
3649:       .test(pytorch_q8gemm_dq_ukernel_4x4c2__sse2);
3650: }
3651: 
3652: TEST(Q8GEMM_DQ_4x4c2__SSE2, k_gt_8) {
3653:   TEST_REQUIRES_X86_SSE2;
3654:   for (size_t k = 9; k < 16; k++) {
3655:     GemmMicrokernelTester().mr(4).nr(4).np(4).kr(2).m(4).n(4).k(k).test(
3656:         pytorch_q8gemm_dq_ukernel_4x4c2__sse2);
3657:   }
3658: }
3659: 
3660: TEST(Q8GEMM_DQ_4x4c2__SSE2, k_gt_8_strided_a) {
3661:   TEST_REQUIRES_X86_SSE2;
3662:   for (size_t k = 9; k < 16; k++) {
3663:     GemmMicrokernelTester()
3664:         .mr(4)
3665:         .nr(4)
3666:         .np(4)
3667:         .kr(2)
3668:         .m(4)
3669:         .n(4)
3670:         .k(k)
3671:         .aStride(37)
3672:         .test(pytorch_q8gemm_dq_ukernel_4x4c2__sse2);
3673:   }
3674: }
3675: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `GemmMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`GemmMicrokernelTester`。

### Lines 3676-3726 / 第 3676-3726 行

```cpp
3676: TEST(Q8GEMM_DQ_4x4c2__SSE2, k_gt_8_strided_c) {
3677:   TEST_REQUIRES_X86_SSE2;
3678:   for (size_t k = 9; k < 16; k++) {
3679:     GemmMicrokernelTester()
3680:         .mr(4)
3681:         .nr(4)
3682:         .np(4)
3683:         .kr(2)
3684:         .m(4)
3685:         .n(4)
3686:         .k(k)
3687:         .cStride(17)
3688:         .test(pytorch_q8gemm_dq_ukernel_4x4c2__sse2);
3689:   }
3690: }
3691: 
3692: TEST(Q8GEMM_DQ_4x4c2__SSE2, k_gt_8_azp0) {
3693:   TEST_REQUIRES_X86_SSE2;
3694:   for (size_t k = 9; k < 16; k++) {
3695:     GemmMicrokernelTester()
3696:         .mr(4)
3697:         .nr(4)
3698:         .np(4)
3699:         .kr(2)
3700:         .m(4)
3701:         .n(4)
3702:         .k(k)
3703:         .aZeroPoint(0)
3704:         .test(pytorch_q8gemm_dq_ukernel_4x4c2__sse2);
3705:   }
3706: }
3707: 
3708: TEST(Q8GEMM_DQ_4x4c2__SSE2, k_gt_8_bzp0) {
3709:   TEST_REQUIRES_X86_SSE2;
3710:   for (size_t k = 9; k < 16; k++) {
3711:     GemmMicrokernelTester()
3712:         .mr(4)
3713:         .nr(4)
3714:         .np(4)
3715:         .kr(2)
3716:         .m(4)
3717:         .n(4)
3718:         .k(k)
3719:         .bZeroPoint(0)
3720:         .test(pytorch_q8gemm_dq_ukernel_4x4c2__sse2);
3721:   }
3722: }
3723: 
3724: TEST(Q8GEMM_DQ_4x4c2__SSE2, k_gt_8_nozp) {
3725:   TEST_REQUIRES_X86_SSE2;
3726:   for (size_t k = 9; k < 16; k++) {
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `GemmMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`GemmMicrokernelTester`。

### Lines 3727-3784 / 第 3727-3784 行

```cpp
3727:     GemmMicrokernelTester()
3728:         .mr(4)
3729:         .nr(4)
3730:         .np(4)
3731:         .kr(2)
3732:         .m(4)
3733:         .n(4)
3734:         .k(k)
3735:         .aZeroPoint(0)
3736:         .bZeroPoint(0)
3737:         .test(pytorch_q8gemm_dq_ukernel_4x4c2__sse2);
3738:   }
3739: }
3740: 
3741: TEST(Q8GEMM_DQ_4x4c2__SSE2, k_gt_8_subtile) {
3742:   TEST_REQUIRES_X86_SSE2;
3743:   for (size_t k = 9; k < 16; k++) {
3744:     for (uint32_t m = 1; m <= 4; m++) {
3745:       for (uint32_t n = 1; n <= 4; n++) {
3746:         GemmMicrokernelTester()
3747:             .mr(4)
3748:             .nr(4)
3749:             .np(4)
3750:             .kr(2)
3751:             .m(m)
3752:             .n(n)
3753:             .k(k)
3754:             .iterations(3)
3755:             .test(pytorch_q8gemm_dq_ukernel_4x4c2__sse2);
3756:       }
3757:     }
3758:   }
3759: }
3760: 
3761: TEST(Q8GEMM_DQ_4x4c2__SSE2, k_div_8) {
3762:   TEST_REQUIRES_X86_SSE2;
3763:   for (size_t k = 16; k < 128; k += 8) {
3764:     GemmMicrokernelTester().mr(4).nr(4).np(4).kr(2).m(4).n(4).k(k).test(
3765:         pytorch_q8gemm_dq_ukernel_4x4c2__sse2);
3766:   }
3767: }
3768: 
3769: TEST(Q8GEMM_DQ_4x4c2__SSE2, k_div_8_strided_a) {
3770:   TEST_REQUIRES_X86_SSE2;
3771:   for (size_t k = 16; k < 128; k += 8) {
3772:     GemmMicrokernelTester()
3773:         .mr(4)
3774:         .nr(4)
3775:         .np(4)
3776:         .kr(2)
3777:         .m(4)
3778:         .n(4)
3779:         .k(k)
3780:         .aStride(171)
3781:         .test(pytorch_q8gemm_dq_ukernel_4x4c2__sse2);
3782:   }
3783: }
3784: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `GemmMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`GemmMicrokernelTester`。

### Lines 3785-3820 / 第 3785-3820 行

```cpp
3785: TEST(Q8GEMM_DQ_4x4c2__SSE2, k_div_8_strided_c) {
3786:   TEST_REQUIRES_X86_SSE2;
3787:   for (size_t k = 16; k < 128; k += 8) {
3788:     GemmMicrokernelTester()
3789:         .mr(4)
3790:         .nr(4)
3791:         .np(4)
3792:         .kr(2)
3793:         .m(4)
3794:         .n(4)
3795:         .k(k)
3796:         .cStride(17)
3797:         .test(pytorch_q8gemm_dq_ukernel_4x4c2__sse2);
3798:   }
3799: }
3800: 
3801: TEST(Q8GEMM_DQ_4x4c2__SSE2, k_div_8_subtile) {
3802:   TEST_REQUIRES_X86_SSE2;
3803:   for (size_t k = 16; k < 128; k += 24) {
3804:     for (uint32_t m = 1; m <= 4; m++) {
3805:       for (uint32_t n = 1; n <= 4; n++) {
3806:         GemmMicrokernelTester()
3807:             .mr(4)
3808:             .nr(4)
3809:             .np(4)
3810:             .kr(2)
3811:             .m(m)
3812:             .n(n)
3813:             .k(k)
3814:             .iterations(3)
3815:             .test(pytorch_q8gemm_dq_ukernel_4x4c2__sse2);
3816:       }
3817:     }
3818:   }
3819: }
3820: #endif
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
- **External includes / 外部头文件**: `cpuinfo.h`, `gtest/gtest.h`, `qnnpack/isa-checks.h`, `qnnpack/q8gemm.h`, `gemm-microkernel-tester.h`
- **Namespaces / 命名空间**: 无
- **Representative symbols / 代表性符号**: `GemmMicrokernelTester`
