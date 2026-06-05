# fully-connected.cc — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `aten/src/ATen/native/quantized/cpu/qnnpack/test/fully-connected.cc`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements ATen native operators, checks, helper kernels, and backend dispatch behavior. This specific file centers on `fully-connected.cc`. The file header highlights: "All rights reserved. This source code is licensed under the BSD-style license found in the LICENSE file in the root directory of this source tree.."
- **Purpose (CN)**: 实现 ATen 原生算子、运行时检查、辅助内核以及后端分发行为。 该文件具体围绕 `fully-connected.cc` 展开。 文件头部注释也概括了其核心职责。

## Line-by-Line Analysis / 逐行分析
### Lines 1-21 / 第 1-21 行

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
0011: #include "fully-connected-operator-tester.h"
0012: 
0013: TEST(FULLY_CONNECTED_OP, integration_test_static) {
0014:   FullyConnectedOperatorTester()
0015:       .batchSize(4)
0016:       .inputChannels(4)
0017:       .outputChannels(4)
0018:       .iterations(3)
0019:       .testQ8(FullyConnectedOperatorTester::Mode::Static);
0020: }
0021: 
```

- **EN:** This block implements local helper logic for `fully-connected`. Key symbols: `FullyConnectedOperatorTester`.
- **CN:** 该代码块实现与 `fully-connected` 相关的局部辅助逻辑。关键符号：`FullyConnectedOperatorTester`。

### Lines 22-48 / 第 22-48 行

```cpp
0022: TEST(FULLY_CONNECTED_OP, integration_test_runtime) {
0023:   FullyConnectedOperatorTester()
0024:       .batchSize(4)
0025:       .inputChannels(4)
0026:       .outputChannels(4)
0027:       .iterations(3)
0028:       .testQ8(FullyConnectedOperatorTester::Mode::Runtime);
0029: }
0030: 
0031: TEST(FULLY_CONNECTED_OP, integration_test_dynamic) {
0032:   FullyConnectedOperatorTester()
0033:       .batchSize(4)
0034:       .inputChannels(4)
0035:       .outputChannels(4)
0036:       .iterations(3)
0037:       .testQ8(FullyConnectedOperatorTester::Mode::Dynamic);
0038: }
0039: 
0040: TEST(FULLY_CONNECTED_OP, zero_batch_static) {
0041:   FullyConnectedOperatorTester()
0042:       .batchSize(0)
0043:       .inputChannels(2)
0044:       .outputChannels(2)
0045:       .iterations(1)
0046:       .testQ8(FullyConnectedOperatorTester::Mode::Static);
0047: }
0048: 
```

- **EN:** This block implements local helper logic for `fully-connected`. Key symbols: `FullyConnectedOperatorTester`.
- **CN:** 该代码块实现与 `fully-connected` 相关的局部辅助逻辑。关键符号：`FullyConnectedOperatorTester`。

### Lines 49-75 / 第 49-75 行

```cpp
0049: TEST(FULLY_CONNECTED_OP, zero_batch_runtime) {
0050:   FullyConnectedOperatorTester()
0051:       .batchSize(0)
0052:       .inputChannels(2)
0053:       .outputChannels(2)
0054:       .iterations(1)
0055:       .testQ8(FullyConnectedOperatorTester::Mode::Runtime);
0056: }
0057: 
0058: TEST(FULLY_CONNECTED_OP, zero_batch_dynamic) {
0059:   FullyConnectedOperatorTester()
0060:       .batchSize(0)
0061:       .inputChannels(2)
0062:       .outputChannels(2)
0063:       .iterations(1)
0064:       .testQ8(FullyConnectedOperatorTester::Mode::Dynamic);
0065: }
0066: 
0067: TEST(FULLY_CONNECTED_OP, unit_batch_static) {
0068:   FullyConnectedOperatorTester()
0069:       .batchSize(1)
0070:       .inputChannels(23)
0071:       .outputChannels(19)
0072:       .iterations(3)
0073:       .testQ8(FullyConnectedOperatorTester::Mode::Static);
0074: }
0075: 
```

- **EN:** This block implements local helper logic for `fully-connected`. Key symbols: `FullyConnectedOperatorTester`.
- **CN:** 该代码块实现与 `fully-connected` 相关的局部辅助逻辑。关键符号：`FullyConnectedOperatorTester`。

### Lines 76-103 / 第 76-103 行

```cpp
0076: TEST(FULLY_CONNECTED_OP, unit_batch_runtime) {
0077:   FullyConnectedOperatorTester()
0078:       .batchSize(1)
0079:       .inputChannels(23)
0080:       .outputChannels(19)
0081:       .iterations(3)
0082:       .testQ8(FullyConnectedOperatorTester::Mode::Runtime);
0083: }
0084: 
0085: TEST(FULLY_CONNECTED_OP, unit_batch_dynamic) {
0086:   FullyConnectedOperatorTester()
0087:       .batchSize(1)
0088:       .inputChannels(23)
0089:       .outputChannels(19)
0090:       .iterations(3)
0091:       .testQ8(FullyConnectedOperatorTester::Mode::Dynamic);
0092: }
0093: 
0094: TEST(FULLY_CONNECTED_OP, unit_batch_with_qmin_static) {
0095:   FullyConnectedOperatorTester()
0096:       .batchSize(1)
0097:       .inputChannels(23)
0098:       .outputChannels(19)
0099:       .qmin(128)
0100:       .iterations(3)
0101:       .testQ8(FullyConnectedOperatorTester::Mode::Static);
0102: }
0103: 
```

- **EN:** This block implements local helper logic for `fully-connected`. Key symbols: `FullyConnectedOperatorTester`.
- **CN:** 该代码块实现与 `fully-connected` 相关的局部辅助逻辑。关键符号：`FullyConnectedOperatorTester`。

### Lines 104-123 / 第 104-123 行

```cpp
0104: TEST(FULLY_CONNECTED_OP, unit_batch_with_qmin_runtime) {
0105:   FullyConnectedOperatorTester()
0106:       .batchSize(1)
0107:       .inputChannels(23)
0108:       .outputChannels(19)
0109:       .qmin(128)
0110:       .iterations(3)
0111:       .testQ8(FullyConnectedOperatorTester::Mode::Runtime);
0112: }
0113: 
0114: TEST(FULLY_CONNECTED_OP, unit_batch_with_qmin_dynamic) {
0115:   FullyConnectedOperatorTester()
0116:       .batchSize(1)
0117:       .inputChannels(23)
0118:       .outputChannels(19)
0119:       .qmin(128)
0120:       .iterations(3)
0121:       .testQ8(FullyConnectedOperatorTester::Mode::Dynamic);
0122: }
0123: 
```

- **EN:** This block implements local helper logic for `fully-connected`. Key symbols: `FullyConnectedOperatorTester`.
- **CN:** 该代码块实现与 `fully-connected` 相关的局部辅助逻辑。关键符号：`FullyConnectedOperatorTester`。

### Lines 124-143 / 第 124-143 行

```cpp
0124: TEST(FULLY_CONNECTED_OP, unit_batch_with_qmax_static) {
0125:   FullyConnectedOperatorTester()
0126:       .batchSize(1)
0127:       .inputChannels(23)
0128:       .outputChannels(19)
0129:       .qmax(128)
0130:       .iterations(3)
0131:       .testQ8(FullyConnectedOperatorTester::Mode::Static);
0132: }
0133: 
0134: TEST(FULLY_CONNECTED_OP, unit_batch_with_qmax_runtime) {
0135:   FullyConnectedOperatorTester()
0136:       .batchSize(1)
0137:       .inputChannels(23)
0138:       .outputChannels(19)
0139:       .qmax(128)
0140:       .iterations(3)
0141:       .testQ8(FullyConnectedOperatorTester::Mode::Runtime);
0142: }
0143: 
```

- **EN:** This block implements local helper logic for `fully-connected`. Key symbols: `FullyConnectedOperatorTester`.
- **CN:** 该代码块实现与 `fully-connected` 相关的局部辅助逻辑。关键符号：`FullyConnectedOperatorTester`。

### Lines 144-163 / 第 144-163 行

```cpp
0144: TEST(FULLY_CONNECTED_OP, unit_batch_with_qmax_dynamic) {
0145:   FullyConnectedOperatorTester()
0146:       .batchSize(1)
0147:       .inputChannels(23)
0148:       .outputChannels(19)
0149:       .qmax(128)
0150:       .iterations(3)
0151:       .testQ8(FullyConnectedOperatorTester::Mode::Dynamic);
0152: }
0153: 
0154: TEST(FULLY_CONNECTED_OP, unit_batch_with_input_stride_static) {
0155:   FullyConnectedOperatorTester()
0156:       .batchSize(1)
0157:       .inputChannels(23)
0158:       .inputStride(28)
0159:       .outputChannels(19)
0160:       .iterations(3)
0161:       .testQ8(FullyConnectedOperatorTester::Mode::Static);
0162: }
0163: 
```

- **EN:** This block implements local helper logic for `fully-connected`. Key symbols: `FullyConnectedOperatorTester`.
- **CN:** 该代码块实现与 `fully-connected` 相关的局部辅助逻辑。关键符号：`FullyConnectedOperatorTester`。

### Lines 164-183 / 第 164-183 行

```cpp
0164: TEST(FULLY_CONNECTED_OP, unit_batch_with_input_stride_runtime) {
0165:   FullyConnectedOperatorTester()
0166:       .batchSize(1)
0167:       .inputChannels(23)
0168:       .inputStride(28)
0169:       .outputChannels(19)
0170:       .iterations(3)
0171:       .testQ8(FullyConnectedOperatorTester::Mode::Runtime);
0172: }
0173: 
0174: TEST(FULLY_CONNECTED_OP, unit_batch_with_input_stride_dynamic) {
0175:   FullyConnectedOperatorTester()
0176:       .batchSize(1)
0177:       .inputChannels(23)
0178:       .inputStride(28)
0179:       .outputChannels(19)
0180:       .iterations(3)
0181:       .testQ8(FullyConnectedOperatorTester::Mode::Dynamic);
0182: }
0183: 
```

- **EN:** This block implements local helper logic for `fully-connected`. Key symbols: `FullyConnectedOperatorTester`.
- **CN:** 该代码块实现与 `fully-connected` 相关的局部辅助逻辑。关键符号：`FullyConnectedOperatorTester`。

### Lines 184-203 / 第 184-203 行

```cpp
0184: TEST(FULLY_CONNECTED_OP, unit_batch_with_output_stride_static) {
0185:   FullyConnectedOperatorTester()
0186:       .batchSize(1)
0187:       .inputChannels(23)
0188:       .outputChannels(19)
0189:       .outputStride(29)
0190:       .iterations(3)
0191:       .testQ8(FullyConnectedOperatorTester::Mode::Static);
0192: }
0193: 
0194: TEST(FULLY_CONNECTED_OP, unit_batch_with_output_stride_runtime) {
0195:   FullyConnectedOperatorTester()
0196:       .batchSize(1)
0197:       .inputChannels(23)
0198:       .outputChannels(19)
0199:       .outputStride(29)
0200:       .iterations(3)
0201:       .testQ8(FullyConnectedOperatorTester::Mode::Runtime);
0202: }
0203: 
```

- **EN:** This block implements local helper logic for `fully-connected`. Key symbols: `FullyConnectedOperatorTester`.
- **CN:** 该代码块实现与 `fully-connected` 相关的局部辅助逻辑。关键符号：`FullyConnectedOperatorTester`。

### Lines 204-231 / 第 204-231 行

```cpp
0204: TEST(FULLY_CONNECTED_OP, unit_batch_with_output_stride_dynamic) {
0205:   FullyConnectedOperatorTester()
0206:       .batchSize(1)
0207:       .inputChannels(23)
0208:       .outputChannels(19)
0209:       .outputStride(29)
0210:       .iterations(3)
0211:       .testQ8(FullyConnectedOperatorTester::Mode::Dynamic);
0212: }
0213: 
0214: TEST(FULLY_CONNECTED_OP, small_batch_static) {
0215:   FullyConnectedOperatorTester()
0216:       .batchSize(12)
0217:       .inputChannels(23)
0218:       .outputChannels(19)
0219:       .iterations(3)
0220:       .testQ8(FullyConnectedOperatorTester::Mode::Static);
0221: }
0222: 
0223: TEST(FULLY_CONNECTED_OP, small_batch_runtime) {
0224:   FullyConnectedOperatorTester()
0225:       .batchSize(12)
0226:       .inputChannels(23)
0227:       .outputChannels(19)
0228:       .iterations(3)
0229:       .testQ8(FullyConnectedOperatorTester::Mode::Runtime);
0230: }
0231: 
```

- **EN:** This block implements local helper logic for `fully-connected`. Key symbols: `FullyConnectedOperatorTester`.
- **CN:** 该代码块实现与 `fully-connected` 相关的局部辅助逻辑。关键符号：`FullyConnectedOperatorTester`。

### Lines 232-260 / 第 232-260 行

```cpp
0232: TEST(FULLY_CONNECTED_OP, small_batch_dynamic) {
0233:   FullyConnectedOperatorTester()
0234:       .batchSize(12)
0235:       .inputChannels(23)
0236:       .outputChannels(19)
0237:       .iterations(3)
0238:       .testQ8(FullyConnectedOperatorTester::Mode::Dynamic);
0239: }
0240: 
0241: TEST(FULLY_CONNECTED_OP, small_batch_with_qmin_static) {
0242:   FullyConnectedOperatorTester()
0243:       .batchSize(12)
0244:       .inputChannels(23)
0245:       .outputChannels(19)
0246:       .qmin(128)
0247:       .iterations(3)
0248:       .testQ8(FullyConnectedOperatorTester::Mode::Static);
0249: }
0250: 
0251: TEST(FULLY_CONNECTED_OP, small_batch_with_qmin_runtime) {
0252:   FullyConnectedOperatorTester()
0253:       .batchSize(12)
0254:       .inputChannels(23)
0255:       .outputChannels(19)
0256:       .qmin(128)
0257:       .iterations(3)
0258:       .testQ8(FullyConnectedOperatorTester::Mode::Runtime);
0259: }
0260: 
```

- **EN:** This block implements local helper logic for `fully-connected`. Key symbols: `FullyConnectedOperatorTester`.
- **CN:** 该代码块实现与 `fully-connected` 相关的局部辅助逻辑。关键符号：`FullyConnectedOperatorTester`。

### Lines 261-280 / 第 261-280 行

```cpp
0261: TEST(FULLY_CONNECTED_OP, small_batch_with_qmin_dynamic) {
0262:   FullyConnectedOperatorTester()
0263:       .batchSize(12)
0264:       .inputChannels(23)
0265:       .outputChannels(19)
0266:       .qmin(128)
0267:       .iterations(3)
0268:       .testQ8(FullyConnectedOperatorTester::Mode::Dynamic);
0269: }
0270: 
0271: TEST(FULLY_CONNECTED_OP, small_batch_with_qmax) {
0272:   FullyConnectedOperatorTester()
0273:       .batchSize(12)
0274:       .inputChannels(23)
0275:       .outputChannels(19)
0276:       .qmax(128)
0277:       .iterations(3)
0278:       .testQ8(FullyConnectedOperatorTester::Mode::Static);
0279: }
0280: 
```

- **EN:** This block implements local helper logic for `fully-connected`. Key symbols: `FullyConnectedOperatorTester`.
- **CN:** 该代码块实现与 `fully-connected` 相关的局部辅助逻辑。关键符号：`FullyConnectedOperatorTester`。

### Lines 281-300 / 第 281-300 行

```cpp
0281: TEST(FULLY_CONNECTED_OP, small_batch_with_qmax_runtime) {
0282:   FullyConnectedOperatorTester()
0283:       .batchSize(12)
0284:       .inputChannels(23)
0285:       .outputChannels(19)
0286:       .qmax(128)
0287:       .iterations(3)
0288:       .testQ8(FullyConnectedOperatorTester::Mode::Runtime);
0289: }
0290: 
0291: TEST(FULLY_CONNECTED_OP, small_batch_with_qmax_dynamic) {
0292:   FullyConnectedOperatorTester()
0293:       .batchSize(12)
0294:       .inputChannels(23)
0295:       .outputChannels(19)
0296:       .qmax(128)
0297:       .iterations(3)
0298:       .testQ8(FullyConnectedOperatorTester::Mode::Dynamic);
0299: }
0300: 
```

- **EN:** This block implements local helper logic for `fully-connected`. Key symbols: `FullyConnectedOperatorTester`.
- **CN:** 该代码块实现与 `fully-connected` 相关的局部辅助逻辑。关键符号：`FullyConnectedOperatorTester`。

### Lines 301-321 / 第 301-321 行

```cpp
0301: TEST(FULLY_CONNECTED_OP, small_batch_with_input_stride_static) {
0302:   FullyConnectedOperatorTester()
0303:       .batchSize(12)
0304:       .inputChannels(23)
0305:       .inputStride(28)
0306:       .outputChannels(19)
0307:       .iterations(3)
0308:       .testQ8(FullyConnectedOperatorTester::Mode::Static);
0309: }
0310: 
0311: // TODO: Broken
0312: // TEST(FULLY_CONNECTED_OP, small_batch_with_input_stride_runtime) {
0313: //   FullyConnectedOperatorTester()
0314: //       .batchSize(12)
0315: //       .inputChannels(23)
0316: //       .inputStride(28)
0317: //       .outputChannels(19)
0318: //       .iterations(3)
0319: //       .testQ8(FullyConnectedOperatorTester::Mode::Runtime);
0320: // }
0321: 
```

- **EN:** This block implements local helper logic for `fully-connected`. Key symbols: `FullyConnectedOperatorTester`.
- **CN:** 该代码块实现与 `fully-connected` 相关的局部辅助逻辑。关键符号：`FullyConnectedOperatorTester`。

### Lines 322-341 / 第 322-341 行

```cpp
0322: // TEST(FULLY_CONNECTED_OP, small_batch_with_input_stride_dynamic) {
0323: //   FullyConnectedOperatorTester()
0324: //       .batchSize(12)
0325: //       .inputChannels(23)
0326: //       .inputStride(28)
0327: //       .outputChannels(19)
0328: //       .iterations(3)
0329: //       .testQ8(FullyConnectedOperatorTester::Mode::Dynamic);
0330: // }
0331: 
0332: TEST(FULLY_CONNECTED_OP, small_batch_with_output_stride_static) {
0333:   FullyConnectedOperatorTester()
0334:       .batchSize(12)
0335:       .inputChannels(23)
0336:       .outputChannels(19)
0337:       .outputStride(29)
0338:       .iterations(3)
0339:       .testQ8(FullyConnectedOperatorTester::Mode::Static);
0340: }
0341: 
```

- **EN:** This block implements local helper logic for `fully-connected`. Key symbols: `FullyConnectedOperatorTester`.
- **CN:** 该代码块实现与 `fully-connected` 相关的局部辅助逻辑。关键符号：`FullyConnectedOperatorTester`。

### Lines 342-361 / 第 342-361 行

```cpp
0342: // TEST(FULLY_CONNECTED_OP, small_batch_with_output_stride_runtime) {
0343: //   FullyConnectedOperatorTester()
0344: //       .batchSize(12)
0345: //       .inputChannels(23)
0346: //       .outputChannels(19)
0347: //       .outputStride(29)
0348: //       .iterations(3)
0349: //       .testQ8(FullyConnectedOperatorTester::Mode::Runtime);
0350: // }
0351: 
0352: // TEST(FULLY_CONNECTED_OP, small_batch_with_output_stride_dynamic) {
0353: //   FullyConnectedOperatorTester()
0354: //       .batchSize(12)
0355: //       .inputChannels(23)
0356: //       .outputChannels(19)
0357: //       .outputStride(29)
0358: //       .iterations(3)
0359: //       .testQ8(FullyConnectedOperatorTester::Mode::Dynamic);
0360: // }
0361: 
```

- **EN:** Documents design intent, constraints, compatibility notes, or usage expectations for the surrounding implementation.
- **CN:** 说明周围实现的设计目标、约束条件、兼容性注意事项或使用预期。

### Lines 362-381 / 第 362-381 行

```cpp
0362: TEST(FULLY_CONNECTED_OP, integration_test_static_per_channel) {
0363:   FullyConnectedOperatorTester()
0364:       .batchSize(4)
0365:       .inputChannels(4)
0366:       .outputChannels(4)
0367:       .iterations(3)
0368:       .per_channel(true)
0369:       .testQ8(FullyConnectedOperatorTester::Mode::Static);
0370: }
0371: 
0372: TEST(FULLY_CONNECTED_OP, integration_test_runtime_per_channel) {
0373:   FullyConnectedOperatorTester()
0374:       .batchSize(4)
0375:       .inputChannels(4)
0376:       .outputChannels(4)
0377:       .iterations(3)
0378:       .per_channel(true)
0379:       .testQ8(FullyConnectedOperatorTester::Mode::Runtime);
0380: }
0381: 
```

- **EN:** This block implements local helper logic for `fully-connected`. Key symbols: `FullyConnectedOperatorTester`.
- **CN:** 该代码块实现与 `fully-connected` 相关的局部辅助逻辑。关键符号：`FullyConnectedOperatorTester`。

### Lines 382-401 / 第 382-401 行

```cpp
0382: TEST(FULLY_CONNECTED_OP, integration_test_dynamic_per_channel) {
0383:   FullyConnectedOperatorTester()
0384:       .batchSize(4)
0385:       .inputChannels(4)
0386:       .outputChannels(4)
0387:       .iterations(3)
0388:       .per_channel(true)
0389:       .testQ8(FullyConnectedOperatorTester::Mode::Dynamic);
0390: }
0391: 
0392: TEST(FULLY_CONNECTED_OP, zero_batch_static_per_channel) {
0393:   FullyConnectedOperatorTester()
0394:       .batchSize(0)
0395:       .inputChannels(2)
0396:       .outputChannels(2)
0397:       .iterations(1)
0398:       .per_channel(true)
0399:       .testQ8(FullyConnectedOperatorTester::Mode::Static);
0400: }
0401: 
```

- **EN:** This block implements local helper logic for `fully-connected`. Key symbols: `FullyConnectedOperatorTester`.
- **CN:** 该代码块实现与 `fully-connected` 相关的局部辅助逻辑。关键符号：`FullyConnectedOperatorTester`。

### Lines 402-421 / 第 402-421 行

```cpp
0402: TEST(FULLY_CONNECTED_OP, zero_batch_runtime_per_channel) {
0403:   FullyConnectedOperatorTester()
0404:       .batchSize(0)
0405:       .inputChannels(2)
0406:       .outputChannels(2)
0407:       .iterations(1)
0408:       .per_channel(true)
0409:       .testQ8(FullyConnectedOperatorTester::Mode::Runtime);
0410: }
0411: 
0412: TEST(FULLY_CONNECTED_OP, zero_batch_dynamic_per_channel) {
0413:   FullyConnectedOperatorTester()
0414:       .batchSize(0)
0415:       .inputChannels(2)
0416:       .outputChannels(2)
0417:       .iterations(1)
0418:       .per_channel(true)
0419:       .testQ8(FullyConnectedOperatorTester::Mode::Dynamic);
0420: }
0421: 
```

- **EN:** This block implements local helper logic for `fully-connected`. Key symbols: `FullyConnectedOperatorTester`.
- **CN:** 该代码块实现与 `fully-connected` 相关的局部辅助逻辑。关键符号：`FullyConnectedOperatorTester`。

### Lines 422-441 / 第 422-441 行

```cpp
0422: TEST(FULLY_CONNECTED_OP, unit_batch_static_per_channel) {
0423:   FullyConnectedOperatorTester()
0424:       .batchSize(1)
0425:       .inputChannels(23)
0426:       .outputChannels(19)
0427:       .iterations(3)
0428:       .per_channel(true)
0429:       .testQ8(FullyConnectedOperatorTester::Mode::Static);
0430: }
0431: 
0432: TEST(FULLY_CONNECTED_OP, unit_batch_runtime_per_channel) {
0433:   FullyConnectedOperatorTester()
0434:       .batchSize(1)
0435:       .inputChannels(23)
0436:       .outputChannels(19)
0437:       .iterations(3)
0438:       .per_channel(true)
0439:       .testQ8(FullyConnectedOperatorTester::Mode::Runtime);
0440: }
0441: 
```

- **EN:** This block implements local helper logic for `fully-connected`. Key symbols: `FullyConnectedOperatorTester`.
- **CN:** 该代码块实现与 `fully-connected` 相关的局部辅助逻辑。关键符号：`FullyConnectedOperatorTester`。

### Lines 442-462 / 第 442-462 行

```cpp
0442: TEST(FULLY_CONNECTED_OP, unit_batch_dynamic_per_channel) {
0443:   FullyConnectedOperatorTester()
0444:       .batchSize(1)
0445:       .inputChannels(23)
0446:       .outputChannels(19)
0447:       .iterations(3)
0448:       .per_channel(true)
0449:       .testQ8(FullyConnectedOperatorTester::Mode::Dynamic);
0450: }
0451: 
0452: TEST(FULLY_CONNECTED_OP, unit_batch_with_qmin_static_per_channel) {
0453:   FullyConnectedOperatorTester()
0454:       .batchSize(1)
0455:       .inputChannels(23)
0456:       .outputChannels(19)
0457:       .qmin(128)
0458:       .iterations(3)
0459:       .per_channel(true)
0460:       .testQ8(FullyConnectedOperatorTester::Mode::Static);
0461: }
0462: 
```

- **EN:** This block implements local helper logic for `fully-connected`. Key symbols: `FullyConnectedOperatorTester`.
- **CN:** 该代码块实现与 `fully-connected` 相关的局部辅助逻辑。关键符号：`FullyConnectedOperatorTester`。

### Lines 463-484 / 第 463-484 行

```cpp
0463: TEST(FULLY_CONNECTED_OP, unit_batch_with_qmin_runtime_per_channel) {
0464:   FullyConnectedOperatorTester()
0465:       .batchSize(1)
0466:       .inputChannels(23)
0467:       .outputChannels(19)
0468:       .qmin(128)
0469:       .iterations(3)
0470:       .per_channel(true)
0471:       .testQ8(FullyConnectedOperatorTester::Mode::Runtime);
0472: }
0473: 
0474: TEST(FULLY_CONNECTED_OP, unit_batch_with_qmin_dynamic_per_channel) {
0475:   FullyConnectedOperatorTester()
0476:       .batchSize(1)
0477:       .inputChannels(23)
0478:       .outputChannels(19)
0479:       .qmin(128)
0480:       .iterations(3)
0481:       .per_channel(true)
0482:       .testQ8(FullyConnectedOperatorTester::Mode::Dynamic);
0483: }
0484: 
```

- **EN:** This block implements local helper logic for `fully-connected`. Key symbols: `FullyConnectedOperatorTester`.
- **CN:** 该代码块实现与 `fully-connected` 相关的局部辅助逻辑。关键符号：`FullyConnectedOperatorTester`。

### Lines 485-506 / 第 485-506 行

```cpp
0485: TEST(FULLY_CONNECTED_OP, unit_batch_with_qmax_static_per_channel) {
0486:   FullyConnectedOperatorTester()
0487:       .batchSize(1)
0488:       .inputChannels(23)
0489:       .outputChannels(19)
0490:       .qmax(128)
0491:       .iterations(3)
0492:       .per_channel(true)
0493:       .testQ8(FullyConnectedOperatorTester::Mode::Static);
0494: }
0495: 
0496: TEST(FULLY_CONNECTED_OP, unit_batch_with_qmax_runtime_per_channel) {
0497:   FullyConnectedOperatorTester()
0498:       .batchSize(1)
0499:       .inputChannels(23)
0500:       .outputChannels(19)
0501:       .qmax(128)
0502:       .iterations(3)
0503:       .per_channel(true)
0504:       .testQ8(FullyConnectedOperatorTester::Mode::Runtime);
0505: }
0506: 
```

- **EN:** This block implements local helper logic for `fully-connected`. Key symbols: `FullyConnectedOperatorTester`.
- **CN:** 该代码块实现与 `fully-connected` 相关的局部辅助逻辑。关键符号：`FullyConnectedOperatorTester`。

### Lines 507-528 / 第 507-528 行

```cpp
0507: TEST(FULLY_CONNECTED_OP, unit_batch_with_qmax_dynamic_per_channel) {
0508:   FullyConnectedOperatorTester()
0509:       .batchSize(1)
0510:       .inputChannels(23)
0511:       .outputChannels(19)
0512:       .qmax(128)
0513:       .iterations(3)
0514:       .per_channel(true)
0515:       .testQ8(FullyConnectedOperatorTester::Mode::Dynamic);
0516: }
0517: 
0518: TEST(FULLY_CONNECTED_OP, unit_batch_with_input_stride_static_per_channel) {
0519:   FullyConnectedOperatorTester()
0520:       .batchSize(1)
0521:       .inputChannels(23)
0522:       .inputStride(28)
0523:       .outputChannels(19)
0524:       .iterations(3)
0525:       .per_channel(true)
0526:       .testQ8(FullyConnectedOperatorTester::Mode::Static);
0527: }
0528: 
```

- **EN:** This block implements local helper logic for `fully-connected`. Key symbols: `FullyConnectedOperatorTester`.
- **CN:** 该代码块实现与 `fully-connected` 相关的局部辅助逻辑。关键符号：`FullyConnectedOperatorTester`。

### Lines 529-550 / 第 529-550 行

```cpp
0529: TEST(FULLY_CONNECTED_OP, unit_batch_with_input_stride_runtime_per_channel) {
0530:   FullyConnectedOperatorTester()
0531:       .batchSize(1)
0532:       .inputChannels(23)
0533:       .inputStride(28)
0534:       .outputChannels(19)
0535:       .iterations(3)
0536:       .per_channel(true)
0537:       .testQ8(FullyConnectedOperatorTester::Mode::Runtime);
0538: }
0539: 
0540: TEST(FULLY_CONNECTED_OP, unit_batch_with_input_stride_dynamic_per_channel) {
0541:   FullyConnectedOperatorTester()
0542:       .batchSize(1)
0543:       .inputChannels(23)
0544:       .inputStride(28)
0545:       .outputChannels(19)
0546:       .iterations(3)
0547:       .per_channel(true)
0548:       .testQ8(FullyConnectedOperatorTester::Mode::Dynamic);
0549: }
0550: 
```

- **EN:** This block implements local helper logic for `fully-connected`. Key symbols: `FullyConnectedOperatorTester`.
- **CN:** 该代码块实现与 `fully-connected` 相关的局部辅助逻辑。关键符号：`FullyConnectedOperatorTester`。

### Lines 551-572 / 第 551-572 行

```cpp
0551: TEST(FULLY_CONNECTED_OP, unit_batch_with_output_stride_static_per_channel) {
0552:   FullyConnectedOperatorTester()
0553:       .batchSize(1)
0554:       .inputChannels(23)
0555:       .outputChannels(19)
0556:       .outputStride(29)
0557:       .iterations(3)
0558:       .per_channel(true)
0559:       .testQ8(FullyConnectedOperatorTester::Mode::Static);
0560: }
0561: 
0562: TEST(FULLY_CONNECTED_OP, unit_batch_with_output_stride_runtime_per_channel) {
0563:   FullyConnectedOperatorTester()
0564:       .batchSize(1)
0565:       .inputChannels(23)
0566:       .outputChannels(19)
0567:       .outputStride(29)
0568:       .iterations(3)
0569:       .per_channel(true)
0570:       .testQ8(FullyConnectedOperatorTester::Mode::Runtime);
0571: }
0572: 
```

- **EN:** This block implements local helper logic for `fully-connected`. Key symbols: `FullyConnectedOperatorTester`.
- **CN:** 该代码块实现与 `fully-connected` 相关的局部辅助逻辑。关键符号：`FullyConnectedOperatorTester`。

### Lines 573-593 / 第 573-593 行

```cpp
0573: TEST(FULLY_CONNECTED_OP, unit_batch_with_output_stride_dynamic_per_channel) {
0574:   FullyConnectedOperatorTester()
0575:       .batchSize(1)
0576:       .inputChannels(23)
0577:       .outputChannels(19)
0578:       .outputStride(29)
0579:       .iterations(3)
0580:       .per_channel(true)
0581:       .testQ8(FullyConnectedOperatorTester::Mode::Dynamic);
0582: }
0583: 
0584: TEST(FULLY_CONNECTED_OP, small_batch_static_per_channel) {
0585:   FullyConnectedOperatorTester()
0586:       .batchSize(12)
0587:       .inputChannels(23)
0588:       .outputChannels(19)
0589:       .iterations(3)
0590:       .per_channel(true)
0591:       .testQ8(FullyConnectedOperatorTester::Mode::Static);
0592: }
0593: 
```

- **EN:** This block implements local helper logic for `fully-connected`. Key symbols: `FullyConnectedOperatorTester`.
- **CN:** 该代码块实现与 `fully-connected` 相关的局部辅助逻辑。关键符号：`FullyConnectedOperatorTester`。

### Lines 594-613 / 第 594-613 行

```cpp
0594: TEST(FULLY_CONNECTED_OP, small_batch_runtime_per_channel_per_channel) {
0595:   FullyConnectedOperatorTester()
0596:       .batchSize(12)
0597:       .inputChannels(23)
0598:       .outputChannels(19)
0599:       .iterations(3)
0600:       .per_channel(true)
0601:       .testQ8(FullyConnectedOperatorTester::Mode::Runtime);
0602: }
0603: 
0604: TEST(FULLY_CONNECTED_OP, small_batch_dynamic_per_channel) {
0605:   FullyConnectedOperatorTester()
0606:       .batchSize(12)
0607:       .inputChannels(23)
0608:       .outputChannels(19)
0609:       .iterations(3)
0610:       .per_channel(true)
0611:       .testQ8(FullyConnectedOperatorTester::Mode::Dynamic);
0612: }
0613: 
```

- **EN:** This block implements local helper logic for `fully-connected`. Key symbols: `FullyConnectedOperatorTester`.
- **CN:** 该代码块实现与 `fully-connected` 相关的局部辅助逻辑。关键符号：`FullyConnectedOperatorTester`。

### Lines 614-635 / 第 614-635 行

```cpp
0614: TEST(FULLY_CONNECTED_OP, small_batch_with_qmin_static_per_channel) {
0615:   FullyConnectedOperatorTester()
0616:       .batchSize(12)
0617:       .inputChannels(23)
0618:       .outputChannels(19)
0619:       .qmin(128)
0620:       .iterations(3)
0621:       .per_channel(true)
0622:       .testQ8(FullyConnectedOperatorTester::Mode::Static);
0623: }
0624: 
0625: TEST(FULLY_CONNECTED_OP, small_batch_with_qmin_runtime_per_channel) {
0626:   FullyConnectedOperatorTester()
0627:       .batchSize(12)
0628:       .inputChannels(23)
0629:       .outputChannels(19)
0630:       .qmin(128)
0631:       .iterations(3)
0632:       .per_channel(true)
0633:       .testQ8(FullyConnectedOperatorTester::Mode::Runtime);
0634: }
0635: 
```

- **EN:** This block implements local helper logic for `fully-connected`. Key symbols: `FullyConnectedOperatorTester`.
- **CN:** 该代码块实现与 `fully-connected` 相关的局部辅助逻辑。关键符号：`FullyConnectedOperatorTester`。

### Lines 636-657 / 第 636-657 行

```cpp
0636: TEST(FULLY_CONNECTED_OP, small_batch_with_qmin_dynamic_per_channel) {
0637:   FullyConnectedOperatorTester()
0638:       .batchSize(12)
0639:       .inputChannels(23)
0640:       .outputChannels(19)
0641:       .qmin(128)
0642:       .iterations(3)
0643:       .per_channel(true)
0644:       .testQ8(FullyConnectedOperatorTester::Mode::Dynamic);
0645: }
0646: 
0647: TEST(FULLY_CONNECTED_OP, small_batch_with_qmax_per_channel) {
0648:   FullyConnectedOperatorTester()
0649:       .batchSize(12)
0650:       .inputChannels(23)
0651:       .outputChannels(19)
0652:       .qmax(128)
0653:       .iterations(3)
0654:       .per_channel(true)
0655:       .testQ8(FullyConnectedOperatorTester::Mode::Static);
0656: }
0657: 
```

- **EN:** This block implements local helper logic for `fully-connected`. Key symbols: `FullyConnectedOperatorTester`.
- **CN:** 该代码块实现与 `fully-connected` 相关的局部辅助逻辑。关键符号：`FullyConnectedOperatorTester`。

### Lines 658-679 / 第 658-679 行

```cpp
0658: TEST(FULLY_CONNECTED_OP, small_batch_with_qmax_runtime_per_channel) {
0659:   FullyConnectedOperatorTester()
0660:       .batchSize(12)
0661:       .inputChannels(23)
0662:       .outputChannels(19)
0663:       .qmax(128)
0664:       .iterations(3)
0665:       .per_channel(true)
0666:       .testQ8(FullyConnectedOperatorTester::Mode::Runtime);
0667: }
0668: 
0669: TEST(FULLY_CONNECTED_OP, small_batch_with_qmax_dynamic_per_channel) {
0670:   FullyConnectedOperatorTester()
0671:       .batchSize(12)
0672:       .inputChannels(23)
0673:       .outputChannels(19)
0674:       .qmax(128)
0675:       .iterations(3)
0676:       .per_channel(true)
0677:       .testQ8(FullyConnectedOperatorTester::Mode::Dynamic);
0678: }
0679: 
```

- **EN:** This block implements local helper logic for `fully-connected`. Key symbols: `FullyConnectedOperatorTester`.
- **CN:** 该代码块实现与 `fully-connected` 相关的局部辅助逻辑。关键符号：`FullyConnectedOperatorTester`。

### Lines 680-700 / 第 680-700 行

```cpp
0680: TEST(FULLY_CONNECTED_OP, small_batch_with_input_stride_static_per_channel) {
0681:   FullyConnectedOperatorTester()
0682:       .batchSize(12)
0683:       .inputChannels(23)
0684:       .inputStride(28)
0685:       .outputChannels(19)
0686:       .iterations(3)
0687:       .per_channel(true)
0688:       .testQ8(FullyConnectedOperatorTester::Mode::Static);
0689: }
0690: 
0691: TEST(FULLY_CONNECTED_OP, small_batch_with_output_stride_static_per_channel) {
0692:   FullyConnectedOperatorTester()
0693:       .batchSize(12)
0694:       .inputChannels(23)
0695:       .outputChannels(19)
0696:       .outputStride(29)
0697:       .iterations(3)
0698:       .per_channel(true)
0699:       .testQ8(FullyConnectedOperatorTester::Mode::Static);
0700: }
```

- **EN:** This block implements local helper logic for `fully-connected`. Key symbols: `FullyConnectedOperatorTester`.
- **CN:** 该代码块实现与 `fully-connected` 相关的局部辅助逻辑。关键符号：`FullyConnectedOperatorTester`。


## Key Concepts / 关键概念
- **ATen native operator logic** — ATen 原生算子逻辑
- **Core symbols: FullyConnectedOperatorTester** — 核心符号：FullyConnectedOperatorTester

## Dependencies / 依赖关系
- **Internal includes / 内部头文件**: 无
- **External includes / 外部头文件**: `gtest/gtest.h`, `fully-connected-operator-tester.h`
- **Namespaces / 命名空间**: 无
- **Representative symbols / 代表性符号**: `FullyConnectedOperatorTester`
