# convolution.cc — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `aten/src/ATen/native/quantized/cpu/qnnpack/test/convolution.cc`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements ATen native operators, checks, helper kernels, and backend dispatch behavior. This specific file centers on `convolution.cc`. The file header highlights: "All rights reserved. This source code is licensed under the BSD-style license found in the LICENSE file in the root directory of this source tree.."
- **Purpose (CN)**: 实现 ATen 原生算子、运行时检查、辅助内核以及后端分发行为。 该文件具体围绕 `convolution.cc` 展开。 文件头部注释也概括了其核心职责。

## Line-by-Line Analysis / 逐行分析
### Lines 1-44 / 第 1-44 行

```cpp
0001: /*
0002:  * Copyright (c) Facebook, Inc. and its affiliates.
0003:  * All rights reserved.
0004:  *
0005:  * This source code is licensed under the BSD-style license found in the
0006:  * LICENSE file in the root directory of this source tree.
0007:  */
0008: 
0009: #include <gtest/gtest.h>
0010: 
0011: #include <qnnpack/params.h>
0012: 
0013: #include "convolution-operator-tester.h"
0014: 
0015: using namespace qnnpack::testing;
0016: 
0017: _STATIC_AND_RUNTIME_TEST(CONVOLUTION_OP, zero_batch,
0018:   ConvolutionOperatorTester()
0019:       .batchSize(0)
0020:       .inputSize(5, 5)
0021:       .kernelSize(1, 1)
0022:       .groupInputChannels(2)
0023:       .groupOutputChannels(2)
0024:       .iterations(1)
0025: )
0026: 
0027: _STATIC_AND_RUNTIME_TEST(CONVOLUTION_OP, 1x1,
0028:   ConvolutionOperatorTester()
0029:       .inputSize(27, 29)
0030:       .kernelSize(1, 1)
0031:       .groupInputChannels(23)
0032:       .groupOutputChannels(19)
0033:       .iterations(3)
0034: )
0035: 
0036: _STATIC_AND_RUNTIME_TEST(CONVOLUTION_OP, 1x1_runtime_quant,
0037:   ConvolutionOperatorTester()
0038:       .inputSize(27, 29)
0039:       .kernelSize(1, 1)
0040:       .groupInputChannels(23)
0041:       .groupOutputChannels(19)
0042:       .iterations(3)
0043: )
0044: 
```

- **EN:** This block implements local helper logic for `convolution`. Key symbols: no prominent local symbols.
- **CN:** 该代码块实现与 `convolution` 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 45-84 / 第 45-84 行

```cpp
0045: _STATIC_AND_RUNTIME_TEST(CONVOLUTION_OP, 1x1_with_qmin,
0046:   ConvolutionOperatorTester()
0047:       .inputSize(27, 29)
0048:       .kernelSize(1, 1)
0049:       .groupInputChannels(23)
0050:       .groupOutputChannels(19)
0051:       .qmin(128)
0052:       .iterations(3)
0053: )
0054: 
0055: _STATIC_AND_RUNTIME_TEST(CONVOLUTION_OP, 1x1_with_qmax,
0056:   ConvolutionOperatorTester()
0057:       .inputSize(27, 29)
0058:       .kernelSize(1, 1)
0059:       .groupInputChannels(23)
0060:       .groupOutputChannels(19)
0061:       .qmax(128)
0062:       .iterations(3)
0063: )
0064: 
0065: _STATIC_TEST(CONVOLUTION_OP, 1x1_with_input_stride,
0066:   ConvolutionOperatorTester()
0067:       .inputSize(27, 29)
0068:       .kernelSize(1, 1)
0069:       .inputPixelStride(28)
0070:       .groupInputChannels(23)
0071:       .groupOutputChannels(19)
0072:       .iterations(3)
0073: )
0074: 
0075: _STATIC_TEST(CONVOLUTION_OP, 1x1_with_output_stride,
0076:   ConvolutionOperatorTester()
0077:       .inputSize(27, 29)
0078:       .kernelSize(1, 1)
0079:       .outputPixelStride(29)
0080:       .groupInputChannels(23)
0081:       .groupOutputChannels(19)
0082:       .iterations(3)
0083: )
0084: 
```

- **EN:** This block implements local helper logic for `convolution`. Key symbols: no prominent local symbols.
- **CN:** 该代码块实现与 `convolution` 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 85-131 / 第 85-131 行

```cpp
0085: _STATIC_AND_RUNTIME_TEST(CONVOLUTION_OP, 1x1_with_batch,
0086:   ConvolutionOperatorTester()
0087:       .inputSize(13, 14)
0088:       .kernelSize(1, 1)
0089:       .batchSize(3)
0090:       .groupInputChannels(23)
0091:       .groupOutputChannels(19)
0092:       .iterations(3)
0093: )
0094: 
0095: _STATIC_AND_RUNTIME_TEST(CONVOLUTION_OP, grouped_1x1,
0096:   ConvolutionOperatorTester()
0097:       .inputSize(24, 25)
0098:       .kernelSize(1, 1)
0099:       .groups(2)
0100:       .groupInputChannels(17)
0101:       .groupOutputChannels(19)
0102:       .iterations(3)
0103: )
0104: 
0105: TEST(CONVOLUTION_OP, xzp_1x1) {
0106:   ASSERT_EQ(pytorch_qnnp_status_success, pytorch_qnnp_initialize());
0107:   if (pytorch_qnnp_params.q8conv_xzp.kthreshold != SIZE_MAX) {
0108:     ConvolutionOperatorTester()
0109:         .inputSize(27, 29)
0110:         .kernelSize(1, 1)
0111:         .groupInputChannels(pytorch_qnnp_params.q8conv_xzp.kthreshold + 1)
0112:         .groupOutputChannels(19)
0113:         .iterations(3)
0114:         .testQ8();
0115:   }
0116: }
0117: 
0118: TEST(CONVOLUTION_OP, xzp_1x1_with_qmin) {
0119:   ASSERT_EQ(pytorch_qnnp_status_success, pytorch_qnnp_initialize());
0120:   if (pytorch_qnnp_params.q8conv_xzp.kthreshold != SIZE_MAX) {
0121:     ConvolutionOperatorTester()
0122:         .inputSize(27, 29)
0123:         .kernelSize(1, 1)
0124:         .groupInputChannels(pytorch_qnnp_params.q8conv_xzp.kthreshold + 1)
0125:         .groupOutputChannels(19)
0126:         .qmin(128)
0127:         .iterations(3)
0128:         .testQ8();
0129:   }
0130: }
0131: 
```

- **EN:** This block handles conditional branches and special cases. Key symbols: `ConvolutionOperatorTester`.
- **CN:** 该代码块处理条件分支与特殊情况。关键符号：`ConvolutionOperatorTester`。

### Lines 132-173 / 第 132-173 行

```cpp
0132: TEST(CONVOLUTION_OP, xzp_1x1_with_qmax) {
0133:   ASSERT_EQ(pytorch_qnnp_status_success, pytorch_qnnp_initialize());
0134:   if (pytorch_qnnp_params.q8conv_xzp.kthreshold != SIZE_MAX) {
0135:     ConvolutionOperatorTester()
0136:         .inputSize(27, 29)
0137:         .kernelSize(1, 1)
0138:         .groupInputChannels(pytorch_qnnp_params.q8conv_xzp.kthreshold + 1)
0139:         .groupOutputChannels(19)
0140:         .qmax(128)
0141:         .iterations(3)
0142:         .testQ8();
0143:   }
0144: }
0145: 
0146: TEST(CONVOLUTION_OP, xzp_1x1_with_input_stride) {
0147:   ASSERT_EQ(pytorch_qnnp_status_success, pytorch_qnnp_initialize());
0148:   if (pytorch_qnnp_params.q8conv_xzp.kthreshold != SIZE_MAX) {
0149:     ConvolutionOperatorTester()
0150:         .inputSize(27, 29)
0151:         .kernelSize(1, 1)
0152:         .inputPixelStride(pytorch_qnnp_params.q8conv_xzp.kthreshold + 5)
0153:         .groupInputChannels(pytorch_qnnp_params.q8conv_xzp.kthreshold + 1)
0154:         .groupOutputChannels(19)
0155:         .iterations(3)
0156:         .testQ8();
0157:   }
0158: }
0159: 
0160: TEST(CONVOLUTION_OP, xzp_1x1_with_output_stride) {
0161:   ASSERT_EQ(pytorch_qnnp_status_success, pytorch_qnnp_initialize());
0162:   if (pytorch_qnnp_params.q8conv_xzp.kthreshold != SIZE_MAX) {
0163:     ConvolutionOperatorTester()
0164:         .inputSize(27, 29)
0165:         .kernelSize(1, 1)
0166:         .outputPixelStride(29)
0167:         .groupInputChannels(pytorch_qnnp_params.q8conv_xzp.kthreshold + 1)
0168:         .groupOutputChannels(19)
0169:         .iterations(3)
0170:         .testQ8();
0171:   }
0172: }
0173: 
```

- **EN:** This block handles conditional branches and special cases. Key symbols: `ConvolutionOperatorTester`.
- **CN:** 该代码块处理条件分支与特殊情况。关键符号：`ConvolutionOperatorTester`。

### Lines 174-215 / 第 174-215 行

```cpp
0174: TEST(CONVOLUTION_OP, xzp_1x1_with_batch) {
0175:   ASSERT_EQ(pytorch_qnnp_status_success, pytorch_qnnp_initialize());
0176:   if (pytorch_qnnp_params.q8conv_xzp.kthreshold != SIZE_MAX) {
0177:     ConvolutionOperatorTester()
0178:         .inputSize(13, 14)
0179:         .kernelSize(1, 1)
0180:         .batchSize(3)
0181:         .groupInputChannels(pytorch_qnnp_params.q8conv_xzp.kthreshold + 1)
0182:         .groupOutputChannels(19)
0183:         .iterations(3)
0184:         .testQ8();
0185:   }
0186: }
0187: 
0188: TEST(CONVOLUTION_OP, grouped_xzp_1x1) {
0189:   ASSERT_EQ(pytorch_qnnp_status_success, pytorch_qnnp_initialize());
0190:   if (pytorch_qnnp_params.q8conv_xzp.kthreshold != SIZE_MAX) {
0191:     ConvolutionOperatorTester()
0192:         .inputSize(24, 25)
0193:         .kernelSize(1, 1)
0194:         .groups(2)
0195:         .groupInputChannels(pytorch_qnnp_params.q8conv_xzp.kthreshold + 1)
0196:         .groupOutputChannels(19)
0197:         .iterations(3)
0198:         .testQ8();
0199:   }
0200: }
0201: 
0202: TEST(CONVOLUTION_OP, grouped_xzp_1x1_runtime_quant) {
0203:   ASSERT_EQ(pytorch_qnnp_status_success, pytorch_qnnp_initialize());
0204:   if (pytorch_qnnp_params.q8conv_xzp.kthreshold != SIZE_MAX) {
0205:     ConvolutionOperatorTester()
0206:         .inputSize(24, 25)
0207:         .kernelSize(1, 1)
0208:         .groups(2)
0209:         .groupInputChannels(pytorch_qnnp_params.q8conv_xzp.kthreshold + 1)
0210:         .groupOutputChannels(19)
0211:         .iterations(3)
0212:         .testQ8(Mode::Runtime);
0213:   }
0214: }
0215: 
```

- **EN:** This block handles conditional branches and special cases. Key symbols: `ConvolutionOperatorTester`.
- **CN:** 该代码块处理条件分支与特殊情况。关键符号：`ConvolutionOperatorTester`。

### Lines 216-257 / 第 216-257 行

```cpp
0216: _STATIC_AND_RUNTIME_TEST(CONVOLUTION_OP, 1x3,
0217:   ConvolutionOperatorTester()
0218:       .inputSize(20, 19)
0219:       .paddingWidth(1)
0220:       .kernelSize(1, 3)
0221:       .groupInputChannels(17)
0222:       .groupOutputChannels(15)
0223:       .iterations(3)
0224: )
0225: 
0226: _STATIC_AND_RUNTIME_TEST(CONVOLUTION_OP, grouped_1x3,
0227:   ConvolutionOperatorTester()
0228:       .inputSize(20, 19)
0229:       .paddingWidth(1)
0230:       .kernelSize(1, 3)
0231:       .groups(2)
0232:       .groupInputChannels(17)
0233:       .groupOutputChannels(15)
0234:       .iterations(3)
0235: )
0236: 
0237: _STATIC_AND_RUNTIME_TEST(CONVOLUTION_OP, 3x1,
0238:   ConvolutionOperatorTester()
0239:       .inputSize(19, 20)
0240:       .paddingHeight(1)
0241:       .kernelSize(3, 1)
0242:       .groupInputChannels(17)
0243:       .groupOutputChannels(15)
0244:       .iterations(3)
0245: )
0246: 
0247: _STATIC_AND_RUNTIME_TEST(CONVOLUTION_OP, grouped_3x1,
0248:   ConvolutionOperatorTester()
0249:       .inputSize(19, 20)
0250:       .paddingHeight(1)
0251:       .kernelSize(3, 1)
0252:       .groups(2)
0253:       .groupInputChannels(17)
0254:       .groupOutputChannels(15)
0255:       .iterations(3)
0256: )
0257: 
```

- **EN:** This block implements local helper logic for `convolution`. Key symbols: no prominent local symbols.
- **CN:** 该代码块实现与 `convolution` 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 258-298 / 第 258-298 行

```cpp
0258: _STATIC_AND_RUNTIME_TEST(CONVOLUTION_OP, 3x3,
0259:   ConvolutionOperatorTester()
0260:       .inputSize(13, 12)
0261:       .padding(1)
0262:       .kernelSize(3, 3)
0263:       .groupInputChannels(15)
0264:       .groupOutputChannels(17)
0265:       .iterations(3)
0266: )
0267: 
0268: _STATIC_AND_RUNTIME_TEST(CONVOLUTION_OP, 3x3_without_padding,
0269:   ConvolutionOperatorTester()
0270:       .inputSize(13, 12)
0271:       .kernelSize(3, 3)
0272:       .groupInputChannels(15)
0273:       .groupOutputChannels(17)
0274:       .iterations(3)
0275: )
0276: 
0277: _STATIC_AND_RUNTIME_TEST(
0278:     CONVOLUTION_OP,
0279:     3x3_with_width_padding,
0280:     ConvolutionOperatorTester()
0281:         .inputSize(13, 12)
0282:         .paddingWidth(1)
0283:         .kernelSize(3, 3)
0284:         .groupInputChannels(15)
0285:         .groupOutputChannels(17)
0286:         .iterations(3))
0287: 
0288: _STATIC_AND_RUNTIME_TEST(
0289:     CONVOLUTION_OP,
0290:     3x3_with_height_padding,
0291:     ConvolutionOperatorTester()
0292:         .inputSize(13, 12)
0293:         .paddingHeight(1)
0294:         .kernelSize(3, 3)
0295:         .groupInputChannels(15)
0296:         .groupOutputChannels(17)
0297:         .iterations(3))
0298: 
```

- **EN:** This block implements local helper logic for `convolution`. Key symbols: no prominent local symbols.
- **CN:** 该代码块实现与 `convolution` 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 299-342 / 第 299-342 行

```cpp
0299: _STATIC_TEST(CONVOLUTION_OP, 3x3_with_input_stride,
0300:   ConvolutionOperatorTester()
0301:       .inputSize(13, 12)
0302:       .padding(1)
0303:       .kernelSize(3, 3)
0304:       .inputPixelStride(22)
0305:       .groupInputChannels(15)
0306:       .groupOutputChannels(17)
0307:       .iterations(3)
0308: )
0309: 
0310: _STATIC_TEST(CONVOLUTION_OP, 3x3_with_output_stride,
0311:   ConvolutionOperatorTester()
0312:       .inputSize(13, 12)
0313:       .padding(1)
0314:       .kernelSize(3, 3)
0315:       .outputPixelStride(23)
0316:       .groupInputChannels(15)
0317:       .groupOutputChannels(17)
0318:       .iterations(3)
0319: )
0320: 
0321: _STATIC_AND_RUNTIME_TEST(CONVOLUTION_OP, 3x3_with_batch,
0322:   ConvolutionOperatorTester()
0323:       .inputSize(10, 9)
0324:       .padding(1)
0325:       .kernelSize(3, 3)
0326:       .batchSize(3)
0327:       .groupInputChannels(15)
0328:       .groupOutputChannels(17)
0329:       .iterations(3)
0330: )
0331: 
0332: _STATIC_AND_RUNTIME_TEST(CONVOLUTION_OP, grouped_3x3,
0333:   ConvolutionOperatorTester()
0334:       .inputSize(10, 11)
0335:       .padding(1)
0336:       .kernelSize(3, 3)
0337:       .groups(2)
0338:       .groupInputChannels(14)
0339:       .groupOutputChannels(13)
0340:       .iterations(3)
0341: )
0342: 
```

- **EN:** This block implements local helper logic for `convolution`. Key symbols: no prominent local symbols.
- **CN:** 该代码块实现与 `convolution` 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 343-386 / 第 343-386 行

```cpp
0343: _STATIC_AND_RUNTIME_TEST(CONVOLUTION_OP, 3x3s2,
0344:   ConvolutionOperatorTester()
0345:       .inputSize(19, 21)
0346:       .padding(1)
0347:       .kernelSize(3, 3)
0348:       .subsampling(2)
0349:       .groupInputChannels(27)
0350:       .groupOutputChannels(19)
0351:       .iterations(3)
0352: )
0353: 
0354: _STATIC_AND_RUNTIME_TEST(CONVOLUTION_OP, 3x3s1x2,
0355:   ConvolutionOperatorTester()
0356:       .inputSize(13, 13)
0357:       .padding(1)
0358:       .kernelSize(3, 3)
0359:       .subsampling(1, 2)
0360:       .groupInputChannels(27)
0361:       .groupOutputChannels(19)
0362:       .iterations(3)
0363: )
0364: 
0365: _STATIC_AND_RUNTIME_TEST(CONVOLUTION_OP, 3x3s2x1,
0366:   ConvolutionOperatorTester()
0367:       .inputSize(13, 13)
0368:       .padding(1)
0369:       .kernelSize(3, 3)
0370:       .subsampling(2, 1)
0371:       .groupInputChannels(27)
0372:       .groupOutputChannels(19)
0373:       .iterations(3)
0374: )
0375: 
0376: _STATIC_AND_RUNTIME_TEST(CONVOLUTION_OP, 3x3d2,
0377:   ConvolutionOperatorTester()
0378:       .inputSize(13, 14)
0379:       .padding(2)
0380:       .kernelSize(3, 3)
0381:       .dilation(2)
0382:       .groupInputChannels(27)
0383:       .groupOutputChannels(19)
0384:       .iterations(3)
0385: )
0386: 
```

- **EN:** This block implements local helper logic for `convolution`. Key symbols: no prominent local symbols.
- **CN:** 该代码块实现与 `convolution` 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 387-427 / 第 387-427 行

```cpp
0387: _STATIC_AND_RUNTIME_TEST(CONVOLUTION_OP, 3x3d1x2,
0388:   ConvolutionOperatorTester()
0389:       .inputSize(14, 15)
0390:       .padding(1, 2)
0391:       .kernelSize(3, 3)
0392:       .dilation(1, 2)
0393:       .groupInputChannels(27)
0394:       .groupOutputChannels(19)
0395:       .iterations(3)
0396: )
0397: 
0398: _STATIC_AND_RUNTIME_TEST(CONVOLUTION_OP, 3x3d2x1,
0399:   ConvolutionOperatorTester()
0400:       .inputSize(15, 14)
0401:       .padding(2, 1)
0402:       .kernelSize(3, 3)
0403:       .dilation(2, 1)
0404:       .groupInputChannels(27)
0405:       .groupOutputChannels(19)
0406:       .iterations(3)
0407: )
0408: 
0409: _STATIC_AND_RUNTIME_TEST(CONVOLUTION_OP, depthwise_3x3,
0410:   ConvolutionOperatorTester()
0411:       .inputSize(15, 14)
0412:       .padding(1, 1)
0413:       .kernelSize(3, 3)
0414:       .groups(27)
0415:       .iterations(3)
0416: )
0417: 
0418: _STATIC_AND_RUNTIME_TEST(CONVOLUTION_OP, depthwise_3x3s2,
0419:   ConvolutionOperatorTester()
0420:       .inputSize(15, 14)
0421:       .padding(1, 1)
0422:       .kernelSize(3, 3)
0423:       .subsampling(2)
0424:       .groups(27)
0425:       .iterations(3)
0426: )
0427: 
```

- **EN:** This block implements local helper logic for `convolution`. Key symbols: no prominent local symbols.
- **CN:** 该代码块实现与 `convolution` 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 428-467 / 第 428-467 行

```cpp
0428: _STATIC_AND_RUNTIME_TEST(CONVOLUTION_OP, depthwise_3x3s1x2,
0429:   ConvolutionOperatorTester()
0430:       .inputSize(15, 14)
0431:       .padding(1, 1)
0432:       .kernelSize(3, 3)
0433:       .subsampling(1, 2)
0434:       .groups(27)
0435:       .iterations(3)
0436: )
0437: 
0438: _STATIC_AND_RUNTIME_TEST(CONVOLUTION_OP, depthwise_3x3s2x1,
0439:   ConvolutionOperatorTester()
0440:       .inputSize(15, 14)
0441:       .padding(1, 1)
0442:       .kernelSize(3, 3)
0443:       .subsampling(2, 1)
0444:       .groups(27)
0445:       .iterations(3)
0446: )
0447: 
0448: _STATIC_AND_RUNTIME_TEST(CONVOLUTION_OP, depthwise_3x3d2,
0449:   ConvolutionOperatorTester()
0450:       .inputSize(15, 14)
0451:       .padding(1, 1)
0452:       .kernelSize(3, 3)
0453:       .dilation(2)
0454:       .groups(27)
0455:       .iterations(3)
0456: )
0457: 
0458: _STATIC_AND_RUNTIME_TEST(CONVOLUTION_OP, depthwise_3x3d1x2,
0459:   ConvolutionOperatorTester()
0460:       .inputSize(15, 14)
0461:       .padding(1, 1)
0462:       .kernelSize(3, 3)
0463:       .dilation(1, 2)
0464:       .groups(27)
0465:       .iterations(3)
0466: )
0467: 
```

- **EN:** This block implements local helper logic for `convolution`. Key symbols: no prominent local symbols.
- **CN:** 该代码块实现与 `convolution` 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 468-516 / 第 468-516 行

```cpp
0468: _STATIC_AND_RUNTIME_TEST(CONVOLUTION_OP, depthwise_3x3d2x1,
0469:   ConvolutionOperatorTester()
0470:       .inputSize(15, 14)
0471:       .padding(1, 1)
0472:       .kernelSize(3, 3)
0473:       .dilation(2, 1)
0474:       .groups(27)
0475:       .iterations(3)
0476: )
0477: 
0478: _STATIC_AND_RUNTIME_TEST(CONVOLUTION_OP, depthwise_5x5,
0479:   ConvolutionOperatorTester()
0480:       .inputSize(15, 14)
0481:       .padding(2, 2)
0482:       .kernelSize(5, 5)
0483:       .groups(27)
0484:       .iterations(3)
0485: )
0486: 
0487: _STATIC_AND_RUNTIME_TEST(CONVOLUTION_OP, depthwise_5x5s2,
0488:   ConvolutionOperatorTester()
0489:       .inputSize(15, 14)
0490:       .padding(2, 2)
0491:       .kernelSize(5, 5)
0492:       .subsampling(2)
0493:       .groups(27)
0494:       .iterations(3)
0495: )
0496: 
0497: _STATIC_AND_RUNTIME_TEST(CONVOLUTION_OP, depthwise_5x5s1x2,
0498:   ConvolutionOperatorTester()
0499:       .inputSize(15, 14)
0500:       .padding(2, 2)
0501:       .kernelSize(5, 5)
0502:       .subsampling(1, 2)
0503:       .groups(27)
0504:       .iterations(3)
0505: )
0506: 
0507: _STATIC_AND_RUNTIME_TEST(CONVOLUTION_OP, depthwise_5x5s2x1,
0508:   ConvolutionOperatorTester()
0509:       .inputSize(15, 14)
0510:       .padding(2, 2)
0511:       .kernelSize(5, 5)
0512:       .subsampling(2, 1)
0513:       .groups(27)
0514:       .iterations(3)
0515: )
0516: 
```

- **EN:** This block implements local helper logic for `convolution`. Key symbols: no prominent local symbols.
- **CN:** 该代码块实现与 `convolution` 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 517-557 / 第 517-557 行

```cpp
0517: _STATIC_AND_RUNTIME_TEST(CONVOLUTION_OP, depthwise_5x5d2,
0518:   ConvolutionOperatorTester()
0519:       .inputSize(15, 14)
0520:       .padding(2, 2)
0521:       .kernelSize(5, 5)
0522:       .dilation(2)
0523:       .groups(27)
0524:       .iterations(3)
0525: )
0526: 
0527: _STATIC_AND_RUNTIME_TEST(CONVOLUTION_OP, depthwise_5x5d1x2,
0528:   ConvolutionOperatorTester()
0529:       .inputSize(15, 14)
0530:       .padding(2, 2)
0531:       .kernelSize(5, 5)
0532:       .dilation(1, 2)
0533:       .groups(27)
0534:       .iterations(3)
0535: )
0536: 
0537: _STATIC_AND_RUNTIME_TEST(CONVOLUTION_OP, depthwise_5x5d2x1,
0538:   ConvolutionOperatorTester()
0539:       .inputSize(15, 14)
0540:       .padding(2, 2)
0541:       .kernelSize(5, 5)
0542:       .dilation(2, 1)
0543:       .groups(27)
0544:       .iterations(3)
0545: )
0546: 
0547: _STATIC_AND_RUNTIME_TEST(CONVOLUTION_OP, zero_batch_per_channel,
0548:   ConvolutionOperatorTester()
0549:       .batchSize(0)
0550:       .inputSize(5, 5)
0551:       .kernelSize(1, 1)
0552:       .groupInputChannels(2)
0553:       .groupOutputChannels(2)
0554:       .iterations(1)
0555:       .per_channel(true)
0556: )
0557: 
```

- **EN:** This block implements local helper logic for `convolution`. Key symbols: no prominent local symbols.
- **CN:** 该代码块实现与 `convolution` 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 558-600 / 第 558-600 行

```cpp
0558: _STATIC_AND_RUNTIME_TEST(CONVOLUTION_OP, 1x1_per_channel,
0559:   ConvolutionOperatorTester()
0560:       .inputSize(27, 29)
0561:       .kernelSize(1, 1)
0562:       .groupInputChannels(23)
0563:       .groupOutputChannels(19)
0564:       .iterations(3)
0565:       .per_channel(true)
0566: )
0567: 
0568: _STATIC_AND_RUNTIME_TEST(CONVOLUTION_OP, 1x1_with_qmin_per_channel,
0569:   ConvolutionOperatorTester()
0570:       .inputSize(27, 29)
0571:       .kernelSize(1, 1)
0572:       .groupInputChannels(23)
0573:       .groupOutputChannels(19)
0574:       .qmin(128)
0575:       .iterations(3)
0576:       .per_channel(true)
0577: )
0578: 
0579: _STATIC_AND_RUNTIME_TEST(CONVOLUTION_OP, 1x1_with_qmax_per_channel,
0580:   ConvolutionOperatorTester()
0581:       .inputSize(27, 29)
0582:       .kernelSize(1, 1)
0583:       .groupInputChannels(23)
0584:       .groupOutputChannels(19)
0585:       .qmax(128)
0586:       .iterations(3)
0587:       .per_channel(true)
0588: )
0589: 
0590: _STATIC_TEST(CONVOLUTION_OP, 1x1_with_input_stride_per_channel,
0591:   ConvolutionOperatorTester()
0592:       .inputSize(27, 29)
0593:       .kernelSize(1, 1)
0594:       .inputPixelStride(28)
0595:       .groupInputChannels(23)
0596:       .groupOutputChannels(19)
0597:       .iterations(3)
0598:       .per_channel(true)
0599: )
0600: 
```

- **EN:** This block implements local helper logic for `convolution`. Key symbols: no prominent local symbols.
- **CN:** 该代码块实现与 `convolution` 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 601-647 / 第 601-647 行

```cpp
0601: _STATIC_TEST(CONVOLUTION_OP, 1x1_with_output_stride_per_channel,
0602:   ConvolutionOperatorTester()
0603:       .inputSize(27, 29)
0604:       .kernelSize(1, 1)
0605:       .outputPixelStride(29)
0606:       .groupInputChannels(23)
0607:       .groupOutputChannels(19)
0608:       .iterations(3)
0609:       .per_channel(true)
0610: )
0611: 
0612: _STATIC_AND_RUNTIME_TEST(CONVOLUTION_OP, 1x1_with_batch_per_channel,
0613:   ConvolutionOperatorTester()
0614:       .inputSize(13, 14)
0615:       .kernelSize(1, 1)
0616:       .batchSize(3)
0617:       .groupInputChannels(23)
0618:       .groupOutputChannels(19)
0619:       .iterations(3)
0620:       .per_channel(true)
0621: )
0622: 
0623: _STATIC_AND_RUNTIME_TEST(CONVOLUTION_OP, grouped_1x1_per_channel,
0624:   ConvolutionOperatorTester()
0625:       .inputSize(24, 25)
0626:       .kernelSize(1, 1)
0627:       .groups(2)
0628:       .groupInputChannels(17)
0629:       .groupOutputChannels(19)
0630:       .iterations(3)
0631:       .per_channel(true)
0632: )
0633: 
0634: TEST(CONVOLUTION_OP, xzp_1x1_per_channel) {
0635:   ASSERT_EQ(pytorch_qnnp_status_success, pytorch_qnnp_initialize());
0636:   if (pytorch_qnnp_params.q8conv_xzp.kthreshold != SIZE_MAX) {
0637:     ConvolutionOperatorTester()
0638:         .inputSize(27, 29)
0639:         .kernelSize(1, 1)
0640:         .groupInputChannels(pytorch_qnnp_params.q8conv_xzp.kthreshold + 1)
0641:         .groupOutputChannels(19)
0642:         .iterations(3)
0643:         .per_channel(true)
0644:         .testQ8();
0645:   }
0646: }
0647: 
```

- **EN:** This block handles conditional branches and special cases. Key symbols: `ConvolutionOperatorTester`.
- **CN:** 该代码块处理条件分支与特殊情况。关键符号：`ConvolutionOperatorTester`。

### Lines 648-692 / 第 648-692 行

```cpp
0648: TEST(CONVOLUTION_OP, xzp_1x1_with_qmin_per_channel) {
0649:   ASSERT_EQ(pytorch_qnnp_status_success, pytorch_qnnp_initialize());
0650:   if (pytorch_qnnp_params.q8conv_xzp.kthreshold != SIZE_MAX) {
0651:     ConvolutionOperatorTester()
0652:         .inputSize(27, 29)
0653:         .kernelSize(1, 1)
0654:         .groupInputChannels(pytorch_qnnp_params.q8conv_xzp.kthreshold + 1)
0655:         .groupOutputChannels(19)
0656:         .qmin(128)
0657:         .iterations(3)
0658:         .per_channel(true)
0659:         .testQ8();
0660:   }
0661: }
0662: 
0663: TEST(CONVOLUTION_OP, xzp_1x1_with_qmax_per_channel) {
0664:   ASSERT_EQ(pytorch_qnnp_status_success, pytorch_qnnp_initialize());
0665:   if (pytorch_qnnp_params.q8conv_xzp.kthreshold != SIZE_MAX) {
0666:     ConvolutionOperatorTester()
0667:         .inputSize(27, 29)
0668:         .kernelSize(1, 1)
0669:         .groupInputChannels(pytorch_qnnp_params.q8conv_xzp.kthreshold + 1)
0670:         .groupOutputChannels(19)
0671:         .qmax(128)
0672:         .iterations(3)
0673:         .per_channel(true)
0674:         .testQ8();
0675:   }
0676: }
0677: 
0678: TEST(CONVOLUTION_OP, xzp_1x1_with_input_stride_per_channel) {
0679:   ASSERT_EQ(pytorch_qnnp_status_success, pytorch_qnnp_initialize());
0680:   if (pytorch_qnnp_params.q8conv_xzp.kthreshold != SIZE_MAX) {
0681:     ConvolutionOperatorTester()
0682:         .inputSize(27, 29)
0683:         .kernelSize(1, 1)
0684:         .inputPixelStride(pytorch_qnnp_params.q8conv_xzp.kthreshold + 5)
0685:         .groupInputChannels(pytorch_qnnp_params.q8conv_xzp.kthreshold + 1)
0686:         .groupOutputChannels(19)
0687:         .iterations(3)
0688:         .per_channel(true)
0689:         .testQ8();
0690:   }
0691: }
0692: 
```

- **EN:** This block handles conditional branches and special cases. Key symbols: `ConvolutionOperatorTester`.
- **CN:** 该代码块处理条件分支与特殊情况。关键符号：`ConvolutionOperatorTester`。

### Lines 693-737 / 第 693-737 行

```cpp
0693: TEST(CONVOLUTION_OP, xzp_1x1_with_output_stride_per_channel) {
0694:   ASSERT_EQ(pytorch_qnnp_status_success, pytorch_qnnp_initialize());
0695:   if (pytorch_qnnp_params.q8conv_xzp.kthreshold != SIZE_MAX) {
0696:     ConvolutionOperatorTester()
0697:         .inputSize(27, 29)
0698:         .kernelSize(1, 1)
0699:         .outputPixelStride(29)
0700:         .groupInputChannels(pytorch_qnnp_params.q8conv_xzp.kthreshold + 1)
0701:         .groupOutputChannels(19)
0702:         .iterations(3)
0703:         .per_channel(true)
0704:         .testQ8();
0705:   }
0706: }
0707: 
0708: TEST(CONVOLUTION_OP, xzp_1x1_with_batch_per_channel) {
0709:   ASSERT_EQ(pytorch_qnnp_status_success, pytorch_qnnp_initialize());
0710:   if (pytorch_qnnp_params.q8conv_xzp.kthreshold != SIZE_MAX) {
0711:     ConvolutionOperatorTester()
0712:         .inputSize(13, 14)
0713:         .kernelSize(1, 1)
0714:         .batchSize(3)
0715:         .groupInputChannels(pytorch_qnnp_params.q8conv_xzp.kthreshold + 1)
0716:         .groupOutputChannels(19)
0717:         .iterations(3)
0718:         .per_channel(true)
0719:         .testQ8();
0720:   }
0721: }
0722: 
0723: TEST(CONVOLUTION_OP, grouped_xzp_1x1_per_channel) {
0724:   ASSERT_EQ(pytorch_qnnp_status_success, pytorch_qnnp_initialize());
0725:   if (pytorch_qnnp_params.q8conv_xzp.kthreshold != SIZE_MAX) {
0726:     ConvolutionOperatorTester()
0727:         .inputSize(24, 25)
0728:         .kernelSize(1, 1)
0729:         .groups(2)
0730:         .groupInputChannels(pytorch_qnnp_params.q8conv_xzp.kthreshold + 1)
0731:         .groupOutputChannels(19)
0732:         .iterations(3)
0733:         .per_channel(true)
0734:         .testQ8();
0735:   }
0736: }
0737: 
```

- **EN:** This block handles conditional branches and special cases. Key symbols: `ConvolutionOperatorTester`.
- **CN:** 该代码块处理条件分支与特殊情况。关键符号：`ConvolutionOperatorTester`。

### Lines 738-786 / 第 738-786 行

```cpp
0738: TEST(CONVOLUTION_OP, grouped_xzp_1x1_runtime_quant_per_channel) {
0739:   ASSERT_EQ(pytorch_qnnp_status_success, pytorch_qnnp_initialize());
0740:   if (pytorch_qnnp_params.q8conv_xzp.kthreshold != SIZE_MAX) {
0741:     ConvolutionOperatorTester()
0742:         .inputSize(24, 25)
0743:         .kernelSize(1, 1)
0744:         .groups(2)
0745:         .groupInputChannels(pytorch_qnnp_params.q8conv_xzp.kthreshold + 1)
0746:         .groupOutputChannels(19)
0747:         .iterations(3)
0748:         .per_channel(true)
0749:         .testQ8(Mode::Runtime);
0750:   }
0751: }
0752: 
0753: _STATIC_AND_RUNTIME_TEST(CONVOLUTION_OP, 1x3_per_channel,
0754:   ConvolutionOperatorTester()
0755:       .inputSize(20, 19)
0756:       .paddingWidth(1)
0757:       .kernelSize(1, 3)
0758:       .groupInputChannels(17)
0759:       .groupOutputChannels(15)
0760:       .iterations(3)
0761:       .per_channel(true)
0762: )
0763: 
0764: _STATIC_AND_RUNTIME_TEST(CONVOLUTION_OP, grouped_1x3_per_channel,
0765:   ConvolutionOperatorTester()
0766:       .inputSize(20, 19)
0767:       .paddingWidth(1)
0768:       .kernelSize(1, 3)
0769:       .groups(2)
0770:       .groupInputChannels(17)
0771:       .groupOutputChannels(15)
0772:       .iterations(3)
0773:       .per_channel(true)
0774: )
0775: 
0776: _STATIC_AND_RUNTIME_TEST(CONVOLUTION_OP, 3x1_per_channel,
0777:   ConvolutionOperatorTester()
0778:       .inputSize(19, 20)
0779:       .paddingHeight(1)
0780:       .kernelSize(3, 1)
0781:       .groupInputChannels(17)
0782:       .groupOutputChannels(15)
0783:       .iterations(3)
0784:       .per_channel(true)
0785: )
0786: 
```

- **EN:** This block handles conditional branches and special cases. Key symbols: `ConvolutionOperatorTester`.
- **CN:** 该代码块处理条件分支与特殊情况。关键符号：`ConvolutionOperatorTester`。

### Lines 787-831 / 第 787-831 行

```cpp
0787: _STATIC_AND_RUNTIME_TEST(CONVOLUTION_OP, grouped_3x1_per_channel,
0788:   ConvolutionOperatorTester()
0789:       .inputSize(19, 20)
0790:       .paddingHeight(1)
0791:       .kernelSize(3, 1)
0792:       .groups(2)
0793:       .groupInputChannels(17)
0794:       .groupOutputChannels(15)
0795:       .iterations(3)
0796:       .per_channel(true)
0797: )
0798: 
0799: _STATIC_AND_RUNTIME_TEST(CONVOLUTION_OP, 3x3_per_channel,
0800:   ConvolutionOperatorTester()
0801:       .inputSize(13, 12)
0802:       .padding(1)
0803:       .kernelSize(3, 3)
0804:       .groupInputChannels(15)
0805:       .groupOutputChannels(17)
0806:       .iterations(3)
0807:       .per_channel(true)
0808: )
0809: 
0810: _STATIC_AND_RUNTIME_TEST(CONVOLUTION_OP, 3x3_without_padding_per_channel,
0811:   ConvolutionOperatorTester()
0812:       .inputSize(13, 12)
0813:       .kernelSize(3, 3)
0814:       .groupInputChannels(15)
0815:       .groupOutputChannels(17)
0816:       .iterations(3)
0817:       .per_channel(true)
0818: )
0819: 
0820: _STATIC_AND_RUNTIME_TEST(
0821:     CONVOLUTION_OP,
0822:     3x3_with_width_padding_per_channel,
0823:     ConvolutionOperatorTester()
0824:         .inputSize(13, 12)
0825:         .paddingWidth(1)
0826:         .kernelSize(3, 3)
0827:         .groupInputChannels(15)
0828:         .groupOutputChannels(17)
0829:         .iterations(3)
0830:         .per_channel(true))
0831: 
```

- **EN:** This block implements local helper logic for `convolution`. Key symbols: no prominent local symbols.
- **CN:** 该代码块实现与 `convolution` 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 832-879 / 第 832-879 行

```cpp
0832: _STATIC_AND_RUNTIME_TEST(
0833:     CONVOLUTION_OP,
0834:     3x3_with_height_padding_per_channel,
0835:     ConvolutionOperatorTester()
0836:         .inputSize(13, 12)
0837:         .paddingHeight(1)
0838:         .kernelSize(3, 3)
0839:         .groupInputChannels(15)
0840:         .groupOutputChannels(17)
0841:         .iterations(3)
0842:         .per_channel(true))
0843: 
0844: _STATIC_TEST(CONVOLUTION_OP, 3x3_with_input_stride_per_channel,
0845:   ConvolutionOperatorTester()
0846:       .inputSize(13, 12)
0847:       .padding(1)
0848:       .kernelSize(3, 3)
0849:       .inputPixelStride(22)
0850:       .groupInputChannels(15)
0851:       .groupOutputChannels(17)
0852:       .iterations(3)
0853:       .per_channel(true)
0854: )
0855: 
0856: _STATIC_TEST(CONVOLUTION_OP, 3x3_with_output_stride_per_channel,
0857:   ConvolutionOperatorTester()
0858:       .inputSize(13, 12)
0859:       .padding(1)
0860:       .kernelSize(3, 3)
0861:       .outputPixelStride(23)
0862:       .groupInputChannels(15)
0863:       .groupOutputChannels(17)
0864:       .iterations(3)
0865:       .per_channel(true)
0866: )
0867: 
0868: _STATIC_AND_RUNTIME_TEST(CONVOLUTION_OP, 3x3_with_batch_per_channel,
0869:   ConvolutionOperatorTester()
0870:       .inputSize(10, 9)
0871:       .padding(1)
0872:       .kernelSize(3, 3)
0873:       .batchSize(3)
0874:       .groupInputChannels(15)
0875:       .groupOutputChannels(17)
0876:       .iterations(3)
0877:       .per_channel(true)
0878: )
0879: 
```

- **EN:** This block implements local helper logic for `convolution`. Key symbols: no prominent local symbols.
- **CN:** 该代码块实现与 `convolution` 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 880-927 / 第 880-927 行

```cpp
0880: _STATIC_AND_RUNTIME_TEST(CONVOLUTION_OP, grouped_3x3_per_channel,
0881:   ConvolutionOperatorTester()
0882:       .inputSize(10, 11)
0883:       .padding(1)
0884:       .kernelSize(3, 3)
0885:       .groups(2)
0886:       .groupInputChannels(14)
0887:       .groupOutputChannels(13)
0888:       .iterations(3)
0889:       .per_channel(true)
0890: )
0891: 
0892: _STATIC_AND_RUNTIME_TEST(CONVOLUTION_OP, 3x3s2_per_channel,
0893:   ConvolutionOperatorTester()
0894:       .inputSize(19, 21)
0895:       .padding(1)
0896:       .kernelSize(3, 3)
0897:       .subsampling(2)
0898:       .groupInputChannels(27)
0899:       .groupOutputChannels(19)
0900:       .iterations(3)
0901:       .per_channel(true)
0902: )
0903: 
0904: _STATIC_AND_RUNTIME_TEST(CONVOLUTION_OP, 3x3s1x2_per_channel,
0905:   ConvolutionOperatorTester()
0906:       .inputSize(13, 13)
0907:       .padding(1)
0908:       .kernelSize(3, 3)
0909:       .subsampling(1, 2)
0910:       .groupInputChannels(27)
0911:       .groupOutputChannels(19)
0912:       .iterations(3)
0913:       .per_channel(true)
0914: )
0915: 
0916: _STATIC_AND_RUNTIME_TEST(CONVOLUTION_OP, 3x3s2x1_per_channel,
0917:   ConvolutionOperatorTester()
0918:       .inputSize(13, 13)
0919:       .padding(1)
0920:       .kernelSize(3, 3)
0921:       .subsampling(2, 1)
0922:       .groupInputChannels(27)
0923:       .groupOutputChannels(19)
0924:       .iterations(3)
0925:       .per_channel(true)
0926: )
0927: 
```

- **EN:** This block implements local helper logic for `convolution`. Key symbols: no prominent local symbols.
- **CN:** 该代码块实现与 `convolution` 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 928-973 / 第 928-973 行

```cpp
0928: _STATIC_AND_RUNTIME_TEST(CONVOLUTION_OP, 3x3d2_per_channel,
0929:   ConvolutionOperatorTester()
0930:       .inputSize(13, 14)
0931:       .padding(2)
0932:       .kernelSize(3, 3)
0933:       .dilation(2)
0934:       .groupInputChannels(27)
0935:       .groupOutputChannels(19)
0936:       .iterations(3)
0937:       .per_channel(true)
0938: )
0939: 
0940: _STATIC_AND_RUNTIME_TEST(CONVOLUTION_OP, 3x3d1x2_per_channel,
0941:   ConvolutionOperatorTester()
0942:       .inputSize(14, 15)
0943:       .padding(1, 2)
0944:       .kernelSize(3, 3)
0945:       .dilation(1, 2)
0946:       .groupInputChannels(27)
0947:       .groupOutputChannels(19)
0948:       .iterations(3)
0949:       .per_channel(true)
0950: )
0951: 
0952: _STATIC_AND_RUNTIME_TEST(CONVOLUTION_OP, 3x3d2x1_per_channel,
0953:   ConvolutionOperatorTester()
0954:       .inputSize(15, 14)
0955:       .padding(2, 1)
0956:       .kernelSize(3, 3)
0957:       .dilation(2, 1)
0958:       .groupInputChannels(27)
0959:       .groupOutputChannels(19)
0960:       .iterations(3)
0961:       .per_channel(true)
0962: )
0963: 
0964: _STATIC_AND_RUNTIME_TEST(CONVOLUTION_OP, depthwise_3x3_per_channel,
0965:   ConvolutionOperatorTester()
0966:       .inputSize(15, 14)
0967:       .padding(1, 1)
0968:       .kernelSize(3, 3)
0969:       .groups(27)
0970:       .iterations(3)
0971:       .per_channel(true)
0972: )
0973: 
```

- **EN:** This block implements local helper logic for `convolution`. Key symbols: no prominent local symbols.
- **CN:** 该代码块实现与 `convolution` 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 974-1017 / 第 974-1017 行

```cpp
0974: _STATIC_AND_RUNTIME_TEST(CONVOLUTION_OP, depthwise_3x3s2_per_channel,
0975:   ConvolutionOperatorTester()
0976:       .inputSize(15, 14)
0977:       .padding(1, 1)
0978:       .kernelSize(3, 3)
0979:       .subsampling(2)
0980:       .groups(27)
0981:       .iterations(3)
0982:       .per_channel(true)
0983: )
0984: 
0985: _STATIC_AND_RUNTIME_TEST(CONVOLUTION_OP, depthwise_3x3s1x2_per_channel,
0986:   ConvolutionOperatorTester()
0987:       .inputSize(15, 14)
0988:       .padding(1, 1)
0989:       .kernelSize(3, 3)
0990:       .subsampling(1, 2)
0991:       .groups(27)
0992:       .iterations(3)
0993:       .per_channel(true)
0994: )
0995: 
0996: _STATIC_AND_RUNTIME_TEST(CONVOLUTION_OP, depthwise_3x3s2x1_per_channel,
0997:   ConvolutionOperatorTester()
0998:       .inputSize(15, 14)
0999:       .padding(1, 1)
1000:       .kernelSize(3, 3)
1001:       .subsampling(2, 1)
1002:       .groups(27)
1003:       .iterations(3)
1004:       .per_channel(true)
1005: )
1006: 
1007: _STATIC_AND_RUNTIME_TEST(CONVOLUTION_OP, depthwise_3x3d2_per_channel,
1008:   ConvolutionOperatorTester()
1009:       .inputSize(15, 14)
1010:       .padding(1, 1)
1011:       .kernelSize(3, 3)
1012:       .dilation(2)
1013:       .groups(27)
1014:       .iterations(3)
1015:       .per_channel(true)
1016: )
1017: 
```

- **EN:** This block implements local helper logic for `convolution`. Key symbols: no prominent local symbols.
- **CN:** 该代码块实现与 `convolution` 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 1018-1060 / 第 1018-1060 行

```cpp
1018: _STATIC_AND_RUNTIME_TEST(CONVOLUTION_OP, depthwise_3x3d1x2_per_channel,
1019:   ConvolutionOperatorTester()
1020:       .inputSize(15, 14)
1021:       .padding(1, 1)
1022:       .kernelSize(3, 3)
1023:       .dilation(1, 2)
1024:       .groups(27)
1025:       .iterations(3)
1026:       .per_channel(true)
1027: )
1028: 
1029: _STATIC_AND_RUNTIME_TEST(CONVOLUTION_OP, depthwise_3x3d2x1_per_channel,
1030:   ConvolutionOperatorTester()
1031:       .inputSize(15, 14)
1032:       .padding(1, 1)
1033:       .kernelSize(3, 3)
1034:       .dilation(2, 1)
1035:       .groups(27)
1036:       .iterations(3)
1037:       .per_channel(true)
1038: )
1039: 
1040: _STATIC_AND_RUNTIME_TEST(CONVOLUTION_OP, depthwise_5x5_per_channel,
1041:   ConvolutionOperatorTester()
1042:       .inputSize(15, 14)
1043:       .padding(2, 2)
1044:       .kernelSize(5, 5)
1045:       .groups(27)
1046:       .iterations(3)
1047:       .per_channel(true)
1048: )
1049: 
1050: _STATIC_AND_RUNTIME_TEST(CONVOLUTION_OP, depthwise_5x5s2_per_channel,
1051:   ConvolutionOperatorTester()
1052:       .inputSize(15, 14)
1053:       .padding(2, 2)
1054:       .kernelSize(5, 5)
1055:       .subsampling(2)
1056:       .groups(27)
1057:       .iterations(3)
1058:       .per_channel(true)
1059: )
1060: 
```

- **EN:** This block implements local helper logic for `convolution`. Key symbols: no prominent local symbols.
- **CN:** 该代码块实现与 `convolution` 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 1061-1103 / 第 1061-1103 行

```cpp
1061: _STATIC_AND_RUNTIME_TEST(CONVOLUTION_OP, depthwise_5x5s1x2_per_channel,
1062:   ConvolutionOperatorTester()
1063:       .inputSize(15, 14)
1064:       .padding(2, 2)
1065:       .kernelSize(5, 5)
1066:       .subsampling(1, 2)
1067:       .groups(27)
1068:       .iterations(3)
1069:       .per_channel(true)
1070: )
1071: 
1072: _STATIC_AND_RUNTIME_TEST(CONVOLUTION_OP, depthwise_5x5s2x1_per_channel,
1073:   ConvolutionOperatorTester()
1074:       .inputSize(15, 14)
1075:       .padding(2, 2)
1076:       .kernelSize(5, 5)
1077:       .subsampling(2, 1)
1078:       .groups(27)
1079:       .iterations(3)
1080:       .per_channel(true)
1081: )
1082: 
1083: _STATIC_AND_RUNTIME_TEST(CONVOLUTION_OP, depthwise_5x5d2_per_channel,
1084:   ConvolutionOperatorTester()
1085:       .inputSize(15, 14)
1086:       .padding(2, 2)
1087:       .kernelSize(5, 5)
1088:       .dilation(2)
1089:       .groups(27)
1090:       .iterations(3)
1091:       .per_channel(true)
1092: )
1093: 
1094: _STATIC_AND_RUNTIME_TEST(CONVOLUTION_OP, depthwise_5x5d1x2_per_channel,
1095:   ConvolutionOperatorTester()
1096:       .inputSize(15, 14)
1097:       .padding(2, 2)
1098:       .kernelSize(5, 5)
1099:       .dilation(1, 2)
1100:       .groups(27)
1101:       .iterations(3)
1102: )
1103: 
```

- **EN:** This block implements local helper logic for `convolution`. Key symbols: no prominent local symbols.
- **CN:** 该代码块实现与 `convolution` 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 1104-1148 / 第 1104-1148 行

```cpp
1104: _STATIC_AND_RUNTIME_TEST(CONVOLUTION_OP, depthwise_5x5d2x1_per_channel,
1105:   ConvolutionOperatorTester()
1106:       .inputSize(15, 14)
1107:       .padding(2, 2)
1108:       .kernelSize(5, 5)
1109:       .dilation(2, 1)
1110:       .groups(27)
1111:       .iterations(3)
1112:       .per_channel(true)
1113: )
1114: 
1115: _STATIC_AND_RUNTIME_TEST(
1116:     CONVOLUTION_3D_OP,
1117:     zero_batch,
1118:     ConvolutionOperatorTester()
1119:         .dimensionality(3)
1120:         .batchSize(0)
1121:         .inputSize(5, 5, 5)
1122:         .kernelSize(1, 1, 1)
1123:         .groupInputChannels(2)
1124:         .groupOutputChannels(2)
1125:         .iterations(1))
1126: 
1127: _STATIC_AND_RUNTIME_TEST(
1128:     CONVOLUTION_3D_OP,
1129:     1x1x1,
1130:     ConvolutionOperatorTester()
1131:         .dimensionality(3)
1132:         .inputSize(8, 9, 10)
1133:         .kernelSize(1, 1, 1)
1134:         .groupInputChannels(5)
1135:         .groupInputChannels(6)
1136:         .iterations(3))
1137: 
1138: _STATIC_AND_RUNTIME_TEST(
1139:     CONVOLUTION_3D_OP,
1140:     1x1x1_runtime_quant,
1141:     ConvolutionOperatorTester()
1142:         .dimensionality(3)
1143:         .inputSize(8, 9, 10)
1144:         .kernelSize(1, 1, 1)
1145:         .groupInputChannels(5)
1146:         .groupInputChannels(6)
1147:         .iterations(3))
1148: 
```

- **EN:** This block implements local helper logic for `convolution`. Key symbols: no prominent local symbols.
- **CN:** 该代码块实现与 `convolution` 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 1149-1196 / 第 1149-1196 行

```cpp
1149: _STATIC_AND_RUNTIME_TEST(
1150:     CONVOLUTION_3D_OP,
1151:     1x1x1_with_qmin,
1152:     ConvolutionOperatorTester()
1153:         .dimensionality(3)
1154:         .inputSize(8, 9, 10)
1155:         .kernelSize(1, 1, 1)
1156:         .groupInputChannels(5)
1157:         .groupInputChannels(6)
1158:         .qmin(128)
1159:         .iterations(3))
1160: 
1161: _STATIC_AND_RUNTIME_TEST(
1162:     CONVOLUTION_3D_OP,
1163:     1x1x1_with_qmax,
1164:     ConvolutionOperatorTester()
1165:         .dimensionality(3)
1166:         .inputSize(8, 9, 10)
1167:         .kernelSize(1, 1, 1)
1168:         .groupInputChannels(5)
1169:         .groupInputChannels(6)
1170:         .qmax(128)
1171:         .iterations(3))
1172: 
1173: _STATIC_TEST(
1174:     CONVOLUTION_3D_OP,
1175:     1x1x1_with_input_stride,
1176:     ConvolutionOperatorTester()
1177:         .dimensionality(3)
1178:         .inputSize(8, 9, 10)
1179:         .kernelSize(1, 1, 1)
1180:         .inputPixelStride(28)
1181:         .groupInputChannels(5)
1182:         .groupInputChannels(6)
1183:         .iterations(3))
1184: 
1185: _STATIC_TEST(
1186:     CONVOLUTION_3D_OP,
1187:     1x1x1_with_output_stride,
1188:     ConvolutionOperatorTester()
1189:         .dimensionality(3)
1190:         .inputSize(8, 9, 10)
1191:         .kernelSize(1, 1, 1)
1192:         .outputPixelStride(7)
1193:         .groupInputChannels(5)
1194:         .groupInputChannels(6)
1195:         .iterations(3))
1196: 
```

- **EN:** This block implements local helper logic for `convolution`. Key symbols: no prominent local symbols.
- **CN:** 该代码块实现与 `convolution` 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 1197-1237 / 第 1197-1237 行

```cpp
1197: _STATIC_AND_RUNTIME_TEST(
1198:     CONVOLUTION_3D_OP,
1199:     1x1x1_with_batch,
1200:     ConvolutionOperatorTester()
1201:         .dimensionality(3)
1202:         .inputSize(7, 8, 10)
1203:         .kernelSize(1, 1, 1)
1204:         .batchSize(3)
1205:         .groupInputChannels(5)
1206:         .groupInputChannels(6)
1207:         .iterations(3))
1208: 
1209: _STATIC_AND_RUNTIME_TEST(
1210:     CONVOLUTION_3D_OP,
1211:     grouped_1x1x1,
1212:     ConvolutionOperatorTester()
1213:         .dimensionality(3)
1214:         .inputSize(8, 10, 11)
1215:         .kernelSize(1, 1, 1)
1216:         .groups(2)
1217:         .groupInputChannels(5)
1218:         .groupInputChannels(6)
1219:         .iterations(3))
1220: 
1221: TEST(CONVOLUTION_3D_OP, xzp_1x1x1) {
1222:   ASSERT_EQ(pytorch_qnnp_status_success, pytorch_qnnp_initialize());
1223:   if (pytorch_qnnp_params.q8conv_xzp.kthreshold != SIZE_MAX) {
1224:     ConvolutionOperatorTester()
1225:         .dimensionality(3)
1226:         .inputSize(8, 9, 10)
1227:         .kernelSize(1, 1, 1)
1228:         .groupInputChannels(pytorch_qnnp_params.q8conv_xzp.kthreshold + 1)
1229:         .groupInputChannels(6)
1230:         .iterations(3)
1231:         .testQ8();
1232:   }
1233: }
1234: 
1235: TEST(CONVOLUTION_3D_OP, xzp_1x1x1_with_qmin) {
1236:   ASSERT_EQ(pytorch_qnnp_status_success, pytorch_qnnp_initialize());
1237:   if (pytorch_qnnp_params.q8conv_xzp.kthreshold != SIZE_MAX) {
```

- **EN:** This block handles conditional branches and special cases. Key symbols: `ConvolutionOperatorTester`.
- **CN:** 该代码块处理条件分支与特殊情况。关键符号：`ConvolutionOperatorTester`。

### Lines 1238-1279 / 第 1238-1279 行

```cpp
1238:     ConvolutionOperatorTester()
1239:         .dimensionality(3)
1240:         .inputSize(8, 9, 10)
1241:         .kernelSize(1, 1, 1)
1242:         .groupInputChannels(pytorch_qnnp_params.q8conv_xzp.kthreshold + 1)
1243:         .groupInputChannels(6)
1244:         .qmin(128)
1245:         .iterations(3)
1246:         .testQ8();
1247:   }
1248: }
1249: 
1250: TEST(CONVOLUTION_3D_OP, xzp_1x1x1_with_qmax) {
1251:   ASSERT_EQ(pytorch_qnnp_status_success, pytorch_qnnp_initialize());
1252:   if (pytorch_qnnp_params.q8conv_xzp.kthreshold != SIZE_MAX) {
1253:     ConvolutionOperatorTester()
1254:         .dimensionality(3)
1255:         .inputSize(8, 9, 10)
1256:         .kernelSize(1, 1, 1)
1257:         .groupInputChannels(pytorch_qnnp_params.q8conv_xzp.kthreshold + 1)
1258:         .groupInputChannels(6)
1259:         .qmax(128)
1260:         .iterations(3)
1261:         .testQ8();
1262:   }
1263: }
1264: 
1265: TEST(CONVOLUTION_3D_OP, xzp_1x1x1_with_input_stride) {
1266:   ASSERT_EQ(pytorch_qnnp_status_success, pytorch_qnnp_initialize());
1267:   if (pytorch_qnnp_params.q8conv_xzp.kthreshold != SIZE_MAX) {
1268:     ConvolutionOperatorTester()
1269:         .dimensionality(3)
1270:         .inputSize(8, 9, 10)
1271:         .kernelSize(1, 1, 1)
1272:         .inputPixelStride(pytorch_qnnp_params.q8conv_xzp.kthreshold + 5)
1273:         .groupInputChannels(pytorch_qnnp_params.q8conv_xzp.kthreshold + 1)
1274:         .groupInputChannels(6)
1275:         .iterations(3)
1276:         .testQ8();
1277:   }
1278: }
1279: 
```

- **EN:** This block handles conditional branches and special cases. Key symbols: `ConvolutionOperatorTester`.
- **CN:** 该代码块处理条件分支与特殊情况。关键符号：`ConvolutionOperatorTester`。

### Lines 1280-1324 / 第 1280-1324 行

```cpp
1280: TEST(CONVOLUTION_3D_OP, xzp_1x1x1_with_output_stride) {
1281:   ASSERT_EQ(pytorch_qnnp_status_success, pytorch_qnnp_initialize());
1282:   if (pytorch_qnnp_params.q8conv_xzp.kthreshold != SIZE_MAX) {
1283:     ConvolutionOperatorTester()
1284:         .dimensionality(3)
1285:         .inputSize(8, 9, 10)
1286:         .kernelSize(1, 1, 1)
1287:         .outputPixelStride(7)
1288:         .groupInputChannels(pytorch_qnnp_params.q8conv_xzp.kthreshold + 1)
1289:         .groupInputChannels(6)
1290:         .iterations(3)
1291:         .testQ8();
1292:   }
1293: }
1294: 
1295: TEST(CONVOLUTION_3D_OP, xzp_1x1x1_with_batch) {
1296:   ASSERT_EQ(pytorch_qnnp_status_success, pytorch_qnnp_initialize());
1297:   if (pytorch_qnnp_params.q8conv_xzp.kthreshold != SIZE_MAX) {
1298:     ConvolutionOperatorTester()
1299:         .dimensionality(3)
1300:         .inputSize(7, 8, 10)
1301:         .kernelSize(1, 1, 1)
1302:         .batchSize(3)
1303:         .groupInputChannels(pytorch_qnnp_params.q8conv_xzp.kthreshold + 1)
1304:         .groupInputChannels(6)
1305:         .iterations(3)
1306:         .testQ8();
1307:   }
1308: }
1309: 
1310: TEST(CONVOLUTION_3D_OP, grouped_xzp_1x1x1) {
1311:   ASSERT_EQ(pytorch_qnnp_status_success, pytorch_qnnp_initialize());
1312:   if (pytorch_qnnp_params.q8conv_xzp.kthreshold != SIZE_MAX) {
1313:     ConvolutionOperatorTester()
1314:         .dimensionality(3)
1315:         .inputSize(8, 10, 11)
1316:         .kernelSize(1, 1, 1)
1317:         .groups(2)
1318:         .groupInputChannels(pytorch_qnnp_params.q8conv_xzp.kthreshold + 1)
1319:         .groupInputChannels(6)
1320:         .iterations(3)
1321:         .testQ8();
1322:   }
1323: }
1324: 
```

- **EN:** This block handles conditional branches and special cases. Key symbols: `ConvolutionOperatorTester`.
- **CN:** 该代码块处理条件分支与特殊情况。关键符号：`ConvolutionOperatorTester`。

### Lines 1325-1364 / 第 1325-1364 行

```cpp
1325: TEST(CONVOLUTION_3D_OP, grouped_xzp_1x1x1_runtime_quant) {
1326:   ASSERT_EQ(pytorch_qnnp_status_success, pytorch_qnnp_initialize());
1327:   if (pytorch_qnnp_params.q8conv_xzp.kthreshold != SIZE_MAX) {
1328:     ConvolutionOperatorTester()
1329:         .dimensionality(3)
1330:         .inputSize(8, 10, 11)
1331:         .kernelSize(1, 1, 1)
1332:         .groups(2)
1333:         .groupInputChannels(pytorch_qnnp_params.q8conv_xzp.kthreshold + 1)
1334:         .groupInputChannels(6)
1335:         .iterations(3)
1336:         .testQ8(Mode::Runtime);
1337:   }
1338: }
1339: 
1340: _STATIC_AND_RUNTIME_TEST(
1341:     CONVOLUTION_3D_OP,
1342:     1x1x3,
1343:     ConvolutionOperatorTester()
1344:         .dimensionality(3)
1345:         .inputSize(8, 7, 10)
1346:         .paddingWidth(1)
1347:         .kernelSize(1, 1, 3)
1348:         .groupInputChannels(5)
1349:         .groupInputChannels(6)
1350:         .iterations(3))
1351: 
1352: _STATIC_AND_RUNTIME_TEST(
1353:     CONVOLUTION_3D_OP,
1354:     grouped_1x1x3,
1355:     ConvolutionOperatorTester()
1356:         .dimensionality(3)
1357:         .inputSize(8, 7, 10)
1358:         .paddingWidth(1)
1359:         .kernelSize(1, 1, 3)
1360:         .groups(2)
1361:         .groupInputChannels(5)
1362:         .groupInputChannels(6)
1363:         .iterations(3))
1364: 
```

- **EN:** This block handles conditional branches and special cases. Key symbols: `ConvolutionOperatorTester`.
- **CN:** 该代码块处理条件分支与特殊情况。关键符号：`ConvolutionOperatorTester`。

### Lines 1365-1412 / 第 1365-1412 行

```cpp
1365: _STATIC_AND_RUNTIME_TEST(
1366:     CONVOLUTION_3D_OP,
1367:     3x3x1,
1368:     ConvolutionOperatorTester()
1369:         .dimensionality(3)
1370:         .inputSize(8, 9, 7)
1371:         .paddingHeight(1)
1372:         .kernelSize(3, 3, 1)
1373:         .groupInputChannels(5)
1374:         .groupInputChannels(6)
1375:         .iterations(3))
1376: 
1377: _STATIC_AND_RUNTIME_TEST(
1378:     CONVOLUTION_3D_OP,
1379:     grouped_3x3x1,
1380:     ConvolutionOperatorTester()
1381:         .dimensionality(3)
1382:         .inputSize(8, 9, 7)
1383:         .paddingHeight(1)
1384:         .kernelSize(3, 3, 1)
1385:         .groups(2)
1386:         .groupInputChannels(5)
1387:         .groupInputChannels(6)
1388:         .iterations(3))
1389: 
1390: _STATIC_AND_RUNTIME_TEST(
1391:     CONVOLUTION_3D_OP,
1392:     3x3x3,
1393:     ConvolutionOperatorTester()
1394:         .dimensionality(3)
1395:         .inputSize(8, 10, 9)
1396:         .padding(1)
1397:         .kernelSize(3, 3, 3)
1398:         .groupInputChannels(5)
1399:         .groupInputChannels(6)
1400:         .iterations(3))
1401: 
1402: _STATIC_AND_RUNTIME_TEST(
1403:     CONVOLUTION_3D_OP,
1404:     3x3x3_without_padding,
1405:     ConvolutionOperatorTester()
1406:         .dimensionality(3)
1407:         .inputSize(8, 10, 9)
1408:         .kernelSize(3, 3, 3)
1409:         .groupInputChannels(5)
1410:         .groupInputChannels(6)
1411:         .iterations(3))
1412: 
```

- **EN:** This block implements local helper logic for `convolution`. Key symbols: no prominent local symbols.
- **CN:** 该代码块实现与 `convolution` 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 1413-1461 / 第 1413-1461 行

```cpp
1413: _STATIC_AND_RUNTIME_TEST(
1414:     CONVOLUTION_3D_OP,
1415:     3x3x3_with_width_padding,
1416:     ConvolutionOperatorTester()
1417:         .dimensionality(3)
1418:         .inputSize(8, 10, 9)
1419:         .paddingWidth(1)
1420:         .kernelSize(3, 3, 3)
1421:         .groupInputChannels(5)
1422:         .groupInputChannels(6)
1423:         .iterations(3))
1424: 
1425: _STATIC_AND_RUNTIME_TEST(
1426:     CONVOLUTION_3D_OP,
1427:     3x3x3_with_height_padding,
1428:     ConvolutionOperatorTester()
1429:         .dimensionality(3)
1430:         .inputSize(8, 10, 9)
1431:         .paddingHeight(1)
1432:         .kernelSize(3, 3, 3)
1433:         .groupInputChannels(5)
1434:         .groupInputChannels(6)
1435:         .iterations(3))
1436: 
1437: _STATIC_AND_RUNTIME_TEST(
1438:     CONVOLUTION_3D_OP,
1439:     3x3x3_with_depth_padding,
1440:     ConvolutionOperatorTester()
1441:         .dimensionality(3)
1442:         .inputSize(8, 10, 9)
1443:         .paddingDepth(1)
1444:         .kernelSize(3, 3, 3)
1445:         .groupInputChannels(5)
1446:         .groupInputChannels(6)
1447:         .iterations(3))
1448: 
1449: _STATIC_TEST(
1450:     CONVOLUTION_3D_OP,
1451:     3x3x3_with_input_stride,
1452:     ConvolutionOperatorTester()
1453:         .dimensionality(3)
1454:         .inputSize(8, 10, 9)
1455:         .padding(1)
1456:         .kernelSize(3, 3, 3)
1457:         .inputPixelStride(22)
1458:         .groupInputChannels(5)
1459:         .groupInputChannels(6)
1460:         .iterations(3))
1461: 
```

- **EN:** This block implements local helper logic for `convolution`. Key symbols: no prominent local symbols.
- **CN:** 该代码块实现与 `convolution` 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 1462-1513 / 第 1462-1513 行

```cpp
1462: _STATIC_TEST(
1463:     CONVOLUTION_3D_OP,
1464:     3x3x3_with_output_stride,
1465:     ConvolutionOperatorTester()
1466:         .dimensionality(3)
1467:         .inputSize(8, 10, 9)
1468:         .padding(1)
1469:         .kernelSize(3, 3, 3)
1470:         .outputPixelStride(23)
1471:         .groupInputChannels(5)
1472:         .groupInputChannels(6)
1473:         .iterations(3))
1474: 
1475: _STATIC_AND_RUNTIME_TEST(
1476:     CONVOLUTION_3D_OP,
1477:     3x3x3_with_batch,
1478:     ConvolutionOperatorTester()
1479:         .dimensionality(3)
1480:         .inputSize(10, 10, 9)
1481:         .padding(1)
1482:         .kernelSize(3, 3, 3)
1483:         .batchSize(3)
1484:         .groupInputChannels(5)
1485:         .groupInputChannels(6)
1486:         .iterations(3))
1487: 
1488: _STATIC_AND_RUNTIME_TEST(
1489:     CONVOLUTION_3D_OP,
1490:     grouped_3x3x3,
1491:     ConvolutionOperatorTester()
1492:         .dimensionality(3)
1493:         .inputSize(10, 10, 11)
1494:         .padding(1)
1495:         .kernelSize(3, 3, 3)
1496:         .groups(2)
1497:         .groupInputChannels(5)
1498:         .groupInputChannels(6)
1499:         .iterations(3))
1500: 
1501: _STATIC_AND_RUNTIME_TEST(
1502:     CONVOLUTION_3D_OP,
1503:     3x3x3s2,
1504:     ConvolutionOperatorTester()
1505:         .dimensionality(3)
1506:         .inputSize(8, 10, 12)
1507:         .padding(1)
1508:         .kernelSize(3, 3, 3)
1509:         .subsampling(2)
1510:         .groupInputChannels(5)
1511:         .groupInputChannels(6)
1512:         .iterations(3))
1513: 
```

- **EN:** This block implements local helper logic for `convolution`. Key symbols: no prominent local symbols.
- **CN:** 该代码块实现与 `convolution` 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 1514-1565 / 第 1514-1565 行

```cpp
1514: _STATIC_AND_RUNTIME_TEST(
1515:     CONVOLUTION_3D_OP,
1516:     3x3x3s1x2,
1517:     ConvolutionOperatorTester()
1518:         .dimensionality(3)
1519:         .inputSize(8, 9, 10)
1520:         .padding(1)
1521:         .kernelSize(3, 3, 3)
1522:         .subsampling(1, 1, 2)
1523:         .groupInputChannels(5)
1524:         .groupInputChannels(6)
1525:         .iterations(3))
1526: 
1527: _STATIC_AND_RUNTIME_TEST(
1528:     CONVOLUTION_3D_OP,
1529:     3x3x3s2x1,
1530:     ConvolutionOperatorTester()
1531:         .dimensionality(3)
1532:         .inputSize(8, 9, 10)
1533:         .padding(1)
1534:         .kernelSize(3, 3, 3)
1535:         .subsampling(2, 2, 1)
1536:         .groupInputChannels(5)
1537:         .groupInputChannels(6)
1538:         .iterations(3))
1539: 
1540: _STATIC_AND_RUNTIME_TEST(
1541:     CONVOLUTION_3D_OP,
1542:     3x3x3d2,
1543:     ConvolutionOperatorTester()
1544:         .dimensionality(3)
1545:         .inputSize(7, 8, 10)
1546:         .padding(2)
1547:         .kernelSize(3, 3, 3)
1548:         .dilation(2)
1549:         .groupInputChannels(5)
1550:         .groupInputChannels(6)
1551:         .iterations(3))
1552: 
1553: _STATIC_AND_RUNTIME_TEST(
1554:     CONVOLUTION_3D_OP,
1555:     3x3x3d1x2,
1556:     ConvolutionOperatorTester()
1557:         .dimensionality(3)
1558:         .inputSize(11, 10, 8)
1559:         .padding(1, 1, 2)
1560:         .kernelSize(3, 3, 3)
1561:         .dilation(1, 1, 2)
1562:         .groupInputChannels(5)
1563:         .groupInputChannels(6)
1564:         .iterations(3))
1565: 
```

- **EN:** This block implements local helper logic for `convolution`. Key symbols: no prominent local symbols.
- **CN:** 该代码块实现与 `convolution` 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 1566-1613 / 第 1566-1613 行

```cpp
1566: _STATIC_AND_RUNTIME_TEST(
1567:     CONVOLUTION_3D_OP,
1568:     3x3x3d2x1,
1569:     ConvolutionOperatorTester()
1570:         .dimensionality(3)
1571:         .inputSize(10, 9, 7)
1572:         .padding(2, 2, 1)
1573:         .kernelSize(3, 3, 3)
1574:         .dilation(2, 2, 1)
1575:         .groupInputChannels(5)
1576:         .groupInputChannels(6)
1577:         .iterations(3))
1578: 
1579: _STATIC_AND_RUNTIME_TEST(
1580:     CONVOLUTION_3D_OP,
1581:     depthwise_3x3x3,
1582:     ConvolutionOperatorTester()
1583:         .dimensionality(3)
1584:         .inputSize(10, 9, 7)
1585:         .padding(1, 1, 1)
1586:         .kernelSize(3, 3, 3)
1587:         .groups(27)
1588:         .iterations(3))
1589: 
1590: _STATIC_AND_RUNTIME_TEST(
1591:     CONVOLUTION_3D_OP,
1592:     depthwise_3x3x3s2,
1593:     ConvolutionOperatorTester()
1594:         .dimensionality(3)
1595:         .inputSize(10, 9, 7)
1596:         .padding(1, 1, 1)
1597:         .kernelSize(3, 3, 3)
1598:         .subsampling(2)
1599:         .groups(27)
1600:         .iterations(3))
1601: 
1602: _STATIC_AND_RUNTIME_TEST(
1603:     CONVOLUTION_3D_OP,
1604:     depthwise_3x3x3s1x2,
1605:     ConvolutionOperatorTester()
1606:         .dimensionality(3)
1607:         .inputSize(10, 9, 7)
1608:         .padding(1, 1, 1)
1609:         .kernelSize(3, 3, 3)
1610:         .subsampling(1, 1, 2)
1611:         .groups(27)
1612:         .iterations(3))
1613: 
```

- **EN:** This block implements local helper logic for `convolution`. Key symbols: no prominent local symbols.
- **CN:** 该代码块实现与 `convolution` 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 1614-1661 / 第 1614-1661 行

```cpp
1614: _STATIC_AND_RUNTIME_TEST(
1615:     CONVOLUTION_3D_OP,
1616:     depthwise_3x3x3s2x1,
1617:     ConvolutionOperatorTester()
1618:         .dimensionality(3)
1619:         .inputSize(10, 9, 7)
1620:         .padding(1, 1, 1)
1621:         .kernelSize(3, 3, 3)
1622:         .subsampling(2, 2, 1)
1623:         .groups(27)
1624:         .iterations(3))
1625: 
1626: _STATIC_AND_RUNTIME_TEST(
1627:     CONVOLUTION_3D_OP,
1628:     depthwise_3x3x3d2,
1629:     ConvolutionOperatorTester()
1630:         .dimensionality(3)
1631:         .inputSize(10, 9, 7)
1632:         .padding(1, 1, 1)
1633:         .kernelSize(3, 3, 3)
1634:         .dilation(2)
1635:         .groups(27)
1636:         .iterations(3))
1637: 
1638: _STATIC_AND_RUNTIME_TEST(
1639:     CONVOLUTION_3D_OP,
1640:     depthwise_3x3x3d1x2,
1641:     ConvolutionOperatorTester()
1642:         .dimensionality(3)
1643:         .inputSize(10, 9, 7)
1644:         .padding(1, 1, 1)
1645:         .kernelSize(3, 3, 3)
1646:         .dilation(1, 1, 2)
1647:         .groups(27)
1648:         .iterations(3))
1649: 
1650: _STATIC_AND_RUNTIME_TEST(
1651:     CONVOLUTION_3D_OP,
1652:     depthwise_3x3x3d2x1,
1653:     ConvolutionOperatorTester()
1654:         .dimensionality(3)
1655:         .inputSize(10, 9, 7)
1656:         .padding(1, 1, 1)
1657:         .kernelSize(3, 3, 3)
1658:         .dilation(2, 2, 1)
1659:         .groups(27)
1660:         .iterations(3))
1661: 
```

- **EN:** This block implements local helper logic for `convolution`. Key symbols: no prominent local symbols.
- **CN:** 该代码块实现与 `convolution` 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 1662-1712 / 第 1662-1712 行

```cpp
1662: _STATIC_AND_RUNTIME_TEST(
1663:     CONVOLUTION_3D_OP,
1664:     zero_batch_per_channel,
1665:     ConvolutionOperatorTester()
1666:         .dimensionality(3)
1667:         .batchSize(0)
1668:         .inputSize(5, 5, 5)
1669:         .kernelSize(1, 1, 1)
1670:         .groupInputChannels(2)
1671:         .groupOutputChannels(2)
1672:         .iterations(1)
1673:         .per_channel(true))
1674: 
1675: _STATIC_AND_RUNTIME_TEST(
1676:     CONVOLUTION_3D_OP,
1677:     1x1x1_per_channel,
1678:     ConvolutionOperatorTester()
1679:         .dimensionality(3)
1680:         .inputSize(8, 9, 10)
1681:         .kernelSize(1, 1, 1)
1682:         .groupInputChannels(5)
1683:         .groupInputChannels(6)
1684:         .iterations(3)
1685:         .per_channel(true))
1686: 
1687: _STATIC_AND_RUNTIME_TEST(
1688:     CONVOLUTION_3D_OP,
1689:     1x1x1_with_qmin_per_channel,
1690:     ConvolutionOperatorTester()
1691:         .dimensionality(3)
1692:         .inputSize(8, 9, 10)
1693:         .kernelSize(1, 1, 1)
1694:         .groupInputChannels(5)
1695:         .groupInputChannels(6)
1696:         .qmin(128)
1697:         .iterations(3)
1698:         .per_channel(true))
1699: 
1700: _STATIC_AND_RUNTIME_TEST(
1701:     CONVOLUTION_3D_OP,
1702:     1x1x1_with_qmax_per_channel,
1703:     ConvolutionOperatorTester()
1704:         .dimensionality(3)
1705:         .inputSize(8, 9, 10)
1706:         .kernelSize(1, 1, 1)
1707:         .groupInputChannels(5)
1708:         .groupInputChannels(6)
1709:         .qmax(128)
1710:         .iterations(3)
1711:         .per_channel(true))
1712: 
```

- **EN:** This block implements local helper logic for `convolution`. Key symbols: no prominent local symbols.
- **CN:** 该代码块实现与 `convolution` 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 1713-1764 / 第 1713-1764 行

```cpp
1713: _STATIC_TEST(
1714:     CONVOLUTION_3D_OP,
1715:     1x1x1_with_input_stride_per_channel,
1716:     ConvolutionOperatorTester()
1717:         .dimensionality(3)
1718:         .inputSize(8, 9, 10)
1719:         .kernelSize(1, 1, 1)
1720:         .inputPixelStride(28)
1721:         .groupInputChannels(5)
1722:         .groupInputChannels(6)
1723:         .iterations(3)
1724:         .per_channel(true))
1725: 
1726: _STATIC_TEST(
1727:     CONVOLUTION_3D_OP,
1728:     1x1x1_with_output_stride_per_channel,
1729:     ConvolutionOperatorTester()
1730:         .dimensionality(3)
1731:         .inputSize(8, 9, 10)
1732:         .kernelSize(1, 1, 1)
1733:         .outputPixelStride(7)
1734:         .groupInputChannels(5)
1735:         .groupInputChannels(6)
1736:         .iterations(3)
1737:         .per_channel(true))
1738: 
1739: _STATIC_AND_RUNTIME_TEST(
1740:     CONVOLUTION_3D_OP,
1741:     1x1x1_with_batch_per_channel,
1742:     ConvolutionOperatorTester()
1743:         .dimensionality(3)
1744:         .inputSize(7, 8, 10)
1745:         .kernelSize(1, 1, 1)
1746:         .batchSize(3)
1747:         .groupInputChannels(5)
1748:         .groupInputChannels(6)
1749:         .iterations(3)
1750:         .per_channel(true))
1751: 
1752: _STATIC_AND_RUNTIME_TEST(
1753:     CONVOLUTION_3D_OP,
1754:     grouped_1x1x1_per_channel,
1755:     ConvolutionOperatorTester()
1756:         .dimensionality(3)
1757:         .inputSize(8, 10, 11)
1758:         .kernelSize(1, 1, 1)
1759:         .groups(2)
1760:         .groupInputChannels(5)
1761:         .groupInputChannels(6)
1762:         .iterations(3)
1763:         .per_channel(true))
1764: 
```

- **EN:** This block implements local helper logic for `convolution`. Key symbols: no prominent local symbols.
- **CN:** 该代码块实现与 `convolution` 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 1765-1811 / 第 1765-1811 行

```cpp
1765: TEST(CONVOLUTION_3D_OP, xzp_1x1x1_per_channel) {
1766:   ASSERT_EQ(pytorch_qnnp_status_success, pytorch_qnnp_initialize());
1767:   if (pytorch_qnnp_params.q8conv_xzp.kthreshold != SIZE_MAX) {
1768:     ConvolutionOperatorTester()
1769:         .dimensionality(3)
1770:         .inputSize(8, 9, 10)
1771:         .kernelSize(1, 1, 1)
1772:         .groupInputChannels(pytorch_qnnp_params.q8conv_xzp.kthreshold + 1)
1773:         .groupInputChannels(6)
1774:         .iterations(3)
1775:         .per_channel(true)
1776:         .testQ8();
1777:   }
1778: }
1779: 
1780: TEST(CONVOLUTION_3D_OP, xzp_1x1x1_with_qmin_per_channel) {
1781:   ASSERT_EQ(pytorch_qnnp_status_success, pytorch_qnnp_initialize());
1782:   if (pytorch_qnnp_params.q8conv_xzp.kthreshold != SIZE_MAX) {
1783:     ConvolutionOperatorTester()
1784:         .dimensionality(3)
1785:         .inputSize(8, 9, 10)
1786:         .kernelSize(1, 1, 1)
1787:         .groupInputChannels(pytorch_qnnp_params.q8conv_xzp.kthreshold + 1)
1788:         .groupInputChannels(6)
1789:         .qmin(128)
1790:         .iterations(3)
1791:         .per_channel(true)
1792:         .testQ8();
1793:   }
1794: }
1795: 
1796: TEST(CONVOLUTION_3D_OP, xzp_1x1x1_with_qmax_per_channel) {
1797:   ASSERT_EQ(pytorch_qnnp_status_success, pytorch_qnnp_initialize());
1798:   if (pytorch_qnnp_params.q8conv_xzp.kthreshold != SIZE_MAX) {
1799:     ConvolutionOperatorTester()
1800:         .dimensionality(3)
1801:         .inputSize(8, 9, 10)
1802:         .kernelSize(1, 1, 1)
1803:         .groupInputChannels(pytorch_qnnp_params.q8conv_xzp.kthreshold + 1)
1804:         .groupInputChannels(6)
1805:         .qmax(128)
1806:         .iterations(3)
1807:         .per_channel(true)
1808:         .testQ8();
1809:   }
1810: }
1811: 
```

- **EN:** This block handles conditional branches and special cases. Key symbols: `ConvolutionOperatorTester`.
- **CN:** 该代码块处理条件分支与特殊情况。关键符号：`ConvolutionOperatorTester`。

### Lines 1812-1859 / 第 1812-1859 行

```cpp
1812: TEST(CONVOLUTION_3D_OP, xzp_1x1x1_with_input_stride_per_channel) {
1813:   ASSERT_EQ(pytorch_qnnp_status_success, pytorch_qnnp_initialize());
1814:   if (pytorch_qnnp_params.q8conv_xzp.kthreshold != SIZE_MAX) {
1815:     ConvolutionOperatorTester()
1816:         .dimensionality(3)
1817:         .inputSize(8, 9, 10)
1818:         .kernelSize(1, 1, 1)
1819:         .inputPixelStride(pytorch_qnnp_params.q8conv_xzp.kthreshold + 5)
1820:         .groupInputChannels(pytorch_qnnp_params.q8conv_xzp.kthreshold + 1)
1821:         .groupInputChannels(6)
1822:         .iterations(3)
1823:         .per_channel(true)
1824:         .testQ8();
1825:   }
1826: }
1827: 
1828: TEST(CONVOLUTION_3D_OP, xzp_1x1x1_with_output_stride_per_channel) {
1829:   ASSERT_EQ(pytorch_qnnp_status_success, pytorch_qnnp_initialize());
1830:   if (pytorch_qnnp_params.q8conv_xzp.kthreshold != SIZE_MAX) {
1831:     ConvolutionOperatorTester()
1832:         .dimensionality(3)
1833:         .inputSize(8, 9, 10)
1834:         .kernelSize(1, 1, 1)
1835:         .outputPixelStride(7)
1836:         .groupInputChannels(pytorch_qnnp_params.q8conv_xzp.kthreshold + 1)
1837:         .groupInputChannels(6)
1838:         .iterations(3)
1839:         .per_channel(true)
1840:         .testQ8();
1841:   }
1842: }
1843: 
1844: TEST(CONVOLUTION_3D_OP, xzp_1x1x1_with_batch_per_channel) {
1845:   ASSERT_EQ(pytorch_qnnp_status_success, pytorch_qnnp_initialize());
1846:   if (pytorch_qnnp_params.q8conv_xzp.kthreshold != SIZE_MAX) {
1847:     ConvolutionOperatorTester()
1848:         .dimensionality(3)
1849:         .inputSize(7, 8, 10)
1850:         .kernelSize(1, 1, 1)
1851:         .batchSize(3)
1852:         .groupInputChannels(pytorch_qnnp_params.q8conv_xzp.kthreshold + 1)
1853:         .groupInputChannels(6)
1854:         .iterations(3)
1855:         .per_channel(true)
1856:         .testQ8();
1857:   }
1858: }
1859: 
```

- **EN:** This block handles conditional branches and special cases. Key symbols: `ConvolutionOperatorTester`.
- **CN:** 该代码块处理条件分支与特殊情况。关键符号：`ConvolutionOperatorTester`。

### Lines 1860-1904 / 第 1860-1904 行

```cpp
1860: TEST(CONVOLUTION_3D_OP, grouped_xzp_1x1x1_per_channel) {
1861:   ASSERT_EQ(pytorch_qnnp_status_success, pytorch_qnnp_initialize());
1862:   if (pytorch_qnnp_params.q8conv_xzp.kthreshold != SIZE_MAX) {
1863:     ConvolutionOperatorTester()
1864:         .dimensionality(3)
1865:         .inputSize(8, 10, 11)
1866:         .kernelSize(1, 1, 1)
1867:         .groups(2)
1868:         .groupInputChannels(pytorch_qnnp_params.q8conv_xzp.kthreshold + 1)
1869:         .groupInputChannels(6)
1870:         .iterations(3)
1871:         .per_channel(true)
1872:         .testQ8();
1873:   }
1874: }
1875: 
1876: TEST(CONVOLUTION_3D_OP, grouped_xzp_1x1x1_runtime_quant_per_channel) {
1877:   ASSERT_EQ(pytorch_qnnp_status_success, pytorch_qnnp_initialize());
1878:   if (pytorch_qnnp_params.q8conv_xzp.kthreshold != SIZE_MAX) {
1879:     ConvolutionOperatorTester()
1880:         .dimensionality(3)
1881:         .inputSize(8, 10, 11)
1882:         .kernelSize(1, 1, 1)
1883:         .groups(2)
1884:         .groupInputChannels(pytorch_qnnp_params.q8conv_xzp.kthreshold + 1)
1885:         .groupInputChannels(6)
1886:         .iterations(3)
1887:         .per_channel(true)
1888:         .testQ8(Mode::Runtime);
1889:   }
1890: }
1891: 
1892: _STATIC_AND_RUNTIME_TEST(
1893:     CONVOLUTION_3D_OP,
1894:     1x1x3_per_channel,
1895:     ConvolutionOperatorTester()
1896:         .dimensionality(3)
1897:         .inputSize(8, 7, 10)
1898:         .paddingWidth(1)
1899:         .kernelSize(1, 1, 3)
1900:         .groupInputChannels(5)
1901:         .groupInputChannels(6)
1902:         .iterations(3)
1903:         .per_channel(true))
1904: 
```

- **EN:** This block handles conditional branches and special cases. Key symbols: `ConvolutionOperatorTester`.
- **CN:** 该代码块处理条件分支与特殊情况。关键符号：`ConvolutionOperatorTester`。

### Lines 1905-1945 / 第 1905-1945 行

```cpp
1905: _STATIC_AND_RUNTIME_TEST(
1906:     CONVOLUTION_3D_OP,
1907:     grouped_1x1x3_per_channel,
1908:     ConvolutionOperatorTester()
1909:         .dimensionality(3)
1910:         .inputSize(8, 7, 10)
1911:         .paddingWidth(1)
1912:         .kernelSize(1, 1, 3)
1913:         .groups(2)
1914:         .groupInputChannels(5)
1915:         .groupInputChannels(6)
1916:         .iterations(3)
1917:         .per_channel(true))
1918: 
1919: _STATIC_AND_RUNTIME_TEST(
1920:     CONVOLUTION_3D_OP,
1921:     3x3x1_per_channel,
1922:     ConvolutionOperatorTester()
1923:         .dimensionality(3)
1924:         .inputSize(8, 9, 7)
1925:         .paddingHeight(1)
1926:         .kernelSize(3, 3, 1)
1927:         .groupInputChannels(5)
1928:         .groupInputChannels(6)
1929:         .iterations(3)
1930:         .per_channel(true))
1931: 
1932: _STATIC_AND_RUNTIME_TEST(
1933:     CONVOLUTION_3D_OP,
1934:     grouped_3x3x1_per_channel,
1935:     ConvolutionOperatorTester()
1936:         .dimensionality(3)
1937:         .inputSize(8, 9, 7)
1938:         .paddingHeight(1)
1939:         .kernelSize(3, 3, 1)
1940:         .groups(2)
1941:         .groupInputChannels(5)
1942:         .groupInputChannels(6)
1943:         .iterations(3)
1944:         .per_channel(true))
1945: 
```

- **EN:** This block implements local helper logic for `convolution`. Key symbols: no prominent local symbols.
- **CN:** 该代码块实现与 `convolution` 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 1946-1996 / 第 1946-1996 行

```cpp
1946: _STATIC_AND_RUNTIME_TEST(
1947:     CONVOLUTION_3D_OP,
1948:     3x3x3_per_channel,
1949:     ConvolutionOperatorTester()
1950:         .dimensionality(3)
1951:         .inputSize(8, 10, 9)
1952:         .padding(1)
1953:         .kernelSize(3, 3, 3)
1954:         .groupInputChannels(5)
1955:         .groupInputChannels(6)
1956:         .iterations(3)
1957:         .per_channel(true))
1958: 
1959: _STATIC_AND_RUNTIME_TEST(
1960:     CONVOLUTION_3D_OP,
1961:     3x3x3_without_padding_per_channel,
1962:     ConvolutionOperatorTester()
1963:         .dimensionality(3)
1964:         .inputSize(8, 10, 9)
1965:         .kernelSize(3, 3, 3)
1966:         .groupInputChannels(5)
1967:         .groupInputChannels(6)
1968:         .iterations(3)
1969:         .per_channel(true))
1970: 
1971: _STATIC_AND_RUNTIME_TEST(
1972:     CONVOLUTION_3D_OP,
1973:     3x3x3_with_width_padding_per_channel,
1974:     ConvolutionOperatorTester()
1975:         .dimensionality(3)
1976:         .inputSize(8, 10, 9)
1977:         .paddingWidth(1)
1978:         .kernelSize(3, 3, 3)
1979:         .groupInputChannels(5)
1980:         .groupInputChannels(6)
1981:         .iterations(3)
1982:         .per_channel(true))
1983: 
1984: _STATIC_AND_RUNTIME_TEST(
1985:     CONVOLUTION_3D_OP,
1986:     3x3x3_with_height_padding_per_channel,
1987:     ConvolutionOperatorTester()
1988:         .dimensionality(3)
1989:         .inputSize(8, 10, 9)
1990:         .paddingHeight(1)
1991:         .kernelSize(3, 3, 3)
1992:         .groupInputChannels(5)
1993:         .groupInputChannels(6)
1994:         .iterations(3)
1995:         .per_channel(true))
1996: 
```

- **EN:** This block implements local helper logic for `convolution`. Key symbols: no prominent local symbols.
- **CN:** 该代码块实现与 `convolution` 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 1997-2037 / 第 1997-2037 行

```cpp
1997: _STATIC_AND_RUNTIME_TEST(
1998:     CONVOLUTION_3D_OP,
1999:     3x3x3_with_depth_padding_per_channel,
2000:     ConvolutionOperatorTester()
2001:         .dimensionality(3)
2002:         .inputSize(8, 10, 9)
2003:         .paddingDepth(1)
2004:         .kernelSize(3, 3, 3)
2005:         .groupInputChannels(5)
2006:         .groupInputChannels(6)
2007:         .iterations(3)
2008:         .per_channel(true))
2009: 
2010: _STATIC_TEST(
2011:     CONVOLUTION_3D_OP,
2012:     3x3x3_with_input_stride_per_channel,
2013:     ConvolutionOperatorTester()
2014:         .dimensionality(3)
2015:         .inputSize(8, 10, 9)
2016:         .padding(1)
2017:         .kernelSize(3, 3, 3)
2018:         .inputPixelStride(22)
2019:         .groupInputChannels(5)
2020:         .groupInputChannels(6)
2021:         .iterations(3)
2022:         .per_channel(true))
2023: 
2024: _STATIC_TEST(
2025:     CONVOLUTION_3D_OP,
2026:     3x3x3_with_output_stride_per_channel,
2027:     ConvolutionOperatorTester()
2028:         .dimensionality(3)
2029:         .inputSize(8, 10, 9)
2030:         .padding(1)
2031:         .kernelSize(3, 3, 3)
2032:         .outputPixelStride(23)
2033:         .groupInputChannels(5)
2034:         .groupInputChannels(6)
2035:         .iterations(3)
2036:         .per_channel(true))
2037: 
```

- **EN:** This block implements local helper logic for `convolution`. Key symbols: no prominent local symbols.
- **CN:** 该代码块实现与 `convolution` 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 2038-2079 / 第 2038-2079 行

```cpp
2038: _STATIC_AND_RUNTIME_TEST(
2039:     CONVOLUTION_3D_OP,
2040:     3x3x3_with_batch_per_channel,
2041:     ConvolutionOperatorTester()
2042:         .dimensionality(3)
2043:         .inputSize(10, 10, 9)
2044:         .padding(1)
2045:         .kernelSize(3, 3, 3)
2046:         .batchSize(3)
2047:         .groupInputChannels(5)
2048:         .groupInputChannels(6)
2049:         .iterations(3)
2050:         .per_channel(true))
2051: 
2052: _STATIC_AND_RUNTIME_TEST(
2053:     CONVOLUTION_3D_OP,
2054:     grouped_3x3x3_per_channel,
2055:     ConvolutionOperatorTester()
2056:         .dimensionality(3)
2057:         .inputSize(10, 10, 11)
2058:         .padding(1)
2059:         .kernelSize(3, 3, 3)
2060:         .groups(2)
2061:         .groupInputChannels(5)
2062:         .groupInputChannels(6)
2063:         .iterations(3)
2064:         .per_channel(true))
2065: 
2066: _STATIC_AND_RUNTIME_TEST(
2067:     CONVOLUTION_3D_OP,
2068:     3x3x3s2_per_channel,
2069:     ConvolutionOperatorTester()
2070:         .dimensionality(3)
2071:         .inputSize(8, 10, 12)
2072:         .padding(1)
2073:         .kernelSize(3, 3, 3)
2074:         .subsampling(2)
2075:         .groupInputChannels(5)
2076:         .groupInputChannels(6)
2077:         .iterations(3)
2078:         .per_channel(true))
2079: 
```

- **EN:** This block implements local helper logic for `convolution`. Key symbols: no prominent local symbols.
- **CN:** 该代码块实现与 `convolution` 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 2080-2121 / 第 2080-2121 行

```cpp
2080: _STATIC_AND_RUNTIME_TEST(
2081:     CONVOLUTION_3D_OP,
2082:     3x3x3s1x2_per_channel,
2083:     ConvolutionOperatorTester()
2084:         .dimensionality(3)
2085:         .inputSize(8, 9, 10)
2086:         .padding(1)
2087:         .kernelSize(3, 3, 3)
2088:         .subsampling(1, 1, 2)
2089:         .groupInputChannels(5)
2090:         .groupInputChannels(6)
2091:         .iterations(3)
2092:         .per_channel(true))
2093: 
2094: _STATIC_AND_RUNTIME_TEST(
2095:     CONVOLUTION_3D_OP,
2096:     3x3x3s2x1_per_channel,
2097:     ConvolutionOperatorTester()
2098:         .dimensionality(3)
2099:         .inputSize(8, 9, 10)
2100:         .padding(1)
2101:         .kernelSize(3, 3, 3)
2102:         .subsampling(2, 2, 1)
2103:         .groupInputChannels(5)
2104:         .groupInputChannels(6)
2105:         .iterations(3)
2106:         .per_channel(true))
2107: 
2108: _STATIC_AND_RUNTIME_TEST(
2109:     CONVOLUTION_3D_OP,
2110:     3x3x3d2_per_channel,
2111:     ConvolutionOperatorTester()
2112:         .dimensionality(3)
2113:         .inputSize(7, 8, 10)
2114:         .padding(2)
2115:         .kernelSize(3, 3, 3)
2116:         .dilation(2)
2117:         .groupInputChannels(5)
2118:         .groupInputChannels(6)
2119:         .iterations(3)
2120:         .per_channel(true))
2121: 
```

- **EN:** This block implements local helper logic for `convolution`. Key symbols: no prominent local symbols.
- **CN:** 该代码块实现与 `convolution` 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 2122-2161 / 第 2122-2161 行

```cpp
2122: _STATIC_AND_RUNTIME_TEST(
2123:     CONVOLUTION_3D_OP,
2124:     3x3x3d1x2_per_channel,
2125:     ConvolutionOperatorTester()
2126:         .dimensionality(3)
2127:         .inputSize(11, 10, 8)
2128:         .padding(1, 1, 2)
2129:         .kernelSize(3, 3, 3)
2130:         .dilation(1, 1, 2)
2131:         .groupInputChannels(5)
2132:         .groupInputChannels(6)
2133:         .iterations(3)
2134:         .per_channel(true))
2135: 
2136: _STATIC_AND_RUNTIME_TEST(
2137:     CONVOLUTION_3D_OP,
2138:     3x3x3d2x1_per_channel,
2139:     ConvolutionOperatorTester()
2140:         .dimensionality(3)
2141:         .inputSize(10, 9, 7)
2142:         .padding(2, 2, 1)
2143:         .kernelSize(3, 3, 3)
2144:         .dilation(2, 2, 1)
2145:         .groupInputChannels(5)
2146:         .groupInputChannels(6)
2147:         .iterations(3)
2148:         .per_channel(true))
2149: 
2150: _STATIC_AND_RUNTIME_TEST(
2151:     CONVOLUTION_3D_OP,
2152:     depthwise_3x3x3_per_channel,
2153:     ConvolutionOperatorTester()
2154:         .dimensionality(3)
2155:         .inputSize(10, 9, 7)
2156:         .padding(1, 1, 1)
2157:         .kernelSize(3, 3, 3)
2158:         .groups(27)
2159:         .iterations(3)
2160:         .per_channel(true))
2161: 
```

- **EN:** This block implements local helper logic for `convolution`. Key symbols: no prominent local symbols.
- **CN:** 该代码块实现与 `convolution` 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 2162-2213 / 第 2162-2213 行

```cpp
2162: _STATIC_AND_RUNTIME_TEST(
2163:     CONVOLUTION_3D_OP,
2164:     depthwise_3x3x3s2_per_channel,
2165:     ConvolutionOperatorTester()
2166:         .dimensionality(3)
2167:         .inputSize(10, 9, 7)
2168:         .padding(1, 1, 1)
2169:         .kernelSize(3, 3, 3)
2170:         .subsampling(2)
2171:         .groups(27)
2172:         .iterations(3)
2173:         .per_channel(true))
2174: 
2175: _STATIC_AND_RUNTIME_TEST(
2176:     CONVOLUTION_3D_OP,
2177:     depthwise_3x3x3s1x2_per_channel,
2178:     ConvolutionOperatorTester()
2179:         .dimensionality(3)
2180:         .inputSize(10, 9, 7)
2181:         .padding(1, 1, 1)
2182:         .kernelSize(3, 3, 3)
2183:         .subsampling(1, 1, 2)
2184:         .groups(27)
2185:         .iterations(3)
2186:         .per_channel(true))
2187: 
2188: _STATIC_AND_RUNTIME_TEST(
2189:     CONVOLUTION_3D_OP,
2190:     depthwise_3x3x3s2x1_per_channel,
2191:     ConvolutionOperatorTester()
2192:         .dimensionality(3)
2193:         .inputSize(10, 9, 7)
2194:         .padding(1, 1, 1)
2195:         .kernelSize(3, 3, 3)
2196:         .subsampling(2, 2, 1)
2197:         .groups(27)
2198:         .iterations(3)
2199:         .per_channel(true))
2200: 
2201: _STATIC_AND_RUNTIME_TEST(
2202:     CONVOLUTION_3D_OP,
2203:     depthwise_3x3x3d2_per_channel,
2204:     ConvolutionOperatorTester()
2205:         .dimensionality(3)
2206:         .inputSize(10, 9, 7)
2207:         .padding(1, 1, 1)
2208:         .kernelSize(3, 3, 3)
2209:         .dilation(2)
2210:         .groups(27)
2211:         .iterations(3)
2212:         .per_channel(true))
2213: 
```

- **EN:** This block implements local helper logic for `convolution`. Key symbols: no prominent local symbols.
- **CN:** 该代码块实现与 `convolution` 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 2214-2238 / 第 2214-2238 行

```cpp
2214: _STATIC_AND_RUNTIME_TEST(
2215:     CONVOLUTION_3D_OP,
2216:     depthwise_3x3x3d1x2_per_channel,
2217:     ConvolutionOperatorTester()
2218:         .dimensionality(3)
2219:         .inputSize(10, 9, 7)
2220:         .padding(1, 1, 1)
2221:         .kernelSize(3, 3, 3)
2222:         .dilation(1, 1, 2)
2223:         .groups(27)
2224:         .iterations(3)
2225:         .per_channel(true))
2226: 
2227: _STATIC_AND_RUNTIME_TEST(
2228:     CONVOLUTION_3D_OP,
2229:     depthwise_3x3x3d2x1_per_channel,
2230:     ConvolutionOperatorTester()
2231:         .dimensionality(3)
2232:         .inputSize(10, 9, 7)
2233:         .padding(1, 1, 1)
2234:         .kernelSize(3, 3, 3)
2235:         .dilation(2, 2, 1)
2236:         .groups(27)
2237:         .iterations(3)
2238:         .per_channel(true))
```

- **EN:** This block implements local helper logic for `convolution`. Key symbols: no prominent local symbols.
- **CN:** 该代码块实现与 `convolution` 相关的局部辅助逻辑。关键符号：无明显局部符号。


## Key Concepts / 关键概念
- **ATen native operator logic** — ATen 原生算子逻辑
- **GPU kernel/runtime interaction** — GPU 内核/运行时交互
- **Mixed-precision/autocast logic** — 混合精度/自动混精逻辑
- **Runtime validation and invariants** — 运行时校验与不变量
- **Core symbols: ConvolutionOperatorTester** — 核心符号：ConvolutionOperatorTester

## Dependencies / 依赖关系
- **Internal includes / 内部头文件**: 无
- **External includes / 外部头文件**: `gtest/gtest.h`, `qnnpack/params.h`, `convolution-operator-tester.h`
- **Namespaces / 命名空间**: `qnnpack::testing;`
- **Representative symbols / 代表性符号**: `ConvolutionOperatorTester`
