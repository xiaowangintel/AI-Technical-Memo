# q8avgpool.cc — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `aten/src/ATen/native/quantized/cpu/qnnpack/test/q8avgpool.cc`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements ATen native operators, checks, helper kernels, and backend dispatch behavior. This specific file centers on `q8avgpool.cc`. The file header highlights: "All rights reserved. This source code is licensed under the BSD-style license found in the LICENSE file in the root directory of this source tree.."
- **Purpose (CN)**: 实现 ATen 原生算子、运行时检查、辅助内核以及后端分发行为。 该文件具体围绕 `q8avgpool.cc` 展开。 文件头部注释也概括了其核心职责。

## Line-by-Line Analysis / 逐行分析
### Lines 1-45 / 第 1-45 行

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
0013: #include <qnnpack/q8avgpool.h>
0014: 
0015: #include "avgpool-microkernel-tester.h"
0016: 
0017: #if CPUINFO_ARCH_ARM || CPUINFO_ARCH_ARM64
0018: TEST(Q8AVGPOOL_UP8xM__NEON, kc_lt_8_small_ks) {
0019:   TEST_REQUIRES_ARM_NEON;
0020:   for (size_t kc = 1; kc < 8; kc++) {
0021:     for (size_t ks = 1; ks < 8; ks++) {
0022:       for (size_t kh = 1; kh <= ks; kh++) {
0023:         for (size_t kw = 1; kw <= ks; kw++) {
0024:           if (kh * kw == ks) {
0025:             AvgPoolMicrokernelTester().kr(8).kh(kh).kw(kw).kc(kc).test(
0026:                 pytorch_q8avgpool_ukernel_up8xm__neon);
0027:           }
0028:         }
0029:       }
0030:     }
0031:   }
0032: }
0033: 
0034: TEST(Q8AVGPOOL_UP8xM__NEON, kc_lt_8_large_ks) {
0035:   TEST_REQUIRES_ARM_NEON;
0036:   for (size_t kc = 1; kc < 8; kc++) {
0037:     for (size_t ks = 8; ks < 16; ks++) {
0038:       AvgPoolMicrokernelTester().kr(8).kh(ks).kw(1).kc(kc).test(
0039:           pytorch_q8avgpool_ukernel_up8xm__neon);
0040:       AvgPoolMicrokernelTester().kr(8).kh(1).kw(ks).kc(kc).test(
0041:           pytorch_q8avgpool_ukernel_up8xm__neon);
0042:     }
0043:   }
0044: }
0045: 
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends; handles conditional branches and special cases; iterates over tensors, descriptors, options, or runtime state. Key symbols: `AvgPoolMicrokernelTester`.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端；处理条件分支与特殊情况；遍历张量、描述符、配置项或运行时状态。关键符号：`AvgPoolMicrokernelTester`。

### Lines 46-87 / 第 46-87 行

```cpp
0046: TEST(Q8AVGPOOL_UP8xM__NEON, kc_lt_8_with_x_scale) {
0047:   TEST_REQUIRES_ARM_NEON;
0048:   for (size_t n = 1; n <= 3; n += 2) {
0049:     for (size_t kc = 1; kc < 8; kc++) {
0050:       for (size_t ks : std::vector<size_t>{{2, 3, 5}}) {
0051:         for (float xScale = 0.01f; xScale < 100.0f; xScale *= 3.14159265f) {
0052:           AvgPoolMicrokernelTester()
0053:               .kr(8)
0054:               .n(n)
0055:               .kh(ks)
0056:               .kw(ks)
0057:               .kc(kc)
0058:               .xScale(xScale)
0059:               .iterations(1)
0060:               .test(pytorch_q8avgpool_ukernel_up8xm__neon);
0061:         }
0062:       }
0063:     }
0064:   }
0065: }
0066: 
0067: TEST(Q8AVGPOOL_UP8xM__NEON, kc_lt_8_with_x_zero_point) {
0068:   TEST_REQUIRES_ARM_NEON;
0069:   for (size_t n = 1; n <= 3; n += 2) {
0070:     for (size_t kc = 1; kc < 8; kc++) {
0071:       for (size_t ks : std::vector<size_t>{{2, 3, 5}}) {
0072:         for (int32_t xZeroPoint = 0; xZeroPoint <= 255; xZeroPoint += 51) {
0073:           AvgPoolMicrokernelTester()
0074:               .kr(8)
0075:               .n(n)
0076:               .kh(ks)
0077:               .kw(ks)
0078:               .kc(kc)
0079:               .xZeroPoint(uint8_t(xZeroPoint))
0080:               .iterations(1)
0081:               .test(pytorch_q8avgpool_ukernel_up8xm__neon);
0082:         }
0083:       }
0084:     }
0085:   }
0086: }
0087: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `AvgPoolMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`AvgPoolMicrokernelTester`。

### Lines 88-129 / 第 88-129 行

```cpp
0088: TEST(Q8AVGPOOL_UP8xM__NEON, kc_lt_8_with_y_scale) {
0089:   TEST_REQUIRES_ARM_NEON;
0090:   for (size_t n = 1; n <= 3; n += 2) {
0091:     for (size_t kc = 1; kc < 8; kc++) {
0092:       for (size_t ks : std::vector<size_t>{{2, 3, 5}}) {
0093:         for (float yScale = 0.01f; yScale < 100.0f; yScale *= 3.14159265f) {
0094:           AvgPoolMicrokernelTester()
0095:               .kr(8)
0096:               .n(n)
0097:               .kh(ks)
0098:               .kw(ks)
0099:               .kc(kc)
0100:               .yScale(yScale)
0101:               .iterations(1)
0102:               .test(pytorch_q8avgpool_ukernel_up8xm__neon);
0103:         }
0104:       }
0105:     }
0106:   }
0107: }
0108: 
0109: TEST(Q8AVGPOOL_UP8xM__NEON, kc_lt_8_with_y_zero_point) {
0110:   TEST_REQUIRES_ARM_NEON;
0111:   for (size_t n = 1; n <= 3; n += 2) {
0112:     for (size_t kc = 1; kc < 8; kc++) {
0113:       for (size_t ks : std::vector<size_t>{{2, 3, 5}}) {
0114:         for (int32_t yZeroPoint = 0; yZeroPoint <= 255; yZeroPoint += 51) {
0115:           AvgPoolMicrokernelTester()
0116:               .kr(8)
0117:               .n(n)
0118:               .kh(ks)
0119:               .kw(ks)
0120:               .kc(kc)
0121:               .yZeroPoint(uint8_t(yZeroPoint))
0122:               .iterations(1)
0123:               .test(pytorch_q8avgpool_ukernel_up8xm__neon);
0124:         }
0125:       }
0126:     }
0127:   }
0128: }
0129: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `AvgPoolMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`AvgPoolMicrokernelTester`。

### Lines 130-175 / 第 130-175 行

```cpp
0130: TEST(Q8AVGPOOL_UP8xM__NEON, kc_lt_8_with_y_max) {
0131:   TEST_REQUIRES_ARM_NEON;
0132:   for (size_t n = 1; n <= 3; n += 2) {
0133:     for (size_t kc = 1; kc < 8; kc++) {
0134:       for (size_t ks : std::vector<size_t>{{2, 3, 5}}) {
0135:         AvgPoolMicrokernelTester()
0136:             .kr(8)
0137:             .n(n)
0138:             .kh(ks)
0139:             .kw(ks)
0140:             .kc(kc)
0141:             .xZeroPoint(128)
0142:             .yZeroPoint(128)
0143:             .xScale(1.0f)
0144:             .yScale(1.0f)
0145:             .yMax(128)
0146:             .iterations(3)
0147:             .test(pytorch_q8avgpool_ukernel_up8xm__neon);
0148:       }
0149:     }
0150:   }
0151: }
0152: 
0153: TEST(Q8AVGPOOL_UP8xM__NEON, kc_lt_8_with_y_min) {
0154:   TEST_REQUIRES_ARM_NEON;
0155:   for (size_t n = 1; n <= 3; n += 2) {
0156:     for (size_t kc = 1; kc < 8; kc++) {
0157:       for (size_t ks : std::vector<size_t>{{2, 3, 5}}) {
0158:         AvgPoolMicrokernelTester()
0159:             .kr(8)
0160:             .n(n)
0161:             .kh(ks)
0162:             .kw(ks)
0163:             .kc(kc)
0164:             .xZeroPoint(128)
0165:             .yZeroPoint(128)
0166:             .xScale(1.0f)
0167:             .yScale(1.0f)
0168:             .yMin(128)
0169:             .iterations(3)
0170:             .test(pytorch_q8avgpool_ukernel_up8xm__neon);
0171:       }
0172:     }
0173:   }
0174: }
0175: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `AvgPoolMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`AvgPoolMicrokernelTester`。

### Lines 176-215 / 第 176-215 行

```cpp
0176: TEST(Q8AVGPOOL_UP8xM__NEON, small_n) {
0177:   TEST_REQUIRES_ARM_NEON;
0178:   for (size_t n = 2; n < 5; n++) {
0179:     for (size_t ks : std::vector<size_t>{{2, 3, 5}}) {
0180:       for (size_t kc = 1; kc < 8; kc++) {
0181:         AvgPoolMicrokernelTester()
0182:             .kr(8)
0183:             .n(n)
0184:             .kh(ks)
0185:             .kw(ks)
0186:             .kc(kc)
0187:             .iterations(3)
0188:             .test(pytorch_q8avgpool_ukernel_up8xm__neon);
0189:       }
0190:     }
0191:   }
0192: }
0193: 
0194: TEST(Q8AVGPOOL_UP8xM__NEON, small_n_with_x_stride) {
0195:   TEST_REQUIRES_ARM_NEON;
0196:   for (size_t n = 2; n < 5; n++) {
0197:     for (size_t ks : std::vector<size_t>{{2, 3, 5}}) {
0198:       for (size_t kc = 1; kc < 8; kc++) {
0199:         AvgPoolMicrokernelTester()
0200:             .kr(8)
0201:             .n(n)
0202:             .kh(ks)
0203:             .kw(ks)
0204:             .kc(kc)
0205:             .xStride(11)
0206:             .iterations(3)
0207:             .test(pytorch_q8avgpool_ukernel_up8xm__neon);
0208:       }
0209:     }
0210:   }
0211: }
0212: 
0213: TEST(Q8AVGPOOL_UP8xM__NEON, small_n_with_y_stride) {
0214:   TEST_REQUIRES_ARM_NEON;
0215:   for (size_t n = 2; n < 5; n++) {
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `AvgPoolMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`AvgPoolMicrokernelTester`。

### Lines 216-256 / 第 216-256 行

```cpp
0216:     for (size_t ks : std::vector<size_t>{{2, 3, 5}}) {
0217:       for (size_t kc = 1; kc < 8; kc++) {
0218:         AvgPoolMicrokernelTester()
0219:             .kr(8)
0220:             .n(n)
0221:             .kh(ks)
0222:             .kw(ks)
0223:             .kc(kc)
0224:             .yStride(13)
0225:             .iterations(3)
0226:             .test(pytorch_q8avgpool_ukernel_up8xm__neon);
0227:       }
0228:     }
0229:   }
0230: }
0231: 
0232: TEST(Q8AVGPOOL_UP8xM__NEON, small_n_with_s) {
0233:   TEST_REQUIRES_ARM_NEON;
0234:   for (size_t n = 2; n < 5; n++) {
0235:     for (size_t ks : std::vector<size_t>{{2, 3, 5}}) {
0236:       for (size_t s = 2; s <= 5; s++) {
0237:         for (size_t kc = 1; kc < 8; kc++) {
0238:           AvgPoolMicrokernelTester()
0239:               .kr(8)
0240:               .n(n)
0241:               .kh(ks)
0242:               .kw(ks)
0243:               .kc(kc)
0244:               .s(s)
0245:               .iterations(1)
0246:               .test(pytorch_q8avgpool_ukernel_up8xm__neon);
0247:         }
0248:       }
0249:     }
0250:   }
0251: }
0252: 
0253: TEST(Q8AVGPOOL_UP8x9__NEON, kc_eq_8_fulltile) {
0254:   TEST_REQUIRES_ARM_NEON;
0255:   auto tester = AvgPoolMicrokernelTester().kr(8).mr(9).kc(8);
0256:   for (size_t kh = 1; kh <= tester.mr(); kh++) {
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `AvgPoolMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`AvgPoolMicrokernelTester`。

### Lines 257-296 / 第 257-296 行

```cpp
0257:     for (size_t kw = 1; kw <= tester.mr(); kw++) {
0258:       if (kh * kw == tester.mr()) {
0259:         tester.kh(kh).kw(kw).test(pytorch_q8avgpool_ukernel_up8x9__neon);
0260:       }
0261:     }
0262:   }
0263: }
0264: 
0265: TEST(Q8AVGPOOL_UP8x9__NEON, kc_eq_8_subtile) {
0266:   TEST_REQUIRES_ARM_NEON;
0267:   auto tester = AvgPoolMicrokernelTester().kr(8).mr(9).kc(8);
0268:   for (size_t ks = 2; ks < tester.mr(); ks++) {
0269:     for (size_t kh = 1; kh <= ks; kh++) {
0270:       for (size_t kw = 1; kw <= ks; kw++) {
0271:         if (kh * kw == ks) {
0272:           tester.kh(kh).kw(kw).test(pytorch_q8avgpool_ukernel_up8x9__neon);
0273:         }
0274:       }
0275:     }
0276:   }
0277: }
0278: 
0279: TEST(Q8AVGPOOL_UP8x9__NEON, kc_div_8_fulltile) {
0280:   TEST_REQUIRES_ARM_NEON;
0281:   auto tester = AvgPoolMicrokernelTester().kr(8).mr(9);
0282:   for (size_t kh = 1; kh <= tester.mr(); kh++) {
0283:     for (size_t kw = 1; kw <= tester.mr(); kw++) {
0284:       if (kh * kw == tester.mr()) {
0285:         for (size_t kc = 8; kc < 128; kc += 24) {
0286:           tester.kh(kh).kw(kw).kc(kc).test(pytorch_q8avgpool_ukernel_up8x9__neon);
0287:         }
0288:       }
0289:     }
0290:   }
0291: }
0292: 
0293: TEST(Q8AVGPOOL_UP8x9__NEON, kc_div_8_subtile) {
0294:   TEST_REQUIRES_ARM_NEON;
0295:   auto tester = AvgPoolMicrokernelTester().kr(8).mr(9).iterations(3);
0296:   for (size_t ks = 2; ks < tester.mr(); ks++) {
```

- **EN:** This block handles conditional branches and special cases; iterates over tensors, descriptors, options, or runtime state. Key symbols: no prominent local symbols.
- **CN:** 该代码块处理条件分支与特殊情况；遍历张量、描述符、配置项或运行时状态。关键符号：无明显局部符号。

### Lines 297-337 / 第 297-337 行

```cpp
0297:     for (size_t kh = 1; kh <= ks; kh++) {
0298:       for (size_t kw = 1; kw <= ks; kw++) {
0299:         if (kh * kw == ks) {
0300:           for (size_t kc = 8; kc < 128; kc += 24) {
0301:             tester.kh(kh).kw(kw).kc(kc).test(pytorch_q8avgpool_ukernel_up8x9__neon);
0302:           }
0303:         }
0304:       }
0305:     }
0306:   }
0307: }
0308: 
0309: TEST(Q8AVGPOOL_UP8x9__NEON, kc_div_8_fulltile_with_x_stride) {
0310:   TEST_REQUIRES_ARM_NEON;
0311:   auto tester = AvgPoolMicrokernelTester().kr(8).mr(9).iterations(3);
0312:   for (size_t kh = 1; kh <= tester.mr(); kh++) {
0313:     for (size_t kw = 1; kw <= tester.mr(); kw++) {
0314:       if (kh * kw == tester.mr()) {
0315:         for (size_t kc = 8; kc < 128; kc += 24) {
0316:           tester.kh(kh).kw(kw).kc(kc).xStride(131).test(
0317:               pytorch_q8avgpool_ukernel_up8x9__neon);
0318:         }
0319:       }
0320:     }
0321:   }
0322: }
0323: 
0324: TEST(Q8AVGPOOL_UP8x9__NEON, kc_gt_8_fulltile) {
0325:   TEST_REQUIRES_ARM_NEON;
0326:   auto tester = AvgPoolMicrokernelTester().kr(8).mr(9);
0327:   for (size_t kh = 1; kh <= tester.mr(); kh++) {
0328:     for (size_t kw = 1; kw <= tester.mr(); kw++) {
0329:       if (kh * kw == tester.mr()) {
0330:         for (size_t kc = 9; kc < 16; kc++) {
0331:           tester.kh(kh).kw(kw).kc(kc).test(pytorch_q8avgpool_ukernel_up8x9__neon);
0332:         }
0333:       }
0334:     }
0335:   }
0336: }
0337: 
```

- **EN:** This block handles conditional branches and special cases; iterates over tensors, descriptors, options, or runtime state. Key symbols: no prominent local symbols.
- **CN:** 该代码块处理条件分支与特殊情况；遍历张量、描述符、配置项或运行时状态。关键符号：无明显局部符号。

### Lines 338-388 / 第 338-388 行

```cpp
0338: TEST(Q8AVGPOOL_UP8x9__NEON, kc_gt_8_subtile) {
0339:   TEST_REQUIRES_ARM_NEON;
0340:   auto tester = AvgPoolMicrokernelTester().kr(8).mr(9).iterations(3);
0341:   for (size_t ks = 2; ks < tester.mr(); ks++) {
0342:     for (size_t kh = 1; kh <= ks; kh++) {
0343:       for (size_t kw = 1; kw <= ks; kw++) {
0344:         if (kh * kw == ks) {
0345:           for (size_t kc = 9; kc < 16; kc++) {
0346:             tester.kh(kh).kw(kw).kc(kc).test(pytorch_q8avgpool_ukernel_up8x9__neon);
0347:           }
0348:         }
0349:       }
0350:     }
0351:   }
0352: }
0353: 
0354: TEST(Q8AVGPOOL_UP8x9__NEON, kc_gt_8_fulltile_with_x_stride) {
0355:   TEST_REQUIRES_ARM_NEON;
0356:   auto tester = AvgPoolMicrokernelTester().kr(8).mr(9).iterations(3);
0357:   for (size_t kh = 1; kh <= tester.mr(); kh++) {
0358:     for (size_t kw = 1; kw <= tester.mr(); kw++) {
0359:       if (kh * kw == tester.mr()) {
0360:         for (size_t kc = 9; kc < 16; kc++) {
0361:           tester.kh(kh).kw(kw).kc(kc).xStride(23).test(
0362:               pytorch_q8avgpool_ukernel_up8x9__neon);
0363:         }
0364:       }
0365:     }
0366:   }
0367: }
0368: 
0369: TEST(Q8AVGPOOL_UP8x9__NEON, kc_div_8_with_x_scale) {
0370:   TEST_REQUIRES_ARM_NEON;
0371:   for (size_t n = 1; n <= 5; n += 2) {
0372:     for (size_t kc = 8; kc < 128; kc += 24) {
0373:       for (float xScale = 0.01f; xScale < 100.0f; xScale *= 3.14159265f) {
0374:         AvgPoolMicrokernelTester()
0375:             .kr(8)
0376:             .mr(9)
0377:             .n(n)
0378:             .kh(3)
0379:             .kw(3)
0380:             .kc(kc)
0381:             .xScale(xScale)
0382:             .iterations(2)
0383:             .test(pytorch_q8avgpool_ukernel_up8x9__neon);
0384:       }
0385:     }
0386:   }
0387: }
0388: 
```

- **EN:** This block handles conditional branches and special cases; iterates over tensors, descriptors, options, or runtime state. Key symbols: `AvgPoolMicrokernelTester`.
- **CN:** 该代码块处理条件分支与特殊情况；遍历张量、描述符、配置项或运行时状态。关键符号：`AvgPoolMicrokernelTester`。

### Lines 389-428 / 第 389-428 行

```cpp
0389: TEST(Q8AVGPOOL_UP8x9__NEON, kc_div_8_with_x_zero_point) {
0390:   TEST_REQUIRES_ARM_NEON;
0391:   for (size_t n = 1; n <= 5; n += 2) {
0392:     for (size_t kc = 8; kc < 128; kc += 24) {
0393:       for (int32_t xZeroPoint = 0; xZeroPoint <= 255; xZeroPoint += 51) {
0394:         AvgPoolMicrokernelTester()
0395:             .kr(8)
0396:             .mr(9)
0397:             .n(n)
0398:             .kh(3)
0399:             .kw(3)
0400:             .kc(kc)
0401:             .xZeroPoint(uint8_t(xZeroPoint))
0402:             .iterations(3)
0403:             .test(pytorch_q8avgpool_ukernel_up8x9__neon);
0404:       }
0405:     }
0406:   }
0407: }
0408: 
0409: TEST(Q8AVGPOOL_UP8x9__NEON, kc_div_8_with_y_scale) {
0410:   TEST_REQUIRES_ARM_NEON;
0411:   for (size_t n = 1; n <= 5; n += 2) {
0412:     for (size_t kc = 8; kc < 128; kc += 24) {
0413:       for (float yScale = 0.01f; yScale < 100.0f; yScale *= 3.14159265f) {
0414:         AvgPoolMicrokernelTester()
0415:             .kr(8)
0416:             .mr(9)
0417:             .n(n)
0418:             .kh(3)
0419:             .kw(3)
0420:             .kc(kc)
0421:             .yScale(yScale)
0422:             .iterations(2)
0423:             .test(pytorch_q8avgpool_ukernel_up8x9__neon);
0424:       }
0425:     }
0426:   }
0427: }
0428: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `AvgPoolMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`AvgPoolMicrokernelTester`。

### Lines 429-469 / 第 429-469 行

```cpp
0429: TEST(Q8AVGPOOL_UP8x9__NEON, kc_div_8_with_y_zero_point) {
0430:   TEST_REQUIRES_ARM_NEON;
0431:   for (size_t n = 1; n <= 5; n += 2) {
0432:     for (size_t kc = 8; kc < 128; kc += 24) {
0433:       for (int32_t yZeroPoint = 0; yZeroPoint <= 255; yZeroPoint += 51) {
0434:         AvgPoolMicrokernelTester()
0435:             .kr(8)
0436:             .mr(9)
0437:             .n(n)
0438:             .kh(3)
0439:             .kw(3)
0440:             .kc(kc)
0441:             .yZeroPoint(uint8_t(yZeroPoint))
0442:             .iterations(3)
0443:             .test(pytorch_q8avgpool_ukernel_up8x9__neon);
0444:       }
0445:     }
0446:   }
0447: }
0448: 
0449: TEST(Q8AVGPOOL_UP8x9__NEON, kc_div_8_with_y_max) {
0450:   TEST_REQUIRES_ARM_NEON;
0451:   for (size_t n = 1; n <= 5; n += 2) {
0452:     for (size_t kc = 8; kc < 128; kc += 24) {
0453:       AvgPoolMicrokernelTester()
0454:           .kr(8)
0455:           .mr(9)
0456:           .n(n)
0457:           .kh(3)
0458:           .kw(3)
0459:           .kc(kc)
0460:           .xZeroPoint(128)
0461:           .yZeroPoint(128)
0462:           .xScale(1.0f)
0463:           .yScale(1.0f)
0464:           .yMax(128)
0465:           .test(pytorch_q8avgpool_ukernel_up8x9__neon);
0466:     }
0467:   }
0468: }
0469: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `AvgPoolMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`AvgPoolMicrokernelTester`。

### Lines 470-521 / 第 470-521 行

```cpp
0470: TEST(Q8AVGPOOL_UP8x9__NEON, kc_div_8_with_y_min) {
0471:   TEST_REQUIRES_ARM_NEON;
0472:   for (size_t n = 1; n <= 5; n += 2) {
0473:     for (size_t kc = 8; kc < 128; kc += 24) {
0474:       AvgPoolMicrokernelTester()
0475:           .kr(8)
0476:           .mr(9)
0477:           .n(n)
0478:           .kh(3)
0479:           .kw(3)
0480:           .kc(kc)
0481:           .xZeroPoint(128)
0482:           .yZeroPoint(128)
0483:           .xScale(1.0f)
0484:           .yScale(1.0f)
0485:           .yMin(128)
0486:           .test(pytorch_q8avgpool_ukernel_up8x9__neon);
0487:     }
0488:   }
0489: }
0490: 
0491: TEST(Q8AVGPOOL_UP8x9__NEON, small_n) {
0492:   TEST_REQUIRES_ARM_NEON;
0493:   for (size_t n = 2; n < 5; n++) {
0494:     for (size_t ks : std::vector<size_t>{{2, 3}}) {
0495:       for (size_t kc = 8; kc < 25; kc += 5) {
0496:         AvgPoolMicrokernelTester().kr(8).mr(9).n(n).kh(ks).kw(ks).kc(kc).test(
0497:             pytorch_q8avgpool_ukernel_up8x9__neon);
0498:       }
0499:     }
0500:   }
0501: }
0502: 
0503: TEST(Q8AVGPOOL_UP8x9__NEON, small_n_with_x_stride) {
0504:   TEST_REQUIRES_ARM_NEON;
0505:   for (size_t n = 2; n < 5; n++) {
0506:     for (size_t ks : std::vector<size_t>{{2, 3}}) {
0507:       for (size_t kc = 8; kc < 25; kc += 5) {
0508:         AvgPoolMicrokernelTester()
0509:             .kr(8)
0510:             .mr(9)
0511:             .n(n)
0512:             .kh(ks)
0513:             .kw(ks)
0514:             .kc(kc)
0515:             .xStride(29)
0516:             .test(pytorch_q8avgpool_ukernel_up8x9__neon);
0517:       }
0518:     }
0519:   }
0520: }
0521: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `AvgPoolMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`AvgPoolMicrokernelTester`。

### Lines 522-561 / 第 522-561 行

```cpp
0522: TEST(Q8AVGPOOL_UP8x9__NEON, small_n_with_y_stride) {
0523:   TEST_REQUIRES_ARM_NEON;
0524:   for (size_t n = 2; n < 5; n++) {
0525:     for (size_t ks : std::vector<size_t>{{2, 3}}) {
0526:       for (size_t kc = 8; kc < 25; kc += 5) {
0527:         AvgPoolMicrokernelTester()
0528:             .kr(8)
0529:             .mr(9)
0530:             .n(n)
0531:             .kh(ks)
0532:             .kw(ks)
0533:             .kc(kc)
0534:             .yStride(31)
0535:             .test(pytorch_q8avgpool_ukernel_up8x9__neon);
0536:       }
0537:     }
0538:   }
0539: }
0540: 
0541: TEST(Q8AVGPOOL_UP8x9__NEON, small_n_with_s) {
0542:   TEST_REQUIRES_ARM_NEON;
0543:   for (size_t n = 2; n < 5; n++) {
0544:     for (size_t ks : std::vector<size_t>{{2, 3}}) {
0545:       for (size_t kc = 8; kc < 25; kc += 5) {
0546:         for (size_t s = 2; s <= ks; s++) {
0547:           AvgPoolMicrokernelTester()
0548:               .kr(8)
0549:               .mr(9)
0550:               .n(n)
0551:               .kh(ks)
0552:               .kw(ks)
0553:               .kc(kc)
0554:               .s(s)
0555:               .test(pytorch_q8avgpool_ukernel_up8x9__neon);
0556:         }
0557:       }
0558:     }
0559:   }
0560: }
0561: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `AvgPoolMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`AvgPoolMicrokernelTester`。

### Lines 562-602 / 第 562-602 行

```cpp
0562: TEST(Q8AVGPOOL_MP8x9P8Q__NEON, kc_eq_8_twopass_fulltile) {
0563:   TEST_REQUIRES_ARM_NEON;
0564:   auto tester = AvgPoolMicrokernelTester().kr(8).mr(9).qr(8).kc(8);
0565:   const size_t ks = tester.mr() + tester.qr();
0566:   for (size_t kh = 1; kh <= ks; kh++) {
0567:     for (size_t kw = 1; kw <= ks; kw++) {
0568:       if (kh * kw == ks) {
0569:         tester.kh(kh).kw(kw).test(pytorch_q8avgpool_ukernel_mp8x9p8q__neon);
0570:       }
0571:     }
0572:   }
0573: }
0574: 
0575: TEST(Q8AVGPOOL_MP8x9P8Q__NEON, kc_eq_8_twopass_subtile) {
0576:   TEST_REQUIRES_ARM_NEON;
0577:   auto tester = AvgPoolMicrokernelTester().kr(8).mr(9).qr(8).kc(8);
0578:   for (size_t ks = 10; ks < tester.mr() + tester.qr(); ks++) {
0579:     tester.kh(ks).kw(1).test(pytorch_q8avgpool_ukernel_mp8x9p8q__neon);
0580:     tester.kh(1).kw(ks).test(pytorch_q8avgpool_ukernel_mp8x9p8q__neon);
0581:   }
0582: }
0583: 
0584: TEST(Q8AVGPOOL_MP8x9P8Q__NEON, kc_eq_8_multipass_fulltile) {
0585:   TEST_REQUIRES_ARM_NEON;
0586:   for (size_t ks : std::vector<size_t>{{25, 49}}) {
0587:     auto tester = AvgPoolMicrokernelTester().kr(8).mr(9).qr(8).kc(8);
0588:     for (size_t kh = 1; kh <= ks; kh++) {
0589:       for (size_t kw = 1; kw <= ks; kw++) {
0590:         if (kh * kw == ks) {
0591:           tester.kh(kh).kw(kw).test(pytorch_q8avgpool_ukernel_mp8x9p8q__neon);
0592:         }
0593:       }
0594:     }
0595:   }
0596: }
0597: 
0598: TEST(Q8AVGPOOL_MP8x9P8Q__NEON, kc_eq_8_multipass_subtile) {
0599:   TEST_REQUIRES_ARM_NEON;
0600:   for (size_t ksMax : std::vector<size_t>{{25, 49}}) {
0601:     auto tester = AvgPoolMicrokernelTester().kr(8).mr(9).qr(8).kc(8);
0602:     for (size_t ks = ksMax - tester.qr() + 1; ks < ksMax; ks++) {
```

- **EN:** This block handles conditional branches and special cases; iterates over tensors, descriptors, options, or runtime state. Key symbols: no prominent local symbols.
- **CN:** 该代码块处理条件分支与特殊情况；遍历张量、描述符、配置项或运行时状态。关键符号：无明显局部符号。

### Lines 603-645 / 第 603-645 行

```cpp
0603:       tester.kh(ks).kw(1).test(pytorch_q8avgpool_ukernel_mp8x9p8q__neon);
0604:       tester.kh(1).kw(ks).test(pytorch_q8avgpool_ukernel_mp8x9p8q__neon);
0605:     }
0606:   }
0607: }
0608: 
0609: TEST(Q8AVGPOOL_MP8x9P8Q__NEON, kc_div_8_twopass_fulltile) {
0610:   TEST_REQUIRES_ARM_NEON;
0611:   auto tester = AvgPoolMicrokernelTester().kr(8).mr(9).qr(8).iterations(3);
0612:   const size_t ks = 17;
0613:   for (size_t kc = 8; kc < 128; kc += 24) {
0614:     tester.kc(kc).kh(ks).kw(1).test(pytorch_q8avgpool_ukernel_mp8x9p8q__neon);
0615:     tester.kc(kc).kh(1).kw(ks).test(pytorch_q8avgpool_ukernel_mp8x9p8q__neon);
0616:   }
0617: }
0618: 
0619: TEST(Q8AVGPOOL_MP8x9P8Q__NEON, kc_div_8_twopass_subtile) {
0620:   TEST_REQUIRES_ARM_NEON;
0621:   auto tester = AvgPoolMicrokernelTester().kr(8).mr(9).qr(8).iterations(3);
0622:   for (size_t ks = 10; ks < tester.mr() + tester.qr(); ks++) {
0623:     for (size_t kc = 8; kc < 128; kc += 24) {
0624:       tester.kc(kc).kh(ks).kw(1).test(pytorch_q8avgpool_ukernel_mp8x9p8q__neon);
0625:       tester.kc(kc).kh(1).kw(ks).test(pytorch_q8avgpool_ukernel_mp8x9p8q__neon);
0626:     }
0627:   }
0628: }
0629: 
0630: TEST(Q8AVGPOOL_MP8x9P8Q__NEON, kc_div_8_twopass_fulltile_with_x_stride) {
0631:   TEST_REQUIRES_ARM_NEON;
0632:   auto tester = AvgPoolMicrokernelTester().kr(8).mr(9).qr(8).iterations(3);
0633:   const size_t ks = tester.mr() + tester.qr();
0634:   for (size_t kh = 1; kh <= ks; kh++) {
0635:     for (size_t kw = 1; kw <= ks; kw++) {
0636:       if (kh * kw == ks) {
0637:         for (size_t kc = 8; kc < 128; kc += 24) {
0638:           tester.kh(kh).kw(kw).kc(kc).xStride(131).test(
0639:               pytorch_q8avgpool_ukernel_mp8x9p8q__neon);
0640:         }
0641:       }
0642:     }
0643:   }
0644: }
0645: 
```

- **EN:** This block handles conditional branches and special cases; iterates over tensors, descriptors, options, or runtime state. Key symbols: no prominent local symbols.
- **CN:** 该代码块处理条件分支与特殊情况；遍历张量、描述符、配置项或运行时状态。关键符号：无明显局部符号。

### Lines 646-691 / 第 646-691 行

```cpp
0646: TEST(Q8AVGPOOL_MP8x9P8Q__NEON, kc_div_8_multipass_fulltile) {
0647:   TEST_REQUIRES_ARM_NEON;
0648:   for (size_t ks : std::vector<size_t>{{25, 49}}) {
0649:     auto tester = AvgPoolMicrokernelTester().kr(8).mr(9).qr(8).iterations(3);
0650:     for (size_t kh = 1; kh <= ks; kh++) {
0651:       for (size_t kw = 1; kw <= ks; kw++) {
0652:         if (kh * kw == ks) {
0653:           for (size_t kc = 8; kc < 128; kc += 24) {
0654:             tester.kh(kh).kw(kw).kc(kc).test(pytorch_q8avgpool_ukernel_mp8x9p8q__neon);
0655:           }
0656:         }
0657:       }
0658:     }
0659:   }
0660: }
0661: 
0662: TEST(Q8AVGPOOL_MP8x9P8Q__NEON, kc_div_8_multipass_subtile) {
0663:   TEST_REQUIRES_ARM_NEON;
0664:   for (size_t ksMax : std::vector<size_t>{{25, 49}}) {
0665:     auto tester = AvgPoolMicrokernelTester().kr(8).mr(9).qr(8).iterations(3);
0666:     for (size_t ks = ksMax - tester.qr() + 1; ks < ksMax; ks++) {
0667:       for (size_t kc = 8; kc < 128; kc += 24) {
0668:         tester.kc(kc).kh(ks).kw(1).test(pytorch_q8avgpool_ukernel_mp8x9p8q__neon);
0669:         tester.kc(kc).kh(1).kw(ks).test(pytorch_q8avgpool_ukernel_mp8x9p8q__neon);
0670:       }
0671:     }
0672:   }
0673: }
0674: 
0675: TEST(Q8AVGPOOL_MP8x9P8Q__NEON, kc_div_8_multipass_fulltile_with_x_stride) {
0676:   TEST_REQUIRES_ARM_NEON;
0677:   for (size_t ks : std::vector<size_t>{{25, 49}}) {
0678:     auto tester = AvgPoolMicrokernelTester().kr(8).mr(9).qr(8).iterations(3);
0679:     for (size_t kh = 1; kh <= ks; kh++) {
0680:       for (size_t kw = 1; kw <= ks; kw++) {
0681:         if (kh * kw == ks) {
0682:           for (size_t kc = 8; kc < 128; kc += 24) {
0683:             tester.kh(kh).kw(kw).kc(kc).xStride(131).test(
0684:                 pytorch_q8avgpool_ukernel_mp8x9p8q__neon);
0685:           }
0686:         }
0687:       }
0688:     }
0689:   }
0690: }
0691: 
```

- **EN:** This block handles conditional branches and special cases; iterates over tensors, descriptors, options, or runtime state. Key symbols: no prominent local symbols.
- **CN:** 该代码块处理条件分支与特殊情况；遍历张量、描述符、配置项或运行时状态。关键符号：无明显局部符号。

### Lines 692-733 / 第 692-733 行

```cpp
0692: TEST(Q8AVGPOOL_MP8x9P8Q__NEON, kc_gt_8_twopass_fulltile) {
0693:   TEST_REQUIRES_ARM_NEON;
0694:   auto tester = AvgPoolMicrokernelTester().kr(8).mr(9).qr(8).iterations(3);
0695:   const size_t ks = tester.mr() + tester.qr();
0696:   for (size_t kh = 1; kh <= ks; kh++) {
0697:     for (size_t kw = 1; kw <= ks; kw++) {
0698:       if (kh * kw == ks) {
0699:         for (size_t kc = 9; kc < 16; kc++) {
0700:           tester.kh(kh).kw(kw).kc(kc).test(pytorch_q8avgpool_ukernel_mp8x9p8q__neon);
0701:         }
0702:       }
0703:     }
0704:   }
0705: }
0706: 
0707: TEST(Q8AVGPOOL_MP8x9P8Q__NEON, kc_gt_8_twopass_subtile) {
0708:   TEST_REQUIRES_ARM_NEON;
0709:   auto tester = AvgPoolMicrokernelTester().kr(8).mr(9).qr(8).iterations(3);
0710:   for (size_t ks = 10; ks < tester.mr() + tester.qr(); ks++) {
0711:     for (size_t kc = 9; kc < 16; kc++) {
0712:       tester.kc(kc).kh(ks).kw(1).test(pytorch_q8avgpool_ukernel_mp8x9p8q__neon);
0713:       tester.kc(kc).kh(1).kw(ks).test(pytorch_q8avgpool_ukernel_mp8x9p8q__neon);
0714:     }
0715:   }
0716: }
0717: 
0718: TEST(Q8AVGPOOL_MP8x9P8Q__NEON, kc_gt_8_twopass_fulltile_with_x_stride) {
0719:   TEST_REQUIRES_ARM_NEON;
0720:   auto tester = AvgPoolMicrokernelTester().kr(8).mr(9).qr(8).iterations(3);
0721:   const size_t ks = tester.mr() + tester.qr();
0722:   for (size_t kh = 1; kh <= ks; kh++) {
0723:     for (size_t kw = 1; kw <= ks; kw++) {
0724:       if (kh * kw == ks) {
0725:         for (size_t kc = 9; kc < 16; kc++) {
0726:           tester.kh(kh).kw(kw).kc(kc).xStride(23).test(
0727:               pytorch_q8avgpool_ukernel_mp8x9p8q__neon);
0728:         }
0729:       }
0730:     }
0731:   }
0732: }
0733: 
```

- **EN:** This block handles conditional branches and special cases; iterates over tensors, descriptors, options, or runtime state. Key symbols: no prominent local symbols.
- **CN:** 该代码块处理条件分支与特殊情况；遍历张量、描述符、配置项或运行时状态。关键符号：无明显局部符号。

### Lines 734-779 / 第 734-779 行

```cpp
0734: TEST(Q8AVGPOOL_MP8x9P8Q__NEON, kc_gt_8_multipass_fulltile) {
0735:   TEST_REQUIRES_ARM_NEON;
0736:   for (size_t ks : std::vector<size_t>{{25, 49}}) {
0737:     auto tester = AvgPoolMicrokernelTester().kr(8).mr(9).qr(8).iterations(3);
0738:     for (size_t kh = 1; kh <= ks; kh++) {
0739:       for (size_t kw = 1; kw <= ks; kw++) {
0740:         if (kh * kw == ks) {
0741:           for (size_t kc = 9; kc < 16; kc++) {
0742:             tester.kh(kh).kw(kw).kc(kc).test(pytorch_q8avgpool_ukernel_mp8x9p8q__neon);
0743:           }
0744:         }
0745:       }
0746:     }
0747:   }
0748: }
0749: 
0750: TEST(Q8AVGPOOL_MP8x9P8Q__NEON, kc_gt_8_multipass_subtile) {
0751:   TEST_REQUIRES_ARM_NEON;
0752:   for (size_t ksMax : std::vector<size_t>{{25, 49}}) {
0753:     auto tester = AvgPoolMicrokernelTester().kr(8).mr(9).qr(8).iterations(3);
0754:     for (size_t ks = ksMax - tester.qr() + 1; ks < ksMax; ks++) {
0755:       for (size_t kc = 9; kc < 16; kc++) {
0756:         tester.kc(kc).kh(ks).kw(1).test(pytorch_q8avgpool_ukernel_mp8x9p8q__neon);
0757:         tester.kc(kc).kh(1).kw(ks).test(pytorch_q8avgpool_ukernel_mp8x9p8q__neon);
0758:       }
0759:     }
0760:   }
0761: }
0762: 
0763: TEST(Q8AVGPOOL_MP8x9P8Q__NEON, kc_gt_8_multipass_fulltile_with_x_stride) {
0764:   TEST_REQUIRES_ARM_NEON;
0765:   for (size_t ks : std::vector<size_t>{{25, 49}}) {
0766:     auto tester = AvgPoolMicrokernelTester().kr(8).mr(9).qr(8).iterations(3);
0767:     for (size_t kh = 1; kh <= ks; kh++) {
0768:       for (size_t kw = 1; kw <= ks; kw++) {
0769:         if (kh * kw == ks) {
0770:           for (size_t kc = 9; kc < 16; kc++) {
0771:             tester.kh(kh).kw(kw).kc(kc).xStride(23).test(
0772:                 pytorch_q8avgpool_ukernel_mp8x9p8q__neon);
0773:           }
0774:         }
0775:       }
0776:     }
0777:   }
0778: }
0779: 
```

- **EN:** This block handles conditional branches and special cases; iterates over tensors, descriptors, options, or runtime state. Key symbols: no prominent local symbols.
- **CN:** 该代码块处理条件分支与特殊情况；遍历张量、描述符、配置项或运行时状态。关键符号：无明显局部符号。

### Lines 780-821 / 第 780-821 行

```cpp
0780: TEST(Q8AVGPOOL_MP8x9P8Q__NEON, kc_div_8_with_x_scale) {
0781:   TEST_REQUIRES_ARM_NEON;
0782:   for (size_t n = 1; n <= 5; n += 2) {
0783:     for (size_t kc = 8; kc < 128; kc += 24) {
0784:       for (float xScale = 0.01f; xScale < 100.0f; xScale *= 3.14159265f) {
0785:         AvgPoolMicrokernelTester()
0786:             .kr(8)
0787:             .mr(9)
0788:             .qr(8)
0789:             .n(n)
0790:             .kh(5)
0791:             .kw(5)
0792:             .kc(kc)
0793:             .xScale(xScale)
0794:             .iterations(1)
0795:             .test(pytorch_q8avgpool_ukernel_mp8x9p8q__neon);
0796:       }
0797:     }
0798:   }
0799: }
0800: 
0801: TEST(Q8AVGPOOL_MP8x9P8Q__NEON, kc_div_8_with_x_zero_point) {
0802:   TEST_REQUIRES_ARM_NEON;
0803:   for (size_t n = 1; n <= 5; n += 2) {
0804:     for (size_t kc = 8; kc < 128; kc += 24) {
0805:       for (int32_t xZeroPoint = 0; xZeroPoint <= 255; xZeroPoint += 51) {
0806:         AvgPoolMicrokernelTester()
0807:             .kr(8)
0808:             .mr(9)
0809:             .qr(8)
0810:             .n(n)
0811:             .kh(5)
0812:             .kw(5)
0813:             .kc(kc)
0814:             .xZeroPoint(uint8_t(xZeroPoint))
0815:             .iterations(1)
0816:             .test(pytorch_q8avgpool_ukernel_mp8x9p8q__neon);
0817:       }
0818:     }
0819:   }
0820: }
0821: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `AvgPoolMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`AvgPoolMicrokernelTester`。

### Lines 822-863 / 第 822-863 行

```cpp
0822: TEST(Q8AVGPOOL_MP8x9P8Q__NEON, kc_div_8_with_y_scale) {
0823:   TEST_REQUIRES_ARM_NEON;
0824:   for (size_t n = 1; n <= 5; n += 2) {
0825:     for (size_t kc = 8; kc < 128; kc += 24) {
0826:       for (float yScale = 0.01f; yScale < 100.0f; yScale *= 3.14159265f) {
0827:         AvgPoolMicrokernelTester()
0828:             .kr(8)
0829:             .mr(9)
0830:             .qr(8)
0831:             .n(n)
0832:             .kh(5)
0833:             .kw(5)
0834:             .kc(kc)
0835:             .yScale(yScale)
0836:             .iterations(1)
0837:             .test(pytorch_q8avgpool_ukernel_mp8x9p8q__neon);
0838:       }
0839:     }
0840:   }
0841: }
0842: 
0843: TEST(Q8AVGPOOL_MP8x9P8Q__NEON, kc_div_8_with_y_zero_point) {
0844:   TEST_REQUIRES_ARM_NEON;
0845:   for (size_t n = 1; n <= 5; n += 2) {
0846:     for (size_t kc = 8; kc < 128; kc += 24) {
0847:       for (int32_t yZeroPoint = 0; yZeroPoint <= 255; yZeroPoint += 51) {
0848:         AvgPoolMicrokernelTester()
0849:             .kr(8)
0850:             .mr(9)
0851:             .qr(8)
0852:             .n(n)
0853:             .kh(5)
0854:             .kw(5)
0855:             .kc(kc)
0856:             .yZeroPoint(uint8_t(yZeroPoint))
0857:             .iterations(1)
0858:             .test(pytorch_q8avgpool_ukernel_mp8x9p8q__neon);
0859:       }
0860:     }
0861:   }
0862: }
0863: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `AvgPoolMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`AvgPoolMicrokernelTester`。

### Lines 864-909 / 第 864-909 行

```cpp
0864: TEST(Q8AVGPOOL_MP8x9P8Q__NEON, kc_div_8_with_y_max) {
0865:   TEST_REQUIRES_ARM_NEON;
0866:   for (size_t n = 1; n <= 5; n += 2) {
0867:     for (size_t kc = 8; kc < 128; kc += 24) {
0868:       AvgPoolMicrokernelTester()
0869:           .kr(8)
0870:           .mr(9)
0871:           .qr(8)
0872:           .n(n)
0873:           .kh(5)
0874:           .kw(5)
0875:           .kc(kc)
0876:           .xZeroPoint(128)
0877:           .yZeroPoint(128)
0878:           .xScale(1.0f)
0879:           .yScale(1.0f)
0880:           .yMax(128)
0881:           .iterations(3)
0882:           .test(pytorch_q8avgpool_ukernel_mp8x9p8q__neon);
0883:     }
0884:   }
0885: }
0886: 
0887: TEST(Q8AVGPOOL_MP8x9P8Q__NEON, kc_div_8_with_y_min) {
0888:   TEST_REQUIRES_ARM_NEON;
0889:   for (size_t n = 1; n <= 5; n += 2) {
0890:     for (size_t kc = 8; kc < 128; kc += 24) {
0891:       AvgPoolMicrokernelTester()
0892:           .kr(8)
0893:           .mr(9)
0894:           .qr(8)
0895:           .n(n)
0896:           .kh(5)
0897:           .kw(5)
0898:           .kc(kc)
0899:           .xZeroPoint(128)
0900:           .yZeroPoint(128)
0901:           .xScale(1.0f)
0902:           .yScale(1.0f)
0903:           .yMin(128)
0904:           .iterations(3)
0905:           .test(pytorch_q8avgpool_ukernel_mp8x9p8q__neon);
0906:     }
0907:   }
0908: }
0909: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `AvgPoolMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`AvgPoolMicrokernelTester`。

### Lines 910-951 / 第 910-951 行

```cpp
0910: TEST(Q8AVGPOOL_MP8x9P8Q__NEON, small_n) {
0911:   TEST_REQUIRES_ARM_NEON;
0912:   for (size_t n = 2; n < 5; n++) {
0913:     for (size_t ks : std::vector<size_t>{{5, 7}}) {
0914:       for (size_t kc = 8; kc < 25; kc += 5) {
0915:         AvgPoolMicrokernelTester()
0916:             .kr(8)
0917:             .mr(9)
0918:             .qr(8)
0919:             .n(n)
0920:             .kh(ks)
0921:             .kw(ks)
0922:             .kc(kc)
0923:             .test(pytorch_q8avgpool_ukernel_mp8x9p8q__neon);
0924:       }
0925:     }
0926:   }
0927: }
0928: 
0929: TEST(Q8AVGPOOL_MP8x9P8Q__NEON, small_n_with_x_stride) {
0930:   TEST_REQUIRES_ARM_NEON;
0931:   for (size_t n = 2; n < 5; n++) {
0932:     for (size_t ks : std::vector<size_t>{{5, 7}}) {
0933:       for (size_t kc = 8; kc < 25; kc += 5) {
0934:         AvgPoolMicrokernelTester()
0935:             .kr(8)
0936:             .mr(9)
0937:             .qr(8)
0938:             .n(n)
0939:             .kh(ks)
0940:             .kw(ks)
0941:             .kc(kc)
0942:             .xStride(29)
0943:             .test(pytorch_q8avgpool_ukernel_mp8x9p8q__neon);
0944:       }
0945:     }
0946:   }
0947: }
0948: 
0949: TEST(Q8AVGPOOL_MP8x9P8Q__NEON, small_n_with_y_stride) {
0950:   TEST_REQUIRES_ARM_NEON;
0951:   for (size_t n = 2; n < 5; n++) {
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `AvgPoolMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`AvgPoolMicrokernelTester`。

### Lines 952-991 / 第 952-991 行

```cpp
0952:     for (size_t ks : std::vector<size_t>{{5, 7}}) {
0953:       for (size_t kc = 8; kc < 25; kc += 5) {
0954:         AvgPoolMicrokernelTester()
0955:             .kr(8)
0956:             .mr(9)
0957:             .qr(8)
0958:             .n(n)
0959:             .kh(ks)
0960:             .kw(ks)
0961:             .kc(kc)
0962:             .yStride(31)
0963:             .test(pytorch_q8avgpool_ukernel_mp8x9p8q__neon);
0964:       }
0965:     }
0966:   }
0967: }
0968: 
0969: TEST(Q8AVGPOOL_MP8x9P8Q__NEON, small_n_with_s) {
0970:   TEST_REQUIRES_ARM_NEON;
0971:   for (size_t n = 2; n < 5; n++) {
0972:     for (size_t ks : std::vector<size_t>{{5, 7}}) {
0973:       for (size_t s = 2; s <= 5; s++) {
0974:         for (size_t kc = 8; kc < 25; kc += 5) {
0975:           AvgPoolMicrokernelTester()
0976:               .kr(8)
0977:               .mr(9)
0978:               .qr(8)
0979:               .n(n)
0980:               .kh(ks)
0981:               .kw(ks)
0982:               .kc(kc)
0983:               .s(s)
0984:               .test(pytorch_q8avgpool_ukernel_mp8x9p8q__neon);
0985:         }
0986:       }
0987:     }
0988:   }
0989: }
0990: #endif /* CPUINFO_ARCH_ARM || CPUINFO_ARCH_ARM64 */
0991: 
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends; iterates over tensors, descriptors, options, or runtime state. Key symbols: `AvgPoolMicrokernelTester`.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端；遍历张量、描述符、配置项或运行时状态。关键符号：`AvgPoolMicrokernelTester`。

### Lines 992-1041 / 第 992-1041 行

```cpp
0992: #if CPUINFO_ARCH_X86 || CPUINFO_ARCH_X86_64
0993: TEST(Q8AVGPOOL_UP8xM__SSE2, kc_lt_8_small_ks) {
0994:   TEST_REQUIRES_X86_SSE2;
0995:   for (size_t kc = 1; kc < 8; kc++) {
0996:     for (size_t ks = 1; ks < 8; ks++) {
0997:       for (size_t kh = 1; kh <= ks; kh++) {
0998:         for (size_t kw = 1; kw <= ks; kw++) {
0999:           if (kh * kw == ks) {
1000:             AvgPoolMicrokernelTester().kr(8).kh(kh).kw(kw).kc(kc).test(
1001:                 pytorch_q8avgpool_ukernel_up8xm__sse2);
1002:           }
1003:         }
1004:       }
1005:     }
1006:   }
1007: }
1008: 
1009: TEST(Q8AVGPOOL_UP8xM__SSE2, kc_lt_8_large_ks) {
1010:   TEST_REQUIRES_X86_SSE2;
1011:   for (size_t kc = 1; kc < 8; kc++) {
1012:     for (size_t ks = 8; ks < 16; ks++) {
1013:       AvgPoolMicrokernelTester().kr(8).kh(ks).kw(1).kc(kc).test(
1014:           pytorch_q8avgpool_ukernel_up8xm__sse2);
1015:       AvgPoolMicrokernelTester().kr(8).kh(1).kw(ks).kc(kc).test(
1016:           pytorch_q8avgpool_ukernel_up8xm__sse2);
1017:     }
1018:   }
1019: }
1020: 
1021: TEST(Q8AVGPOOL_UP8xM__SSE2, kc_lt_8_with_x_scale) {
1022:   TEST_REQUIRES_X86_SSE2;
1023:   for (size_t n = 1; n <= 3; n += 2) {
1024:     for (size_t kc = 1; kc < 8; kc++) {
1025:       for (size_t ks : std::vector<size_t>{{2, 3, 5}}) {
1026:         for (float xScale = 0.01f; xScale < 100.0f; xScale *= 3.14159265f) {
1027:           AvgPoolMicrokernelTester()
1028:               .kr(8)
1029:               .n(n)
1030:               .kh(ks)
1031:               .kw(ks)
1032:               .kc(kc)
1033:               .xScale(xScale)
1034:               .iterations(1)
1035:               .test(pytorch_q8avgpool_ukernel_up8xm__sse2);
1036:         }
1037:       }
1038:     }
1039:   }
1040: }
1041: 
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends; handles conditional branches and special cases; iterates over tensors, descriptors, options, or runtime state. Key symbols: `AvgPoolMicrokernelTester`.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端；处理条件分支与特殊情况；遍历张量、描述符、配置项或运行时状态。关键符号：`AvgPoolMicrokernelTester`。

### Lines 1042-1083 / 第 1042-1083 行

```cpp
1042: TEST(Q8AVGPOOL_UP8xM__SSE2, kc_lt_8_with_x_zero_point) {
1043:   TEST_REQUIRES_X86_SSE2;
1044:   for (size_t n = 1; n <= 3; n += 2) {
1045:     for (size_t kc = 1; kc < 8; kc++) {
1046:       for (size_t ks : std::vector<size_t>{{2, 3, 5}}) {
1047:         for (int32_t xZeroPoint = 0; xZeroPoint <= 255; xZeroPoint += 51) {
1048:           AvgPoolMicrokernelTester()
1049:               .kr(8)
1050:               .n(n)
1051:               .kh(ks)
1052:               .kw(ks)
1053:               .kc(kc)
1054:               .xZeroPoint(uint8_t(xZeroPoint))
1055:               .iterations(1)
1056:               .test(pytorch_q8avgpool_ukernel_up8xm__sse2);
1057:         }
1058:       }
1059:     }
1060:   }
1061: }
1062: 
1063: TEST(Q8AVGPOOL_UP8xM__SSE2, kc_lt_8_with_y_scale) {
1064:   TEST_REQUIRES_X86_SSE2;
1065:   for (size_t n = 1; n <= 3; n += 2) {
1066:     for (size_t kc = 1; kc < 8; kc++) {
1067:       for (size_t ks : std::vector<size_t>{{2, 3, 5}}) {
1068:         for (float yScale = 0.01f; yScale < 100.0f; yScale *= 3.14159265f) {
1069:           AvgPoolMicrokernelTester()
1070:               .kr(8)
1071:               .n(n)
1072:               .kh(ks)
1073:               .kw(ks)
1074:               .kc(kc)
1075:               .yScale(yScale)
1076:               .iterations(1)
1077:               .test(pytorch_q8avgpool_ukernel_up8xm__sse2);
1078:         }
1079:       }
1080:     }
1081:   }
1082: }
1083: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `AvgPoolMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`AvgPoolMicrokernelTester`。

### Lines 1084-1127 / 第 1084-1127 行

```cpp
1084: TEST(Q8AVGPOOL_UP8xM__SSE2, kc_lt_8_with_y_zero_point) {
1085:   TEST_REQUIRES_X86_SSE2;
1086:   for (size_t n = 1; n <= 3; n += 2) {
1087:     for (size_t kc = 1; kc < 8; kc++) {
1088:       for (size_t ks : std::vector<size_t>{{2, 3, 5}}) {
1089:         for (int32_t yZeroPoint = 0; yZeroPoint <= 255; yZeroPoint += 51) {
1090:           AvgPoolMicrokernelTester()
1091:               .kr(8)
1092:               .n(n)
1093:               .kh(ks)
1094:               .kw(ks)
1095:               .kc(kc)
1096:               .yZeroPoint(uint8_t(yZeroPoint))
1097:               .iterations(1)
1098:               .test(pytorch_q8avgpool_ukernel_up8xm__sse2);
1099:         }
1100:       }
1101:     }
1102:   }
1103: }
1104: 
1105: TEST(Q8AVGPOOL_UP8xM__SSE2, kc_lt_8_with_y_max) {
1106:   TEST_REQUIRES_X86_SSE2;
1107:   for (size_t n = 1; n <= 3; n += 2) {
1108:     for (size_t kc = 1; kc < 8; kc++) {
1109:       for (size_t ks : std::vector<size_t>{{2, 3, 5}}) {
1110:         AvgPoolMicrokernelTester()
1111:             .kr(8)
1112:             .n(n)
1113:             .kh(ks)
1114:             .kw(ks)
1115:             .kc(kc)
1116:             .xZeroPoint(128)
1117:             .yZeroPoint(128)
1118:             .xScale(1.0f)
1119:             .yScale(1.0f)
1120:             .yMax(128)
1121:             .iterations(3)
1122:             .test(pytorch_q8avgpool_ukernel_up8xm__sse2);
1123:       }
1124:     }
1125:   }
1126: }
1127: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `AvgPoolMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`AvgPoolMicrokernelTester`。

### Lines 1128-1168 / 第 1128-1168 行

```cpp
1128: TEST(Q8AVGPOOL_UP8xM__SSE2, kc_lt_8_with_y_min) {
1129:   TEST_REQUIRES_X86_SSE2;
1130:   for (size_t n = 1; n <= 3; n += 2) {
1131:     for (size_t kc = 1; kc < 8; kc++) {
1132:       for (size_t ks : std::vector<size_t>{{2, 3, 5}}) {
1133:         AvgPoolMicrokernelTester()
1134:             .kr(8)
1135:             .n(n)
1136:             .kh(ks)
1137:             .kw(ks)
1138:             .kc(kc)
1139:             .xZeroPoint(128)
1140:             .yZeroPoint(128)
1141:             .xScale(1.0f)
1142:             .yScale(1.0f)
1143:             .yMin(128)
1144:             .iterations(3)
1145:             .test(pytorch_q8avgpool_ukernel_up8xm__sse2);
1146:       }
1147:     }
1148:   }
1149: }
1150: 
1151: TEST(Q8AVGPOOL_UP8xM__SSE2, small_n) {
1152:   TEST_REQUIRES_X86_SSE2;
1153:   for (size_t n = 2; n < 5; n++) {
1154:     for (size_t ks : std::vector<size_t>{{2, 3, 5}}) {
1155:       for (size_t kc = 1; kc < 8; kc++) {
1156:         AvgPoolMicrokernelTester()
1157:             .kr(8)
1158:             .n(n)
1159:             .kh(ks)
1160:             .kw(ks)
1161:             .kc(kc)
1162:             .iterations(3)
1163:             .test(pytorch_q8avgpool_ukernel_up8xm__sse2);
1164:       }
1165:     }
1166:   }
1167: }
1168: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `AvgPoolMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`AvgPoolMicrokernelTester`。

### Lines 1169-1209 / 第 1169-1209 行

```cpp
1169: TEST(Q8AVGPOOL_UP8xM__SSE2, small_n_with_x_stride) {
1170:   TEST_REQUIRES_X86_SSE2;
1171:   for (size_t n = 2; n < 5; n++) {
1172:     for (size_t ks : std::vector<size_t>{{2, 3, 5}}) {
1173:       for (size_t kc = 1; kc < 8; kc++) {
1174:         AvgPoolMicrokernelTester()
1175:             .kr(8)
1176:             .n(n)
1177:             .kh(ks)
1178:             .kw(ks)
1179:             .kc(kc)
1180:             .xStride(11)
1181:             .iterations(3)
1182:             .test(pytorch_q8avgpool_ukernel_up8xm__sse2);
1183:       }
1184:     }
1185:   }
1186: }
1187: 
1188: TEST(Q8AVGPOOL_UP8xM__SSE2, small_n_with_y_stride) {
1189:   TEST_REQUIRES_X86_SSE2;
1190:   for (size_t n = 2; n < 5; n++) {
1191:     for (size_t ks : std::vector<size_t>{{2, 3, 5}}) {
1192:       for (size_t kc = 1; kc < 8; kc++) {
1193:         AvgPoolMicrokernelTester()
1194:             .kr(8)
1195:             .n(n)
1196:             .kh(ks)
1197:             .kw(ks)
1198:             .kc(kc)
1199:             .yStride(13)
1200:             .iterations(3)
1201:             .test(pytorch_q8avgpool_ukernel_up8xm__sse2);
1202:       }
1203:     }
1204:   }
1205: }
1206: 
1207: TEST(Q8AVGPOOL_UP8xM__SSE2, small_n_with_s) {
1208:   TEST_REQUIRES_X86_SSE2;
1209:   for (size_t n = 2; n < 5; n++) {
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `AvgPoolMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`AvgPoolMicrokernelTester`。

### Lines 1210-1253 / 第 1210-1253 行

```cpp
1210:     for (size_t ks : std::vector<size_t>{{2, 3, 5}}) {
1211:       for (size_t s = 2; s <= 5; s++) {
1212:         for (size_t kc = 1; kc < 8; kc++) {
1213:           AvgPoolMicrokernelTester()
1214:               .kr(8)
1215:               .n(n)
1216:               .kh(ks)
1217:               .kw(ks)
1218:               .kc(kc)
1219:               .s(s)
1220:               .iterations(1)
1221:               .test(pytorch_q8avgpool_ukernel_up8xm__sse2);
1222:         }
1223:       }
1224:     }
1225:   }
1226: }
1227: 
1228: TEST(Q8AVGPOOL_UP8x9__SSE2, kc_eq_8_fulltile) {
1229:   TEST_REQUIRES_X86_SSE2;
1230:   auto tester = AvgPoolMicrokernelTester().kr(8).mr(9).kc(8);
1231:   for (size_t kh = 1; kh <= tester.mr(); kh++) {
1232:     for (size_t kw = 1; kw <= tester.mr(); kw++) {
1233:       if (kh * kw == tester.mr()) {
1234:         tester.kh(kh).kw(kw).test(pytorch_q8avgpool_ukernel_up8x9__sse2);
1235:       }
1236:     }
1237:   }
1238: }
1239: 
1240: TEST(Q8AVGPOOL_UP8x9__SSE2, kc_eq_8_subtile) {
1241:   TEST_REQUIRES_X86_SSE2;
1242:   auto tester = AvgPoolMicrokernelTester().kr(8).mr(9).kc(8);
1243:   for (size_t ks = 2; ks < tester.mr(); ks++) {
1244:     for (size_t kh = 1; kh <= ks; kh++) {
1245:       for (size_t kw = 1; kw <= ks; kw++) {
1246:         if (kh * kw == ks) {
1247:           tester.kh(kh).kw(kw).test(pytorch_q8avgpool_ukernel_up8x9__sse2);
1248:         }
1249:       }
1250:     }
1251:   }
1252: }
1253: 
```

- **EN:** This block handles conditional branches and special cases; iterates over tensors, descriptors, options, or runtime state. Key symbols: `AvgPoolMicrokernelTester`.
- **CN:** 该代码块处理条件分支与特殊情况；遍历张量、描述符、配置项或运行时状态。关键符号：`AvgPoolMicrokernelTester`。

### Lines 1254-1298 / 第 1254-1298 行

```cpp
1254: TEST(Q8AVGPOOL_UP8x9__SSE2, kc_div_8_fulltile) {
1255:   TEST_REQUIRES_X86_SSE2;
1256:   auto tester = AvgPoolMicrokernelTester().kr(8).mr(9);
1257:   for (size_t kh = 1; kh <= tester.mr(); kh++) {
1258:     for (size_t kw = 1; kw <= tester.mr(); kw++) {
1259:       if (kh * kw == tester.mr()) {
1260:         for (size_t kc = 8; kc < 128; kc += 24) {
1261:           tester.kh(kh).kw(kw).kc(kc).test(pytorch_q8avgpool_ukernel_up8x9__sse2);
1262:         }
1263:       }
1264:     }
1265:   }
1266: }
1267: 
1268: TEST(Q8AVGPOOL_UP8x9__SSE2, kc_div_8_subtile) {
1269:   TEST_REQUIRES_X86_SSE2;
1270:   auto tester = AvgPoolMicrokernelTester().kr(8).mr(9).iterations(3);
1271:   for (size_t ks = 2; ks < tester.mr(); ks++) {
1272:     for (size_t kh = 1; kh <= ks; kh++) {
1273:       for (size_t kw = 1; kw <= ks; kw++) {
1274:         if (kh * kw == ks) {
1275:           for (size_t kc = 8; kc < 128; kc += 24) {
1276:             tester.kh(kh).kw(kw).kc(kc).test(pytorch_q8avgpool_ukernel_up8x9__sse2);
1277:           }
1278:         }
1279:       }
1280:     }
1281:   }
1282: }
1283: 
1284: TEST(Q8AVGPOOL_UP8x9__SSE2, kc_div_8_fulltile_with_x_stride) {
1285:   TEST_REQUIRES_X86_SSE2;
1286:   auto tester = AvgPoolMicrokernelTester().kr(8).mr(9).iterations(3);
1287:   for (size_t kh = 1; kh <= tester.mr(); kh++) {
1288:     for (size_t kw = 1; kw <= tester.mr(); kw++) {
1289:       if (kh * kw == tester.mr()) {
1290:         for (size_t kc = 8; kc < 128; kc += 24) {
1291:           tester.kh(kh).kw(kw).kc(kc).xStride(131).test(
1292:               pytorch_q8avgpool_ukernel_up8x9__sse2);
1293:         }
1294:       }
1295:     }
1296:   }
1297: }
1298: 
```

- **EN:** This block handles conditional branches and special cases; iterates over tensors, descriptors, options, or runtime state. Key symbols: no prominent local symbols.
- **CN:** 该代码块处理条件分支与特殊情况；遍历张量、描述符、配置项或运行时状态。关键符号：无明显局部符号。

### Lines 1299-1343 / 第 1299-1343 行

```cpp
1299: TEST(Q8AVGPOOL_UP8x9__SSE2, kc_gt_8_fulltile) {
1300:   TEST_REQUIRES_X86_SSE2;
1301:   auto tester = AvgPoolMicrokernelTester().kr(8).mr(9);
1302:   for (size_t kh = 1; kh <= tester.mr(); kh++) {
1303:     for (size_t kw = 1; kw <= tester.mr(); kw++) {
1304:       if (kh * kw == tester.mr()) {
1305:         for (size_t kc = 9; kc < 16; kc++) {
1306:           tester.kh(kh).kw(kw).kc(kc).test(pytorch_q8avgpool_ukernel_up8x9__sse2);
1307:         }
1308:       }
1309:     }
1310:   }
1311: }
1312: 
1313: TEST(Q8AVGPOOL_UP8x9__SSE2, kc_gt_8_subtile) {
1314:   TEST_REQUIRES_X86_SSE2;
1315:   auto tester = AvgPoolMicrokernelTester().kr(8).mr(9).iterations(3);
1316:   for (size_t ks = 2; ks < tester.mr(); ks++) {
1317:     for (size_t kh = 1; kh <= ks; kh++) {
1318:       for (size_t kw = 1; kw <= ks; kw++) {
1319:         if (kh * kw == ks) {
1320:           for (size_t kc = 9; kc < 16; kc++) {
1321:             tester.kh(kh).kw(kw).kc(kc).test(pytorch_q8avgpool_ukernel_up8x9__sse2);
1322:           }
1323:         }
1324:       }
1325:     }
1326:   }
1327: }
1328: 
1329: TEST(Q8AVGPOOL_UP8x9__SSE2, kc_gt_8_fulltile_with_x_stride) {
1330:   TEST_REQUIRES_X86_SSE2;
1331:   auto tester = AvgPoolMicrokernelTester().kr(8).mr(9).iterations(3);
1332:   for (size_t kh = 1; kh <= tester.mr(); kh++) {
1333:     for (size_t kw = 1; kw <= tester.mr(); kw++) {
1334:       if (kh * kw == tester.mr()) {
1335:         for (size_t kc = 9; kc < 16; kc++) {
1336:           tester.kh(kh).kw(kw).kc(kc).xStride(23).test(
1337:               pytorch_q8avgpool_ukernel_up8x9__sse2);
1338:         }
1339:       }
1340:     }
1341:   }
1342: }
1343: 
```

- **EN:** This block handles conditional branches and special cases; iterates over tensors, descriptors, options, or runtime state. Key symbols: no prominent local symbols.
- **CN:** 该代码块处理条件分支与特殊情况；遍历张量、描述符、配置项或运行时状态。关键符号：无明显局部符号。

### Lines 1344-1383 / 第 1344-1383 行

```cpp
1344: TEST(Q8AVGPOOL_UP8x9__SSE2, kc_div_8_with_x_scale) {
1345:   TEST_REQUIRES_X86_SSE2;
1346:   for (size_t n = 1; n <= 5; n += 2) {
1347:     for (size_t kc = 8; kc < 128; kc += 24) {
1348:       for (float xScale = 0.01f; xScale < 100.0f; xScale *= 3.14159265f) {
1349:         AvgPoolMicrokernelTester()
1350:             .kr(8)
1351:             .mr(9)
1352:             .n(n)
1353:             .kh(3)
1354:             .kw(3)
1355:             .kc(kc)
1356:             .xScale(xScale)
1357:             .iterations(2)
1358:             .test(pytorch_q8avgpool_ukernel_up8x9__sse2);
1359:       }
1360:     }
1361:   }
1362: }
1363: 
1364: TEST(Q8AVGPOOL_UP8x9__SSE2, kc_div_8_with_x_zero_point) {
1365:   TEST_REQUIRES_X86_SSE2;
1366:   for (size_t n = 1; n <= 5; n += 2) {
1367:     for (size_t kc = 8; kc < 128; kc += 24) {
1368:       for (int32_t xZeroPoint = 0; xZeroPoint <= 255; xZeroPoint += 51) {
1369:         AvgPoolMicrokernelTester()
1370:             .kr(8)
1371:             .mr(9)
1372:             .n(n)
1373:             .kh(3)
1374:             .kw(3)
1375:             .kc(kc)
1376:             .xZeroPoint(uint8_t(xZeroPoint))
1377:             .iterations(3)
1378:             .test(pytorch_q8avgpool_ukernel_up8x9__sse2);
1379:       }
1380:     }
1381:   }
1382: }
1383: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `AvgPoolMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`AvgPoolMicrokernelTester`。

### Lines 1384-1423 / 第 1384-1423 行

```cpp
1384: TEST(Q8AVGPOOL_UP8x9__SSE2, kc_div_8_with_y_scale) {
1385:   TEST_REQUIRES_X86_SSE2;
1386:   for (size_t n = 1; n <= 5; n += 2) {
1387:     for (size_t kc = 8; kc < 128; kc += 24) {
1388:       for (float yScale = 0.01f; yScale < 100.0f; yScale *= 3.14159265f) {
1389:         AvgPoolMicrokernelTester()
1390:             .kr(8)
1391:             .mr(9)
1392:             .n(n)
1393:             .kh(3)
1394:             .kw(3)
1395:             .kc(kc)
1396:             .yScale(yScale)
1397:             .iterations(2)
1398:             .test(pytorch_q8avgpool_ukernel_up8x9__sse2);
1399:       }
1400:     }
1401:   }
1402: }
1403: 
1404: TEST(Q8AVGPOOL_UP8x9__SSE2, kc_div_8_with_y_zero_point) {
1405:   TEST_REQUIRES_X86_SSE2;
1406:   for (size_t n = 1; n <= 5; n += 2) {
1407:     for (size_t kc = 8; kc < 128; kc += 24) {
1408:       for (int32_t yZeroPoint = 0; yZeroPoint <= 255; yZeroPoint += 51) {
1409:         AvgPoolMicrokernelTester()
1410:             .kr(8)
1411:             .mr(9)
1412:             .n(n)
1413:             .kh(3)
1414:             .kw(3)
1415:             .kc(kc)
1416:             .yZeroPoint(uint8_t(yZeroPoint))
1417:             .iterations(3)
1418:             .test(pytorch_q8avgpool_ukernel_up8x9__sse2);
1419:       }
1420:     }
1421:   }
1422: }
1423: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `AvgPoolMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`AvgPoolMicrokernelTester`。

### Lines 1424-1465 / 第 1424-1465 行

```cpp
1424: TEST(Q8AVGPOOL_UP8x9__SSE2, kc_div_8_with_y_max) {
1425:   TEST_REQUIRES_X86_SSE2;
1426:   for (size_t n = 1; n <= 5; n += 2) {
1427:     for (size_t kc = 8; kc < 128; kc += 24) {
1428:       AvgPoolMicrokernelTester()
1429:           .kr(8)
1430:           .mr(9)
1431:           .n(n)
1432:           .kh(3)
1433:           .kw(3)
1434:           .kc(kc)
1435:           .xZeroPoint(128)
1436:           .yZeroPoint(128)
1437:           .xScale(1.0f)
1438:           .yScale(1.0f)
1439:           .yMax(128)
1440:           .test(pytorch_q8avgpool_ukernel_up8x9__sse2);
1441:     }
1442:   }
1443: }
1444: 
1445: TEST(Q8AVGPOOL_UP8x9__SSE2, kc_div_8_with_y_min) {
1446:   TEST_REQUIRES_X86_SSE2;
1447:   for (size_t n = 1; n <= 5; n += 2) {
1448:     for (size_t kc = 8; kc < 128; kc += 24) {
1449:       AvgPoolMicrokernelTester()
1450:           .kr(8)
1451:           .mr(9)
1452:           .n(n)
1453:           .kh(3)
1454:           .kw(3)
1455:           .kc(kc)
1456:           .xZeroPoint(128)
1457:           .yZeroPoint(128)
1458:           .xScale(1.0f)
1459:           .yScale(1.0f)
1460:           .yMin(128)
1461:           .test(pytorch_q8avgpool_ukernel_up8x9__sse2);
1462:     }
1463:   }
1464: }
1465: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `AvgPoolMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`AvgPoolMicrokernelTester`。

### Lines 1466-1515 / 第 1466-1515 行

```cpp
1466: TEST(Q8AVGPOOL_UP8x9__SSE2, small_n) {
1467:   TEST_REQUIRES_X86_SSE2;
1468:   for (size_t n = 2; n < 5; n++) {
1469:     for (size_t ks : std::vector<size_t>{{2, 3}}) {
1470:       for (size_t kc = 8; kc < 25; kc += 5) {
1471:         AvgPoolMicrokernelTester().kr(8).mr(9).n(n).kh(ks).kw(ks).kc(kc).test(
1472:             pytorch_q8avgpool_ukernel_up8x9__sse2);
1473:       }
1474:     }
1475:   }
1476: }
1477: 
1478: TEST(Q8AVGPOOL_UP8x9__SSE2, small_n_with_x_stride) {
1479:   TEST_REQUIRES_X86_SSE2;
1480:   for (size_t n = 2; n < 5; n++) {
1481:     for (size_t ks : std::vector<size_t>{{2, 3}}) {
1482:       for (size_t kc = 8; kc < 25; kc += 5) {
1483:         AvgPoolMicrokernelTester()
1484:             .kr(8)
1485:             .mr(9)
1486:             .n(n)
1487:             .kh(ks)
1488:             .kw(ks)
1489:             .kc(kc)
1490:             .xStride(29)
1491:             .test(pytorch_q8avgpool_ukernel_up8x9__sse2);
1492:       }
1493:     }
1494:   }
1495: }
1496: 
1497: TEST(Q8AVGPOOL_UP8x9__SSE2, small_n_with_y_stride) {
1498:   TEST_REQUIRES_X86_SSE2;
1499:   for (size_t n = 2; n < 5; n++) {
1500:     for (size_t ks : std::vector<size_t>{{2, 3}}) {
1501:       for (size_t kc = 8; kc < 25; kc += 5) {
1502:         AvgPoolMicrokernelTester()
1503:             .kr(8)
1504:             .mr(9)
1505:             .n(n)
1506:             .kh(ks)
1507:             .kw(ks)
1508:             .kc(kc)
1509:             .yStride(31)
1510:             .test(pytorch_q8avgpool_ukernel_up8x9__sse2);
1511:       }
1512:     }
1513:   }
1514: }
1515: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `AvgPoolMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`AvgPoolMicrokernelTester`。

### Lines 1516-1558 / 第 1516-1558 行

```cpp
1516: TEST(Q8AVGPOOL_UP8x9__SSE2, small_n_with_s) {
1517:   TEST_REQUIRES_X86_SSE2;
1518:   for (size_t n = 2; n < 5; n++) {
1519:     for (size_t ks : std::vector<size_t>{{2, 3}}) {
1520:       for (size_t kc = 8; kc < 25; kc += 5) {
1521:         for (size_t s = 2; s <= ks; s++) {
1522:           AvgPoolMicrokernelTester()
1523:               .kr(8)
1524:               .mr(9)
1525:               .n(n)
1526:               .kh(ks)
1527:               .kw(ks)
1528:               .kc(kc)
1529:               .s(s)
1530:               .test(pytorch_q8avgpool_ukernel_up8x9__sse2);
1531:         }
1532:       }
1533:     }
1534:   }
1535: }
1536: 
1537: TEST(Q8AVGPOOL_MP8x9P8Q__SSE2, kc_eq_8_twopass_fulltile) {
1538:   TEST_REQUIRES_X86_SSE2;
1539:   auto tester = AvgPoolMicrokernelTester().kr(8).mr(9).qr(8).kc(8);
1540:   const size_t ks = tester.mr() + tester.qr();
1541:   for (size_t kh = 1; kh <= ks; kh++) {
1542:     for (size_t kw = 1; kw <= ks; kw++) {
1543:       if (kh * kw == ks) {
1544:         tester.kh(kh).kw(kw).test(pytorch_q8avgpool_ukernel_mp8x9p8q__sse2);
1545:       }
1546:     }
1547:   }
1548: }
1549: 
1550: TEST(Q8AVGPOOL_MP8x9P8Q__SSE2, kc_eq_8_twopass_subtile) {
1551:   TEST_REQUIRES_X86_SSE2;
1552:   auto tester = AvgPoolMicrokernelTester().kr(8).mr(9).qr(8).kc(8);
1553:   for (size_t ks = 10; ks < tester.mr() + tester.qr(); ks++) {
1554:     tester.kh(ks).kw(1).test(pytorch_q8avgpool_ukernel_mp8x9p8q__sse2);
1555:     tester.kh(1).kw(ks).test(pytorch_q8avgpool_ukernel_mp8x9p8q__sse2);
1556:   }
1557: }
1558: 
```

- **EN:** This block handles conditional branches and special cases; iterates over tensors, descriptors, options, or runtime state. Key symbols: `AvgPoolMicrokernelTester`.
- **CN:** 该代码块处理条件分支与特殊情况；遍历张量、描述符、配置项或运行时状态。关键符号：`AvgPoolMicrokernelTester`。

### Lines 1559-1598 / 第 1559-1598 行

```cpp
1559: TEST(Q8AVGPOOL_MP8x9P8Q__SSE2, kc_eq_8_multipass_fulltile) {
1560:   TEST_REQUIRES_X86_SSE2;
1561:   for (size_t ks : std::vector<size_t>{{25, 49}}) {
1562:     auto tester = AvgPoolMicrokernelTester().kr(8).mr(9).qr(8).kc(8);
1563:     for (size_t kh = 1; kh <= ks; kh++) {
1564:       for (size_t kw = 1; kw <= ks; kw++) {
1565:         if (kh * kw == ks) {
1566:           tester.kh(kh).kw(kw).test(pytorch_q8avgpool_ukernel_mp8x9p8q__sse2);
1567:         }
1568:       }
1569:     }
1570:   }
1571: }
1572: 
1573: TEST(Q8AVGPOOL_MP8x9P8Q__SSE2, kc_eq_8_multipass_subtile) {
1574:   TEST_REQUIRES_X86_SSE2;
1575:   for (size_t ksMax : std::vector<size_t>{{25, 49}}) {
1576:     auto tester = AvgPoolMicrokernelTester().kr(8).mr(9).qr(8).kc(8);
1577:     for (size_t ks = ksMax - tester.qr() + 1; ks < ksMax; ks++) {
1578:       tester.kh(ks).kw(1).test(pytorch_q8avgpool_ukernel_mp8x9p8q__sse2);
1579:       tester.kh(1).kw(ks).test(pytorch_q8avgpool_ukernel_mp8x9p8q__sse2);
1580:     }
1581:   }
1582: }
1583: 
1584: TEST(Q8AVGPOOL_MP8x9P8Q__SSE2, kc_div_8_twopass_fulltile) {
1585:   TEST_REQUIRES_X86_SSE2;
1586:   auto tester = AvgPoolMicrokernelTester().kr(8).mr(9).qr(8).iterations(3);
1587:   const size_t ks = 17;
1588:   for (size_t kc = 8; kc < 128; kc += 24) {
1589:     tester.kc(kc).kh(ks).kw(1).test(pytorch_q8avgpool_ukernel_mp8x9p8q__sse2);
1590:     tester.kc(kc).kh(1).kw(ks).test(pytorch_q8avgpool_ukernel_mp8x9p8q__sse2);
1591:   }
1592: }
1593: 
1594: TEST(Q8AVGPOOL_MP8x9P8Q__SSE2, kc_div_8_twopass_subtile) {
1595:   TEST_REQUIRES_X86_SSE2;
1596:   auto tester = AvgPoolMicrokernelTester().kr(8).mr(9).qr(8).iterations(3);
1597:   for (size_t ks = 10; ks < tester.mr() + tester.qr(); ks++) {
1598:     for (size_t kc = 8; kc < 128; kc += 24) {
```

- **EN:** This block handles conditional branches and special cases; iterates over tensors, descriptors, options, or runtime state. Key symbols: no prominent local symbols.
- **CN:** 该代码块处理条件分支与特殊情况；遍历张量、描述符、配置项或运行时状态。关键符号：无明显局部符号。

### Lines 1599-1639 / 第 1599-1639 行

```cpp
1599:       tester.kc(kc).kh(ks).kw(1).test(pytorch_q8avgpool_ukernel_mp8x9p8q__sse2);
1600:       tester.kc(kc).kh(1).kw(ks).test(pytorch_q8avgpool_ukernel_mp8x9p8q__sse2);
1601:     }
1602:   }
1603: }
1604: 
1605: TEST(Q8AVGPOOL_MP8x9P8Q__SSE2, kc_div_8_twopass_fulltile_with_x_stride) {
1606:   TEST_REQUIRES_X86_SSE2;
1607:   auto tester = AvgPoolMicrokernelTester().kr(8).mr(9).qr(8).iterations(3);
1608:   const size_t ks = tester.mr() + tester.qr();
1609:   for (size_t kh = 1; kh <= ks; kh++) {
1610:     for (size_t kw = 1; kw <= ks; kw++) {
1611:       if (kh * kw == ks) {
1612:         for (size_t kc = 8; kc < 128; kc += 24) {
1613:           tester.kh(kh).kw(kw).kc(kc).xStride(131).test(
1614:               pytorch_q8avgpool_ukernel_mp8x9p8q__sse2);
1615:         }
1616:       }
1617:     }
1618:   }
1619: }
1620: 
1621: TEST(Q8AVGPOOL_MP8x9P8Q__SSE2, kc_div_8_multipass_fulltile) {
1622:   TEST_REQUIRES_X86_SSE2;
1623:   for (size_t ks : std::vector<size_t>{{25, 49}}) {
1624:     auto tester = AvgPoolMicrokernelTester().kr(8).mr(9).qr(8).iterations(3);
1625:     for (size_t kh = 1; kh <= ks; kh++) {
1626:       for (size_t kw = 1; kw <= ks; kw++) {
1627:         if (kh * kw == ks) {
1628:           for (size_t kc = 8; kc < 128; kc += 24) {
1629:             tester.kh(kh).kw(kw).kc(kc).test(pytorch_q8avgpool_ukernel_mp8x9p8q__sse2);
1630:           }
1631:         }
1632:       }
1633:     }
1634:   }
1635: }
1636: 
1637: TEST(Q8AVGPOOL_MP8x9P8Q__SSE2, kc_div_8_multipass_subtile) {
1638:   TEST_REQUIRES_X86_SSE2;
1639:   for (size_t ksMax : std::vector<size_t>{{25, 49}}) {
```

- **EN:** This block handles conditional branches and special cases; iterates over tensors, descriptors, options, or runtime state. Key symbols: no prominent local symbols.
- **CN:** 该代码块处理条件分支与特殊情况；遍历张量、描述符、配置项或运行时状态。关键符号：无明显局部符号。

### Lines 1640-1681 / 第 1640-1681 行

```cpp
1640:     auto tester = AvgPoolMicrokernelTester().kr(8).mr(9).qr(8).iterations(3);
1641:     for (size_t ks = ksMax - tester.qr() + 1; ks < ksMax; ks++) {
1642:       for (size_t kc = 8; kc < 128; kc += 24) {
1643:         tester.kc(kc).kh(ks).kw(1).test(pytorch_q8avgpool_ukernel_mp8x9p8q__sse2);
1644:         tester.kc(kc).kh(1).kw(ks).test(pytorch_q8avgpool_ukernel_mp8x9p8q__sse2);
1645:       }
1646:     }
1647:   }
1648: }
1649: 
1650: TEST(Q8AVGPOOL_MP8x9P8Q__SSE2, kc_div_8_multipass_fulltile_with_x_stride) {
1651:   TEST_REQUIRES_X86_SSE2;
1652:   for (size_t ks : std::vector<size_t>{{25, 49}}) {
1653:     auto tester = AvgPoolMicrokernelTester().kr(8).mr(9).qr(8).iterations(3);
1654:     for (size_t kh = 1; kh <= ks; kh++) {
1655:       for (size_t kw = 1; kw <= ks; kw++) {
1656:         if (kh * kw == ks) {
1657:           for (size_t kc = 8; kc < 128; kc += 24) {
1658:             tester.kh(kh).kw(kw).kc(kc).xStride(131).test(
1659:                 pytorch_q8avgpool_ukernel_mp8x9p8q__sse2);
1660:           }
1661:         }
1662:       }
1663:     }
1664:   }
1665: }
1666: 
1667: TEST(Q8AVGPOOL_MP8x9P8Q__SSE2, kc_gt_8_twopass_fulltile) {
1668:   TEST_REQUIRES_X86_SSE2;
1669:   auto tester = AvgPoolMicrokernelTester().kr(8).mr(9).qr(8).iterations(3);
1670:   const size_t ks = tester.mr() + tester.qr();
1671:   for (size_t kh = 1; kh <= ks; kh++) {
1672:     for (size_t kw = 1; kw <= ks; kw++) {
1673:       if (kh * kw == ks) {
1674:         for (size_t kc = 9; kc < 16; kc++) {
1675:           tester.kh(kh).kw(kw).kc(kc).test(pytorch_q8avgpool_ukernel_mp8x9p8q__sse2);
1676:         }
1677:       }
1678:     }
1679:   }
1680: }
1681: 
```

- **EN:** This block handles conditional branches and special cases; iterates over tensors, descriptors, options, or runtime state. Key symbols: no prominent local symbols.
- **CN:** 该代码块处理条件分支与特殊情况；遍历张量、描述符、配置项或运行时状态。关键符号：无明显局部符号。

### Lines 1682-1724 / 第 1682-1724 行

```cpp
1682: TEST(Q8AVGPOOL_MP8x9P8Q__SSE2, kc_gt_8_twopass_subtile) {
1683:   TEST_REQUIRES_X86_SSE2;
1684:   auto tester = AvgPoolMicrokernelTester().kr(8).mr(9).qr(8).iterations(3);
1685:   for (size_t ks = 10; ks < tester.mr() + tester.qr(); ks++) {
1686:     for (size_t kc = 9; kc < 16; kc++) {
1687:       tester.kc(kc).kh(ks).kw(1).test(pytorch_q8avgpool_ukernel_mp8x9p8q__sse2);
1688:       tester.kc(kc).kh(1).kw(ks).test(pytorch_q8avgpool_ukernel_mp8x9p8q__sse2);
1689:     }
1690:   }
1691: }
1692: 
1693: TEST(Q8AVGPOOL_MP8x9P8Q__SSE2, kc_gt_8_twopass_fulltile_with_x_stride) {
1694:   TEST_REQUIRES_X86_SSE2;
1695:   auto tester = AvgPoolMicrokernelTester().kr(8).mr(9).qr(8).iterations(3);
1696:   const size_t ks = tester.mr() + tester.qr();
1697:   for (size_t kh = 1; kh <= ks; kh++) {
1698:     for (size_t kw = 1; kw <= ks; kw++) {
1699:       if (kh * kw == ks) {
1700:         for (size_t kc = 9; kc < 16; kc++) {
1701:           tester.kh(kh).kw(kw).kc(kc).xStride(23).test(
1702:               pytorch_q8avgpool_ukernel_mp8x9p8q__sse2);
1703:         }
1704:       }
1705:     }
1706:   }
1707: }
1708: 
1709: TEST(Q8AVGPOOL_MP8x9P8Q__SSE2, kc_gt_8_multipass_fulltile) {
1710:   TEST_REQUIRES_X86_SSE2;
1711:   for (size_t ks : std::vector<size_t>{{25, 49}}) {
1712:     auto tester = AvgPoolMicrokernelTester().kr(8).mr(9).qr(8).iterations(3);
1713:     for (size_t kh = 1; kh <= ks; kh++) {
1714:       for (size_t kw = 1; kw <= ks; kw++) {
1715:         if (kh * kw == ks) {
1716:           for (size_t kc = 9; kc < 16; kc++) {
1717:             tester.kh(kh).kw(kw).kc(kc).test(pytorch_q8avgpool_ukernel_mp8x9p8q__sse2);
1718:           }
1719:         }
1720:       }
1721:     }
1722:   }
1723: }
1724: 
```

- **EN:** This block handles conditional branches and special cases; iterates over tensors, descriptors, options, or runtime state. Key symbols: no prominent local symbols.
- **CN:** 该代码块处理条件分支与特殊情况；遍历张量、描述符、配置项或运行时状态。关键符号：无明显局部符号。

### Lines 1725-1775 / 第 1725-1775 行

```cpp
1725: TEST(Q8AVGPOOL_MP8x9P8Q__SSE2, kc_gt_8_multipass_subtile) {
1726:   TEST_REQUIRES_X86_SSE2;
1727:   for (size_t ksMax : std::vector<size_t>{{25, 49}}) {
1728:     auto tester = AvgPoolMicrokernelTester().kr(8).mr(9).qr(8).iterations(3);
1729:     for (size_t ks = ksMax - tester.qr() + 1; ks < ksMax; ks++) {
1730:       for (size_t kc = 9; kc < 16; kc++) {
1731:         tester.kc(kc).kh(ks).kw(1).test(pytorch_q8avgpool_ukernel_mp8x9p8q__sse2);
1732:         tester.kc(kc).kh(1).kw(ks).test(pytorch_q8avgpool_ukernel_mp8x9p8q__sse2);
1733:       }
1734:     }
1735:   }
1736: }
1737: 
1738: TEST(Q8AVGPOOL_MP8x9P8Q__SSE2, kc_gt_8_multipass_fulltile_with_x_stride) {
1739:   TEST_REQUIRES_X86_SSE2;
1740:   for (size_t ks : std::vector<size_t>{{25, 49}}) {
1741:     auto tester = AvgPoolMicrokernelTester().kr(8).mr(9).qr(8).iterations(3);
1742:     for (size_t kh = 1; kh <= ks; kh++) {
1743:       for (size_t kw = 1; kw <= ks; kw++) {
1744:         if (kh * kw == ks) {
1745:           for (size_t kc = 9; kc < 16; kc++) {
1746:             tester.kh(kh).kw(kw).kc(kc).xStride(23).test(
1747:                 pytorch_q8avgpool_ukernel_mp8x9p8q__sse2);
1748:           }
1749:         }
1750:       }
1751:     }
1752:   }
1753: }
1754: 
1755: TEST(Q8AVGPOOL_MP8x9P8Q__SSE2, kc_div_8_with_x_scale) {
1756:   TEST_REQUIRES_X86_SSE2;
1757:   for (size_t n = 1; n <= 5; n += 2) {
1758:     for (size_t kc = 8; kc < 128; kc += 24) {
1759:       for (float xScale = 0.01f; xScale < 100.0f; xScale *= 3.14159265f) {
1760:         AvgPoolMicrokernelTester()
1761:             .kr(8)
1762:             .mr(9)
1763:             .qr(8)
1764:             .n(n)
1765:             .kh(5)
1766:             .kw(5)
1767:             .kc(kc)
1768:             .xScale(xScale)
1769:             .iterations(1)
1770:             .test(pytorch_q8avgpool_ukernel_mp8x9p8q__sse2);
1771:       }
1772:     }
1773:   }
1774: }
1775: 
```

- **EN:** This block handles conditional branches and special cases; iterates over tensors, descriptors, options, or runtime state. Key symbols: `AvgPoolMicrokernelTester`.
- **CN:** 该代码块处理条件分支与特殊情况；遍历张量、描述符、配置项或运行时状态。关键符号：`AvgPoolMicrokernelTester`。

### Lines 1776-1817 / 第 1776-1817 行

```cpp
1776: TEST(Q8AVGPOOL_MP8x9P8Q__SSE2, kc_div_8_with_x_zero_point) {
1777:   TEST_REQUIRES_X86_SSE2;
1778:   for (size_t n = 1; n <= 5; n += 2) {
1779:     for (size_t kc = 8; kc < 128; kc += 24) {
1780:       for (int32_t xZeroPoint = 0; xZeroPoint <= 255; xZeroPoint += 51) {
1781:         AvgPoolMicrokernelTester()
1782:             .kr(8)
1783:             .mr(9)
1784:             .qr(8)
1785:             .n(n)
1786:             .kh(5)
1787:             .kw(5)
1788:             .kc(kc)
1789:             .xZeroPoint(uint8_t(xZeroPoint))
1790:             .iterations(1)
1791:             .test(pytorch_q8avgpool_ukernel_mp8x9p8q__sse2);
1792:       }
1793:     }
1794:   }
1795: }
1796: 
1797: TEST(Q8AVGPOOL_MP8x9P8Q__SSE2, kc_div_8_with_y_scale) {
1798:   TEST_REQUIRES_X86_SSE2;
1799:   for (size_t n = 1; n <= 5; n += 2) {
1800:     for (size_t kc = 8; kc < 128; kc += 24) {
1801:       for (float yScale = 0.01f; yScale < 100.0f; yScale *= 3.14159265f) {
1802:         AvgPoolMicrokernelTester()
1803:             .kr(8)
1804:             .mr(9)
1805:             .qr(8)
1806:             .n(n)
1807:             .kh(5)
1808:             .kw(5)
1809:             .kc(kc)
1810:             .yScale(yScale)
1811:             .iterations(1)
1812:             .test(pytorch_q8avgpool_ukernel_mp8x9p8q__sse2);
1813:       }
1814:     }
1815:   }
1816: }
1817: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `AvgPoolMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`AvgPoolMicrokernelTester`。

### Lines 1818-1861 / 第 1818-1861 行

```cpp
1818: TEST(Q8AVGPOOL_MP8x9P8Q__SSE2, kc_div_8_with_y_zero_point) {
1819:   TEST_REQUIRES_X86_SSE2;
1820:   for (size_t n = 1; n <= 5; n += 2) {
1821:     for (size_t kc = 8; kc < 128; kc += 24) {
1822:       for (int32_t yZeroPoint = 0; yZeroPoint <= 255; yZeroPoint += 51) {
1823:         AvgPoolMicrokernelTester()
1824:             .kr(8)
1825:             .mr(9)
1826:             .qr(8)
1827:             .n(n)
1828:             .kh(5)
1829:             .kw(5)
1830:             .kc(kc)
1831:             .yZeroPoint(uint8_t(yZeroPoint))
1832:             .iterations(1)
1833:             .test(pytorch_q8avgpool_ukernel_mp8x9p8q__sse2);
1834:       }
1835:     }
1836:   }
1837: }
1838: 
1839: TEST(Q8AVGPOOL_MP8x9P8Q__SSE2, kc_div_8_with_y_max) {
1840:   TEST_REQUIRES_X86_SSE2;
1841:   for (size_t n = 1; n <= 5; n += 2) {
1842:     for (size_t kc = 8; kc < 128; kc += 24) {
1843:       AvgPoolMicrokernelTester()
1844:           .kr(8)
1845:           .mr(9)
1846:           .qr(8)
1847:           .n(n)
1848:           .kh(5)
1849:           .kw(5)
1850:           .kc(kc)
1851:           .xZeroPoint(128)
1852:           .yZeroPoint(128)
1853:           .xScale(1.0f)
1854:           .yScale(1.0f)
1855:           .yMax(128)
1856:           .iterations(3)
1857:           .test(pytorch_q8avgpool_ukernel_mp8x9p8q__sse2);
1858:     }
1859:   }
1860: }
1861: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `AvgPoolMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`AvgPoolMicrokernelTester`。

### Lines 1862-1903 / 第 1862-1903 行

```cpp
1862: TEST(Q8AVGPOOL_MP8x9P8Q__SSE2, kc_div_8_with_y_min) {
1863:   TEST_REQUIRES_X86_SSE2;
1864:   for (size_t n = 1; n <= 5; n += 2) {
1865:     for (size_t kc = 8; kc < 128; kc += 24) {
1866:       AvgPoolMicrokernelTester()
1867:           .kr(8)
1868:           .mr(9)
1869:           .qr(8)
1870:           .n(n)
1871:           .kh(5)
1872:           .kw(5)
1873:           .kc(kc)
1874:           .xZeroPoint(128)
1875:           .yZeroPoint(128)
1876:           .xScale(1.0f)
1877:           .yScale(1.0f)
1878:           .yMin(128)
1879:           .iterations(3)
1880:           .test(pytorch_q8avgpool_ukernel_mp8x9p8q__sse2);
1881:     }
1882:   }
1883: }
1884: 
1885: TEST(Q8AVGPOOL_MP8x9P8Q__SSE2, small_n) {
1886:   TEST_REQUIRES_X86_SSE2;
1887:   for (size_t n = 2; n < 5; n++) {
1888:     for (size_t ks : std::vector<size_t>{{5, 7}}) {
1889:       for (size_t kc = 8; kc < 25; kc += 5) {
1890:         AvgPoolMicrokernelTester()
1891:             .kr(8)
1892:             .mr(9)
1893:             .qr(8)
1894:             .n(n)
1895:             .kh(ks)
1896:             .kw(ks)
1897:             .kc(kc)
1898:             .test(pytorch_q8avgpool_ukernel_mp8x9p8q__sse2);
1899:       }
1900:     }
1901:   }
1902: }
1903: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `AvgPoolMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`AvgPoolMicrokernelTester`。

### Lines 1904-1943 / 第 1904-1943 行

```cpp
1904: TEST(Q8AVGPOOL_MP8x9P8Q__SSE2, small_n_with_x_stride) {
1905:   TEST_REQUIRES_X86_SSE2;
1906:   for (size_t n = 2; n < 5; n++) {
1907:     for (size_t ks : std::vector<size_t>{{5, 7}}) {
1908:       for (size_t kc = 8; kc < 25; kc += 5) {
1909:         AvgPoolMicrokernelTester()
1910:             .kr(8)
1911:             .mr(9)
1912:             .qr(8)
1913:             .n(n)
1914:             .kh(ks)
1915:             .kw(ks)
1916:             .kc(kc)
1917:             .xStride(29)
1918:             .test(pytorch_q8avgpool_ukernel_mp8x9p8q__sse2);
1919:       }
1920:     }
1921:   }
1922: }
1923: 
1924: TEST(Q8AVGPOOL_MP8x9P8Q__SSE2, small_n_with_y_stride) {
1925:   TEST_REQUIRES_X86_SSE2;
1926:   for (size_t n = 2; n < 5; n++) {
1927:     for (size_t ks : std::vector<size_t>{{5, 7}}) {
1928:       for (size_t kc = 8; kc < 25; kc += 5) {
1929:         AvgPoolMicrokernelTester()
1930:             .kr(8)
1931:             .mr(9)
1932:             .qr(8)
1933:             .n(n)
1934:             .kh(ks)
1935:             .kw(ks)
1936:             .kc(kc)
1937:             .yStride(31)
1938:             .test(pytorch_q8avgpool_ukernel_mp8x9p8q__sse2);
1939:       }
1940:     }
1941:   }
1942: }
1943: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `AvgPoolMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`AvgPoolMicrokernelTester`。

### Lines 1944-1965 / 第 1944-1965 行

```cpp
1944: TEST(Q8AVGPOOL_MP8x9P8Q__SSE2, small_n_with_s) {
1945:   TEST_REQUIRES_X86_SSE2;
1946:   for (size_t n = 2; n < 5; n++) {
1947:     for (size_t ks : std::vector<size_t>{{5, 7}}) {
1948:       for (size_t s = 2; s <= 5; s++) {
1949:         for (size_t kc = 8; kc < 25; kc += 5) {
1950:           AvgPoolMicrokernelTester()
1951:               .kr(8)
1952:               .mr(9)
1953:               .qr(8)
1954:               .n(n)
1955:               .kh(ks)
1956:               .kw(ks)
1957:               .kc(kc)
1958:               .s(s)
1959:               .test(pytorch_q8avgpool_ukernel_mp8x9p8q__sse2);
1960:         }
1961:       }
1962:     }
1963:   }
1964: }
1965: #endif /* CPUINFO_ARCH_X86 || CPUINFO_ARCH_X86_64 */
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends; iterates over tensors, descriptors, options, or runtime state. Key symbols: `AvgPoolMicrokernelTester`.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端；遍历张量、描述符、配置项或运行时状态。关键符号：`AvgPoolMicrokernelTester`。


## Key Concepts / 关键概念
- **ATen native operator logic** — ATen 原生算子逻辑
- **GPU kernel/runtime interaction** — GPU 内核/运行时交互
- **Runtime validation and invariants** — 运行时校验与不变量
- **Core symbols: AvgPoolMicrokernelTester** — 核心符号：AvgPoolMicrokernelTester

## Dependencies / 依赖关系
- **Internal includes / 内部头文件**: 无
- **External includes / 外部头文件**: `cpuinfo.h`, `gtest/gtest.h`, `qnnpack/isa-checks.h`, `qnnpack/q8avgpool.h`, `avgpool-microkernel-tester.h`
- **Namespaces / 命名空间**: 无
- **Representative symbols / 代表性符号**: `AvgPoolMicrokernelTester`
