# average-pooling.cc — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `aten/src/ATen/native/quantized/cpu/qnnpack/test/average-pooling.cc`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements ATen native operators, checks, helper kernels, and backend dispatch behavior. This specific file centers on `average-pooling.cc`. The file header highlights: "All rights reserved. This source code is licensed under the BSD-style license found in the LICENSE file in the root directory of this source tree.."
- **Purpose (CN)**: 实现 ATen 原生算子、运行时检查、辅助内核以及后端分发行为。 该文件具体围绕 `average-pooling.cc` 展开。 文件头部注释也概括了其核心职责。

## Line-by-Line Analysis / 逐行分析
### Lines 1-32 / 第 1-32 行

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
0013: #include "average-pooling-operator-tester.h"
0014: 
0015: TEST(AVERAGE_POOLING_OP, zero_batch) {
0016:   ASSERT_EQ(pytorch_qnnp_status_success, pytorch_qnnp_initialize());
0017:   AveragePoolingOperatorTester()
0018:       .batchSize(0)
0019:       .inputHeight(2)
0020:       .inputWidth(4)
0021:       .poolingHeight(1)
0022:       .poolingWidth(2)
0023:       .channels(4)
0024:       .testQ8();
0025: }
0026: 
0027: TEST(AVERAGE_POOLING_OP, unit_batch_many_channels_small_1xM_pool) {
0028:   ASSERT_EQ(pytorch_qnnp_status_success, pytorch_qnnp_initialize());
0029:   for (size_t channels = pytorch_qnnp_params.q8avgpool.kr;
0030:        channels <= 3 * pytorch_qnnp_params.q8avgpool.kr;
0031:        channels++) {
0032:     for (size_t poolSize = 2; poolSize <= pytorch_qnnp_params.q8avgpool.mr;
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `AveragePoolingOperatorTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`AveragePoolingOperatorTester`。

### Lines 33-67 / 第 33-67 行

```cpp
0033:          poolSize++) {
0034:       AveragePoolingOperatorTester()
0035:           .batchSize(1)
0036:           .inputHeight(2)
0037:           .inputWidth(poolSize + 2)
0038:           .poolingHeight(1)
0039:           .poolingWidth(poolSize)
0040:           .channels(channels)
0041:           .testQ8();
0042:     }
0043:   }
0044: }
0045: 
0046: TEST(AVERAGE_POOLING_OP, unit_batch_many_channels_small_1xM_pool_with_padding) {
0047:   ASSERT_EQ(pytorch_qnnp_status_success, pytorch_qnnp_initialize());
0048:   for (size_t channels = pytorch_qnnp_params.q8avgpool.kr;
0049:        channels <= 3 * pytorch_qnnp_params.q8avgpool.kr;
0050:        channels += 3) {
0051:     for (size_t poolSize = 3; poolSize <= pytorch_qnnp_params.q8avgpool.mr;
0052:          poolSize++) {
0053:       for (size_t paddingWidth = 0; paddingWidth <= 1; paddingWidth++) {
0054:         AveragePoolingOperatorTester()
0055:             .batchSize(1)
0056:             .inputHeight(2)
0057:             .inputWidth(poolSize + 2)
0058:             .paddingWidth(paddingWidth)
0059:             .poolingHeight(1)
0060:             .poolingWidth(poolSize)
0061:             .channels(channels)
0062:             .testQ8();
0063:       }
0064:     }
0065:   }
0066: }
0067: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `AveragePoolingOperatorTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`AveragePoolingOperatorTester`。

### Lines 68-106 / 第 68-106 行

```cpp
0068: TEST(AVERAGE_POOLING_OP, unit_batch_many_channels_small_1xM_pool_with_stride) {
0069:   ASSERT_EQ(pytorch_qnnp_status_success, pytorch_qnnp_initialize());
0070:   for (size_t channels = pytorch_qnnp_params.q8avgpool.kr;
0071:        channels <= 3 * pytorch_qnnp_params.q8avgpool.kr;
0072:        channels += 3) {
0073:     for (size_t poolSize = 2; poolSize <= pytorch_qnnp_params.q8avgpool.mr;
0074:          poolSize++) {
0075:       AveragePoolingOperatorTester()
0076:           .batchSize(1)
0077:           .inputHeight(2)
0078:           .inputWidth(poolSize + 4)
0079:           .poolingHeight(1)
0080:           .poolingWidth(poolSize)
0081:           .strideWidth(2)
0082:           .channels(channels)
0083:           .testQ8();
0084:     }
0085:   }
0086: }
0087: 
0088: TEST(AVERAGE_POOLING_OP, unit_batch_many_channels_small_Mx1_pool) {
0089:   ASSERT_EQ(pytorch_qnnp_status_success, pytorch_qnnp_initialize());
0090:   for (size_t channels = pytorch_qnnp_params.q8avgpool.kr;
0091:        channels <= 3 * pytorch_qnnp_params.q8avgpool.kr;
0092:        channels++) {
0093:     for (size_t poolSize = 2; poolSize <= pytorch_qnnp_params.q8avgpool.mr;
0094:          poolSize++) {
0095:       AveragePoolingOperatorTester()
0096:           .batchSize(1)
0097:           .inputHeight(poolSize + 1)
0098:           .inputWidth(3)
0099:           .poolingHeight(poolSize)
0100:           .poolingWidth(1)
0101:           .channels(channels)
0102:           .testQ8();
0103:     }
0104:   }
0105: }
0106: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `AveragePoolingOperatorTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`AveragePoolingOperatorTester`。

### Lines 107-148 / 第 107-148 行

```cpp
0107: TEST(AVERAGE_POOLING_OP, unit_batch_many_channels_small_Mx1_pool_with_padding) {
0108:   ASSERT_EQ(pytorch_qnnp_status_success, pytorch_qnnp_initialize());
0109:   for (size_t channels = pytorch_qnnp_params.q8avgpool.kr;
0110:        channels <= 3 * pytorch_qnnp_params.q8avgpool.kr;
0111:        channels += 3) {
0112:     for (size_t poolSize = 2; poolSize <= pytorch_qnnp_params.q8avgpool.mr;
0113:          poolSize++) {
0114:       for (size_t paddingHeight = 0; paddingHeight <= 1; paddingHeight++) {
0115:         AveragePoolingOperatorTester()
0116:             .batchSize(1)
0117:             .inputHeight(poolSize + 1)
0118:             .inputWidth(3)
0119:             .paddingHeight(paddingHeight)
0120:             .poolingHeight(poolSize)
0121:             .poolingWidth(1)
0122:             .channels(channels)
0123:             .testQ8();
0124:       }
0125:     }
0126:   }
0127: }
0128: 
0129: TEST(AVERAGE_POOLING_OP, unit_batch_many_channels_small_Mx1_pool_with_stride) {
0130:   ASSERT_EQ(pytorch_qnnp_status_success, pytorch_qnnp_initialize());
0131:   for (size_t channels = pytorch_qnnp_params.q8avgpool.kr;
0132:        channels <= 3 * pytorch_qnnp_params.q8avgpool.kr;
0133:        channels += 3) {
0134:     for (size_t poolSize = 2; poolSize <= pytorch_qnnp_params.q8avgpool.mr;
0135:          poolSize++) {
0136:       AveragePoolingOperatorTester()
0137:           .batchSize(1)
0138:           .inputHeight(poolSize + 3)
0139:           .inputWidth(3)
0140:           .poolingHeight(poolSize)
0141:           .poolingWidth(1)
0142:           .strideHeight(2)
0143:           .channels(channels)
0144:           .testQ8();
0145:     }
0146:   }
0147: }
0148: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `AveragePoolingOperatorTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`AveragePoolingOperatorTester`。

### Lines 149-179 / 第 149-179 行

```cpp
0149: TEST(
0150:     AVERAGE_POOLING_OP,
0151:     unit_batch_many_channels_small_pool_with_input_stride) {
0152:   ASSERT_EQ(pytorch_qnnp_status_success, pytorch_qnnp_initialize());
0153:   for (size_t channels = pytorch_qnnp_params.q8avgpool.kr;
0154:        channels <= 3 * pytorch_qnnp_params.q8avgpool.kr;
0155:        channels += 3) {
0156:     for (size_t poolSize = 2; poolSize <= pytorch_qnnp_params.q8avgpool.mr;
0157:          poolSize++) {
0158:       AveragePoolingOperatorTester()
0159:           .batchSize(1)
0160:           .inputHeight(poolSize + 1)
0161:           .inputWidth(3)
0162:           .poolingHeight(poolSize)
0163:           .poolingWidth(1)
0164:           .channels(channels)
0165:           .inputPixelStride(5 * pytorch_qnnp_params.q8avgpool.kr)
0166:           .testQ8();
0167:       AveragePoolingOperatorTester()
0168:           .batchSize(1)
0169:           .inputHeight(2)
0170:           .inputWidth(poolSize + 2)
0171:           .poolingHeight(1)
0172:           .poolingWidth(poolSize)
0173:           .channels(channels)
0174:           .inputPixelStride(5 * pytorch_qnnp_params.q8avgpool.kr)
0175:           .testQ8();
0176:     }
0177:   }
0178: }
0179: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `AveragePoolingOperatorTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`AveragePoolingOperatorTester`。

### Lines 180-210 / 第 180-210 行

```cpp
0180: TEST(
0181:     AVERAGE_POOLING_OP,
0182:     unit_batch_many_channels_small_pool_with_output_stride) {
0183:   ASSERT_EQ(pytorch_qnnp_status_success, pytorch_qnnp_initialize());
0184:   for (size_t channels = pytorch_qnnp_params.q8avgpool.kr;
0185:        channels <= 3 * pytorch_qnnp_params.q8avgpool.kr;
0186:        channels += 3) {
0187:     for (size_t poolSize = 2; poolSize <= pytorch_qnnp_params.q8avgpool.mr;
0188:          poolSize++) {
0189:       AveragePoolingOperatorTester()
0190:           .batchSize(1)
0191:           .inputHeight(poolSize + 1)
0192:           .inputWidth(3)
0193:           .poolingHeight(poolSize)
0194:           .poolingWidth(1)
0195:           .channels(channels)
0196:           .outputPixelStride(5 * pytorch_qnnp_params.q8avgpool.kr)
0197:           .testQ8();
0198:       AveragePoolingOperatorTester()
0199:           .batchSize(1)
0200:           .inputHeight(2)
0201:           .inputWidth(poolSize + 2)
0202:           .poolingHeight(1)
0203:           .poolingWidth(poolSize)
0204:           .channels(channels)
0205:           .outputPixelStride(5 * pytorch_qnnp_params.q8avgpool.kr)
0206:           .testQ8();
0207:     }
0208:   }
0209: }
0210: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `AveragePoolingOperatorTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`AveragePoolingOperatorTester`。

### Lines 211-242 / 第 211-242 行

```cpp
0211: TEST(AVERAGE_POOLING_OP, unit_batch_many_channels_small_pool_with_input_scale) {
0212:   ASSERT_EQ(pytorch_qnnp_status_success, pytorch_qnnp_initialize());
0213:   for (size_t channels = pytorch_qnnp_params.q8avgpool.kr;
0214:        channels <= 3 * pytorch_qnnp_params.q8avgpool.kr;
0215:        channels += 3) {
0216:     for (size_t poolSize = 2; poolSize <= pytorch_qnnp_params.q8avgpool.mr;
0217:          poolSize++) {
0218:       for (float inputScale = 0.01f; inputScale < 100.0f;
0219:            inputScale *= 3.14159265f) {
0220:         AveragePoolingOperatorTester()
0221:             .batchSize(1)
0222:             .inputHeight(poolSize + 1)
0223:             .inputWidth(3)
0224:             .poolingHeight(poolSize)
0225:             .poolingWidth(1)
0226:             .channels(channels)
0227:             .inputScale(inputScale)
0228:             .testQ8();
0229:         AveragePoolingOperatorTester()
0230:             .batchSize(1)
0231:             .inputHeight(2)
0232:             .inputWidth(poolSize + 2)
0233:             .poolingHeight(1)
0234:             .poolingWidth(poolSize)
0235:             .channels(channels)
0236:             .inputScale(inputScale)
0237:             .testQ8();
0238:       }
0239:     }
0240:   }
0241: }
0242: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `AveragePoolingOperatorTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`AveragePoolingOperatorTester`。

### Lines 243-276 / 第 243-276 行

```cpp
0243: TEST(
0244:     AVERAGE_POOLING_OP,
0245:     unit_batch_many_channels_small_pool_with_input_zero_point) {
0246:   ASSERT_EQ(pytorch_qnnp_status_success, pytorch_qnnp_initialize());
0247:   for (size_t channels = pytorch_qnnp_params.q8avgpool.kr;
0248:        channels <= 3 * pytorch_qnnp_params.q8avgpool.kr;
0249:        channels += 3) {
0250:     for (size_t poolSize = 2; poolSize <= pytorch_qnnp_params.q8avgpool.mr;
0251:          poolSize++) {
0252:       for (int32_t inputZeroPoint = 0; inputZeroPoint <= 255;
0253:            inputZeroPoint += 51) {
0254:         AveragePoolingOperatorTester()
0255:             .batchSize(1)
0256:             .inputHeight(poolSize + 1)
0257:             .inputWidth(3)
0258:             .poolingHeight(poolSize)
0259:             .poolingWidth(1)
0260:             .channels(channels)
0261:             .inputZeroPoint(uint8_t(inputZeroPoint))
0262:             .testQ8();
0263:         AveragePoolingOperatorTester()
0264:             .batchSize(1)
0265:             .inputHeight(2)
0266:             .inputWidth(poolSize + 2)
0267:             .poolingHeight(1)
0268:             .poolingWidth(poolSize)
0269:             .channels(channels)
0270:             .inputZeroPoint(uint8_t(inputZeroPoint))
0271:             .testQ8();
0272:       }
0273:     }
0274:   }
0275: }
0276: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `AveragePoolingOperatorTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`AveragePoolingOperatorTester`。

### Lines 277-310 / 第 277-310 行

```cpp
0277: TEST(
0278:     AVERAGE_POOLING_OP,
0279:     unit_batch_many_channels_small_pool_with_output_scale) {
0280:   ASSERT_EQ(pytorch_qnnp_status_success, pytorch_qnnp_initialize());
0281:   for (size_t channels = pytorch_qnnp_params.q8avgpool.kr;
0282:        channels <= 3 * pytorch_qnnp_params.q8avgpool.kr;
0283:        channels += 3) {
0284:     for (size_t poolSize = 2; poolSize <= pytorch_qnnp_params.q8avgpool.mr;
0285:          poolSize++) {
0286:       for (float outputScale = 0.01f; outputScale < 100.0f;
0287:            outputScale *= 3.14159265f) {
0288:         AveragePoolingOperatorTester()
0289:             .batchSize(1)
0290:             .inputHeight(poolSize + 1)
0291:             .inputWidth(3)
0292:             .poolingHeight(poolSize)
0293:             .poolingWidth(1)
0294:             .channels(channels)
0295:             .outputScale(outputScale)
0296:             .testQ8();
0297:         AveragePoolingOperatorTester()
0298:             .batchSize(1)
0299:             .inputHeight(2)
0300:             .inputWidth(poolSize + 2)
0301:             .poolingHeight(1)
0302:             .poolingWidth(poolSize)
0303:             .channels(channels)
0304:             .outputScale(outputScale)
0305:             .testQ8();
0306:       }
0307:     }
0308:   }
0309: }
0310: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `AveragePoolingOperatorTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`AveragePoolingOperatorTester`。

### Lines 311-344 / 第 311-344 行

```cpp
0311: TEST(
0312:     AVERAGE_POOLING_OP,
0313:     unit_batch_many_channels_small_pool_with_output_zero_point) {
0314:   ASSERT_EQ(pytorch_qnnp_status_success, pytorch_qnnp_initialize());
0315:   for (size_t channels = pytorch_qnnp_params.q8avgpool.kr;
0316:        channels <= 3 * pytorch_qnnp_params.q8avgpool.kr;
0317:        channels += 3) {
0318:     for (size_t poolSize = 2; poolSize <= pytorch_qnnp_params.q8avgpool.mr;
0319:          poolSize++) {
0320:       for (int32_t outputZeroPoint = 0; outputZeroPoint <= 255;
0321:            outputZeroPoint += 51) {
0322:         AveragePoolingOperatorTester()
0323:             .batchSize(1)
0324:             .inputHeight(poolSize + 1)
0325:             .inputWidth(3)
0326:             .poolingHeight(poolSize)
0327:             .poolingWidth(1)
0328:             .channels(channels)
0329:             .outputZeroPoint(uint8_t(outputZeroPoint))
0330:             .testQ8();
0331:         AveragePoolingOperatorTester()
0332:             .batchSize(1)
0333:             .inputHeight(2)
0334:             .inputWidth(poolSize + 2)
0335:             .poolingHeight(1)
0336:             .poolingWidth(poolSize)
0337:             .channels(channels)
0338:             .outputZeroPoint(uint8_t(outputZeroPoint))
0339:             .testQ8();
0340:       }
0341:     }
0342:   }
0343: }
0344: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `AveragePoolingOperatorTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`AveragePoolingOperatorTester`。

### Lines 345-376 / 第 345-376 行

```cpp
0345: TEST(AVERAGE_POOLING_OP, unit_batch_many_channels_small_pool_with_qmin) {
0346:   ASSERT_EQ(pytorch_qnnp_status_success, pytorch_qnnp_initialize());
0347:   for (size_t channels = pytorch_qnnp_params.q8avgpool.kr;
0348:        channels <= 3 * pytorch_qnnp_params.q8avgpool.kr;
0349:        channels += 3) {
0350:     for (size_t poolSize = 2; poolSize <= pytorch_qnnp_params.q8avgpool.mr;
0351:          poolSize++) {
0352:       AveragePoolingOperatorTester()
0353:           .batchSize(1)
0354:           .inputHeight(poolSize + 1)
0355:           .inputWidth(3)
0356:           .poolingHeight(poolSize)
0357:           .poolingWidth(1)
0358:           .channels(channels)
0359:           .qmin(128)
0360:           .testQ8();
0361:       AveragePoolingOperatorTester()
0362:           .batchSize(1)
0363:           .inputHeight(2)
0364:           .inputWidth(poolSize + 2)
0365:           .poolingHeight(1)
0366:           .poolingWidth(poolSize)
0367:           .channels(channels)
0368:           .qmin(128)
0369:           .testQ8();
0370:     }
0371:   }
0372: }
0373: 
0374: TEST(AVERAGE_POOLING_OP, unit_batch_many_channels_small_pool_with_qmax) {
0375:   ASSERT_EQ(pytorch_qnnp_status_success, pytorch_qnnp_initialize());
0376:   for (size_t channels = pytorch_qnnp_params.q8avgpool.kr;
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `AveragePoolingOperatorTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`AveragePoolingOperatorTester`。

### Lines 377-408 / 第 377-408 行

```cpp
0377:        channels <= 3 * pytorch_qnnp_params.q8avgpool.kr;
0378:        channels += 3) {
0379:     for (size_t poolSize = 2; poolSize <= pytorch_qnnp_params.q8avgpool.mr;
0380:          poolSize++) {
0381:       AveragePoolingOperatorTester()
0382:           .batchSize(1)
0383:           .inputHeight(poolSize + 1)
0384:           .inputWidth(3)
0385:           .poolingHeight(poolSize)
0386:           .poolingWidth(1)
0387:           .channels(channels)
0388:           .qmax(128)
0389:           .testQ8();
0390:       AveragePoolingOperatorTester()
0391:           .batchSize(1)
0392:           .inputHeight(2)
0393:           .inputWidth(poolSize + 2)
0394:           .poolingHeight(1)
0395:           .poolingWidth(poolSize)
0396:           .channels(channels)
0397:           .qmax(128)
0398:           .testQ8();
0399:     }
0400:   }
0401: }
0402: 
0403: TEST(AVERAGE_POOLING_OP, unit_batch_many_channels_large_1xM_pool) {
0404:   ASSERT_EQ(pytorch_qnnp_status_success, pytorch_qnnp_initialize());
0405:   for (size_t channels = pytorch_qnnp_params.q8avgpool.kr;
0406:        channels <= 3 * pytorch_qnnp_params.q8avgpool.kr;
0407:        channels++) {
0408:     for (size_t poolSize = pytorch_qnnp_params.q8avgpool.mr + 1; poolSize <=
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `AveragePoolingOperatorTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`AveragePoolingOperatorTester`。

### Lines 409-444 / 第 409-444 行

```cpp
0409:          pytorch_qnnp_params.q8avgpool.mr + pytorch_qnnp_params.q8avgpool.qr;
0410:          poolSize++) {
0411:       AveragePoolingOperatorTester()
0412:           .batchSize(1)
0413:           .inputHeight(2)
0414:           .inputWidth(poolSize + 2)
0415:           .poolingHeight(1)
0416:           .poolingWidth(poolSize)
0417:           .channels(channels)
0418:           .testQ8();
0419:     }
0420:   }
0421: }
0422: 
0423: TEST(AVERAGE_POOLING_OP, unit_batch_many_channels_large_1xM_pool_with_padding) {
0424:   ASSERT_EQ(pytorch_qnnp_status_success, pytorch_qnnp_initialize());
0425:   for (size_t channels = pytorch_qnnp_params.q8avgpool.kr;
0426:        channels <= 3 * pytorch_qnnp_params.q8avgpool.kr;
0427:        channels += 3) {
0428:     for (size_t poolSize = 3; poolSize <= pytorch_qnnp_params.q8avgpool.mr;
0429:          poolSize++) {
0430:       for (size_t paddingWidth = 0; paddingWidth <= 1; paddingWidth++) {
0431:         AveragePoolingOperatorTester()
0432:             .batchSize(1)
0433:             .inputHeight(2)
0434:             .inputWidth(poolSize + 2)
0435:             .paddingWidth(paddingWidth)
0436:             .poolingHeight(1)
0437:             .poolingWidth(poolSize)
0438:             .channels(channels)
0439:             .testQ8();
0440:       }
0441:     }
0442:   }
0443: }
0444: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `AveragePoolingOperatorTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`AveragePoolingOperatorTester`。

### Lines 445-485 / 第 445-485 行

```cpp
0445: TEST(AVERAGE_POOLING_OP, unit_batch_many_channels_large_1xM_pool_with_stride) {
0446:   ASSERT_EQ(pytorch_qnnp_status_success, pytorch_qnnp_initialize());
0447:   for (size_t channels = pytorch_qnnp_params.q8avgpool.kr;
0448:        channels <= 3 * pytorch_qnnp_params.q8avgpool.kr;
0449:        channels++) {
0450:     for (size_t poolSize = pytorch_qnnp_params.q8avgpool.mr + 1; poolSize <=
0451:          pytorch_qnnp_params.q8avgpool.mr + pytorch_qnnp_params.q8avgpool.qr;
0452:          poolSize++) {
0453:       AveragePoolingOperatorTester()
0454:           .batchSize(1)
0455:           .inputHeight(2)
0456:           .inputWidth(poolSize + 4)
0457:           .poolingHeight(1)
0458:           .poolingWidth(poolSize)
0459:           .strideWidth(2)
0460:           .channels(channels)
0461:           .testQ8();
0462:     }
0463:   }
0464: }
0465: 
0466: TEST(AVERAGE_POOLING_OP, unit_batch_many_channels_large_Mx1_pool) {
0467:   ASSERT_EQ(pytorch_qnnp_status_success, pytorch_qnnp_initialize());
0468:   for (size_t channels = pytorch_qnnp_params.q8avgpool.kr;
0469:        channels <= 3 * pytorch_qnnp_params.q8avgpool.kr;
0470:        channels++) {
0471:     for (size_t poolSize = pytorch_qnnp_params.q8avgpool.mr + 1; poolSize <=
0472:          pytorch_qnnp_params.q8avgpool.mr + pytorch_qnnp_params.q8avgpool.qr;
0473:          poolSize++) {
0474:       AveragePoolingOperatorTester()
0475:           .batchSize(1)
0476:           .inputHeight(poolSize + 1)
0477:           .inputWidth(3)
0478:           .poolingHeight(poolSize)
0479:           .poolingWidth(1)
0480:           .channels(channels)
0481:           .testQ8();
0482:     }
0483:   }
0484: }
0485: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `AveragePoolingOperatorTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`AveragePoolingOperatorTester`。

### Lines 486-517 / 第 486-517 行

```cpp
0486: TEST(AVERAGE_POOLING_OP, unit_batch_many_channels_large_Mx1_pool_with_padding) {
0487:   ASSERT_EQ(pytorch_qnnp_status_success, pytorch_qnnp_initialize());
0488:   for (size_t channels = pytorch_qnnp_params.q8avgpool.kr;
0489:        channels <= 3 * pytorch_qnnp_params.q8avgpool.kr;
0490:        channels += 3) {
0491:     for (size_t poolSize = pytorch_qnnp_params.q8avgpool.mr + 1; poolSize <=
0492:          pytorch_qnnp_params.q8avgpool.mr + pytorch_qnnp_params.q8avgpool.qr;
0493:          poolSize++) {
0494:       for (size_t paddingHeight = 0; paddingHeight <= 1; paddingHeight++) {
0495:         AveragePoolingOperatorTester()
0496:             .batchSize(1)
0497:             .inputHeight(poolSize + 1)
0498:             .inputWidth(3)
0499:             .paddingHeight(paddingHeight)
0500:             .poolingHeight(poolSize)
0501:             .poolingWidth(1)
0502:             .channels(channels)
0503:             .testQ8();
0504:       }
0505:     }
0506:   }
0507: }
0508: 
0509: TEST(AVERAGE_POOLING_OP, unit_batch_many_channels_large_Mx1_pool_with_stride) {
0510:   ASSERT_EQ(pytorch_qnnp_status_success, pytorch_qnnp_initialize());
0511:   for (size_t channels = pytorch_qnnp_params.q8avgpool.kr;
0512:        channels <= 3 * pytorch_qnnp_params.q8avgpool.kr;
0513:        channels += 3) {
0514:     for (size_t poolSize = pytorch_qnnp_params.q8avgpool.mr + 1; poolSize <=
0515:          pytorch_qnnp_params.q8avgpool.mr + pytorch_qnnp_params.q8avgpool.qr;
0516:          poolSize++) {
0517:       for (size_t paddingHeight = 0; paddingHeight <= 1; paddingHeight++) {
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `AveragePoolingOperatorTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`AveragePoolingOperatorTester`。

### Lines 518-563 / 第 518-563 行

```cpp
0518:         AveragePoolingOperatorTester()
0519:             .batchSize(1)
0520:             .inputHeight(poolSize + 1)
0521:             .inputWidth(3)
0522:             .paddingHeight(paddingHeight)
0523:             .poolingHeight(poolSize)
0524:             .poolingWidth(1)
0525:             .channels(channels)
0526:             .testQ8();
0527:       }
0528:     }
0529:   }
0530: }
0531: 
0532: TEST(
0533:     AVERAGE_POOLING_OP,
0534:     unit_batch_many_channels_large_pool_with_input_stride) {
0535:   ASSERT_EQ(pytorch_qnnp_status_success, pytorch_qnnp_initialize());
0536:   for (size_t channels = pytorch_qnnp_params.q8avgpool.kr;
0537:        channels <= 3 * pytorch_qnnp_params.q8avgpool.kr;
0538:        channels++) {
0539:     for (size_t poolSize = pytorch_qnnp_params.q8avgpool.mr + 1; poolSize <=
0540:          pytorch_qnnp_params.q8avgpool.mr + pytorch_qnnp_params.q8avgpool.qr;
0541:          poolSize++) {
0542:       AveragePoolingOperatorTester()
0543:           .batchSize(1)
0544:           .inputHeight(poolSize + 1)
0545:           .inputWidth(3)
0546:           .poolingHeight(poolSize)
0547:           .poolingWidth(1)
0548:           .channels(channels)
0549:           .inputPixelStride(5 * pytorch_qnnp_params.q8avgpool.kr)
0550:           .testQ8();
0551:       AveragePoolingOperatorTester()
0552:           .batchSize(1)
0553:           .inputHeight(2)
0554:           .inputWidth(poolSize + 2)
0555:           .poolingHeight(1)
0556:           .poolingWidth(poolSize)
0557:           .channels(channels)
0558:           .inputPixelStride(5 * pytorch_qnnp_params.q8avgpool.kr)
0559:           .testQ8();
0560:     }
0561:   }
0562: }
0563: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `AveragePoolingOperatorTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`AveragePoolingOperatorTester`。

### Lines 564-596 / 第 564-596 行

```cpp
0564: TEST(AVERAGE_POOLING_OP, unit_batch_many_channels_large_pool_with_input_scale) {
0565:   ASSERT_EQ(pytorch_qnnp_status_success, pytorch_qnnp_initialize());
0566:   for (size_t channels = pytorch_qnnp_params.q8avgpool.kr;
0567:        channels <= 3 * pytorch_qnnp_params.q8avgpool.kr;
0568:        channels += 3) {
0569:     for (size_t poolSize = pytorch_qnnp_params.q8avgpool.mr + 1; poolSize <=
0570:          pytorch_qnnp_params.q8avgpool.mr + pytorch_qnnp_params.q8avgpool.qr;
0571:          poolSize++) {
0572:       for (float inputScale = 0.01f; inputScale < 100.0f;
0573:            inputScale *= 3.14159265f) {
0574:         AveragePoolingOperatorTester()
0575:             .batchSize(1)
0576:             .inputHeight(poolSize + 1)
0577:             .inputWidth(3)
0578:             .poolingHeight(poolSize)
0579:             .poolingWidth(1)
0580:             .channels(channels)
0581:             .inputScale(inputScale)
0582:             .testQ8();
0583:         AveragePoolingOperatorTester()
0584:             .batchSize(1)
0585:             .inputHeight(2)
0586:             .inputWidth(poolSize + 2)
0587:             .poolingHeight(1)
0588:             .poolingWidth(poolSize)
0589:             .channels(channels)
0590:             .inputScale(inputScale)
0591:             .testQ8();
0592:       }
0593:     }
0594:   }
0595: }
0596: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `AveragePoolingOperatorTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`AveragePoolingOperatorTester`。

### Lines 597-631 / 第 597-631 行

```cpp
0597: TEST(
0598:     AVERAGE_POOLING_OP,
0599:     unit_batch_many_channels_large_pool_with_input_zero_point) {
0600:   ASSERT_EQ(pytorch_qnnp_status_success, pytorch_qnnp_initialize());
0601:   for (size_t channels = pytorch_qnnp_params.q8avgpool.kr;
0602:        channels <= 3 * pytorch_qnnp_params.q8avgpool.kr;
0603:        channels += 3) {
0604:     for (size_t poolSize = pytorch_qnnp_params.q8avgpool.mr + 1; poolSize <=
0605:          pytorch_qnnp_params.q8avgpool.mr + pytorch_qnnp_params.q8avgpool.qr;
0606:          poolSize++) {
0607:       for (int32_t inputZeroPoint = 0; inputZeroPoint <= 255;
0608:            inputZeroPoint += 51) {
0609:         AveragePoolingOperatorTester()
0610:             .batchSize(1)
0611:             .inputHeight(poolSize + 1)
0612:             .inputWidth(3)
0613:             .poolingHeight(poolSize)
0614:             .poolingWidth(1)
0615:             .channels(channels)
0616:             .inputZeroPoint(uint8_t(inputZeroPoint))
0617:             .testQ8();
0618:         AveragePoolingOperatorTester()
0619:             .batchSize(1)
0620:             .inputHeight(2)
0621:             .inputWidth(poolSize + 2)
0622:             .poolingHeight(1)
0623:             .poolingWidth(poolSize)
0624:             .channels(channels)
0625:             .inputZeroPoint(uint8_t(inputZeroPoint))
0626:             .testQ8();
0627:       }
0628:     }
0629:   }
0630: }
0631: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `AveragePoolingOperatorTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`AveragePoolingOperatorTester`。

### Lines 632-663 / 第 632-663 行

```cpp
0632: TEST(
0633:     AVERAGE_POOLING_OP,
0634:     unit_batch_many_channels_large_pool_with_output_stride) {
0635:   ASSERT_EQ(pytorch_qnnp_status_success, pytorch_qnnp_initialize());
0636:   for (size_t channels = pytorch_qnnp_params.q8avgpool.kr;
0637:        channels <= 3 * pytorch_qnnp_params.q8avgpool.kr;
0638:        channels++) {
0639:     for (size_t poolSize = pytorch_qnnp_params.q8avgpool.mr + 1; poolSize <=
0640:          pytorch_qnnp_params.q8avgpool.mr + pytorch_qnnp_params.q8avgpool.qr;
0641:          poolSize++) {
0642:       AveragePoolingOperatorTester()
0643:           .batchSize(1)
0644:           .inputHeight(poolSize + 1)
0645:           .inputWidth(3)
0646:           .poolingHeight(poolSize)
0647:           .poolingWidth(1)
0648:           .channels(channels)
0649:           .outputPixelStride(5 * pytorch_qnnp_params.q8avgpool.kr)
0650:           .testQ8();
0651:       AveragePoolingOperatorTester()
0652:           .batchSize(1)
0653:           .inputHeight(2)
0654:           .inputWidth(poolSize + 2)
0655:           .poolingHeight(1)
0656:           .poolingWidth(poolSize)
0657:           .channels(channels)
0658:           .outputPixelStride(5 * pytorch_qnnp_params.q8avgpool.kr)
0659:           .testQ8();
0660:     }
0661:   }
0662: }
0663: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `AveragePoolingOperatorTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`AveragePoolingOperatorTester`。

### Lines 664-698 / 第 664-698 行

```cpp
0664: TEST(
0665:     AVERAGE_POOLING_OP,
0666:     unit_batch_many_channels_large_pool_with_output_scale) {
0667:   ASSERT_EQ(pytorch_qnnp_status_success, pytorch_qnnp_initialize());
0668:   for (size_t channels = pytorch_qnnp_params.q8avgpool.kr;
0669:        channels <= 3 * pytorch_qnnp_params.q8avgpool.kr;
0670:        channels += 3) {
0671:     for (size_t poolSize = pytorch_qnnp_params.q8avgpool.mr + 1; poolSize <=
0672:          pytorch_qnnp_params.q8avgpool.mr + pytorch_qnnp_params.q8avgpool.qr;
0673:          poolSize++) {
0674:       for (float outputScale = 0.01f; outputScale < 100.0f;
0675:            outputScale *= 3.14159265f) {
0676:         AveragePoolingOperatorTester()
0677:             .batchSize(1)
0678:             .inputHeight(poolSize + 1)
0679:             .inputWidth(3)
0680:             .poolingHeight(poolSize)
0681:             .poolingWidth(1)
0682:             .channels(channels)
0683:             .outputScale(outputScale)
0684:             .testQ8();
0685:         AveragePoolingOperatorTester()
0686:             .batchSize(1)
0687:             .inputHeight(2)
0688:             .inputWidth(poolSize + 2)
0689:             .poolingHeight(1)
0690:             .poolingWidth(poolSize)
0691:             .channels(channels)
0692:             .outputScale(outputScale)
0693:             .testQ8();
0694:       }
0695:     }
0696:   }
0697: }
0698: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `AveragePoolingOperatorTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`AveragePoolingOperatorTester`。

### Lines 699-733 / 第 699-733 行

```cpp
0699: TEST(
0700:     AVERAGE_POOLING_OP,
0701:     unit_batch_many_channels_large_pool_with_output_zero_point) {
0702:   ASSERT_EQ(pytorch_qnnp_status_success, pytorch_qnnp_initialize());
0703:   for (size_t channels = pytorch_qnnp_params.q8avgpool.kr;
0704:        channels <= 3 * pytorch_qnnp_params.q8avgpool.kr;
0705:        channels += 3) {
0706:     for (size_t poolSize = pytorch_qnnp_params.q8avgpool.mr + 1; poolSize <=
0707:          pytorch_qnnp_params.q8avgpool.mr + pytorch_qnnp_params.q8avgpool.qr;
0708:          poolSize++) {
0709:       for (int32_t outputZeroPoint = 0; outputZeroPoint <= 255;
0710:            outputZeroPoint += 51) {
0711:         AveragePoolingOperatorTester()
0712:             .batchSize(1)
0713:             .inputHeight(poolSize + 1)
0714:             .inputWidth(3)
0715:             .poolingHeight(poolSize)
0716:             .poolingWidth(1)
0717:             .channels(channels)
0718:             .outputZeroPoint(uint8_t(outputZeroPoint))
0719:             .testQ8();
0720:         AveragePoolingOperatorTester()
0721:             .batchSize(1)
0722:             .inputHeight(2)
0723:             .inputWidth(poolSize + 2)
0724:             .poolingHeight(1)
0725:             .poolingWidth(poolSize)
0726:             .channels(channels)
0727:             .outputZeroPoint(uint8_t(outputZeroPoint))
0728:             .testQ8();
0729:       }
0730:     }
0731:   }
0732: }
0733: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `AveragePoolingOperatorTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`AveragePoolingOperatorTester`。

### Lines 734-763 / 第 734-763 行

```cpp
0734: TEST(AVERAGE_POOLING_OP, unit_batch_many_channels_large_pool_with_qmin) {
0735:   ASSERT_EQ(pytorch_qnnp_status_success, pytorch_qnnp_initialize());
0736:   for (size_t channels = pytorch_qnnp_params.q8avgpool.kr;
0737:        channels <= 3 * pytorch_qnnp_params.q8avgpool.kr;
0738:        channels += 3) {
0739:     for (size_t poolSize = pytorch_qnnp_params.q8avgpool.mr + 1; poolSize <=
0740:          pytorch_qnnp_params.q8avgpool.mr + pytorch_qnnp_params.q8avgpool.qr;
0741:          poolSize++) {
0742:       AveragePoolingOperatorTester()
0743:           .batchSize(1)
0744:           .inputHeight(poolSize + 1)
0745:           .inputWidth(3)
0746:           .poolingHeight(poolSize)
0747:           .poolingWidth(1)
0748:           .channels(channels)
0749:           .qmin(128)
0750:           .testQ8();
0751:       AveragePoolingOperatorTester()
0752:           .batchSize(1)
0753:           .inputHeight(2)
0754:           .inputWidth(poolSize + 2)
0755:           .poolingHeight(1)
0756:           .poolingWidth(poolSize)
0757:           .channels(channels)
0758:           .qmin(128)
0759:           .testQ8();
0760:     }
0761:   }
0762: }
0763: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `AveragePoolingOperatorTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`AveragePoolingOperatorTester`。

### Lines 764-793 / 第 764-793 行

```cpp
0764: TEST(AVERAGE_POOLING_OP, unit_batch_many_channels_large_pool_with_qmax) {
0765:   ASSERT_EQ(pytorch_qnnp_status_success, pytorch_qnnp_initialize());
0766:   for (size_t channels = pytorch_qnnp_params.q8avgpool.kr;
0767:        channels <= 3 * pytorch_qnnp_params.q8avgpool.kr;
0768:        channels += 3) {
0769:     for (size_t poolSize = pytorch_qnnp_params.q8avgpool.mr + 1; poolSize <=
0770:          pytorch_qnnp_params.q8avgpool.mr + pytorch_qnnp_params.q8avgpool.qr;
0771:          poolSize++) {
0772:       AveragePoolingOperatorTester()
0773:           .batchSize(1)
0774:           .inputHeight(poolSize + 1)
0775:           .inputWidth(3)
0776:           .poolingHeight(poolSize)
0777:           .poolingWidth(1)
0778:           .channels(channels)
0779:           .qmax(128)
0780:           .testQ8();
0781:       AveragePoolingOperatorTester()
0782:           .batchSize(1)
0783:           .inputHeight(2)
0784:           .inputWidth(poolSize + 2)
0785:           .poolingHeight(1)
0786:           .poolingWidth(poolSize)
0787:           .channels(channels)
0788:           .qmax(128)
0789:           .testQ8();
0790:     }
0791:   }
0792: }
0793: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `AveragePoolingOperatorTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`AveragePoolingOperatorTester`。

### Lines 794-832 / 第 794-832 行

```cpp
0794: TEST(AVERAGE_POOLING_OP, unit_batch_few_channels_1xM_pool) {
0795:   ASSERT_EQ(pytorch_qnnp_status_success, pytorch_qnnp_initialize());
0796:   for (size_t channels = 1; channels < pytorch_qnnp_params.q8avgpool.kr;
0797:        channels++) {
0798:     for (size_t poolSize = 2; poolSize <= 2 * pytorch_qnnp_params.q8avgpool.kr;
0799:          poolSize++) {
0800:       AveragePoolingOperatorTester()
0801:           .batchSize(1)
0802:           .inputHeight(2)
0803:           .inputWidth(poolSize + 2)
0804:           .poolingHeight(1)
0805:           .poolingWidth(poolSize)
0806:           .channels(channels)
0807:           .testQ8();
0808:     }
0809:   }
0810: }
0811: 
0812: TEST(AVERAGE_POOLING_OP, unit_batch_few_channels_1xM_pool_with_padding) {
0813:   ASSERT_EQ(pytorch_qnnp_status_success, pytorch_qnnp_initialize());
0814:   for (size_t channels = 1; channels < pytorch_qnnp_params.q8avgpool.kr;
0815:        channels++) {
0816:     for (size_t poolSize = 3; poolSize <= pytorch_qnnp_params.q8avgpool.mr;
0817:          poolSize++) {
0818:       for (size_t paddingWidth = 0; paddingWidth <= 1; paddingWidth++) {
0819:         AveragePoolingOperatorTester()
0820:             .batchSize(1)
0821:             .inputHeight(2)
0822:             .inputWidth(poolSize + 2)
0823:             .paddingWidth(paddingWidth)
0824:             .poolingHeight(1)
0825:             .poolingWidth(poolSize)
0826:             .channels(channels)
0827:             .testQ8();
0828:       }
0829:     }
0830:   }
0831: }
0832: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `AveragePoolingOperatorTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`AveragePoolingOperatorTester`。

### Lines 833-869 / 第 833-869 行

```cpp
0833: TEST(AVERAGE_POOLING_OP, unit_batch_few_channels_1xM_pool_with_stride) {
0834:   ASSERT_EQ(pytorch_qnnp_status_success, pytorch_qnnp_initialize());
0835:   for (size_t channels = 1; channels < pytorch_qnnp_params.q8avgpool.kr;
0836:        channels++) {
0837:     for (size_t poolSize = 2; poolSize <= 2 * pytorch_qnnp_params.q8avgpool.kr;
0838:          poolSize++) {
0839:       AveragePoolingOperatorTester()
0840:           .batchSize(1)
0841:           .inputHeight(2)
0842:           .inputWidth(poolSize + 4)
0843:           .poolingHeight(1)
0844:           .poolingWidth(poolSize)
0845:           .strideWidth(2)
0846:           .channels(channels)
0847:           .testQ8();
0848:     }
0849:   }
0850: }
0851: 
0852: TEST(AVERAGE_POOLING_OP, unit_batch_few_channels_Mx1_pool) {
0853:   ASSERT_EQ(pytorch_qnnp_status_success, pytorch_qnnp_initialize());
0854:   for (size_t channels = 1; channels < pytorch_qnnp_params.q8avgpool.kr;
0855:        channels++) {
0856:     for (size_t poolSize = 2; poolSize <= 2 * pytorch_qnnp_params.q8avgpool.kr;
0857:          poolSize++) {
0858:       AveragePoolingOperatorTester()
0859:           .batchSize(1)
0860:           .inputHeight(poolSize + 1)
0861:           .inputWidth(3)
0862:           .poolingHeight(poolSize)
0863:           .poolingWidth(1)
0864:           .channels(channels)
0865:           .testQ8();
0866:     }
0867:   }
0868: }
0869: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `AveragePoolingOperatorTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`AveragePoolingOperatorTester`。

### Lines 870-909 / 第 870-909 行

```cpp
0870: TEST(AVERAGE_POOLING_OP, unit_batch_few_channels_Mx1_pool_with_padding) {
0871:   ASSERT_EQ(pytorch_qnnp_status_success, pytorch_qnnp_initialize());
0872:   for (size_t channels = 1; channels < pytorch_qnnp_params.q8avgpool.kr;
0873:        channels++) {
0874:     for (size_t poolSize = 2; poolSize <= 2 * pytorch_qnnp_params.q8avgpool.kr;
0875:          poolSize++) {
0876:       for (size_t paddingHeight = 0; paddingHeight <= 1; paddingHeight++) {
0877:         AveragePoolingOperatorTester()
0878:             .batchSize(1)
0879:             .inputHeight(poolSize + 1)
0880:             .inputWidth(3)
0881:             .paddingHeight(paddingHeight)
0882:             .poolingHeight(poolSize)
0883:             .poolingWidth(1)
0884:             .channels(channels)
0885:             .testQ8();
0886:       }
0887:     }
0888:   }
0889: }
0890: 
0891: TEST(AVERAGE_POOLING_OP, unit_batch_few_channels_Mx1_pool_with_stride) {
0892:   ASSERT_EQ(pytorch_qnnp_status_success, pytorch_qnnp_initialize());
0893:   for (size_t channels = 1; channels < pytorch_qnnp_params.q8avgpool.kr;
0894:        channels++) {
0895:     for (size_t poolSize = 2; poolSize <= 2 * pytorch_qnnp_params.q8avgpool.kr;
0896:          poolSize++) {
0897:       AveragePoolingOperatorTester()
0898:           .batchSize(1)
0899:           .inputHeight(poolSize + 3)
0900:           .inputWidth(3)
0901:           .poolingHeight(poolSize)
0902:           .poolingWidth(1)
0903:           .strideHeight(2)
0904:           .channels(channels)
0905:           .testQ8();
0906:     }
0907:   }
0908: }
0909: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `AveragePoolingOperatorTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`AveragePoolingOperatorTester`。

### Lines 910-940 / 第 910-940 行

```cpp
0910: TEST(AVERAGE_POOLING_OP, unit_batch_few_channels_with_input_stride) {
0911:   ASSERT_EQ(pytorch_qnnp_status_success, pytorch_qnnp_initialize());
0912:   for (size_t channels = 1; channels < pytorch_qnnp_params.q8avgpool.kr;
0913:        channels++) {
0914:     for (size_t poolSize = 2; poolSize <= 2 * pytorch_qnnp_params.q8avgpool.kr;
0915:          poolSize++) {
0916:       AveragePoolingOperatorTester()
0917:           .batchSize(1)
0918:           .inputHeight(poolSize + 1)
0919:           .inputWidth(3)
0920:           .poolingHeight(poolSize)
0921:           .poolingWidth(1)
0922:           .channels(channels)
0923:           .inputPixelStride(5 * pytorch_qnnp_params.q8avgpool.kr)
0924:           .testQ8();
0925:       AveragePoolingOperatorTester()
0926:           .batchSize(1)
0927:           .inputHeight(2)
0928:           .inputWidth(poolSize + 2)
0929:           .poolingHeight(1)
0930:           .poolingWidth(poolSize)
0931:           .channels(channels)
0932:           .inputPixelStride(5 * pytorch_qnnp_params.q8avgpool.kr)
0933:           .testQ8();
0934:     }
0935:   }
0936: }
0937: 
0938: TEST(AVERAGE_POOLING_OP, unit_batch_few_channels_with_input_scale) {
0939:   ASSERT_EQ(pytorch_qnnp_status_success, pytorch_qnnp_initialize());
0940:   for (size_t channels = 1; channels < pytorch_qnnp_params.q8avgpool.kr;
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `AveragePoolingOperatorTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`AveragePoolingOperatorTester`。

### Lines 941-971 / 第 941-971 行

```cpp
0941:        channels++) {
0942:     for (size_t poolSize = 2; poolSize <= 2 * pytorch_qnnp_params.q8avgpool.kr;
0943:          poolSize++) {
0944:       for (float inputScale = 0.01f; inputScale < 100.0f;
0945:            inputScale *= 3.14159265f) {
0946:         AveragePoolingOperatorTester()
0947:             .batchSize(1)
0948:             .inputHeight(poolSize + 1)
0949:             .inputWidth(3)
0950:             .poolingHeight(poolSize)
0951:             .poolingWidth(1)
0952:             .channels(channels)
0953:             .inputScale(inputScale)
0954:             .testQ8();
0955:         AveragePoolingOperatorTester()
0956:             .batchSize(1)
0957:             .inputHeight(2)
0958:             .inputWidth(poolSize + 2)
0959:             .poolingHeight(1)
0960:             .poolingWidth(poolSize)
0961:             .channels(channels)
0962:             .inputScale(inputScale)
0963:             .testQ8();
0964:       }
0965:     }
0966:   }
0967: }
0968: 
0969: TEST(AVERAGE_POOLING_OP, unit_batch_few_channels_with_input_zero_point) {
0970:   ASSERT_EQ(pytorch_qnnp_status_success, pytorch_qnnp_initialize());
0971:   for (size_t channels = 1; channels < pytorch_qnnp_params.q8avgpool.kr;
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `AveragePoolingOperatorTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`AveragePoolingOperatorTester`。

### Lines 972-1002 / 第 972-1002 行

```cpp
0972:        channels++) {
0973:     for (size_t poolSize = 2; poolSize <= 2 * pytorch_qnnp_params.q8avgpool.kr;
0974:          poolSize++) {
0975:       for (int32_t inputZeroPoint = 0; inputZeroPoint <= 255;
0976:            inputZeroPoint += 51) {
0977:         AveragePoolingOperatorTester()
0978:             .batchSize(1)
0979:             .inputHeight(poolSize + 1)
0980:             .inputWidth(3)
0981:             .poolingHeight(poolSize)
0982:             .poolingWidth(1)
0983:             .channels(channels)
0984:             .inputZeroPoint(uint8_t(inputZeroPoint))
0985:             .testQ8();
0986:         AveragePoolingOperatorTester()
0987:             .batchSize(1)
0988:             .inputHeight(2)
0989:             .inputWidth(poolSize + 2)
0990:             .poolingHeight(1)
0991:             .poolingWidth(poolSize)
0992:             .channels(channels)
0993:             .inputZeroPoint(uint8_t(inputZeroPoint))
0994:             .testQ8();
0995:       }
0996:     }
0997:   }
0998: }
0999: 
1000: TEST(AVERAGE_POOLING_OP, unit_batch_few_channels_with_output_stride) {
1001:   ASSERT_EQ(pytorch_qnnp_status_success, pytorch_qnnp_initialize());
1002:   for (size_t channels = 1; channels < pytorch_qnnp_params.q8avgpool.kr;
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `AveragePoolingOperatorTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`AveragePoolingOperatorTester`。

### Lines 1003-1032 / 第 1003-1032 行

```cpp
1003:        channels++) {
1004:     for (size_t poolSize = 2; poolSize <= 2 * pytorch_qnnp_params.q8avgpool.kr;
1005:          poolSize++) {
1006:       AveragePoolingOperatorTester()
1007:           .batchSize(1)
1008:           .inputHeight(poolSize + 1)
1009:           .inputWidth(3)
1010:           .poolingHeight(poolSize)
1011:           .poolingWidth(1)
1012:           .channels(channels)
1013:           .outputPixelStride(5 * pytorch_qnnp_params.q8avgpool.kr)
1014:           .testQ8();
1015:       AveragePoolingOperatorTester()
1016:           .batchSize(1)
1017:           .inputHeight(2)
1018:           .inputWidth(poolSize + 2)
1019:           .poolingHeight(1)
1020:           .poolingWidth(poolSize)
1021:           .channels(channels)
1022:           .outputPixelStride(5 * pytorch_qnnp_params.q8avgpool.kr)
1023:           .testQ8();
1024:     }
1025:   }
1026: }
1027: 
1028: TEST(AVERAGE_POOLING_OP, unit_batch_few_channels_with_output_scale) {
1029:   ASSERT_EQ(pytorch_qnnp_status_success, pytorch_qnnp_initialize());
1030:   for (size_t channels = 1; channels < pytorch_qnnp_params.q8avgpool.kr;
1031:        channels++) {
1032:     for (size_t poolSize = 2; poolSize <= 2 * pytorch_qnnp_params.q8avgpool.kr;
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `AveragePoolingOperatorTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`AveragePoolingOperatorTester`。

### Lines 1033-1063 / 第 1033-1063 行

```cpp
1033:          poolSize++) {
1034:       for (float outputScale = 0.01f; outputScale < 100.0f;
1035:            outputScale *= 3.14159265f) {
1036:         AveragePoolingOperatorTester()
1037:             .batchSize(1)
1038:             .inputHeight(poolSize + 1)
1039:             .inputWidth(3)
1040:             .poolingHeight(poolSize)
1041:             .poolingWidth(1)
1042:             .channels(channels)
1043:             .outputScale(outputScale)
1044:             .testQ8();
1045:         AveragePoolingOperatorTester()
1046:             .batchSize(1)
1047:             .inputHeight(2)
1048:             .inputWidth(poolSize + 2)
1049:             .poolingHeight(1)
1050:             .poolingWidth(poolSize)
1051:             .channels(channels)
1052:             .outputScale(outputScale)
1053:             .testQ8();
1054:       }
1055:     }
1056:   }
1057: }
1058: 
1059: TEST(AVERAGE_POOLING_OP, unit_batch_few_channels_with_output_zero_point) {
1060:   ASSERT_EQ(pytorch_qnnp_status_success, pytorch_qnnp_initialize());
1061:   for (size_t channels = 1; channels < pytorch_qnnp_params.q8avgpool.kr;
1062:        channels++) {
1063:     for (size_t poolSize = 2; poolSize <= 2 * pytorch_qnnp_params.q8avgpool.kr;
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `AveragePoolingOperatorTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`AveragePoolingOperatorTester`。

### Lines 1064-1094 / 第 1064-1094 行

```cpp
1064:          poolSize++) {
1065:       for (int32_t outputZeroPoint = 0; outputZeroPoint <= 255;
1066:            outputZeroPoint += 51) {
1067:         AveragePoolingOperatorTester()
1068:             .batchSize(1)
1069:             .inputHeight(poolSize + 1)
1070:             .inputWidth(3)
1071:             .poolingHeight(poolSize)
1072:             .poolingWidth(1)
1073:             .channels(channels)
1074:             .outputZeroPoint(uint8_t(outputZeroPoint))
1075:             .testQ8();
1076:         AveragePoolingOperatorTester()
1077:             .batchSize(1)
1078:             .inputHeight(2)
1079:             .inputWidth(poolSize + 2)
1080:             .poolingHeight(1)
1081:             .poolingWidth(poolSize)
1082:             .channels(channels)
1083:             .outputZeroPoint(uint8_t(outputZeroPoint))
1084:             .testQ8();
1085:       }
1086:     }
1087:   }
1088: }
1089: 
1090: TEST(AVERAGE_POOLING_OP, unit_batch_few_channels_with_qmin) {
1091:   ASSERT_EQ(pytorch_qnnp_status_success, pytorch_qnnp_initialize());
1092:   for (size_t channels = 1; channels < pytorch_qnnp_params.q8avgpool.kr;
1093:        channels++) {
1094:     for (size_t poolSize = 2; poolSize <= 2 * pytorch_qnnp_params.q8avgpool.kr;
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `AveragePoolingOperatorTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`AveragePoolingOperatorTester`。

### Lines 1095-1145 / 第 1095-1145 行

```cpp
1095:          poolSize++) {
1096:       AveragePoolingOperatorTester()
1097:           .batchSize(1)
1098:           .inputHeight(poolSize + 1)
1099:           .inputWidth(3)
1100:           .poolingHeight(poolSize)
1101:           .poolingWidth(1)
1102:           .channels(channels)
1103:           .qmin(128)
1104:           .testQ8();
1105:       AveragePoolingOperatorTester()
1106:           .batchSize(1)
1107:           .inputHeight(2)
1108:           .inputWidth(poolSize + 1)
1109:           .poolingHeight(1)
1110:           .poolingWidth(poolSize)
1111:           .channels(channels)
1112:           .qmin(128)
1113:           .testQ8();
1114:     }
1115:   }
1116: }
1117: 
1118: TEST(AVERAGE_POOLING_OP, unit_batch_few_channels_with_qmax) {
1119:   ASSERT_EQ(pytorch_qnnp_status_success, pytorch_qnnp_initialize());
1120:   for (size_t channels = 1; channels < pytorch_qnnp_params.q8avgpool.kr;
1121:        channels++) {
1122:     for (size_t poolSize = 2; poolSize <= 2 * pytorch_qnnp_params.q8avgpool.kr;
1123:          poolSize++) {
1124:       AveragePoolingOperatorTester()
1125:           .batchSize(1)
1126:           .inputHeight(poolSize + 1)
1127:           .inputWidth(3)
1128:           .poolingHeight(poolSize)
1129:           .poolingWidth(1)
1130:           .channels(channels)
1131:           .qmax(128)
1132:           .testQ8();
1133:       AveragePoolingOperatorTester()
1134:           .batchSize(1)
1135:           .inputHeight(2)
1136:           .inputWidth(poolSize + 1)
1137:           .poolingHeight(1)
1138:           .poolingWidth(poolSize)
1139:           .channels(channels)
1140:           .qmax(128)
1141:           .testQ8();
1142:     }
1143:   }
1144: }
1145: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `AveragePoolingOperatorTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`AveragePoolingOperatorTester`。

### Lines 1146-1177 / 第 1146-1177 行

```cpp
1146: TEST(AVERAGE_POOLING_OP, small_batch_many_channels_small_pool) {
1147:   ASSERT_EQ(pytorch_qnnp_status_success, pytorch_qnnp_initialize());
1148:   for (size_t channels = pytorch_qnnp_params.q8avgpool.kr;
1149:        channels <= 3 * pytorch_qnnp_params.q8avgpool.kr;
1150:        channels++) {
1151:     for (size_t poolSize = 2; poolSize <= pytorch_qnnp_params.q8avgpool.mr;
1152:          poolSize++) {
1153:       AveragePoolingOperatorTester()
1154:           .batchSize(3)
1155:           .inputHeight(poolSize + 1)
1156:           .inputWidth(3)
1157:           .poolingHeight(poolSize)
1158:           .poolingWidth(1)
1159:           .channels(channels)
1160:           .testQ8();
1161:       AveragePoolingOperatorTester()
1162:           .batchSize(3)
1163:           .inputHeight(2)
1164:           .inputWidth(poolSize + 2)
1165:           .poolingHeight(1)
1166:           .poolingWidth(poolSize)
1167:           .channels(channels)
1168:           .testQ8();
1169:     }
1170:   }
1171: }
1172: 
1173: TEST(
1174:     AVERAGE_POOLING_OP,
1175:     small_batch_many_channels_small_pool_with_input_stride) {
1176:   ASSERT_EQ(pytorch_qnnp_status_success, pytorch_qnnp_initialize());
1177:   for (size_t channels = pytorch_qnnp_params.q8avgpool.kr;
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `AveragePoolingOperatorTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`AveragePoolingOperatorTester`。

### Lines 1178-1208 / 第 1178-1208 行

```cpp
1178:        channels <= 3 * pytorch_qnnp_params.q8avgpool.kr;
1179:        channels += 3) {
1180:     for (size_t poolSize = 2; poolSize <= pytorch_qnnp_params.q8avgpool.mr;
1181:          poolSize++) {
1182:       AveragePoolingOperatorTester()
1183:           .batchSize(3)
1184:           .inputHeight(poolSize + 1)
1185:           .inputWidth(3)
1186:           .poolingHeight(poolSize)
1187:           .poolingWidth(1)
1188:           .channels(channels)
1189:           .inputPixelStride(5 * pytorch_qnnp_params.q8avgpool.kr)
1190:           .testQ8();
1191:       AveragePoolingOperatorTester()
1192:           .batchSize(3)
1193:           .inputHeight(2)
1194:           .inputWidth(poolSize + 1)
1195:           .poolingHeight(1)
1196:           .poolingWidth(poolSize)
1197:           .channels(channels)
1198:           .inputPixelStride(5 * pytorch_qnnp_params.q8avgpool.kr)
1199:           .testQ8();
1200:     }
1201:   }
1202: }
1203: 
1204: TEST(
1205:     AVERAGE_POOLING_OP,
1206:     small_batch_many_channels_small_pool_with_output_stride) {
1207:   ASSERT_EQ(pytorch_qnnp_status_success, pytorch_qnnp_initialize());
1208:   for (size_t channels = pytorch_qnnp_params.q8avgpool.kr;
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `AveragePoolingOperatorTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`AveragePoolingOperatorTester`。

### Lines 1209-1240 / 第 1209-1240 行

```cpp
1209:        channels <= 3 * pytorch_qnnp_params.q8avgpool.kr;
1210:        channels += 3) {
1211:     for (size_t poolSize = 2; poolSize <= pytorch_qnnp_params.q8avgpool.mr;
1212:          poolSize++) {
1213:       AveragePoolingOperatorTester()
1214:           .batchSize(3)
1215:           .inputHeight(poolSize + 1)
1216:           .inputWidth(3)
1217:           .poolingHeight(poolSize)
1218:           .poolingWidth(1)
1219:           .channels(channels)
1220:           .outputPixelStride(5 * pytorch_qnnp_params.q8avgpool.kr)
1221:           .testQ8();
1222:       AveragePoolingOperatorTester()
1223:           .batchSize(3)
1224:           .inputHeight(2)
1225:           .inputWidth(poolSize + 1)
1226:           .poolingHeight(1)
1227:           .poolingWidth(poolSize)
1228:           .channels(channels)
1229:           .outputPixelStride(5 * pytorch_qnnp_params.q8avgpool.kr)
1230:           .testQ8();
1231:     }
1232:   }
1233: }
1234: 
1235: TEST(AVERAGE_POOLING_OP, small_batch_many_channels_large_pool) {
1236:   ASSERT_EQ(pytorch_qnnp_status_success, pytorch_qnnp_initialize());
1237:   for (size_t channels = pytorch_qnnp_params.q8avgpool.kr;
1238:        channels <= 3 * pytorch_qnnp_params.q8avgpool.kr;
1239:        channels++) {
1240:     for (size_t poolSize = pytorch_qnnp_params.q8avgpool.mr + 1; poolSize <=
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `AveragePoolingOperatorTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`AveragePoolingOperatorTester`。

### Lines 1241-1270 / 第 1241-1270 行

```cpp
1241:          pytorch_qnnp_params.q8avgpool.mr + pytorch_qnnp_params.q8avgpool.qr;
1242:          poolSize++) {
1243:       AveragePoolingOperatorTester()
1244:           .batchSize(3)
1245:           .inputHeight(poolSize + 1)
1246:           .inputWidth(3)
1247:           .poolingHeight(poolSize)
1248:           .poolingWidth(1)
1249:           .channels(channels)
1250:           .testQ8();
1251:       AveragePoolingOperatorTester()
1252:           .batchSize(3)
1253:           .inputHeight(2)
1254:           .inputWidth(poolSize + 2)
1255:           .poolingHeight(1)
1256:           .poolingWidth(poolSize)
1257:           .channels(channels)
1258:           .testQ8();
1259:     }
1260:   }
1261: }
1262: 
1263: TEST(
1264:     AVERAGE_POOLING_OP,
1265:     small_batch_many_channels_large_pool_with_input_stride) {
1266:   ASSERT_EQ(pytorch_qnnp_status_success, pytorch_qnnp_initialize());
1267:   for (size_t channels = pytorch_qnnp_params.q8avgpool.kr;
1268:        channels <= 3 * pytorch_qnnp_params.q8avgpool.kr;
1269:        channels += 5) {
1270:     for (size_t poolSize = pytorch_qnnp_params.q8avgpool.mr + 1; poolSize <=
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `AveragePoolingOperatorTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`AveragePoolingOperatorTester`。

### Lines 1271-1302 / 第 1271-1302 行

```cpp
1271:          pytorch_qnnp_params.q8avgpool.mr + pytorch_qnnp_params.q8avgpool.qr;
1272:          poolSize++) {
1273:       AveragePoolingOperatorTester()
1274:           .batchSize(3)
1275:           .inputHeight(poolSize + 1)
1276:           .inputWidth(3)
1277:           .poolingHeight(poolSize)
1278:           .poolingWidth(1)
1279:           .channels(channels)
1280:           .inputPixelStride(5 * pytorch_qnnp_params.q8avgpool.kr)
1281:           .testQ8();
1282:       AveragePoolingOperatorTester()
1283:           .batchSize(3)
1284:           .inputHeight(2)
1285:           .inputWidth(poolSize + 1)
1286:           .poolingHeight(1)
1287:           .poolingWidth(poolSize)
1288:           .channels(channels)
1289:           .inputPixelStride(5 * pytorch_qnnp_params.q8avgpool.kr)
1290:           .testQ8();
1291:     }
1292:   }
1293: }
1294: 
1295: TEST(
1296:     AVERAGE_POOLING_OP,
1297:     small_batch_many_channels_large_pool_with_output_stride) {
1298:   ASSERT_EQ(pytorch_qnnp_status_success, pytorch_qnnp_initialize());
1299:   for (size_t channels = pytorch_qnnp_params.q8avgpool.kr;
1300:        channels <= 3 * pytorch_qnnp_params.q8avgpool.kr;
1301:        channels += 5) {
1302:     for (size_t poolSize = pytorch_qnnp_params.q8avgpool.mr + 1; poolSize <=
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `AveragePoolingOperatorTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`AveragePoolingOperatorTester`。

### Lines 1303-1352 / 第 1303-1352 行

```cpp
1303:          pytorch_qnnp_params.q8avgpool.mr + pytorch_qnnp_params.q8avgpool.qr;
1304:          poolSize++) {
1305:       AveragePoolingOperatorTester()
1306:           .batchSize(3)
1307:           .inputHeight(poolSize + 1)
1308:           .inputWidth(3)
1309:           .poolingHeight(poolSize)
1310:           .poolingWidth(1)
1311:           .channels(channels)
1312:           .outputPixelStride(5 * pytorch_qnnp_params.q8avgpool.kr)
1313:           .testQ8();
1314:       AveragePoolingOperatorTester()
1315:           .batchSize(3)
1316:           .inputHeight(2)
1317:           .inputWidth(poolSize + 1)
1318:           .poolingHeight(1)
1319:           .poolingWidth(poolSize)
1320:           .channels(channels)
1321:           .outputPixelStride(5 * pytorch_qnnp_params.q8avgpool.kr)
1322:           .testQ8();
1323:     }
1324:   }
1325: }
1326: 
1327: TEST(AVERAGE_POOLING_OP, small_batch_few_channels) {
1328:   ASSERT_EQ(pytorch_qnnp_status_success, pytorch_qnnp_initialize());
1329:   for (size_t channels = 1; channels < pytorch_qnnp_params.q8avgpool.kr;
1330:        channels++) {
1331:     for (size_t poolSize = 2; poolSize <= 2 * pytorch_qnnp_params.q8avgpool.kr;
1332:          poolSize++) {
1333:       AveragePoolingOperatorTester()
1334:           .batchSize(3)
1335:           .inputHeight(poolSize + 1)
1336:           .inputWidth(3)
1337:           .poolingHeight(poolSize)
1338:           .poolingWidth(1)
1339:           .channels(channels)
1340:           .testQ8();
1341:       AveragePoolingOperatorTester()
1342:           .batchSize(3)
1343:           .inputHeight(2)
1344:           .inputWidth(poolSize + 2)
1345:           .poolingHeight(1)
1346:           .poolingWidth(poolSize)
1347:           .channels(channels)
1348:           .testQ8();
1349:     }
1350:   }
1351: }
1352: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `AveragePoolingOperatorTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`AveragePoolingOperatorTester`。

### Lines 1353-1383 / 第 1353-1383 行

```cpp
1353: TEST(AVERAGE_POOLING_OP, small_batch_few_channels_with_input_stride) {
1354:   ASSERT_EQ(pytorch_qnnp_status_success, pytorch_qnnp_initialize());
1355:   for (size_t channels = 1; channels < pytorch_qnnp_params.q8avgpool.kr;
1356:        channels++) {
1357:     for (size_t poolSize = 2; poolSize <= 2 * pytorch_qnnp_params.q8avgpool.kr;
1358:          poolSize += 3) {
1359:       AveragePoolingOperatorTester()
1360:           .batchSize(3)
1361:           .inputHeight(poolSize + 1)
1362:           .inputWidth(3)
1363:           .poolingHeight(poolSize)
1364:           .poolingWidth(1)
1365:           .channels(channels)
1366:           .inputPixelStride(5 * pytorch_qnnp_params.q8avgpool.kr)
1367:           .testQ8();
1368:       AveragePoolingOperatorTester()
1369:           .batchSize(3)
1370:           .inputHeight(2)
1371:           .inputWidth(poolSize + 2)
1372:           .poolingHeight(1)
1373:           .poolingWidth(poolSize)
1374:           .channels(channels)
1375:           .inputPixelStride(5 * pytorch_qnnp_params.q8avgpool.kr)
1376:           .testQ8();
1377:     }
1378:   }
1379: }
1380: 
1381: TEST(AVERAGE_POOLING_OP, small_batch_few_channels_with_output_stride) {
1382:   ASSERT_EQ(pytorch_qnnp_status_success, pytorch_qnnp_initialize());
1383:   for (size_t channels = 1; channels < pytorch_qnnp_params.q8avgpool.kr;
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `AveragePoolingOperatorTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`AveragePoolingOperatorTester`。

### Lines 1384-1421 / 第 1384-1421 行

```cpp
1384:        channels++) {
1385:     for (size_t poolSize = 2; poolSize <= 2 * pytorch_qnnp_params.q8avgpool.kr;
1386:          poolSize += 3) {
1387:       AveragePoolingOperatorTester()
1388:           .batchSize(3)
1389:           .inputHeight(poolSize + 1)
1390:           .inputWidth(3)
1391:           .poolingHeight(poolSize)
1392:           .poolingWidth(1)
1393:           .channels(channels)
1394:           .outputPixelStride(5 * pytorch_qnnp_params.q8avgpool.kr)
1395:           .testQ8();
1396:       AveragePoolingOperatorTester()
1397:           .batchSize(3)
1398:           .inputHeight(2)
1399:           .inputWidth(poolSize + 2)
1400:           .poolingHeight(1)
1401:           .poolingWidth(poolSize)
1402:           .channels(channels)
1403:           .outputPixelStride(5 * pytorch_qnnp_params.q8avgpool.kr)
1404:           .testQ8();
1405:     }
1406:   }
1407: }
1408: 
1409: TEST(AVERAGE_POOLING_OP, setup_increasing_batch) {
1410:   ASSERT_EQ(pytorch_qnnp_status_success, pytorch_qnnp_initialize());
1411:   AveragePoolingOperatorTester()
1412:       .batchSize(3)
1413:       .nextBatchSize(5)
1414:       .inputHeight(8)
1415:       .inputWidth(8)
1416:       .poolingHeight(5)
1417:       .poolingWidth(3)
1418:       .channels(24)
1419:       .testSetupQ8();
1420: }
1421: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `AveragePoolingOperatorTester`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`AveragePoolingOperatorTester`。

### Lines 1422-1456 / 第 1422-1456 行

```cpp
1422: TEST(AVERAGE_POOLING_OP, setup_decreasing_batch) {
1423:   ASSERT_EQ(pytorch_qnnp_status_success, pytorch_qnnp_initialize());
1424:   AveragePoolingOperatorTester()
1425:       .batchSize(5)
1426:       .nextBatchSize(3)
1427:       .inputHeight(8)
1428:       .inputWidth(8)
1429:       .poolingHeight(5)
1430:       .poolingWidth(3)
1431:       .channels(24)
1432:       .testSetupQ8();
1433: }
1434: 
1435: TEST(AVERAGE_POOLING_OP, setup_changing_height) {
1436:   ASSERT_EQ(pytorch_qnnp_status_success, pytorch_qnnp_initialize());
1437:   AveragePoolingOperatorTester()
1438:       .batchSize(3)
1439:       .inputHeight(8)
1440:       .inputWidth(8)
1441:       .nextInputHeight(9)
1442:       .poolingHeight(5)
1443:       .poolingWidth(3)
1444:       .channels(24)
1445:       .testSetupQ8();
1446:   AveragePoolingOperatorTester()
1447:       .batchSize(3)
1448:       .inputHeight(8)
1449:       .inputWidth(8)
1450:       .nextInputHeight(7)
1451:       .poolingHeight(5)
1452:       .poolingWidth(3)
1453:       .channels(24)
1454:       .testSetupQ8();
1455: }
1456: 
```

- **EN:** This block implements local helper logic for `average-pooling`. Key symbols: `AveragePoolingOperatorTester`.
- **CN:** 该代码块实现与 `average-pooling` 相关的局部辅助逻辑。关键符号：`AveragePoolingOperatorTester`。

### Lines 1457-1491 / 第 1457-1491 行

```cpp
1457: TEST(AVERAGE_POOLING_OP, setup_changing_width) {
1458:   ASSERT_EQ(pytorch_qnnp_status_success, pytorch_qnnp_initialize());
1459:   AveragePoolingOperatorTester()
1460:       .batchSize(3)
1461:       .inputHeight(8)
1462:       .inputWidth(8)
1463:       .nextInputWidth(9)
1464:       .poolingHeight(5)
1465:       .poolingWidth(3)
1466:       .channels(24)
1467:       .testSetupQ8();
1468:   AveragePoolingOperatorTester()
1469:       .batchSize(3)
1470:       .inputHeight(8)
1471:       .inputWidth(8)
1472:       .nextInputWidth(7)
1473:       .poolingHeight(5)
1474:       .poolingWidth(3)
1475:       .channels(24)
1476:       .testSetupQ8();
1477: }
1478: 
1479: TEST(AVERAGE_POOLING_OP, setup_swap_height_and_width) {
1480:   ASSERT_EQ(pytorch_qnnp_status_success, pytorch_qnnp_initialize());
1481:   AveragePoolingOperatorTester()
1482:       .batchSize(3)
1483:       .inputHeight(9)
1484:       .inputWidth(8)
1485:       .nextInputHeight(8)
1486:       .nextInputWidth(9)
1487:       .poolingHeight(5)
1488:       .poolingWidth(3)
1489:       .channels(24)
1490:       .testSetupQ8();
1491: }
```

- **EN:** This block implements local helper logic for `average-pooling`. Key symbols: `AveragePoolingOperatorTester`.
- **CN:** 该代码块实现与 `average-pooling` 相关的局部辅助逻辑。关键符号：`AveragePoolingOperatorTester`。


## Key Concepts / 关键概念
- **ATen native operator logic** — ATen 原生算子逻辑
- **Runtime validation and invariants** — 运行时校验与不变量
- **Core symbols: AveragePoolingOperatorTester** — 核心符号：AveragePoolingOperatorTester

## Dependencies / 依赖关系
- **Internal includes / 内部头文件**: 无
- **External includes / 外部头文件**: `gtest/gtest.h`, `qnnpack/params.h`, `average-pooling-operator-tester.h`
- **Namespaces / 命名空间**: 无
- **Representative symbols / 代表性符号**: `AveragePoolingOperatorTester`
