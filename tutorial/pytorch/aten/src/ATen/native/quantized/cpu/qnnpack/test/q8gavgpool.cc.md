# q8gavgpool.cc — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `aten/src/ATen/native/quantized/cpu/qnnpack/test/q8gavgpool.cc`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements ATen native operators, checks, helper kernels, and backend dispatch behavior. This specific file centers on `q8gavgpool.cc`. The file header highlights: "All rights reserved. This source code is licensed under the BSD-style license found in the LICENSE file in the root directory of this source tree.."
- **Purpose (CN)**: 实现 ATen 原生算子、运行时检查、辅助内核以及后端分发行为。 该文件具体围绕 `q8gavgpool.cc` 展开。 文件头部注释也概括了其核心职责。

## Line-by-Line Analysis / 逐行分析
### Lines 1-35 / 第 1-35 行

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
0013: #include <qnnpack/q8gavgpool.h>
0014: 
0015: #include "gavgpool-microkernel-tester.h"
0016: 
0017: #if CPUINFO_ARCH_ARM || CPUINFO_ARCH_ARM64
0018: TEST(Q8GAVGPOOL_UP8x7__NEON, n_eq_8_all_m) {
0019:   TEST_REQUIRES_ARM_NEON;
0020:   GAvgPoolMicrokernelTester().m(7).n(8).test(pytorch_q8gavgpool_ukernel_up8x7__neon);
0021: }
0022: 
0023: TEST(Q8GAVGPOOL_UP8x7__NEON, n_eq_8_few_m) {
0024:   TEST_REQUIRES_ARM_NEON;
0025:   for (size_t m = 1; m < 7; m++) {
0026:     GAvgPoolMicrokernelTester().m(m).n(8).test(pytorch_q8gavgpool_ukernel_up8x7__neon);
0027:   }
0028: }
0029: 
0030: TEST(Q8GAVGPOOL_UP8x7__NEON, n_eq_8_all_m_with_x_stride) {
0031:   TEST_REQUIRES_ARM_NEON;
0032:   GAvgPoolMicrokernelTester().m(7).n(8).xStride(11).test(
0033:       pytorch_q8gavgpool_ukernel_up8x7__neon);
0034: }
0035: 
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends; iterates over tensors, descriptors, options, or runtime state. Key symbols: `GAvgPoolMicrokernelTester`.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端；遍历张量、描述符、配置项或运行时状态。关键符号：`GAvgPoolMicrokernelTester`。

### Lines 36-65 / 第 36-65 行

```cpp
0036: TEST(Q8GAVGPOOL_UP8x7__NEON, n_eq_8_all_m_with_x_scale) {
0037:   TEST_REQUIRES_ARM_NEON;
0038:   for (float xScale = 0.01f; xScale < 100.0f; xScale *= 3.14159265f) {
0039:     GAvgPoolMicrokernelTester().m(7).n(8).xScale(xScale).test(
0040:         pytorch_q8gavgpool_ukernel_up8x7__neon);
0041:   }
0042: }
0043: 
0044: TEST(Q8GAVGPOOL_UP8x7__NEON, n_eq_8_all_m_with_x_zero_point) {
0045:   TEST_REQUIRES_ARM_NEON;
0046:   for (int32_t xZeroPoint = 0; xZeroPoint <= 255; xZeroPoint += 51) {
0047:     GAvgPoolMicrokernelTester()
0048:         .m(7)
0049:         .n(8)
0050:         .xZeroPoint(xZeroPoint)
0051:         .test(pytorch_q8gavgpool_ukernel_up8x7__neon);
0052:   }
0053: }
0054: 
0055: TEST(Q8GAVGPOOL_UP8x7__NEON, n_eq_8_all_m_with_y_scale) {
0056:   TEST_REQUIRES_ARM_NEON;
0057:   for (float yScale = 0.01f; yScale < 100.0f; yScale *= 3.14159265f) {
0058:     GAvgPoolMicrokernelTester().m(7).n(8).yScale(yScale).test(
0059:         pytorch_q8gavgpool_ukernel_up8x7__neon);
0060:   }
0061: }
0062: 
0063: TEST(Q8GAVGPOOL_UP8x7__NEON, n_eq_8_all_m_with_y_zero_point) {
0064:   TEST_REQUIRES_ARM_NEON;
0065:   for (int32_t yZeroPoint = 0; yZeroPoint <= 255; yZeroPoint += 51) {
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `GAvgPoolMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`GAvgPoolMicrokernelTester`。

### Lines 66-99 / 第 66-99 行

```cpp
0066:     GAvgPoolMicrokernelTester()
0067:         .m(7)
0068:         .n(8)
0069:         .yZeroPoint(yZeroPoint)
0070:         .test(pytorch_q8gavgpool_ukernel_up8x7__neon);
0071:   }
0072: }
0073: 
0074: TEST(Q8GAVGPOOL_UP8x7__NEON, n_eq_8_all_m_with_y_max) {
0075:   TEST_REQUIRES_ARM_NEON;
0076:   GAvgPoolMicrokernelTester()
0077:       .m(7)
0078:       .n(8)
0079:       .xZeroPoint(128)
0080:       .yZeroPoint(128)
0081:       .xScale(1.0f)
0082:       .yScale(1.0f)
0083:       .yMax(128)
0084:       .test(pytorch_q8gavgpool_ukernel_up8x7__neon);
0085: }
0086: 
0087: TEST(Q8GAVGPOOL_UP8x7__NEON, n_eq_8_all_m_with_y_min) {
0088:   TEST_REQUIRES_ARM_NEON;
0089:   GAvgPoolMicrokernelTester()
0090:       .m(7)
0091:       .n(8)
0092:       .xZeroPoint(128)
0093:       .yZeroPoint(128)
0094:       .xScale(1.0f)
0095:       .yScale(1.0f)
0096:       .yMin(128)
0097:       .test(pytorch_q8gavgpool_ukernel_up8x7__neon);
0098: }
0099: 
```

- **EN:** This block implements local helper logic for `q8gavgpool`. Key symbols: `GAvgPoolMicrokernelTester`.
- **CN:** 该代码块实现与 `q8gavgpool` 相关的局部辅助逻辑。关键符号：`GAvgPoolMicrokernelTester`。

### Lines 100-133 / 第 100-133 行

```cpp
0100: TEST(Q8GAVGPOOL_UP8x7__NEON, n_div_8_all_m) {
0101:   TEST_REQUIRES_ARM_NEON;
0102:   for (size_t n = 8; n < 128; n += 24) {
0103:     GAvgPoolMicrokernelTester().m(7).n(n).test(pytorch_q8gavgpool_ukernel_up8x7__neon);
0104:   }
0105: }
0106: 
0107: TEST(Q8GAVGPOOL_UP8x7__NEON, n_div_8_few_m) {
0108:   TEST_REQUIRES_ARM_NEON;
0109:   for (size_t n = 8; n < 128; n += 24) {
0110:     for (size_t m = 1; m < 7; m++) {
0111:       GAvgPoolMicrokernelTester().m(m).n(n).test(
0112:           pytorch_q8gavgpool_ukernel_up8x7__neon);
0113:     }
0114:   }
0115: }
0116: 
0117: TEST(Q8GAVGPOOL_UP8x7__NEON, n_gt_8_all_m) {
0118:   TEST_REQUIRES_ARM_NEON;
0119:   for (size_t n = 9; n < 16; n++) {
0120:     GAvgPoolMicrokernelTester().m(7).n(n).test(pytorch_q8gavgpool_ukernel_up8x7__neon);
0121:   }
0122: }
0123: 
0124: TEST(Q8GAVGPOOL_UP8x7__NEON, n_gt_8_few_m) {
0125:   TEST_REQUIRES_ARM_NEON;
0126:   for (size_t n = 9; n < 16; n++) {
0127:     for (size_t m = 1; m < 7; m++) {
0128:       GAvgPoolMicrokernelTester().m(m).n(n).test(
0129:           pytorch_q8gavgpool_ukernel_up8x7__neon);
0130:     }
0131:   }
0132: }
0133: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `GAvgPoolMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`GAvgPoolMicrokernelTester`。

### Lines 134-166 / 第 134-166 行

```cpp
0134: TEST(Q8GAVGPOOL_UP8x7__NEON, n_gt_8_all_m_with_x_scale) {
0135:   TEST_REQUIRES_ARM_NEON;
0136:   for (size_t n = 9; n < 16; n++) {
0137:     for (float xScale = 0.01f; xScale < 100.0f; xScale *= 3.14159265f) {
0138:       GAvgPoolMicrokernelTester().m(7).n(n).xScale(xScale).test(
0139:           pytorch_q8gavgpool_ukernel_up8x7__neon);
0140:     }
0141:   }
0142: }
0143: 
0144: TEST(Q8GAVGPOOL_UP8x7__NEON, n_gt_8_all_m_with_x_zero_point) {
0145:   TEST_REQUIRES_ARM_NEON;
0146:   for (size_t n = 9; n < 16; n++) {
0147:     for (int32_t xZeroPoint = 0; xZeroPoint <= 255; xZeroPoint += 51) {
0148:       GAvgPoolMicrokernelTester()
0149:           .m(7)
0150:           .n(n)
0151:           .xZeroPoint(xZeroPoint)
0152:           .test(pytorch_q8gavgpool_ukernel_up8x7__neon);
0153:     }
0154:   }
0155: }
0156: 
0157: TEST(Q8GAVGPOOL_UP8x7__NEON, n_gt_8_all_m_with_y_scale) {
0158:   TEST_REQUIRES_ARM_NEON;
0159:   for (size_t n = 9; n < 16; n++) {
0160:     for (float yScale = 0.01f; yScale < 100.0f; yScale *= 3.14159265f) {
0161:       GAvgPoolMicrokernelTester().m(7).n(n).yScale(yScale).test(
0162:           pytorch_q8gavgpool_ukernel_up8x7__neon);
0163:     }
0164:   }
0165: }
0166: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `GAvgPoolMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`GAvgPoolMicrokernelTester`。

### Lines 167-197 / 第 167-197 行

```cpp
0167: TEST(Q8GAVGPOOL_UP8x7__NEON, n_gt_8_all_m_with_y_zero_point) {
0168:   TEST_REQUIRES_ARM_NEON;
0169:   for (size_t n = 9; n < 16; n++) {
0170:     for (int32_t yZeroPoint = 0; yZeroPoint <= 255; yZeroPoint += 51) {
0171:       GAvgPoolMicrokernelTester()
0172:           .m(7)
0173:           .n(n)
0174:           .yZeroPoint(yZeroPoint)
0175:           .test(pytorch_q8gavgpool_ukernel_up8x7__neon);
0176:     }
0177:   }
0178: }
0179: 
0180: TEST(Q8GAVGPOOL_UP8x7__NEON, n_gt_8_all_m_with_y_max) {
0181:   TEST_REQUIRES_ARM_NEON;
0182:   for (size_t n = 9; n < 16; n++) {
0183:     GAvgPoolMicrokernelTester()
0184:         .m(7)
0185:         .n(n)
0186:         .xZeroPoint(128)
0187:         .yZeroPoint(128)
0188:         .xScale(1.0f)
0189:         .yScale(1.0f)
0190:         .yMax(128)
0191:         .test(pytorch_q8gavgpool_ukernel_up8x7__neon);
0192:   }
0193: }
0194: 
0195: TEST(Q8GAVGPOOL_UP8x7__NEON, n_gt_8_all_m_with_y_min) {
0196:   TEST_REQUIRES_ARM_NEON;
0197:   for (size_t n = 9; n < 16; n++) {
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `GAvgPoolMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`GAvgPoolMicrokernelTester`。

### Lines 198-229 / 第 198-229 行

```cpp
0198:     GAvgPoolMicrokernelTester()
0199:         .m(7)
0200:         .n(n)
0201:         .xZeroPoint(128)
0202:         .yZeroPoint(128)
0203:         .xScale(1.0f)
0204:         .yScale(1.0f)
0205:         .yMin(128)
0206:         .test(pytorch_q8gavgpool_ukernel_up8x7__neon);
0207:   }
0208: }
0209: 
0210: TEST(Q8GAVGPOOL_MP8x7p7q__NEON, n_eq_8_2pass_all_m) {
0211:   TEST_REQUIRES_ARM_NEON;
0212:   GAvgPoolMicrokernelTester().m(14).n(8).nr(8).test(
0213:       pytorch_q8gavgpool_ukernel_mp8x7p7q__neon);
0214: }
0215: 
0216: TEST(Q8GAVGPOOL_MP8x7p7q__NEON, n_eq_8_2pass_all_m_with_x_stride) {
0217:   TEST_REQUIRES_ARM_NEON;
0218:   GAvgPoolMicrokernelTester().m(14).n(8).nr(8).xStride(11).test(
0219:       pytorch_q8gavgpool_ukernel_mp8x7p7q__neon);
0220: }
0221: 
0222: TEST(Q8GAVGPOOL_MP8x7p7q__NEON, n_eq_8_2pass_all_m_with_x_scale) {
0223:   TEST_REQUIRES_ARM_NEON;
0224:   for (float xScale = 0.01f; xScale < 100.0f; xScale *= 3.14159265f) {
0225:     GAvgPoolMicrokernelTester().m(14).n(8).nr(8).xScale(xScale).test(
0226:         pytorch_q8gavgpool_ukernel_mp8x7p7q__neon);
0227:   }
0228: }
0229: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `GAvgPoolMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`GAvgPoolMicrokernelTester`。

### Lines 230-261 / 第 230-261 行

```cpp
0230: TEST(Q8GAVGPOOL_MP8x7p7q__NEON, n_eq_8_2pass_all_m_with_x_zero_point) {
0231:   TEST_REQUIRES_ARM_NEON;
0232:   for (int32_t xZeroPoint = 0; xZeroPoint <= 255; xZeroPoint += 51) {
0233:     GAvgPoolMicrokernelTester()
0234:         .m(14)
0235:         .n(8)
0236:         .nr(8)
0237:         .xZeroPoint(xZeroPoint)
0238:         .test(pytorch_q8gavgpool_ukernel_mp8x7p7q__neon);
0239:   }
0240: }
0241: 
0242: TEST(Q8GAVGPOOL_MP8x7p7q__NEON, n_eq_8_2pass_all_m_with_y_scale) {
0243:   TEST_REQUIRES_ARM_NEON;
0244:   for (float yScale = 0.01f; yScale < 100.0f; yScale *= 3.14159265f) {
0245:     GAvgPoolMicrokernelTester().m(14).n(8).nr(8).yScale(yScale).test(
0246:         pytorch_q8gavgpool_ukernel_mp8x7p7q__neon);
0247:   }
0248: }
0249: 
0250: TEST(Q8GAVGPOOL_MP8x7p7q__NEON, n_eq_8_2pass_all_m_with_y_zero_point) {
0251:   TEST_REQUIRES_ARM_NEON;
0252:   for (int32_t yZeroPoint = 0; yZeroPoint <= 255; yZeroPoint += 51) {
0253:     GAvgPoolMicrokernelTester()
0254:         .m(14)
0255:         .n(8)
0256:         .nr(8)
0257:         .yZeroPoint(yZeroPoint)
0258:         .test(pytorch_q8gavgpool_ukernel_mp8x7p7q__neon);
0259:   }
0260: }
0261: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `GAvgPoolMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`GAvgPoolMicrokernelTester`。

### Lines 262-292 / 第 262-292 行

```cpp
0262: TEST(Q8GAVGPOOL_MP8x7p7q__NEON, n_eq_8_2pass_all_m_with_y_max) {
0263:   TEST_REQUIRES_ARM_NEON;
0264:   GAvgPoolMicrokernelTester()
0265:       .m(14)
0266:       .n(8)
0267:       .nr(8)
0268:       .xZeroPoint(128)
0269:       .yZeroPoint(128)
0270:       .xScale(1.0f)
0271:       .yScale(1.0f)
0272:       .yMax(128)
0273:       .test(pytorch_q8gavgpool_ukernel_mp8x7p7q__neon);
0274: }
0275: 
0276: TEST(Q8GAVGPOOL_MP8x7p7q__NEON, n_eq_8_2pass_all_m_with_y_min) {
0277:   TEST_REQUIRES_ARM_NEON;
0278:   GAvgPoolMicrokernelTester()
0279:       .m(14)
0280:       .n(8)
0281:       .nr(8)
0282:       .xZeroPoint(128)
0283:       .yZeroPoint(128)
0284:       .xScale(1.0f)
0285:       .yScale(1.0f)
0286:       .yMin(128)
0287:       .test(pytorch_q8gavgpool_ukernel_mp8x7p7q__neon);
0288: }
0289: 
0290: TEST(Q8GAVGPOOL_MP8x7p7q__NEON, n_eq_8_2pass_few_m) {
0291:   TEST_REQUIRES_ARM_NEON;
0292:   for (size_t m = 1; m < 7; m++) {
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `GAvgPoolMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`GAvgPoolMicrokernelTester`。

### Lines 293-324 / 第 293-324 行

```cpp
0293:     GAvgPoolMicrokernelTester().m(7 + m).n(8).nr(8).test(
0294:         pytorch_q8gavgpool_ukernel_mp8x7p7q__neon);
0295:   }
0296: }
0297: 
0298: TEST(Q8GAVGPOOL_MP8x7p7q__NEON, n_eq_8_2pass_few_m_with_x_stride) {
0299:   TEST_REQUIRES_ARM_NEON;
0300:   for (size_t m = 1; m < 7; m++) {
0301:     GAvgPoolMicrokernelTester().m(7 + m).n(8).nr(8).xStride(11).test(
0302:         pytorch_q8gavgpool_ukernel_mp8x7p7q__neon);
0303:   }
0304: }
0305: 
0306: TEST(Q8GAVGPOOL_MP8x7p7q__NEON, n_eq_8_multipass_all_m) {
0307:   TEST_REQUIRES_ARM_NEON;
0308:   for (size_t m = 14; m <= 35; m += 7) {
0309:     GAvgPoolMicrokernelTester().m(m).n(8).nr(8).test(
0310:         pytorch_q8gavgpool_ukernel_mp8x7p7q__neon);
0311:   }
0312: }
0313: 
0314: TEST(Q8GAVGPOOL_MP8x7p7q__NEON, n_eq_8_multipass_all_m_with_x_stride) {
0315:   TEST_REQUIRES_ARM_NEON;
0316:   for (size_t m = 14; m <= 35; m += 7) {
0317:     GAvgPoolMicrokernelTester().m(m).n(8).nr(8).test(
0318:         pytorch_q8gavgpool_ukernel_mp8x7p7q__neon);
0319:   }
0320: }
0321: 
0322: TEST(Q8GAVGPOOL_MP8x7p7q__NEON, n_div_8_2pass_all_m) {
0323:   TEST_REQUIRES_ARM_NEON;
0324:   for (size_t n = 8; n < 128; n += 24) {
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `GAvgPoolMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`GAvgPoolMicrokernelTester`。

### Lines 325-359 / 第 325-359 行

```cpp
0325:     GAvgPoolMicrokernelTester().m(14).n(n).nr(8).test(
0326:         pytorch_q8gavgpool_ukernel_mp8x7p7q__neon);
0327:   }
0328: }
0329: 
0330: TEST(Q8GAVGPOOL_MP8x7p7q__NEON, n_div_8_2pass_few_m) {
0331:   TEST_REQUIRES_ARM_NEON;
0332:   for (size_t n = 8; n < 128; n += 24) {
0333:     for (size_t m = 1; m < 7; m++) {
0334:       GAvgPoolMicrokernelTester().m(7 + m).n(n).nr(8).test(
0335:           pytorch_q8gavgpool_ukernel_mp8x7p7q__neon);
0336:     }
0337:   }
0338: }
0339: 
0340: TEST(Q8GAVGPOOL_MP8x7p7q__NEON, n_div_8_multipass_all_m) {
0341:   TEST_REQUIRES_ARM_NEON;
0342:   for (size_t n = 8; n < 128; n += 24) {
0343:     for (size_t m = 14; m <= 35; m += 7) {
0344:       GAvgPoolMicrokernelTester().m(m).n(n).nr(8).nr(8).test(
0345:           pytorch_q8gavgpool_ukernel_mp8x7p7q__neon);
0346:     }
0347:   }
0348: }
0349: 
0350: TEST(Q8GAVGPOOL_MP8x7p7q__NEON, n_div_8_multipass_all_m_with_x_stride) {
0351:   TEST_REQUIRES_ARM_NEON;
0352:   for (size_t n = 8; n < 128; n += 24) {
0353:     for (size_t m = 14; m <= 35; m += 7) {
0354:       GAvgPoolMicrokernelTester().m(m).n(n).nr(8).nr(8).xStride(131).test(
0355:           pytorch_q8gavgpool_ukernel_mp8x7p7q__neon);
0356:     }
0357:   }
0358: }
0359: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `GAvgPoolMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`GAvgPoolMicrokernelTester`。

### Lines 360-391 / 第 360-391 行

```cpp
0360: TEST(Q8GAVGPOOL_MP8x7p7q__NEON, n_gt_8_2pass_all_m) {
0361:   TEST_REQUIRES_ARM_NEON;
0362:   for (size_t n = 9; n < 16; n++) {
0363:     GAvgPoolMicrokernelTester().m(14).n(n).nr(8).test(
0364:         pytorch_q8gavgpool_ukernel_mp8x7p7q__neon);
0365:   }
0366: }
0367: 
0368: TEST(Q8GAVGPOOL_MP8x7p7q__NEON, n_gt_8_2pass_all_m_with_x_scale) {
0369:   TEST_REQUIRES_ARM_NEON;
0370:   for (float xScale = 0.01f; xScale < 100.0f; xScale *= 3.14159265f) {
0371:     for (size_t n = 9; n < 16; n++) {
0372:       GAvgPoolMicrokernelTester().m(14).n(n).nr(8).xScale(xScale).test(
0373:           pytorch_q8gavgpool_ukernel_mp8x7p7q__neon);
0374:     }
0375:   }
0376: }
0377: 
0378: TEST(Q8GAVGPOOL_MP8x7p7q__NEON, n_gt_8_2pass_all_m_with_x_zero_point) {
0379:   TEST_REQUIRES_ARM_NEON;
0380:   for (int32_t xZeroPoint = 0; xZeroPoint <= 255; xZeroPoint += 51) {
0381:     for (size_t n = 9; n < 16; n++) {
0382:       GAvgPoolMicrokernelTester()
0383:           .m(14)
0384:           .n(n)
0385:           .nr(8)
0386:           .xZeroPoint(xZeroPoint)
0387:           .test(pytorch_q8gavgpool_ukernel_mp8x7p7q__neon);
0388:     }
0389:   }
0390: }
0391: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `GAvgPoolMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`GAvgPoolMicrokernelTester`。

### Lines 392-431 / 第 392-431 行

```cpp
0392: TEST(Q8GAVGPOOL_MP8x7p7q__NEON, n_gt_8_2pass_all_m_with_y_scale) {
0393:   TEST_REQUIRES_ARM_NEON;
0394:   for (float yScale = 0.01f; yScale < 100.0f; yScale *= 3.14159265f) {
0395:     for (size_t n = 9; n < 16; n++) {
0396:       GAvgPoolMicrokernelTester().m(14).n(n).nr(8).yScale(yScale).test(
0397:           pytorch_q8gavgpool_ukernel_mp8x7p7q__neon);
0398:     }
0399:   }
0400: }
0401: 
0402: TEST(Q8GAVGPOOL_MP8x7p7q__NEON, n_gt_8_2pass_all_m_with_y_zero_point) {
0403:   TEST_REQUIRES_ARM_NEON;
0404:   for (int32_t yZeroPoint = 0; yZeroPoint <= 255; yZeroPoint += 51) {
0405:     for (size_t n = 9; n < 16; n++) {
0406:       GAvgPoolMicrokernelTester()
0407:           .m(14)
0408:           .n(n)
0409:           .nr(8)
0410:           .yZeroPoint(yZeroPoint)
0411:           .test(pytorch_q8gavgpool_ukernel_mp8x7p7q__neon);
0412:     }
0413:   }
0414: }
0415: 
0416: TEST(Q8GAVGPOOL_MP8x7p7q__NEON, n_gt_8_2pass_all_m_with_y_max) {
0417:   TEST_REQUIRES_ARM_NEON;
0418:   for (size_t n = 9; n < 16; n++) {
0419:     GAvgPoolMicrokernelTester()
0420:         .m(14)
0421:         .n(n)
0422:         .nr(8)
0423:         .xZeroPoint(128)
0424:         .yZeroPoint(128)
0425:         .xScale(1.0f)
0426:         .yScale(1.0f)
0427:         .yMax(128)
0428:         .test(pytorch_q8gavgpool_ukernel_mp8x7p7q__neon);
0429:   }
0430: }
0431: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `GAvgPoolMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`GAvgPoolMicrokernelTester`。

### Lines 432-461 / 第 432-461 行

```cpp
0432: TEST(Q8GAVGPOOL_MP8x7p7q__NEON, n_gt_8_2pass_all_m_with_y_min) {
0433:   TEST_REQUIRES_ARM_NEON;
0434:   for (size_t n = 9; n < 16; n++) {
0435:     GAvgPoolMicrokernelTester()
0436:         .m(14)
0437:         .n(n)
0438:         .nr(8)
0439:         .xZeroPoint(128)
0440:         .yZeroPoint(128)
0441:         .xScale(1.0f)
0442:         .yScale(1.0f)
0443:         .yMin(128)
0444:         .test(pytorch_q8gavgpool_ukernel_mp8x7p7q__neon);
0445:   }
0446: }
0447: 
0448: TEST(Q8GAVGPOOL_MP8x7p7q__NEON, n_gt_8_2pass_few_m) {
0449:   TEST_REQUIRES_ARM_NEON;
0450:   for (size_t n = 9; n < 16; n++) {
0451:     for (size_t m = 1; m < 7; m++) {
0452:       GAvgPoolMicrokernelTester().m(7 + m).n(n).nr(8).test(
0453:           pytorch_q8gavgpool_ukernel_mp8x7p7q__neon);
0454:     }
0455:   }
0456: }
0457: 
0458: TEST(Q8GAVGPOOL_MP8x7p7q__NEON, n_gt_8_multipass_all_m) {
0459:   TEST_REQUIRES_ARM_NEON;
0460:   for (size_t n = 9; n < 16; n++) {
0461:     for (size_t m = 14; m <= 35; m += 7) {
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `GAvgPoolMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`GAvgPoolMicrokernelTester`。

### Lines 462-491 / 第 462-491 行

```cpp
0462:       GAvgPoolMicrokernelTester().m(m).n(n).nr(8).test(
0463:           pytorch_q8gavgpool_ukernel_mp8x7p7q__neon);
0464:     }
0465:   }
0466: }
0467: 
0468: TEST(Q8GAVGPOOL_MP8x7p7q__NEON, n_gt_8_multipass_all_m_with_x_stride) {
0469:   TEST_REQUIRES_ARM_NEON;
0470:   for (size_t n = 9; n < 16; n++) {
0471:     for (size_t m = 14; m <= 35; m += 7) {
0472:       GAvgPoolMicrokernelTester().m(m).n(n).nr(8).xStride(23).test(
0473:           pytorch_q8gavgpool_ukernel_mp8x7p7q__neon);
0474:     }
0475:   }
0476: }
0477: 
0478: TEST(Q8GAVGPOOL_UP8xM__NEON, n_lt_8_small_m) {
0479:   TEST_REQUIRES_ARM_NEON;
0480:   for (size_t n = 1; n < 8; n++) {
0481:     for (size_t m = 1; m < 8; m++) {
0482:       GAvgPoolMicrokernelTester().m(m).n(n).test(
0483:           pytorch_q8gavgpool_ukernel_up8xm__neon);
0484:     }
0485:   }
0486: }
0487: 
0488: TEST(Q8GAVGPOOL_UP8xM__NEON, n_lt_8_large_m) {
0489:   TEST_REQUIRES_ARM_NEON;
0490:   for (size_t n = 1; n < 8; n++) {
0491:     for (size_t m = 8; m < 16; m++) {
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `GAvgPoolMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`GAvgPoolMicrokernelTester`。

### Lines 492-524 / 第 492-524 行

```cpp
0492:       GAvgPoolMicrokernelTester().m(m).n(n).test(
0493:           pytorch_q8gavgpool_ukernel_up8xm__neon);
0494:     }
0495:   }
0496: }
0497: 
0498: TEST(Q8GAVGPOOL_UP8xM__NEON, n_lt_8_with_x_scale) {
0499:   TEST_REQUIRES_ARM_NEON;
0500:   for (size_t n = 1; n < 8; n++) {
0501:     for (size_t m = 1; m < 16; m += 5) {
0502:       for (float xScale = 0.01f; xScale < 100.0f; xScale *= 3.14159265f) {
0503:         GAvgPoolMicrokernelTester().m(m).n(n).xScale(xScale).test(
0504:             pytorch_q8gavgpool_ukernel_up8xm__neon);
0505:       }
0506:     }
0507:   }
0508: }
0509: 
0510: TEST(Q8GAVGPOOL_UP8xM__NEON, n_lt_8_with_x_zero_point) {
0511:   TEST_REQUIRES_ARM_NEON;
0512:   for (size_t n = 1; n < 8; n++) {
0513:     for (size_t m = 1; m < 16; m += 5) {
0514:       for (int32_t xZeroPoint = 0; xZeroPoint <= 255; xZeroPoint += 51) {
0515:         GAvgPoolMicrokernelTester()
0516:             .m(m)
0517:             .n(n)
0518:             .xZeroPoint(xZeroPoint)
0519:             .test(pytorch_q8gavgpool_ukernel_up8xm__neon);
0520:       }
0521:     }
0522:   }
0523: }
0524: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `GAvgPoolMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`GAvgPoolMicrokernelTester`。

### Lines 525-554 / 第 525-554 行

```cpp
0525: TEST(Q8GAVGPOOL_UP8xM__NEON, n_lt_8_with_y_scale) {
0526:   TEST_REQUIRES_ARM_NEON;
0527:   for (size_t n = 1; n < 8; n++) {
0528:     for (size_t m = 1; m < 16; m += 5) {
0529:       for (float yScale = 0.01f; yScale < 100.0f; yScale *= 3.14159265f) {
0530:         GAvgPoolMicrokernelTester().m(m).n(n).yScale(yScale).test(
0531:             pytorch_q8gavgpool_ukernel_up8xm__neon);
0532:       }
0533:     }
0534:   }
0535: }
0536: 
0537: TEST(Q8GAVGPOOL_UP8xM__NEON, n_lt_8_with_y_zero_point) {
0538:   TEST_REQUIRES_ARM_NEON;
0539:   for (size_t n = 1; n < 8; n++) {
0540:     for (size_t m = 1; m < 16; m += 5) {
0541:       for (int32_t yZeroPoint = 0; yZeroPoint <= 255; yZeroPoint += 51) {
0542:         GAvgPoolMicrokernelTester()
0543:             .m(m)
0544:             .n(n)
0545:             .yZeroPoint(yZeroPoint)
0546:             .test(pytorch_q8gavgpool_ukernel_up8xm__neon);
0547:       }
0548:     }
0549:   }
0550: }
0551: 
0552: TEST(Q8GAVGPOOL_UP8xM__NEON, n_lt_8_with_y_max) {
0553:   TEST_REQUIRES_ARM_NEON;
0554:   for (size_t n = 1; n < 8; n++) {
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `GAvgPoolMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`GAvgPoolMicrokernelTester`。

### Lines 555-585 / 第 555-585 行

```cpp
0555:     for (size_t m = 1; m < 16; m += 5) {
0556:       GAvgPoolMicrokernelTester()
0557:           .m(m)
0558:           .n(n)
0559:           .xZeroPoint(128)
0560:           .yZeroPoint(128)
0561:           .xScale(1.0f)
0562:           .yScale(1.0f)
0563:           .yMax(128)
0564:           .test(pytorch_q8gavgpool_ukernel_up8xm__neon);
0565:     }
0566:   }
0567: }
0568: 
0569: TEST(Q8GAVGPOOL_UP8xM__NEON, n_lt_8_with_y_min) {
0570:   TEST_REQUIRES_ARM_NEON;
0571:   for (size_t n = 1; n < 8; n++) {
0572:     for (size_t m = 1; m < 16; m += 5) {
0573:       GAvgPoolMicrokernelTester()
0574:           .m(m)
0575:           .n(n)
0576:           .xZeroPoint(128)
0577:           .yZeroPoint(128)
0578:           .xScale(1.0f)
0579:           .yScale(1.0f)
0580:           .yMin(128)
0581:           .test(pytorch_q8gavgpool_ukernel_up8xm__neon);
0582:     }
0583:   }
0584: }
0585: #endif /* CPUINFO_ARCH_ARM || CPUINFO_ARCH_ARM64 */
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends; iterates over tensors, descriptors, options, or runtime state. Key symbols: `GAvgPoolMicrokernelTester`.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端；遍历张量、描述符、配置项或运行时状态。关键符号：`GAvgPoolMicrokernelTester`。

### Lines 586-616 / 第 586-616 行

```cpp
0586: 
0587: #if CPUINFO_ARCH_X86 || CPUINFO_ARCH_X86_64
0588: TEST(Q8GAVGPOOL_UP8x7__SSE2, n_eq_8_all_m) {
0589:   TEST_REQUIRES_X86_SSE2;
0590:   GAvgPoolMicrokernelTester().m(7).n(8).test(pytorch_q8gavgpool_ukernel_up8x7__sse2);
0591: }
0592: 
0593: TEST(Q8GAVGPOOL_UP8x7__SSE2, n_eq_8_few_m) {
0594:   TEST_REQUIRES_X86_SSE2;
0595:   for (size_t m = 1; m < 7; m++) {
0596:     GAvgPoolMicrokernelTester().m(m).n(8).test(pytorch_q8gavgpool_ukernel_up8x7__sse2);
0597:   }
0598: }
0599: 
0600: TEST(Q8GAVGPOOL_UP8x7__SSE2, n_eq_8_all_m_with_x_stride) {
0601:   TEST_REQUIRES_X86_SSE2;
0602:   GAvgPoolMicrokernelTester().m(7).n(8).xStride(11).test(
0603:       pytorch_q8gavgpool_ukernel_up8x7__sse2);
0604: }
0605: 
0606: TEST(Q8GAVGPOOL_UP8x7__SSE2, n_eq_8_all_m_with_x_scale) {
0607:   TEST_REQUIRES_X86_SSE2;
0608:   for (float xScale = 0.01f; xScale < 100.0f; xScale *= 3.14159265f) {
0609:     GAvgPoolMicrokernelTester().m(7).n(8).xScale(xScale).test(
0610:         pytorch_q8gavgpool_ukernel_up8x7__sse2);
0611:   }
0612: }
0613: 
0614: TEST(Q8GAVGPOOL_UP8x7__SSE2, n_eq_8_all_m_with_x_zero_point) {
0615:   TEST_REQUIRES_X86_SSE2;
0616:   for (int32_t xZeroPoint = 0; xZeroPoint <= 255; xZeroPoint += 51) {
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends; iterates over tensors, descriptors, options, or runtime state. Key symbols: `GAvgPoolMicrokernelTester`.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端；遍历张量、描述符、配置项或运行时状态。关键符号：`GAvgPoolMicrokernelTester`。

### Lines 617-656 / 第 617-656 行

```cpp
0617:     GAvgPoolMicrokernelTester()
0618:         .m(7)
0619:         .n(8)
0620:         .xZeroPoint(xZeroPoint)
0621:         .test(pytorch_q8gavgpool_ukernel_up8x7__sse2);
0622:   }
0623: }
0624: 
0625: TEST(Q8GAVGPOOL_UP8x7__SSE2, n_eq_8_all_m_with_y_scale) {
0626:   TEST_REQUIRES_X86_SSE2;
0627:   for (float yScale = 0.01f; yScale < 100.0f; yScale *= 3.14159265f) {
0628:     GAvgPoolMicrokernelTester().m(7).n(8).yScale(yScale).test(
0629:         pytorch_q8gavgpool_ukernel_up8x7__sse2);
0630:   }
0631: }
0632: 
0633: TEST(Q8GAVGPOOL_UP8x7__SSE2, n_eq_8_all_m_with_y_zero_point) {
0634:   TEST_REQUIRES_X86_SSE2;
0635:   for (int32_t yZeroPoint = 0; yZeroPoint <= 255; yZeroPoint += 51) {
0636:     GAvgPoolMicrokernelTester()
0637:         .m(7)
0638:         .n(8)
0639:         .yZeroPoint(yZeroPoint)
0640:         .test(pytorch_q8gavgpool_ukernel_up8x7__sse2);
0641:   }
0642: }
0643: 
0644: TEST(Q8GAVGPOOL_UP8x7__SSE2, n_eq_8_all_m_with_y_max) {
0645:   TEST_REQUIRES_X86_SSE2;
0646:   GAvgPoolMicrokernelTester()
0647:       .m(7)
0648:       .n(8)
0649:       .xZeroPoint(128)
0650:       .yZeroPoint(128)
0651:       .xScale(1.0f)
0652:       .yScale(1.0f)
0653:       .yMax(128)
0654:       .test(pytorch_q8gavgpool_ukernel_up8x7__sse2);
0655: }
0656: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `GAvgPoolMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`GAvgPoolMicrokernelTester`。

### Lines 657-686 / 第 657-686 行

```cpp
0657: TEST(Q8GAVGPOOL_UP8x7__SSE2, n_eq_8_all_m_with_y_min) {
0658:   TEST_REQUIRES_X86_SSE2;
0659:   GAvgPoolMicrokernelTester()
0660:       .m(7)
0661:       .n(8)
0662:       .xZeroPoint(128)
0663:       .yZeroPoint(128)
0664:       .xScale(1.0f)
0665:       .yScale(1.0f)
0666:       .yMin(128)
0667:       .test(pytorch_q8gavgpool_ukernel_up8x7__sse2);
0668: }
0669: 
0670: TEST(Q8GAVGPOOL_UP8x7__SSE2, n_div_8_all_m) {
0671:   TEST_REQUIRES_X86_SSE2;
0672:   for (size_t n = 8; n < 128; n += 24) {
0673:     GAvgPoolMicrokernelTester().m(7).n(n).test(pytorch_q8gavgpool_ukernel_up8x7__sse2);
0674:   }
0675: }
0676: 
0677: TEST(Q8GAVGPOOL_UP8x7__SSE2, n_div_8_few_m) {
0678:   TEST_REQUIRES_X86_SSE2;
0679:   for (size_t n = 8; n < 128; n += 24) {
0680:     for (size_t m = 1; m < 7; m++) {
0681:       GAvgPoolMicrokernelTester().m(m).n(n).test(
0682:           pytorch_q8gavgpool_ukernel_up8x7__sse2);
0683:     }
0684:   }
0685: }
0686: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `GAvgPoolMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`GAvgPoolMicrokernelTester`。

### Lines 687-716 / 第 687-716 行

```cpp
0687: TEST(Q8GAVGPOOL_UP8x7__SSE2, n_gt_8_all_m) {
0688:   TEST_REQUIRES_X86_SSE2;
0689:   for (size_t n = 9; n < 16; n++) {
0690:     GAvgPoolMicrokernelTester().m(7).n(n).test(pytorch_q8gavgpool_ukernel_up8x7__sse2);
0691:   }
0692: }
0693: 
0694: TEST(Q8GAVGPOOL_UP8x7__SSE2, n_gt_8_few_m) {
0695:   TEST_REQUIRES_X86_SSE2;
0696:   for (size_t n = 9; n < 16; n++) {
0697:     for (size_t m = 1; m < 7; m++) {
0698:       GAvgPoolMicrokernelTester().m(m).n(n).test(
0699:           pytorch_q8gavgpool_ukernel_up8x7__sse2);
0700:     }
0701:   }
0702: }
0703: 
0704: TEST(Q8GAVGPOOL_UP8x7__SSE2, n_gt_8_all_m_with_x_scale) {
0705:   TEST_REQUIRES_X86_SSE2;
0706:   for (size_t n = 9; n < 16; n++) {
0707:     for (float xScale = 0.01f; xScale < 100.0f; xScale *= 3.14159265f) {
0708:       GAvgPoolMicrokernelTester().m(7).n(n).xScale(xScale).test(
0709:           pytorch_q8gavgpool_ukernel_up8x7__sse2);
0710:     }
0711:   }
0712: }
0713: 
0714: TEST(Q8GAVGPOOL_UP8x7__SSE2, n_gt_8_all_m_with_x_zero_point) {
0715:   TEST_REQUIRES_X86_SSE2;
0716:   for (size_t n = 9; n < 16; n++) {
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `GAvgPoolMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`GAvgPoolMicrokernelTester`。

### Lines 717-749 / 第 717-749 行

```cpp
0717:     for (int32_t xZeroPoint = 0; xZeroPoint <= 255; xZeroPoint += 51) {
0718:       GAvgPoolMicrokernelTester()
0719:           .m(7)
0720:           .n(n)
0721:           .xZeroPoint(xZeroPoint)
0722:           .test(pytorch_q8gavgpool_ukernel_up8x7__sse2);
0723:     }
0724:   }
0725: }
0726: 
0727: TEST(Q8GAVGPOOL_UP8x7__SSE2, n_gt_8_all_m_with_y_scale) {
0728:   TEST_REQUIRES_X86_SSE2;
0729:   for (size_t n = 9; n < 16; n++) {
0730:     for (float yScale = 0.01f; yScale < 100.0f; yScale *= 3.14159265f) {
0731:       GAvgPoolMicrokernelTester().m(7).n(n).yScale(yScale).test(
0732:           pytorch_q8gavgpool_ukernel_up8x7__sse2);
0733:     }
0734:   }
0735: }
0736: 
0737: TEST(Q8GAVGPOOL_UP8x7__SSE2, n_gt_8_all_m_with_y_zero_point) {
0738:   TEST_REQUIRES_X86_SSE2;
0739:   for (size_t n = 9; n < 16; n++) {
0740:     for (int32_t yZeroPoint = 0; yZeroPoint <= 255; yZeroPoint += 51) {
0741:       GAvgPoolMicrokernelTester()
0742:           .m(7)
0743:           .n(n)
0744:           .yZeroPoint(yZeroPoint)
0745:           .test(pytorch_q8gavgpool_ukernel_up8x7__sse2);
0746:     }
0747:   }
0748: }
0749: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `GAvgPoolMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`GAvgPoolMicrokernelTester`。

### Lines 750-779 / 第 750-779 行

```cpp
0750: TEST(Q8GAVGPOOL_UP8x7__SSE2, n_gt_8_all_m_with_y_max) {
0751:   TEST_REQUIRES_X86_SSE2;
0752:   for (size_t n = 9; n < 16; n++) {
0753:     GAvgPoolMicrokernelTester()
0754:         .m(7)
0755:         .n(n)
0756:         .xZeroPoint(128)
0757:         .yZeroPoint(128)
0758:         .xScale(1.0f)
0759:         .yScale(1.0f)
0760:         .yMax(128)
0761:         .test(pytorch_q8gavgpool_ukernel_up8x7__sse2);
0762:   }
0763: }
0764: 
0765: TEST(Q8GAVGPOOL_UP8x7__SSE2, n_gt_8_all_m_with_y_min) {
0766:   TEST_REQUIRES_X86_SSE2;
0767:   for (size_t n = 9; n < 16; n++) {
0768:     GAvgPoolMicrokernelTester()
0769:         .m(7)
0770:         .n(n)
0771:         .xZeroPoint(128)
0772:         .yZeroPoint(128)
0773:         .xScale(1.0f)
0774:         .yScale(1.0f)
0775:         .yMin(128)
0776:         .test(pytorch_q8gavgpool_ukernel_up8x7__sse2);
0777:   }
0778: }
0779: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `GAvgPoolMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`GAvgPoolMicrokernelTester`。

### Lines 780-811 / 第 780-811 行

```cpp
0780: TEST(Q8GAVGPOOL_MP8x7p7q__SSE2, n_eq_8_2pass_all_m) {
0781:   TEST_REQUIRES_X86_SSE2;
0782:   GAvgPoolMicrokernelTester().m(14).n(8).nr(8).test(
0783:       pytorch_q8gavgpool_ukernel_mp8x7p7q__sse2);
0784: }
0785: 
0786: TEST(Q8GAVGPOOL_MP8x7p7q__SSE2, n_eq_8_2pass_all_m_with_x_stride) {
0787:   TEST_REQUIRES_X86_SSE2;
0788:   GAvgPoolMicrokernelTester().m(14).n(8).nr(8).xStride(11).test(
0789:       pytorch_q8gavgpool_ukernel_mp8x7p7q__sse2);
0790: }
0791: 
0792: TEST(Q8GAVGPOOL_MP8x7p7q__SSE2, n_eq_8_2pass_all_m_with_x_scale) {
0793:   TEST_REQUIRES_X86_SSE2;
0794:   for (float xScale = 0.01f; xScale < 100.0f; xScale *= 3.14159265f) {
0795:     GAvgPoolMicrokernelTester().m(14).n(8).nr(8).xScale(xScale).test(
0796:         pytorch_q8gavgpool_ukernel_mp8x7p7q__sse2);
0797:   }
0798: }
0799: 
0800: TEST(Q8GAVGPOOL_MP8x7p7q__SSE2, n_eq_8_2pass_all_m_with_x_zero_point) {
0801:   TEST_REQUIRES_X86_SSE2;
0802:   for (int32_t xZeroPoint = 0; xZeroPoint <= 255; xZeroPoint += 51) {
0803:     GAvgPoolMicrokernelTester()
0804:         .m(14)
0805:         .n(8)
0806:         .nr(8)
0807:         .xZeroPoint(xZeroPoint)
0808:         .test(pytorch_q8gavgpool_ukernel_mp8x7p7q__sse2);
0809:   }
0810: }
0811: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `GAvgPoolMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`GAvgPoolMicrokernelTester`。

### Lines 812-845 / 第 812-845 行

```cpp
0812: TEST(Q8GAVGPOOL_MP8x7p7q__SSE2, n_eq_8_2pass_all_m_with_y_scale) {
0813:   TEST_REQUIRES_X86_SSE2;
0814:   for (float yScale = 0.01f; yScale < 100.0f; yScale *= 3.14159265f) {
0815:     GAvgPoolMicrokernelTester().m(14).n(8).nr(8).yScale(yScale).test(
0816:         pytorch_q8gavgpool_ukernel_mp8x7p7q__sse2);
0817:   }
0818: }
0819: 
0820: TEST(Q8GAVGPOOL_MP8x7p7q__SSE2, n_eq_8_2pass_all_m_with_y_zero_point) {
0821:   TEST_REQUIRES_X86_SSE2;
0822:   for (int32_t yZeroPoint = 0; yZeroPoint <= 255; yZeroPoint += 51) {
0823:     GAvgPoolMicrokernelTester()
0824:         .m(14)
0825:         .n(8)
0826:         .nr(8)
0827:         .yZeroPoint(yZeroPoint)
0828:         .test(pytorch_q8gavgpool_ukernel_mp8x7p7q__sse2);
0829:   }
0830: }
0831: 
0832: TEST(Q8GAVGPOOL_MP8x7p7q__SSE2, n_eq_8_2pass_all_m_with_y_max) {
0833:   TEST_REQUIRES_X86_SSE2;
0834:   GAvgPoolMicrokernelTester()
0835:       .m(14)
0836:       .n(8)
0837:       .nr(8)
0838:       .xZeroPoint(128)
0839:       .yZeroPoint(128)
0840:       .xScale(1.0f)
0841:       .yScale(1.0f)
0842:       .yMax(128)
0843:       .test(pytorch_q8gavgpool_ukernel_mp8x7p7q__sse2);
0844: }
0845: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `GAvgPoolMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`GAvgPoolMicrokernelTester`。

### Lines 846-875 / 第 846-875 行

```cpp
0846: TEST(Q8GAVGPOOL_MP8x7p7q__SSE2, n_eq_8_2pass_all_m_with_y_min) {
0847:   TEST_REQUIRES_X86_SSE2;
0848:   GAvgPoolMicrokernelTester()
0849:       .m(14)
0850:       .n(8)
0851:       .nr(8)
0852:       .xZeroPoint(128)
0853:       .yZeroPoint(128)
0854:       .xScale(1.0f)
0855:       .yScale(1.0f)
0856:       .yMin(128)
0857:       .test(pytorch_q8gavgpool_ukernel_mp8x7p7q__sse2);
0858: }
0859: 
0860: TEST(Q8GAVGPOOL_MP8x7p7q__SSE2, n_eq_8_2pass_few_m) {
0861:   TEST_REQUIRES_X86_SSE2;
0862:   for (size_t m = 1; m < 7; m++) {
0863:     GAvgPoolMicrokernelTester().m(7 + m).n(8).nr(8).test(
0864:         pytorch_q8gavgpool_ukernel_mp8x7p7q__sse2);
0865:   }
0866: }
0867: 
0868: TEST(Q8GAVGPOOL_MP8x7p7q__SSE2, n_eq_8_2pass_few_m_with_x_stride) {
0869:   TEST_REQUIRES_X86_SSE2;
0870:   for (size_t m = 1; m < 7; m++) {
0871:     GAvgPoolMicrokernelTester().m(7 + m).n(8).nr(8).xStride(11).test(
0872:         pytorch_q8gavgpool_ukernel_mp8x7p7q__sse2);
0873:   }
0874: }
0875: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `GAvgPoolMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`GAvgPoolMicrokernelTester`。

### Lines 876-909 / 第 876-909 行

```cpp
0876: TEST(Q8GAVGPOOL_MP8x7p7q__SSE2, n_eq_8_multipass_all_m) {
0877:   TEST_REQUIRES_X86_SSE2;
0878:   for (size_t m = 14; m <= 35; m += 7) {
0879:     GAvgPoolMicrokernelTester().m(m).n(8).nr(8).test(
0880:         pytorch_q8gavgpool_ukernel_mp8x7p7q__sse2);
0881:   }
0882: }
0883: 
0884: TEST(Q8GAVGPOOL_MP8x7p7q__SSE2, n_eq_8_multipass_all_m_with_x_stride) {
0885:   TEST_REQUIRES_X86_SSE2;
0886:   for (size_t m = 14; m <= 35; m += 7) {
0887:     GAvgPoolMicrokernelTester().m(m).n(8).nr(8).test(
0888:         pytorch_q8gavgpool_ukernel_mp8x7p7q__sse2);
0889:   }
0890: }
0891: 
0892: TEST(Q8GAVGPOOL_MP8x7p7q__SSE2, n_div_8_2pass_all_m) {
0893:   TEST_REQUIRES_X86_SSE2;
0894:   for (size_t n = 8; n < 128; n += 24) {
0895:     GAvgPoolMicrokernelTester().m(14).n(n).nr(8).test(
0896:         pytorch_q8gavgpool_ukernel_mp8x7p7q__sse2);
0897:   }
0898: }
0899: 
0900: TEST(Q8GAVGPOOL_MP8x7p7q__SSE2, n_div_8_2pass_few_m) {
0901:   TEST_REQUIRES_X86_SSE2;
0902:   for (size_t n = 8; n < 128; n += 24) {
0903:     for (size_t m = 1; m < 7; m++) {
0904:       GAvgPoolMicrokernelTester().m(7 + m).n(n).nr(8).test(
0905:           pytorch_q8gavgpool_ukernel_mp8x7p7q__sse2);
0906:     }
0907:   }
0908: }
0909: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `GAvgPoolMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`GAvgPoolMicrokernelTester`。

### Lines 910-940 / 第 910-940 行

```cpp
0910: TEST(Q8GAVGPOOL_MP8x7p7q__SSE2, n_div_8_multipass_all_m) {
0911:   TEST_REQUIRES_X86_SSE2;
0912:   for (size_t n = 8; n < 128; n += 24) {
0913:     for (size_t m = 14; m <= 35; m += 7) {
0914:       GAvgPoolMicrokernelTester().m(m).n(n).nr(8).nr(8).test(
0915:           pytorch_q8gavgpool_ukernel_mp8x7p7q__sse2);
0916:     }
0917:   }
0918: }
0919: 
0920: TEST(Q8GAVGPOOL_MP8x7p7q__SSE2, n_div_8_multipass_all_m_with_x_stride) {
0921:   TEST_REQUIRES_X86_SSE2;
0922:   for (size_t n = 8; n < 128; n += 24) {
0923:     for (size_t m = 14; m <= 35; m += 7) {
0924:       GAvgPoolMicrokernelTester().m(m).n(n).nr(8).nr(8).xStride(131).test(
0925:           pytorch_q8gavgpool_ukernel_mp8x7p7q__sse2);
0926:     }
0927:   }
0928: }
0929: 
0930: TEST(Q8GAVGPOOL_MP8x7p7q__SSE2, n_gt_8_2pass_all_m) {
0931:   TEST_REQUIRES_X86_SSE2;
0932:   for (size_t n = 9; n < 16; n++) {
0933:     GAvgPoolMicrokernelTester().m(14).n(n).nr(8).test(
0934:         pytorch_q8gavgpool_ukernel_mp8x7p7q__sse2);
0935:   }
0936: }
0937: 
0938: TEST(Q8GAVGPOOL_MP8x7p7q__SSE2, n_gt_8_2pass_all_m_with_x_scale) {
0939:   TEST_REQUIRES_X86_SSE2;
0940:   for (float xScale = 0.01f; xScale < 100.0f; xScale *= 3.14159265f) {
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `GAvgPoolMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`GAvgPoolMicrokernelTester`。

### Lines 941-971 / 第 941-971 行

```cpp
0941:     for (size_t n = 9; n < 16; n++) {
0942:       GAvgPoolMicrokernelTester().m(14).n(n).nr(8).xScale(xScale).test(
0943:           pytorch_q8gavgpool_ukernel_mp8x7p7q__sse2);
0944:     }
0945:   }
0946: }
0947: 
0948: TEST(Q8GAVGPOOL_MP8x7p7q__SSE2, n_gt_8_2pass_all_m_with_x_zero_point) {
0949:   TEST_REQUIRES_X86_SSE2;
0950:   for (int32_t xZeroPoint = 0; xZeroPoint <= 255; xZeroPoint += 51) {
0951:     for (size_t n = 9; n < 16; n++) {
0952:       GAvgPoolMicrokernelTester()
0953:           .m(14)
0954:           .n(n)
0955:           .nr(8)
0956:           .xZeroPoint(xZeroPoint)
0957:           .test(pytorch_q8gavgpool_ukernel_mp8x7p7q__sse2);
0958:     }
0959:   }
0960: }
0961: 
0962: TEST(Q8GAVGPOOL_MP8x7p7q__SSE2, n_gt_8_2pass_all_m_with_y_scale) {
0963:   TEST_REQUIRES_X86_SSE2;
0964:   for (float yScale = 0.01f; yScale < 100.0f; yScale *= 3.14159265f) {
0965:     for (size_t n = 9; n < 16; n++) {
0966:       GAvgPoolMicrokernelTester().m(14).n(n).nr(8).yScale(yScale).test(
0967:           pytorch_q8gavgpool_ukernel_mp8x7p7q__sse2);
0968:     }
0969:   }
0970: }
0971: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `GAvgPoolMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`GAvgPoolMicrokernelTester`。

### Lines 972-1001 / 第 972-1001 行

```cpp
0972: TEST(Q8GAVGPOOL_MP8x7p7q__SSE2, n_gt_8_2pass_all_m_with_y_zero_point) {
0973:   TEST_REQUIRES_X86_SSE2;
0974:   for (int32_t yZeroPoint = 0; yZeroPoint <= 255; yZeroPoint += 51) {
0975:     for (size_t n = 9; n < 16; n++) {
0976:       GAvgPoolMicrokernelTester()
0977:           .m(14)
0978:           .n(n)
0979:           .nr(8)
0980:           .yZeroPoint(yZeroPoint)
0981:           .test(pytorch_q8gavgpool_ukernel_mp8x7p7q__sse2);
0982:     }
0983:   }
0984: }
0985: 
0986: TEST(Q8GAVGPOOL_MP8x7p7q__SSE2, n_gt_8_2pass_all_m_with_y_max) {
0987:   TEST_REQUIRES_X86_SSE2;
0988:   for (size_t n = 9; n < 16; n++) {
0989:     GAvgPoolMicrokernelTester()
0990:         .m(14)
0991:         .n(n)
0992:         .nr(8)
0993:         .xZeroPoint(128)
0994:         .yZeroPoint(128)
0995:         .xScale(1.0f)
0996:         .yScale(1.0f)
0997:         .yMax(128)
0998:         .test(pytorch_q8gavgpool_ukernel_mp8x7p7q__sse2);
0999:   }
1000: }
1001: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `GAvgPoolMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`GAvgPoolMicrokernelTester`。

### Lines 1002-1031 / 第 1002-1031 行

```cpp
1002: TEST(Q8GAVGPOOL_MP8x7p7q__SSE2, n_gt_8_2pass_all_m_with_y_min) {
1003:   TEST_REQUIRES_X86_SSE2;
1004:   for (size_t n = 9; n < 16; n++) {
1005:     GAvgPoolMicrokernelTester()
1006:         .m(14)
1007:         .n(n)
1008:         .nr(8)
1009:         .xZeroPoint(128)
1010:         .yZeroPoint(128)
1011:         .xScale(1.0f)
1012:         .yScale(1.0f)
1013:         .yMin(128)
1014:         .test(pytorch_q8gavgpool_ukernel_mp8x7p7q__sse2);
1015:   }
1016: }
1017: 
1018: TEST(Q8GAVGPOOL_MP8x7p7q__SSE2, n_gt_8_2pass_few_m) {
1019:   TEST_REQUIRES_X86_SSE2;
1020:   for (size_t n = 9; n < 16; n++) {
1021:     for (size_t m = 1; m < 7; m++) {
1022:       GAvgPoolMicrokernelTester().m(7 + m).n(n).nr(8).test(
1023:           pytorch_q8gavgpool_ukernel_mp8x7p7q__sse2);
1024:     }
1025:   }
1026: }
1027: 
1028: TEST(Q8GAVGPOOL_MP8x7p7q__SSE2, n_gt_8_multipass_all_m) {
1029:   TEST_REQUIRES_X86_SSE2;
1030:   for (size_t n = 9; n < 16; n++) {
1031:     for (size_t m = 14; m <= 35; m += 7) {
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `GAvgPoolMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`GAvgPoolMicrokernelTester`。

### Lines 1032-1061 / 第 1032-1061 行

```cpp
1032:       GAvgPoolMicrokernelTester().m(m).n(n).nr(8).test(
1033:           pytorch_q8gavgpool_ukernel_mp8x7p7q__sse2);
1034:     }
1035:   }
1036: }
1037: 
1038: TEST(Q8GAVGPOOL_MP8x7p7q__SSE2, n_gt_8_multipass_all_m_with_x_stride) {
1039:   TEST_REQUIRES_X86_SSE2;
1040:   for (size_t n = 9; n < 16; n++) {
1041:     for (size_t m = 14; m <= 35; m += 7) {
1042:       GAvgPoolMicrokernelTester().m(m).n(n).nr(8).xStride(23).test(
1043:           pytorch_q8gavgpool_ukernel_mp8x7p7q__sse2);
1044:     }
1045:   }
1046: }
1047: 
1048: TEST(Q8GAVGPOOL_UP8xM__SSE2, n_lt_8_small_m) {
1049:   TEST_REQUIRES_X86_SSE2;
1050:   for (size_t n = 1; n < 8; n++) {
1051:     for (size_t m = 1; m < 8; m++) {
1052:       GAvgPoolMicrokernelTester().m(m).n(n).test(
1053:           pytorch_q8gavgpool_ukernel_up8xm__sse2);
1054:     }
1055:   }
1056: }
1057: 
1058: TEST(Q8GAVGPOOL_UP8xM__SSE2, n_lt_8_large_m) {
1059:   TEST_REQUIRES_X86_SSE2;
1060:   for (size_t n = 1; n < 8; n++) {
1061:     for (size_t m = 8; m < 16; m++) {
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `GAvgPoolMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`GAvgPoolMicrokernelTester`。

### Lines 1062-1094 / 第 1062-1094 行

```cpp
1062:       GAvgPoolMicrokernelTester().m(m).n(n).test(
1063:           pytorch_q8gavgpool_ukernel_up8xm__sse2);
1064:     }
1065:   }
1066: }
1067: 
1068: TEST(Q8GAVGPOOL_UP8xM__SSE2, n_lt_8_with_x_scale) {
1069:   TEST_REQUIRES_X86_SSE2;
1070:   for (size_t n = 1; n < 8; n++) {
1071:     for (size_t m = 1; m < 16; m += 5) {
1072:       for (float xScale = 0.01f; xScale < 100.0f; xScale *= 3.14159265f) {
1073:         GAvgPoolMicrokernelTester().m(m).n(n).xScale(xScale).test(
1074:             pytorch_q8gavgpool_ukernel_up8xm__sse2);
1075:       }
1076:     }
1077:   }
1078: }
1079: 
1080: TEST(Q8GAVGPOOL_UP8xM__SSE2, n_lt_8_with_x_zero_point) {
1081:   TEST_REQUIRES_X86_SSE2;
1082:   for (size_t n = 1; n < 8; n++) {
1083:     for (size_t m = 1; m < 16; m += 5) {
1084:       for (int32_t xZeroPoint = 0; xZeroPoint <= 255; xZeroPoint += 51) {
1085:         GAvgPoolMicrokernelTester()
1086:             .m(m)
1087:             .n(n)
1088:             .xZeroPoint(xZeroPoint)
1089:             .test(pytorch_q8gavgpool_ukernel_up8xm__sse2);
1090:       }
1091:     }
1092:   }
1093: }
1094: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `GAvgPoolMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`GAvgPoolMicrokernelTester`。

### Lines 1095-1124 / 第 1095-1124 行

```cpp
1095: TEST(Q8GAVGPOOL_UP8xM__SSE2, n_lt_8_with_y_scale) {
1096:   TEST_REQUIRES_X86_SSE2;
1097:   for (size_t n = 1; n < 8; n++) {
1098:     for (size_t m = 1; m < 16; m += 5) {
1099:       for (float yScale = 0.01f; yScale < 100.0f; yScale *= 3.14159265f) {
1100:         GAvgPoolMicrokernelTester().m(m).n(n).yScale(yScale).test(
1101:             pytorch_q8gavgpool_ukernel_up8xm__sse2);
1102:       }
1103:     }
1104:   }
1105: }
1106: 
1107: TEST(Q8GAVGPOOL_UP8xM__SSE2, n_lt_8_with_y_zero_point) {
1108:   TEST_REQUIRES_X86_SSE2;
1109:   for (size_t n = 1; n < 8; n++) {
1110:     for (size_t m = 1; m < 16; m += 5) {
1111:       for (int32_t yZeroPoint = 0; yZeroPoint <= 255; yZeroPoint += 51) {
1112:         GAvgPoolMicrokernelTester()
1113:             .m(m)
1114:             .n(n)
1115:             .yZeroPoint(yZeroPoint)
1116:             .test(pytorch_q8gavgpool_ukernel_up8xm__sse2);
1117:       }
1118:     }
1119:   }
1120: }
1121: 
1122: TEST(Q8GAVGPOOL_UP8xM__SSE2, n_lt_8_with_y_max) {
1123:   TEST_REQUIRES_X86_SSE2;
1124:   for (size_t n = 1; n < 8; n++) {
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `GAvgPoolMicrokernelTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`GAvgPoolMicrokernelTester`。

### Lines 1125-1155 / 第 1125-1155 行

```cpp
1125:     for (size_t m = 1; m < 16; m += 5) {
1126:       GAvgPoolMicrokernelTester()
1127:           .m(m)
1128:           .n(n)
1129:           .xZeroPoint(128)
1130:           .yZeroPoint(128)
1131:           .xScale(1.0f)
1132:           .yScale(1.0f)
1133:           .yMax(128)
1134:           .test(pytorch_q8gavgpool_ukernel_up8xm__sse2);
1135:     }
1136:   }
1137: }
1138: 
1139: TEST(Q8GAVGPOOL_UP8xM__SSE2, n_lt_8_with_y_min) {
1140:   TEST_REQUIRES_X86_SSE2;
1141:   for (size_t n = 1; n < 8; n++) {
1142:     for (size_t m = 1; m < 16; m += 5) {
1143:       GAvgPoolMicrokernelTester()
1144:           .m(m)
1145:           .n(n)
1146:           .xZeroPoint(128)
1147:           .yZeroPoint(128)
1148:           .xScale(1.0f)
1149:           .yScale(1.0f)
1150:           .yMin(128)
1151:           .test(pytorch_q8gavgpool_ukernel_up8xm__sse2);
1152:     }
1153:   }
1154: }
1155: #endif /* CPUINFO_ARCH_X86 || CPUINFO_ARCH_X86_64 */
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends; iterates over tensors, descriptors, options, or runtime state. Key symbols: `GAvgPoolMicrokernelTester`.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端；遍历张量、描述符、配置项或运行时状态。关键符号：`GAvgPoolMicrokernelTester`。


## Key Concepts / 关键概念
- **ATen native operator logic** — ATen 原生算子逻辑
- **GPU kernel/runtime interaction** — GPU 内核/运行时交互
- **Runtime validation and invariants** — 运行时校验与不变量
- **Core symbols: GAvgPoolMicrokernelTester** — 核心符号：GAvgPoolMicrokernelTester

## Dependencies / 依赖关系
- **Internal includes / 内部头文件**: 无
- **External includes / 外部头文件**: `cpuinfo.h`, `gtest/gtest.h`, `qnnpack/isa-checks.h`, `qnnpack/q8gavgpool.h`, `gavgpool-microkernel-tester.h`
- **Namespaces / 命名空间**: 无
- **Representative symbols / 代表性符号**: `GAvgPoolMicrokernelTester`
