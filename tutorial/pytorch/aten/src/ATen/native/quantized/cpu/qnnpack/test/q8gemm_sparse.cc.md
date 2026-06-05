# q8gemm_sparse.cc — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `aten/src/ATen/native/quantized/cpu/qnnpack/test/q8gemm_sparse.cc`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements ATen native operators, checks, helper kernels, and backend dispatch behavior. This specific file centers on `q8gemm_sparse.cc`. The file header highlights: "All rights reserved. This source code is licensed under the BSD-style license found in the LICENSE file in the root directory of this source tree.." Matrix multiplication and GEMM-style kernels are a central concern.
- **Purpose (CN)**: 实现 ATen 原生算子、运行时检查、辅助内核以及后端分发行为。 该文件具体围绕 `q8gemm_sparse.cc` 展开。 文件头部注释也概括了其核心职责。 矩阵乘法与 GEMM 风格内核是该文件的核心关注点。

## Line-by-Line Analysis / 逐行分析
### Lines 1-80 / 第 1-80 行

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
0013: #include <qnnpack/q8gemm_sparse.h>
0014: 
0015: #include "gemm-block-sparse-microkernel-tester.h"
0016: 
0017: #define TEST_PACKED_ROW_BLOCK_SIZEXCOL_BLOCK_SIZE_SPARSE_OP(MR, \
0018:     NR, row_block_size, col_block_size, \
0019:     prepacking_kernel, compute_kernel_w32, compute_kernel_w16, compute_kernel_w8) \
0020: TEST(Q8GEMM__##MR ## x ##NR ## c##row_block_size ## x ##col_block_size ## __AARCH32_NEON, packedA_k_lt_4) { \
0021:   TEST_REQUIRES_ARM_NEON; \
0022:   GemmBlockSparseMicrokernelTester tester = GemmBlockSparseMicrokernelTester() \
0023:       .mr(MR) \
0024:       .nr(NR) \
0025:       .m(MR) \
0026:       .n(NR) \
0027:       .k(3) \
0028:       .rowBlockSize(row_block_size) \
0029:       .colBlockSize(col_block_size); \
0030:   tester.test_packed<uint32_t>( \
0031:       prepacking_kernel, \
0032:       compute_kernel_w32); \
0033:   tester.test_packed<uint16_t>( \
0034:       prepacking_kernel, \
0035:       compute_kernel_w16); \
0036:   tester.test_packed<uint8_t>( \
0037:       prepacking_kernel, \
0038:       compute_kernel_w8); \
0039: } \
0040: \
0041: TEST(Q8GEMM__##MR ## x ##NR ## c##row_block_size ## x ##col_block_size ## __AARCH32_NEON, packedA_k_lt_4_strided_a) { \
0042:   TEST_REQUIRES_ARM_NEON; \
0043:   GemmBlockSparseMicrokernelTester tester = GemmBlockSparseMicrokernelTester() \
0044:       .mr(MR) \
0045:       .nr(NR) \
0046:       .m(MR) \
0047:       .n(NR) \
0048:       .k(3) \
0049:       .rowBlockSize(row_block_size) \
0050:       .colBlockSize(col_block_size) \
0051:       .aStride(37); \
0052:   tester.test_packed<uint32_t>( \
0053:       prepacking_kernel, \
0054:       compute_kernel_w32); \
0055:   tester.test_packed<uint16_t>( \
0056:       prepacking_kernel, \
0057:       compute_kernel_w16); \
0058:   tester.test_packed<uint8_t>( \
0059:       prepacking_kernel, \
0060:       compute_kernel_w8); \
0061: } \
0062:  \
0063: TEST(Q8GEMM__##MR ## x ##NR ## c##row_block_size ## x ##col_block_size ## __AARCH32_NEON, packedA_k_lt_4_strided_c) { \
0064:   TEST_REQUIRES_ARM_NEON; \
0065:   GemmBlockSparseMicrokernelTester tester = GemmBlockSparseMicrokernelTester() \
0066:       .mr(MR) \
0067:       .nr(NR) \
0068:       .m(MR) \
0069:       .n(NR) \
0070:       .k(3) \
0071:       .rowBlockSize(row_block_size) \
0072:       .colBlockSize(col_block_size) \
0073:       .cStride(17); \
0074:   tester.test_packed<uint32_t>( \
0075:       prepacking_kernel, \
0076:       compute_kernel_w32); \
0077:   tester.test_packed<uint16_t>( \
0078:       prepacking_kernel, \
0079:       compute_kernel_w16); \
0080:   tester.test_packed<uint8_t>( \
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends. Key symbols: no prominent local symbols.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端。关键符号：无明显局部符号。

### Lines 81-160 / 第 81-160 行

```cpp
0081:       prepacking_kernel, \
0082:       compute_kernel_w8); \
0083: } \
0084:  \
0085: TEST(Q8GEMM__##MR ## x ##NR ## c##row_block_size ## x ##col_block_size ## __AARCH32_NEON, packedA_k_lt_4_qmin128) { \
0086:   TEST_REQUIRES_ARM_NEON; \
0087:   GemmBlockSparseMicrokernelTester tester = GemmBlockSparseMicrokernelTester() \
0088:       .mr(MR) \
0089:       .nr(NR) \
0090:       .m(MR) \
0091:       .n(NR) \
0092:       .k(3) \
0093:       .rowBlockSize(row_block_size) \
0094:       .colBlockSize(col_block_size) \
0095:       .qmin(128); \
0096:   tester.test_packed<uint32_t>( \
0097:       prepacking_kernel, \
0098:       compute_kernel_w32); \
0099:   tester.test_packed<uint16_t>( \
0100:       prepacking_kernel, \
0101:       compute_kernel_w16); \
0102:   tester.test_packed<uint8_t>( \
0103:       prepacking_kernel, \
0104:       compute_kernel_w8); \
0105: } \
0106:  \
0107: TEST(Q8GEMM__##MR ## x ##NR ## c##row_block_size ## x ##col_block_size ## __AARCH32_NEON, packedA_k_lt_4_qmax128) { \
0108:   TEST_REQUIRES_ARM_NEON; \
0109:   GemmBlockSparseMicrokernelTester tester = GemmBlockSparseMicrokernelTester() \
0110:       .mr(MR) \
0111:       .nr(NR) \
0112:       .m(MR) \
0113:       .n(NR) \
0114:       .k(3) \
0115:       .rowBlockSize(row_block_size) \
0116:       .colBlockSize(col_block_size) \
0117:       .qmax(128); \
0118:   tester.test_packed<uint32_t>( \
0119:       prepacking_kernel, \
0120:       compute_kernel_w32); \
0121:   tester.test_packed<uint16_t>( \
0122:       prepacking_kernel, \
0123:       compute_kernel_w16); \
0124:   tester.test_packed<uint8_t>( \
0125:       prepacking_kernel, \
0126:       compute_kernel_w8); \
0127: } \
0128:  \
0129: TEST(Q8GEMM__##MR ## x ##NR ## c##row_block_size ## x ##col_block_size ## __AARCH32_NEON, packedA_k_lt_4_azp0) { \
0130:   TEST_REQUIRES_ARM_NEON; \
0131:   GemmBlockSparseMicrokernelTester tester = GemmBlockSparseMicrokernelTester() \
0132:       .mr(MR) \
0133:       .nr(NR) \
0134:       .m(MR) \
0135:       .n(NR) \
0136:       .k(3) \
0137:       .rowBlockSize(row_block_size) \
0138:       .colBlockSize(col_block_size) \
0139:       .aZeroPoint(0); \
0140:   tester.test_packed<uint32_t>( \
0141:       prepacking_kernel, \
0142:       compute_kernel_w32); \
0143:   tester.test_packed<uint16_t>( \
0144:       prepacking_kernel, \
0145:       compute_kernel_w16); \
0146:   tester.test_packed<uint8_t>( \
0147:       prepacking_kernel, \
0148:       compute_kernel_w8); \
0149: } \
0150:  \
0151: TEST(Q8GEMM__##MR ## x ##NR ## c##row_block_size ## x ##col_block_size ## __AARCH32_NEON, packedA_k_lt_4_bzp0) { \
0152:   TEST_REQUIRES_ARM_NEON; \
0153:   GemmBlockSparseMicrokernelTester tester = GemmBlockSparseMicrokernelTester() \
0154:       .mr(MR) \
0155:       .nr(NR) \
0156:       .m(MR) \
0157:       .n(NR) \
0158:       .k(3) \
0159:       .rowBlockSize(row_block_size) \
0160:       .colBlockSize(col_block_size) \
```

- **EN:** This block implements local helper logic for `q8gemm sparse`. Key symbols: no prominent local symbols.
- **CN:** 该代码块实现与 `q8gemm sparse` 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 161-240 / 第 161-240 行

```cpp
0161:       .bZeroPoint(0); \
0162:   tester.test_packed<uint32_t>( \
0163:       prepacking_kernel, \
0164:       compute_kernel_w32); \
0165:   tester.test_packed<uint16_t>( \
0166:       prepacking_kernel, \
0167:       compute_kernel_w16); \
0168:   tester.test_packed<uint8_t>( \
0169:       prepacking_kernel, \
0170:       compute_kernel_w8); \
0171: } \
0172:  \
0173: TEST(Q8GEMM__##MR ## x ##NR ## c##row_block_size ## x ##col_block_size ## __AARCH32_NEON, packedA_k_lt_4_nozp) { \
0174:   TEST_REQUIRES_ARM_NEON; \
0175:   GemmBlockSparseMicrokernelTester tester = GemmBlockSparseMicrokernelTester() \
0176:       .mr(MR) \
0177:       .nr(NR) \
0178:       .m(MR) \
0179:       .n(NR) \
0180:       .k(3) \
0181:       .rowBlockSize(row_block_size) \
0182:       .colBlockSize(col_block_size) \
0183:       .aZeroPoint(0) \
0184:       .bZeroPoint(0); \
0185:   tester.test_packed<uint32_t>( \
0186:       prepacking_kernel, \
0187:       compute_kernel_w32); \
0188:   tester.test_packed<uint16_t>( \
0189:       prepacking_kernel, \
0190:       compute_kernel_w16); \
0191:   tester.test_packed<uint8_t>( \
0192:       prepacking_kernel, \
0193:       compute_kernel_w8); \
0194: } \
0195:  \
0196: TEST(Q8GEMM__##MR ## x ##NR ## c##row_block_size ## x ##col_block_size ## __AARCH32_NEON, packedA_k_lt_8) { \
0197:   TEST_REQUIRES_ARM_NEON; \
0198:   GemmBlockSparseMicrokernelTester tester = GemmBlockSparseMicrokernelTester() \
0199:       .mr(MR) \
0200:       .nr(NR) \
0201:       .m(MR) \
0202:       .n(NR) \
0203:       .k(5) \
0204:       .rowBlockSize(row_block_size) \
0205:       .colBlockSize(col_block_size); \
0206:   tester.test_packed<uint32_t>( \
0207:       prepacking_kernel, \
0208:       compute_kernel_w32); \
0209:   tester.test_packed<uint16_t>( \
0210:       prepacking_kernel, \
0211:       compute_kernel_w16); \
0212:   tester.test_packed<uint8_t>( \
0213:       prepacking_kernel, \
0214:       compute_kernel_w8); \
0215: } \
0216:  \
0217: TEST(Q8GEMM__##MR ## x ##NR ## c##row_block_size ## x ##col_block_size ## __AARCH32_NEON, packedA_k_lt_8_strided_a) { \
0218:   TEST_REQUIRES_ARM_NEON; \
0219:   GemmBlockSparseMicrokernelTester tester = GemmBlockSparseMicrokernelTester() \
0220:       .mr(MR) \
0221:       .nr(NR) \
0222:       .m(MR) \
0223:       .n(NR) \
0224:       .k(5) \
0225:       .rowBlockSize(row_block_size) \
0226:       .colBlockSize(col_block_size) \
0227:       .aStride(37); \
0228:   tester.test_packed<uint32_t>( \
0229:       prepacking_kernel, \
0230:       compute_kernel_w32); \
0231:   tester.test_packed<uint16_t>( \
0232:       prepacking_kernel, \
0233:       compute_kernel_w16); \
0234:   tester.test_packed<uint8_t>( \
0235:       prepacking_kernel, \
0236:       compute_kernel_w8); \
0237: } \
0238:  \
0239: TEST(Q8GEMM__##MR ## x ##NR ## c##row_block_size ## x ##col_block_size ## __AARCH32_NEON, packedA_k_lt_8_strided_c) { \
0240:   TEST_REQUIRES_ARM_NEON; \
```

- **EN:** This block implements local helper logic for `q8gemm sparse`. Key symbols: no prominent local symbols.
- **CN:** 该代码块实现与 `q8gemm sparse` 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 241-320 / 第 241-320 行

```cpp
0241:   GemmBlockSparseMicrokernelTester tester = GemmBlockSparseMicrokernelTester() \
0242:       .mr(MR) \
0243:       .nr(NR) \
0244:       .m(MR) \
0245:       .n(NR) \
0246:       .k(5) \
0247:       .rowBlockSize(row_block_size) \
0248:       .colBlockSize(col_block_size) \
0249:       .cStride(17); \
0250:   tester.test_packed<uint32_t>( \
0251:       prepacking_kernel, \
0252:       compute_kernel_w32); \
0253:   tester.test_packed<uint16_t>( \
0254:       prepacking_kernel, \
0255:       compute_kernel_w16); \
0256:   tester.test_packed<uint8_t>( \
0257:       prepacking_kernel, \
0258:       compute_kernel_w8); \
0259: } \
0260:  \
0261: TEST(Q8GEMM__##MR ## x ##NR ## c##row_block_size ## x ##col_block_size ## __AARCH32_NEON, packedA_k_lt_8_qmin128) { \
0262:   TEST_REQUIRES_ARM_NEON; \
0263:   GemmBlockSparseMicrokernelTester tester = GemmBlockSparseMicrokernelTester() \
0264:       .mr(MR) \
0265:       .nr(NR) \
0266:       .m(MR) \
0267:       .n(NR) \
0268:       .k(5) \
0269:       .rowBlockSize(row_block_size) \
0270:       .colBlockSize(col_block_size) \
0271:       .qmin(128); \
0272:   tester.test_packed<uint32_t>( \
0273:       prepacking_kernel, \
0274:       compute_kernel_w32); \
0275:   tester.test_packed<uint16_t>( \
0276:       prepacking_kernel, \
0277:       compute_kernel_w16); \
0278:   tester.test_packed<uint8_t>( \
0279:       prepacking_kernel, \
0280:       compute_kernel_w8); \
0281: } \
0282:  \
0283: TEST(Q8GEMM__##MR ## x ##NR ## c##row_block_size ## x ##col_block_size ## __AARCH32_NEON, packedA_k_lt_8_qmax128) { \
0284:   TEST_REQUIRES_ARM_NEON; \
0285:   GemmBlockSparseMicrokernelTester tester = GemmBlockSparseMicrokernelTester() \
0286:       .mr(MR) \
0287:       .nr(NR) \
0288:       .m(MR) \
0289:       .n(NR) \
0290:       .k(5) \
0291:       .rowBlockSize(row_block_size) \
0292:       .colBlockSize(col_block_size) \
0293:       .qmax(128); \
0294:   tester.test_packed<uint32_t>( \
0295:       prepacking_kernel, \
0296:       compute_kernel_w32); \
0297:   tester.test_packed<uint16_t>( \
0298:       prepacking_kernel, \
0299:       compute_kernel_w16); \
0300:   tester.test_packed<uint8_t>( \
0301:       prepacking_kernel, \
0302:       compute_kernel_w8); \
0303: } \
0304:  \
0305: TEST(Q8GEMM__##MR ## x ##NR ## c##row_block_size ## x ##col_block_size ## __AARCH32_NEON, packedA_k_lt_8_azp0) { \
0306:   TEST_REQUIRES_ARM_NEON; \
0307:   GemmBlockSparseMicrokernelTester tester = GemmBlockSparseMicrokernelTester() \
0308:       .mr(MR) \
0309:       .nr(NR) \
0310:       .m(MR) \
0311:       .n(NR) \
0312:       .k(5) \
0313:       .rowBlockSize(row_block_size) \
0314:       .colBlockSize(col_block_size) \
0315:       .aZeroPoint(0); \
0316:   tester.test_packed<uint32_t>( \
0317:       prepacking_kernel, \
0318:       compute_kernel_w32); \
0319:   tester.test_packed<uint16_t>( \
0320:       prepacking_kernel, \
```

- **EN:** This block implements local helper logic for `q8gemm sparse`. Key symbols: no prominent local symbols.
- **CN:** 该代码块实现与 `q8gemm sparse` 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 321-400 / 第 321-400 行

```cpp
0321:       compute_kernel_w16); \
0322:   tester.test_packed<uint8_t>( \
0323:       prepacking_kernel, \
0324:       compute_kernel_w8); \
0325: } \
0326:  \
0327: TEST(Q8GEMM__##MR ## x ##NR ## c##row_block_size ## x ##col_block_size ## __AARCH32_NEON, packedA_k_lt_8_bzp0) { \
0328:   TEST_REQUIRES_ARM_NEON; \
0329:   GemmBlockSparseMicrokernelTester tester = GemmBlockSparseMicrokernelTester() \
0330:       .mr(MR) \
0331:       .nr(NR) \
0332:       .m(MR) \
0333:       .n(NR) \
0334:       .k(5) \
0335:       .rowBlockSize(row_block_size) \
0336:       .colBlockSize(col_block_size) \
0337:       .bZeroPoint(0); \
0338:   tester.test_packed<uint32_t>( \
0339:       prepacking_kernel, \
0340:       compute_kernel_w32); \
0341:   tester.test_packed<uint16_t>( \
0342:       prepacking_kernel, \
0343:       compute_kernel_w16); \
0344:   tester.test_packed<uint8_t>( \
0345:       prepacking_kernel, \
0346:       compute_kernel_w8); \
0347: } \
0348:  \
0349: TEST(Q8GEMM__##MR ## x ##NR ## c##row_block_size ## x ##col_block_size ## __AARCH32_NEON, packedA_k_lt_8_nozp) { \
0350:   TEST_REQUIRES_ARM_NEON; \
0351:   GemmBlockSparseMicrokernelTester tester = GemmBlockSparseMicrokernelTester() \
0352:       .mr(MR) \
0353:       .nr(NR) \
0354:       .m(MR) \
0355:       .n(NR) \
0356:       .k(5) \
0357:       .rowBlockSize(row_block_size) \
0358:       .colBlockSize(col_block_size) \
0359:       .aZeroPoint(0) \
0360:       .bZeroPoint(0); \
0361:   tester.test_packed<uint32_t>( \
0362:       prepacking_kernel, \
0363:       compute_kernel_w32); \
0364:   tester.test_packed<uint16_t>( \
0365:       prepacking_kernel, \
0366:       compute_kernel_w16); \
0367:   tester.test_packed<uint8_t>( \
0368:       prepacking_kernel, \
0369:       compute_kernel_w8); \
0370: } \
0371:  \
0372: TEST(Q8GEMM__##MR ## x ##NR ## c##row_block_size ## x ##col_block_size ## __AARCH32_NEON, packedA_k_eq_8) { \
0373:   TEST_REQUIRES_ARM_NEON; \
0374:   GemmBlockSparseMicrokernelTester tester = GemmBlockSparseMicrokernelTester() \
0375:       .mr(MR) \
0376:       .nr(NR) \
0377:       .m(MR) \
0378:       .n(NR) \
0379:       .k(8) \
0380:       .rowBlockSize(row_block_size) \
0381:       .colBlockSize(col_block_size); \
0382:   tester.test_packed<uint32_t>( \
0383:       prepacking_kernel, \
0384:       compute_kernel_w32); \
0385:   tester.test_packed<uint16_t>( \
0386:       prepacking_kernel, \
0387:       compute_kernel_w16); \
0388:   tester.test_packed<uint8_t>( \
0389:       prepacking_kernel, \
0390:       compute_kernel_w8); \
0391: } \
0392:  \
0393: TEST(Q8GEMM__##MR ## x ##NR ## c##row_block_size ## x ##col_block_size ## __AARCH32_NEON, packedA_k_eq_8_strided_a) { \
0394:   TEST_REQUIRES_ARM_NEON; \
0395:   GemmBlockSparseMicrokernelTester tester = GemmBlockSparseMicrokernelTester() \
0396:       .mr(MR) \
0397:       .nr(NR) \
0398:       .m(MR) \
0399:       .n(NR) \
0400:       .k(8) \
```

- **EN:** This block implements local helper logic for `q8gemm sparse`. Key symbols: no prominent local symbols.
- **CN:** 该代码块实现与 `q8gemm sparse` 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 401-480 / 第 401-480 行

```cpp
0401:       .rowBlockSize(row_block_size) \
0402:       .colBlockSize(col_block_size) \
0403:       .aStride(37); \
0404:   tester.test_packed<uint32_t>( \
0405:       prepacking_kernel, \
0406:       compute_kernel_w32); \
0407:   tester.test_packed<uint16_t>( \
0408:       prepacking_kernel, \
0409:       compute_kernel_w16); \
0410:   tester.test_packed<uint8_t>( \
0411:       prepacking_kernel, \
0412:       compute_kernel_w8); \
0413: } \
0414:  \
0415: TEST(Q8GEMM__##MR ## x ##NR ## c##row_block_size ## x ##col_block_size ## __AARCH32_NEON, packedA_k_eq_8_strided_c) { \
0416:   TEST_REQUIRES_ARM_NEON; \
0417:   GemmBlockSparseMicrokernelTester tester = GemmBlockSparseMicrokernelTester() \
0418:       .mr(MR) \
0419:       .nr(NR) \
0420:       .m(MR) \
0421:       .n(NR) \
0422:       .k(8) \
0423:       .rowBlockSize(row_block_size) \
0424:       .colBlockSize(col_block_size) \
0425:       .cStride(17); \
0426:   tester.test_packed<uint32_t>( \
0427:       prepacking_kernel, \
0428:       compute_kernel_w32); \
0429:   tester.test_packed<uint16_t>( \
0430:       prepacking_kernel, \
0431:       compute_kernel_w16); \
0432:   tester.test_packed<uint8_t>( \
0433:       prepacking_kernel, \
0434:       compute_kernel_w8); \
0435: } \
0436:  \
0437: TEST(Q8GEMM__##MR ## x ##NR ## c##row_block_size ## x ##col_block_size ## __AARCH32_NEON, packedA_k_eq_8_qmin128) { \
0438:   TEST_REQUIRES_ARM_NEON; \
0439:   GemmBlockSparseMicrokernelTester tester = GemmBlockSparseMicrokernelTester() \
0440:       .mr(MR) \
0441:       .nr(NR) \
0442:       .m(MR) \
0443:       .n(NR) \
0444:       .k(8) \
0445:       .rowBlockSize(row_block_size) \
0446:       .colBlockSize(col_block_size) \
0447:       .qmin(128); \
0448:   tester.test_packed<uint32_t>( \
0449:       prepacking_kernel, \
0450:       compute_kernel_w32); \
0451:   tester.test_packed<uint16_t>( \
0452:       prepacking_kernel, \
0453:       compute_kernel_w16); \
0454:   tester.test_packed<uint8_t>( \
0455:       prepacking_kernel, \
0456:       compute_kernel_w8); \
0457: } \
0458:  \
0459: TEST(Q8GEMM__##MR ## x ##NR ## c##row_block_size ## x ##col_block_size ## __AARCH32_NEON, packedA_k_eq_8_qmax128) { \
0460:   TEST_REQUIRES_ARM_NEON; \
0461:   GemmBlockSparseMicrokernelTester tester = GemmBlockSparseMicrokernelTester() \
0462:       .mr(MR) \
0463:       .nr(NR) \
0464:       .m(MR) \
0465:       .n(NR) \
0466:       .k(8) \
0467:       .rowBlockSize(row_block_size) \
0468:       .colBlockSize(col_block_size) \
0469:       .qmax(128); \
0470:   tester.test_packed<uint32_t>( \
0471:       prepacking_kernel, \
0472:       compute_kernel_w32); \
0473:   tester.test_packed<uint16_t>( \
0474:       prepacking_kernel, \
0475:       compute_kernel_w16); \
0476:   tester.test_packed<uint8_t>( \
0477:       prepacking_kernel, \
0478:       compute_kernel_w8); \
0479: } \
0480:  \
```

- **EN:** This block implements local helper logic for `q8gemm sparse`. Key symbols: no prominent local symbols.
- **CN:** 该代码块实现与 `q8gemm sparse` 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 481-550 / 第 481-550 行

```cpp
0481: TEST(Q8GEMM__##MR ## x ##NR ## c##row_block_size ## x ##col_block_size ## __AARCH32_NEON, packedA_k_eq_8_azp0) { \
0482:   TEST_REQUIRES_ARM_NEON; \
0483:   GemmBlockSparseMicrokernelTester tester = GemmBlockSparseMicrokernelTester() \
0484:       .mr(MR) \
0485:       .nr(NR) \
0486:       .m(MR) \
0487:       .n(NR) \
0488:       .k(8) \
0489:       .rowBlockSize(row_block_size) \
0490:       .colBlockSize(col_block_size) \
0491:       .aZeroPoint(0); \
0492:   tester.test_packed<uint32_t>( \
0493:       prepacking_kernel, \
0494:       compute_kernel_w32); \
0495:   tester.test_packed<uint16_t>( \
0496:       prepacking_kernel, \
0497:       compute_kernel_w16); \
0498:   tester.test_packed<uint8_t>( \
0499:       prepacking_kernel, \
0500:       compute_kernel_w8); \
0501: } \
0502:  \
0503: TEST(Q8GEMM__##MR ## x ##NR ## c##row_block_size ## x ##col_block_size ## __AARCH32_NEON, packedA_k_eq_8_bzp0) { \
0504:   TEST_REQUIRES_ARM_NEON; \
0505:   GemmBlockSparseMicrokernelTester tester = GemmBlockSparseMicrokernelTester() \
0506:       .mr(MR) \
0507:       .nr(NR) \
0508:       .m(MR) \
0509:       .n(NR) \
0510:       .k(8) \
0511:       .rowBlockSize(row_block_size) \
0512:       .colBlockSize(col_block_size) \
0513:       .bZeroPoint(0); \
0514:   tester.test_packed<uint32_t>( \
0515:       prepacking_kernel, \
0516:       compute_kernel_w32); \
0517:   tester.test_packed<uint16_t>( \
0518:       prepacking_kernel, \
0519:       compute_kernel_w16); \
0520:   tester.test_packed<uint8_t>( \
0521:       prepacking_kernel, \
0522:       compute_kernel_w8); \
0523: } \
0524:  \
0525: TEST(Q8GEMM__##MR ## x ##NR ## c##row_block_size ## x ##col_block_size ## __AARCH32_NEON, packedA_k_eq_8_nozp) { \
0526:   TEST_REQUIRES_ARM_NEON; \
0527:   GemmBlockSparseMicrokernelTester tester = GemmBlockSparseMicrokernelTester() \
0528:       .mr(MR) \
0529:       .nr(NR) \
0530:       .m(MR) \
0531:       .n(NR) \
0532:       .k(8) \
0533:       .rowBlockSize(row_block_size) \
0534:       .colBlockSize(col_block_size) \
0535:       .aZeroPoint(0) \
0536:       .bZeroPoint(0); \
0537:   tester.test_packed<uint32_t>( \
0538:       prepacking_kernel, \
0539:       compute_kernel_w32); \
0540:   tester.test_packed<uint16_t>( \
0541:       prepacking_kernel, \
0542:       compute_kernel_w16); \
0543:   tester.test_packed<uint8_t>( \
0544:       prepacking_kernel, \
0545:       compute_kernel_w8); \
0546: } \
0547:  \
0548: TEST(Q8GEMM__##MR ## x ##NR ## c##row_block_size ## x ##col_block_size ## __AARCH32_NEON, packedA_k_gt_8) { \
0549:   TEST_REQUIRES_ARM_NEON; \
0550:   for (size_t k = 9; k < 16; k++) { \
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: no prominent local symbols.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：无明显局部符号。

### Lines 551-597 / 第 551-597 行

```cpp
0551:     GemmBlockSparseMicrokernelTester tester = GemmBlockSparseMicrokernelTester() \
0552:         .mr(MR) \
0553:         .nr(NR) \
0554:         .m(MR) \
0555:         .n(NR) \
0556:         .k(k) \
0557:         .rowBlockSize(row_block_size) \
0558:         .colBlockSize(col_block_size); \
0559:     tester.test_packed<uint32_t>( \
0560:         prepacking_kernel, \
0561:         compute_kernel_w32); \
0562:     tester.test_packed<uint16_t>( \
0563:         prepacking_kernel, \
0564:         compute_kernel_w16); \
0565:     tester.test_packed<uint8_t>( \
0566:         prepacking_kernel, \
0567:         compute_kernel_w8); \
0568:   } \
0569: } \
0570:  \
0571: TEST(Q8GEMM__##MR ## x ##NR ## c##row_block_size ## x ##col_block_size ## __AARCH32_NEON, packedA_k_gt_8_strided_a) { \
0572:   TEST_REQUIRES_ARM_NEON; \
0573:   for (size_t k = 9; k < 16; k++) { \
0574:     GemmBlockSparseMicrokernelTester tester = GemmBlockSparseMicrokernelTester() \
0575:         .mr(MR) \
0576:         .nr(NR) \
0577:         .m(MR) \
0578:         .n(NR) \
0579:         .k(k) \
0580:         .rowBlockSize(row_block_size) \
0581:         .colBlockSize(col_block_size) \
0582:         .aStride(37); \
0583:     tester.test_packed<uint32_t>( \
0584:         prepacking_kernel, \
0585:         compute_kernel_w32); \
0586:     tester.test_packed<uint16_t>( \
0587:         prepacking_kernel, \
0588:         compute_kernel_w16); \
0589:     tester.test_packed<uint8_t>( \
0590:         prepacking_kernel, \
0591:         compute_kernel_w8); \
0592:   } \
0593: } \
0594:  \
0595: TEST(Q8GEMM__##MR ## x ##NR ## c##row_block_size ## x ##col_block_size ## __AARCH32_NEON, packedA_k_gt_8_strided_c) { \
0596:   TEST_REQUIRES_ARM_NEON; \
0597:   for (size_t k = 9; k < 16; k++) { \
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: no prominent local symbols.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：无明显局部符号。

### Lines 598-645 / 第 598-645 行

```cpp
0598:     GemmBlockSparseMicrokernelTester tester = GemmBlockSparseMicrokernelTester() \
0599:         .mr(MR) \
0600:         .nr(NR) \
0601:         .m(MR) \
0602:         .n(NR) \
0603:         .k(k) \
0604:         .rowBlockSize(row_block_size) \
0605:         .colBlockSize(col_block_size) \
0606:         .cStride(17); \
0607:     tester.test_packed<uint32_t>( \
0608:         prepacking_kernel, \
0609:         compute_kernel_w32); \
0610:     tester.test_packed<uint16_t>( \
0611:         prepacking_kernel, \
0612:         compute_kernel_w16); \
0613:     tester.test_packed<uint8_t>( \
0614:         prepacking_kernel, \
0615:         compute_kernel_w8); \
0616:   } \
0617: } \
0618:  \
0619: TEST(Q8GEMM__##MR ## x ##NR ## c##row_block_size ## x ##col_block_size ## __AARCH32_NEON, packedA_k_gt_8_azp0) { \
0620:   TEST_REQUIRES_ARM_NEON; \
0621:   for (size_t k = 9; k < 16; k++) { \
0622:     GemmBlockSparseMicrokernelTester tester = GemmBlockSparseMicrokernelTester() \
0623:         .mr(MR) \
0624:         .nr(NR) \
0625:         .m(MR) \
0626:         .n(NR) \
0627:         .k(k) \
0628:         .rowBlockSize(row_block_size) \
0629:         .colBlockSize(col_block_size) \
0630:         .aZeroPoint(0); \
0631:     tester.test_packed<uint32_t>( \
0632:         prepacking_kernel, \
0633:         compute_kernel_w32); \
0634:     tester.test_packed<uint16_t>( \
0635:         prepacking_kernel, \
0636:         compute_kernel_w16); \
0637:     tester.test_packed<uint8_t>( \
0638:         prepacking_kernel, \
0639:         compute_kernel_w8); \
0640:   } \
0641: } \
0642:  \
0643: TEST(Q8GEMM__##MR ## x ##NR ## c##row_block_size ## x ##col_block_size ## __AARCH32_NEON, packedA_k_gt_8_bzp0) { \
0644:   TEST_REQUIRES_ARM_NEON; \
0645:   for (size_t k = 9; k < 16; k++) { \
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: no prominent local symbols.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：无明显局部符号。

### Lines 646-694 / 第 646-694 行

```cpp
0646:     GemmBlockSparseMicrokernelTester tester = GemmBlockSparseMicrokernelTester() \
0647:         .mr(MR) \
0648:         .nr(NR) \
0649:         .m(MR) \
0650:         .n(NR) \
0651:         .k(k) \
0652:         .rowBlockSize(row_block_size) \
0653:         .colBlockSize(col_block_size) \
0654:         .bZeroPoint(0); \
0655:     tester.test_packed<uint32_t>( \
0656:         prepacking_kernel, \
0657:         compute_kernel_w32); \
0658:     tester.test_packed<uint16_t>( \
0659:         prepacking_kernel, \
0660:         compute_kernel_w16); \
0661:     tester.test_packed<uint8_t>( \
0662:         prepacking_kernel, \
0663:         compute_kernel_w8); \
0664:   } \
0665: } \
0666:  \
0667: TEST(Q8GEMM__##MR ## x ##NR ## c##row_block_size ## x ##col_block_size ## __AARCH32_NEON, packedA_k_gt_8_nozp) { \
0668:   TEST_REQUIRES_ARM_NEON; \
0669:   for (size_t k = 9; k < 16; k++) { \
0670:     GemmBlockSparseMicrokernelTester tester = GemmBlockSparseMicrokernelTester() \
0671:         .mr(MR) \
0672:         .nr(NR) \
0673:         .m(MR) \
0674:         .n(NR) \
0675:         .k(k) \
0676:         .rowBlockSize(row_block_size) \
0677:         .colBlockSize(col_block_size) \
0678:         .aZeroPoint(0) \
0679:         .bZeroPoint(0); \
0680:     tester.test_packed<uint32_t>( \
0681:         prepacking_kernel, \
0682:         compute_kernel_w32); \
0683:     tester.test_packed<uint16_t>( \
0684:         prepacking_kernel, \
0685:         compute_kernel_w16); \
0686:     tester.test_packed<uint8_t>( \
0687:         prepacking_kernel, \
0688:         compute_kernel_w8); \
0689:   } \
0690: } \
0691:  \
0692: TEST(Q8GEMM__##MR ## x ##NR ## c##row_block_size ## x ##col_block_size ## __AARCH32_NEON, packedA_k_gt_8_subtile) { \
0693:   TEST_REQUIRES_ARM_NEON; \
0694:   for (size_t k = 9; k < 16; k++) { \
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: no prominent local symbols.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：无明显局部符号。

### Lines 695-745 / 第 695-745 行

```cpp
0695:     for (uint32_t m = 1; m <= MR; m++) { \
0696:       for (uint32_t n = 1; n <= NR; n++) { \
0697:         GemmBlockSparseMicrokernelTester tester = GemmBlockSparseMicrokernelTester() \
0698:             .mr(MR) \
0699:             .nr(NR) \
0700:             .m(m) \
0701:             .n(n) \
0702:             .k(k) \
0703:             .rowBlockSize(row_block_size) \
0704:             .colBlockSize(col_block_size) \
0705:             .iterations(3); \
0706:         tester.test_packed<uint32_t>( \
0707:             prepacking_kernel, \
0708:             compute_kernel_w32); \
0709:         tester.test_packed<uint16_t>( \
0710:             prepacking_kernel, \
0711:             compute_kernel_w16); \
0712:         tester.test_packed<uint8_t>( \
0713:             prepacking_kernel, \
0714:             compute_kernel_w8); \
0715:       } \
0716:     } \
0717:   } \
0718: } \
0719:  \
0720: TEST(Q8GEMM__##MR ## x ##NR ## c##row_block_size ## x ##col_block_size ## __AARCH32_NEON, packedA_k_div_8) { \
0721:   TEST_REQUIRES_ARM_NEON; \
0722:   for (size_t k = 16; k < 128; k += 8) { \
0723:     GemmBlockSparseMicrokernelTester tester = GemmBlockSparseMicrokernelTester() \
0724:         .mr(MR) \
0725:         .nr(NR) \
0726:         .m(MR) \
0727:         .n(NR) \
0728:         .k(k) \
0729:         .rowBlockSize(row_block_size) \
0730:         .colBlockSize(col_block_size); \
0731:     tester.test_packed<uint32_t>( \
0732:         prepacking_kernel, \
0733:         compute_kernel_w32); \
0734:     tester.test_packed<uint16_t>( \
0735:         prepacking_kernel, \
0736:         compute_kernel_w16); \
0737:     tester.test_packed<uint8_t>( \
0738:         prepacking_kernel, \
0739:         compute_kernel_w8); \
0740:   } \
0741: } \
0742:  \
0743: TEST(Q8GEMM__##MR ## x ##NR ## c##row_block_size ## x ##col_block_size ## __AARCH32_NEON, packedA_k_div_8_strided_a) { \
0744:   TEST_REQUIRES_ARM_NEON; \
0745:   for (size_t k = 16; k < 128; k += 8) { \
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: no prominent local symbols.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：无明显局部符号。

### Lines 746-793 / 第 746-793 行

```cpp
0746:     GemmBlockSparseMicrokernelTester tester = GemmBlockSparseMicrokernelTester() \
0747:         .mr(MR) \
0748:         .nr(NR) \
0749:         .m(MR) \
0750:         .n(NR) \
0751:         .k(k) \
0752:         .rowBlockSize(row_block_size) \
0753:         .colBlockSize(col_block_size) \
0754:         .aStride(171); \
0755:     tester.test_packed<uint32_t>( \
0756:         prepacking_kernel, \
0757:         compute_kernel_w32); \
0758:     tester.test_packed<uint16_t>( \
0759:         prepacking_kernel, \
0760:         compute_kernel_w16); \
0761:     tester.test_packed<uint8_t>( \
0762:         prepacking_kernel, \
0763:         compute_kernel_w8); \
0764:   } \
0765: } \
0766:  \
0767: TEST(Q8GEMM__##MR ## x ##NR ## c##row_block_size ## x ##col_block_size ## __AARCH32_NEON, packedA_k_div_8_strided_c) { \
0768:   TEST_REQUIRES_ARM_NEON; \
0769:   for (size_t k = 16; k < 128; k += 8) { \
0770:     GemmBlockSparseMicrokernelTester tester = GemmBlockSparseMicrokernelTester() \
0771:         .mr(MR) \
0772:         .nr(NR) \
0773:         .m(MR) \
0774:         .n(NR) \
0775:         .k(k) \
0776:         .rowBlockSize(row_block_size) \
0777:         .colBlockSize(col_block_size) \
0778:         .cStride(17); \
0779:     tester.test_packed<uint32_t>( \
0780:         prepacking_kernel, \
0781:         compute_kernel_w32); \
0782:     tester.test_packed<uint16_t>( \
0783:         prepacking_kernel, \
0784:         compute_kernel_w16); \
0785:     tester.test_packed<uint8_t>( \
0786:         prepacking_kernel, \
0787:         compute_kernel_w8); \
0788:   } \
0789: } \
0790:  \
0791: TEST(Q8GEMM__##MR ## x ##NR ## c##row_block_size ## x ##col_block_size ## __AARCH32_NEON, packedA_k_div_8_subtile) { \
0792:   TEST_REQUIRES_ARM_NEON; \
0793:   for (size_t k = 16; k < 128; k += 24) { \
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: no prominent local symbols.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：无明显局部符号。

### Lines 794-841 / 第 794-841 行

```cpp
0794:     for (uint32_t m = 1; m <= MR; m++) { \
0795:       for (uint32_t n = 1; n <= NR; n++) { \
0796:         GemmBlockSparseMicrokernelTester tester = GemmBlockSparseMicrokernelTester() \
0797:             .mr(MR) \
0798:             .nr(NR) \
0799:             .m(m) \
0800:             .n(n) \
0801:             .k(k) \
0802:             .rowBlockSize(row_block_size) \
0803:             .colBlockSize(col_block_size) \
0804:             .iterations(3); \
0805:         tester.test_packed<uint32_t>( \
0806:             prepacking_kernel, \
0807:             compute_kernel_w32); \
0808:         tester.test_packed<uint16_t>( \
0809:             prepacking_kernel, \
0810:             compute_kernel_w16); \
0811:         tester.test_packed<uint8_t>( \
0812:             prepacking_kernel, \
0813:             compute_kernel_w8); \
0814:       } \
0815:     } \
0816:   } \
0817: }
0818: 
0819: #define TEST_PACKED_1x4_SPARSE_OP(MR, NR, prepacking_kernel, compute_kernel_w32, compute_kernel_w16, compute_kernel_w8) \
0820:   TEST_PACKED_ROW_BLOCK_SIZEXCOL_BLOCK_SIZE_SPARSE_OP(MR, \
0821:       NR, 1, 4, prepacking_kernel, compute_kernel_w32, compute_kernel_w16, compute_kernel_w8)
0822: #define TEST_PACKED_8x1_SPARSE_OP(MR, NR, prepacking_kernel, compute_kernel_w32, compute_kernel_w16, compute_kernel_w8) \
0823:   TEST_PACKED_ROW_BLOCK_SIZEXCOL_BLOCK_SIZE_SPARSE_OP(MR, \
0824:       NR, 8, 1, prepacking_kernel, compute_kernel_w32, compute_kernel_w16, compute_kernel_w8)
0825: 
0826: #if CPUINFO_ARCH_ARM
0827: TEST_PACKED_1x4_SPARSE_OP(
0828:     4,
0829:     8,
0830:     pytorch_q8gemm_sparse_packA_ukernel_4x4__aarch32_neon,
0831:     pytorch_q8gemm_dq_sparse_1x4_ukernel_4x8_packedA_w32__aarch32_neon,
0832:     pytorch_q8gemm_dq_sparse_1x4_ukernel_4x8_packedA_w16__aarch32_neon,
0833:     pytorch_q8gemm_dq_sparse_1x4_ukernel_4x8_packedA_w8__aarch32_neon)
0834: TEST_PACKED_8x1_SPARSE_OP(
0835:     4,
0836:     8,
0837:     pytorch_q8gemm_sparse_packA_ukernel_4x4__aarch32_neon,
0838:     pytorch_q8gemm_dq_sparse_8x1_ukernel_4x8_packedA_w32__aarch32_neon,
0839:     pytorch_q8gemm_dq_sparse_8x1_ukernel_4x8_packedA_w16__aarch32_neon,
0840:     pytorch_q8gemm_dq_sparse_8x1_ukernel_4x8_packedA_w8__aarch32_neon)
0841: 
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends; iterates over tensors, descriptors, options, or runtime state. Key symbols: no prominent local symbols.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端；遍历张量、描述符、配置项或运行时状态。关键符号：无明显局部符号。

### Lines 842-883 / 第 842-883 行

```cpp
0842: #endif
0843: 
0844: #if CPUINFO_ARCH_ARM64
0845: 
0846: TEST_PACKED_1x4_SPARSE_OP(
0847:     8,
0848:     8,
0849:     pytorch_q8gemm_sparse_packA_ukernel_8x4__aarch64_neon,
0850:     pytorch_q8gemm_dq_sparse_1x4_ukernel_8x8_packedA_w32__aarch64_neon,
0851:     pytorch_q8gemm_dq_sparse_1x4_ukernel_8x8_packedA_w16__aarch64_neon,
0852:     pytorch_q8gemm_dq_sparse_1x4_ukernel_8x8_packedA_w8__aarch64_neon)
0853: TEST_PACKED_8x1_SPARSE_OP(
0854:     8,
0855:     8,
0856:     pytorch_q8gemm_sparse_packA_ukernel_8x4__aarch64_neon,
0857:     pytorch_q8gemm_dq_sparse_8x1_ukernel_8x8_packedA_w32__aarch64_neon,
0858:     pytorch_q8gemm_dq_sparse_8x1_ukernel_8x8_packedA_w16__aarch64_neon,
0859:     pytorch_q8gemm_dq_sparse_8x1_ukernel_8x8_packedA_w8__aarch64_neon)
0860: 
0861: #endif
0862: 
0863: #if CPUINFO_ARCH_X86 || CPUINFO_ARCH_X86_64
0864: 
0865: TEST(Q8GEMM_8x4c1x4__SSE2, packedA_k_lt_4) {
0866:   TEST_REQUIRES_X86_SSE2;
0867:   GemmBlockSparseMicrokernelTester tester = GemmBlockSparseMicrokernelTester()
0868:       .mr(8)
0869:       .nr(4)
0870:       .m(8)
0871:       .n(4)
0872:       .k(3);
0873:   tester.test_packed<uint32_t>(
0874:       pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
0875:       pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w32__sse2);
0876:   tester.test_packed<uint16_t>(
0877:       pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
0878:       pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w16__sse2);
0879:   tester.test_packed<uint8_t>(
0880:       pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
0881:       pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w8__sse2);
0882: }
0883: 
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends. Key symbols: `TEST_PACKED_1x4_SPARSE_OP`.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端。关键符号：`TEST_PACKED_1x4_SPARSE_OP`。

### Lines 884-923 / 第 884-923 行

```cpp
0884: TEST(Q8GEMM_8x4c1x4__SSE2, packedA_k_lt_4_strided_a) {
0885:   TEST_REQUIRES_X86_SSE2;
0886:   GemmBlockSparseMicrokernelTester tester = GemmBlockSparseMicrokernelTester()
0887:       .mr(8)
0888:       .nr(4)
0889:       .m(8)
0890:       .n(4)
0891:       .k(3)
0892:       .aStride(37);
0893:   tester.test_packed<uint32_t>(
0894:       pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
0895:       pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w32__sse2);
0896:   tester.test_packed<uint16_t>(
0897:       pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
0898:       pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w16__sse2);
0899:   tester.test_packed<uint8_t>(
0900:       pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
0901:       pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w8__sse2);
0902: }
0903: 
0904: TEST(Q8GEMM_8x4c1x4__SSE2, packedA_k_lt_4_strided_c) {
0905:   TEST_REQUIRES_X86_SSE2;
0906:   GemmBlockSparseMicrokernelTester tester = GemmBlockSparseMicrokernelTester()
0907:       .mr(8)
0908:       .nr(4)
0909:       .m(8)
0910:       .n(4)
0911:       .k(3)
0912:       .cStride(17);
0913:   tester.test_packed<uint32_t>(
0914:       pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
0915:       pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w32__sse2);
0916:   tester.test_packed<uint16_t>(
0917:       pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
0918:       pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w16__sse2);
0919:   tester.test_packed<uint8_t>(
0920:       pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
0921:       pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w8__sse2);
0922: }
0923: 
```

- **EN:** This block implements local helper logic for `q8gemm sparse`. Key symbols: no prominent local symbols.
- **CN:** 该代码块实现与 `q8gemm sparse` 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 924-963 / 第 924-963 行

```cpp
0924: TEST(Q8GEMM_8x4c1x4__SSE2, packedA_k_lt_4_qmin128) {
0925:   TEST_REQUIRES_X86_SSE2;
0926:   GemmBlockSparseMicrokernelTester tester = GemmBlockSparseMicrokernelTester()
0927:       .mr(8)
0928:       .nr(4)
0929:       .m(8)
0930:       .n(4)
0931:       .k(3)
0932:       .qmin(128);
0933:   tester.test_packed<uint32_t>(
0934:       pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
0935:       pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w32__sse2);
0936:   tester.test_packed<uint16_t>(
0937:       pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
0938:       pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w16__sse2);
0939:   tester.test_packed<uint8_t>(
0940:       pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
0941:       pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w8__sse2);
0942: }
0943: 
0944: TEST(Q8GEMM_8x4c1x4__SSE2, packedA_k_lt_4_qmax128) {
0945:   TEST_REQUIRES_X86_SSE2;
0946:   GemmBlockSparseMicrokernelTester tester = GemmBlockSparseMicrokernelTester()
0947:       .mr(8)
0948:       .nr(4)
0949:       .m(8)
0950:       .n(4)
0951:       .k(3)
0952:       .qmax(128);
0953:   tester.test_packed<uint32_t>(
0954:       pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
0955:       pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w32__sse2);
0956:   tester.test_packed<uint16_t>(
0957:       pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
0958:       pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w16__sse2);
0959:   tester.test_packed<uint8_t>(
0960:       pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
0961:       pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w8__sse2);
0962: }
0963: 
```

- **EN:** This block implements local helper logic for `q8gemm sparse`. Key symbols: no prominent local symbols.
- **CN:** 该代码块实现与 `q8gemm sparse` 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 964-1003 / 第 964-1003 行

```cpp
0964: TEST(Q8GEMM_8x4c1x4__SSE2, packedA_k_lt_4_azp0) {
0965:   TEST_REQUIRES_X86_SSE2;
0966:   GemmBlockSparseMicrokernelTester tester = GemmBlockSparseMicrokernelTester()
0967:       .mr(8)
0968:       .nr(4)
0969:       .m(8)
0970:       .n(4)
0971:       .k(3)
0972:       .aZeroPoint(0);
0973:   tester.test_packed<uint32_t>(
0974:       pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
0975:       pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w32__sse2);
0976:   tester.test_packed<uint16_t>(
0977:       pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
0978:       pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w16__sse2);
0979:   tester.test_packed<uint8_t>(
0980:       pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
0981:       pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w8__sse2);
0982: }
0983: 
0984: TEST(Q8GEMM_8x4c1x4__SSE2, packedA_k_lt_4_bzp0) {
0985:   TEST_REQUIRES_X86_SSE2;
0986:   GemmBlockSparseMicrokernelTester tester = GemmBlockSparseMicrokernelTester()
0987:       .mr(8)
0988:       .nr(4)
0989:       .m(8)
0990:       .n(4)
0991:       .k(3)
0992:       .bZeroPoint(0);
0993:   tester.test_packed<uint32_t>(
0994:       pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
0995:       pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w32__sse2);
0996:   tester.test_packed<uint16_t>(
0997:       pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
0998:       pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w16__sse2);
0999:   tester.test_packed<uint8_t>(
1000:       pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
1001:       pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w8__sse2);
1002: }
1003: 
```

- **EN:** This block implements local helper logic for `q8gemm sparse`. Key symbols: no prominent local symbols.
- **CN:** 该代码块实现与 `q8gemm sparse` 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 1004-1043 / 第 1004-1043 行

```cpp
1004: TEST(Q8GEMM_8x4c1x4__SSE2, packedA_k_lt_4_nozp) {
1005:   TEST_REQUIRES_X86_SSE2;
1006:   GemmBlockSparseMicrokernelTester tester = GemmBlockSparseMicrokernelTester()
1007:       .mr(8)
1008:       .nr(4)
1009:       .m(8)
1010:       .n(4)
1011:       .k(3)
1012:       .aZeroPoint(0)
1013:       .bZeroPoint(0);
1014:   tester.test_packed<uint32_t>(
1015:       pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
1016:       pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w32__sse2);
1017:   tester.test_packed<uint16_t>(
1018:       pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
1019:       pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w16__sse2);
1020:   tester.test_packed<uint8_t>(
1021:       pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
1022:       pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w8__sse2);
1023: }
1024: 
1025: TEST(Q8GEMM_8x4c1x4__SSE2, packedA_k_lt_8) {
1026:   TEST_REQUIRES_X86_SSE2;
1027:   GemmBlockSparseMicrokernelTester tester = GemmBlockSparseMicrokernelTester()
1028:       .mr(8)
1029:       .nr(4)
1030:       .m(8)
1031:       .n(4)
1032:       .k(5);
1033:   tester.test_packed<uint32_t>(
1034:       pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
1035:       pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w32__sse2);
1036:   tester.test_packed<uint16_t>(
1037:       pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
1038:       pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w16__sse2);
1039:   tester.test_packed<uint8_t>(
1040:       pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
1041:       pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w8__sse2);
1042: }
1043: 
```

- **EN:** This block implements local helper logic for `q8gemm sparse`. Key symbols: no prominent local symbols.
- **CN:** 该代码块实现与 `q8gemm sparse` 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 1044-1083 / 第 1044-1083 行

```cpp
1044: TEST(Q8GEMM_8x4c1x4__SSE2, packedA_k_lt_8_strided_a) {
1045:   TEST_REQUIRES_X86_SSE2;
1046:   GemmBlockSparseMicrokernelTester tester = GemmBlockSparseMicrokernelTester()
1047:       .mr(8)
1048:       .nr(4)
1049:       .m(8)
1050:       .n(4)
1051:       .k(5)
1052:       .aStride(37);
1053:   tester.test_packed<uint32_t>(
1054:       pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
1055:       pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w32__sse2);
1056:   tester.test_packed<uint16_t>(
1057:       pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
1058:       pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w16__sse2);
1059:   tester.test_packed<uint8_t>(
1060:       pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
1061:       pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w8__sse2);
1062: }
1063: 
1064: TEST(Q8GEMM_8x4c1x4__SSE2, packedA_k_lt_8_strided_c) {
1065:   TEST_REQUIRES_X86_SSE2;
1066:   GemmBlockSparseMicrokernelTester tester = GemmBlockSparseMicrokernelTester()
1067:       .mr(8)
1068:       .nr(4)
1069:       .m(8)
1070:       .n(4)
1071:       .k(5)
1072:       .cStride(17);
1073:   tester.test_packed<uint32_t>(
1074:       pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
1075:       pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w32__sse2);
1076:   tester.test_packed<uint16_t>(
1077:       pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
1078:       pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w16__sse2);
1079:   tester.test_packed<uint8_t>(
1080:       pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
1081:       pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w8__sse2);
1082: }
1083: 
```

- **EN:** This block implements local helper logic for `q8gemm sparse`. Key symbols: no prominent local symbols.
- **CN:** 该代码块实现与 `q8gemm sparse` 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 1084-1123 / 第 1084-1123 行

```cpp
1084: TEST(Q8GEMM_8x4c1x4__SSE2, packedA_k_lt_8_qmin128) {
1085:   TEST_REQUIRES_X86_SSE2;
1086:   GemmBlockSparseMicrokernelTester tester = GemmBlockSparseMicrokernelTester()
1087:       .mr(8)
1088:       .nr(4)
1089:       .m(8)
1090:       .n(4)
1091:       .k(5)
1092:       .qmin(128);
1093:   tester.test_packed<uint32_t>(
1094:       pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
1095:       pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w32__sse2);
1096:   tester.test_packed<uint16_t>(
1097:       pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
1098:       pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w16__sse2);
1099:   tester.test_packed<uint8_t>(
1100:       pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
1101:       pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w8__sse2);
1102: }
1103: 
1104: TEST(Q8GEMM_8x4c1x4__SSE2, packedA_k_lt_8_qmax128) {
1105:   TEST_REQUIRES_X86_SSE2;
1106:   GemmBlockSparseMicrokernelTester tester = GemmBlockSparseMicrokernelTester()
1107:       .mr(8)
1108:       .nr(4)
1109:       .m(8)
1110:       .n(4)
1111:       .k(5)
1112:       .qmax(128);
1113:   tester.test_packed<uint32_t>(
1114:       pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
1115:       pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w32__sse2);
1116:   tester.test_packed<uint16_t>(
1117:       pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
1118:       pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w16__sse2);
1119:   tester.test_packed<uint8_t>(
1120:       pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
1121:       pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w8__sse2);
1122: }
1123: 
```

- **EN:** This block implements local helper logic for `q8gemm sparse`. Key symbols: no prominent local symbols.
- **CN:** 该代码块实现与 `q8gemm sparse` 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 1124-1163 / 第 1124-1163 行

```cpp
1124: TEST(Q8GEMM_8x4c1x4__SSE2, packedA_k_lt_8_azp0) {
1125:   TEST_REQUIRES_X86_SSE2;
1126:   GemmBlockSparseMicrokernelTester tester = GemmBlockSparseMicrokernelTester()
1127:       .mr(8)
1128:       .nr(4)
1129:       .m(8)
1130:       .n(4)
1131:       .k(5)
1132:       .aZeroPoint(0);
1133:   tester.test_packed<uint32_t>(
1134:       pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
1135:       pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w32__sse2);
1136:   tester.test_packed<uint16_t>(
1137:       pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
1138:       pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w16__sse2);
1139:   tester.test_packed<uint8_t>(
1140:       pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
1141:       pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w8__sse2);
1142: }
1143: 
1144: TEST(Q8GEMM_8x4c1x4__SSE2, packedA_k_lt_8_bzp0) {
1145:   TEST_REQUIRES_X86_SSE2;
1146:   GemmBlockSparseMicrokernelTester tester = GemmBlockSparseMicrokernelTester()
1147:       .mr(8)
1148:       .nr(4)
1149:       .m(8)
1150:       .n(4)
1151:       .k(5)
1152:       .bZeroPoint(0);
1153:   tester.test_packed<uint32_t>(
1154:       pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
1155:       pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w32__sse2);
1156:   tester.test_packed<uint16_t>(
1157:       pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
1158:       pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w16__sse2);
1159:   tester.test_packed<uint8_t>(
1160:       pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
1161:       pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w8__sse2);
1162: }
1163: 
```

- **EN:** This block implements local helper logic for `q8gemm sparse`. Key symbols: no prominent local symbols.
- **CN:** 该代码块实现与 `q8gemm sparse` 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 1164-1203 / 第 1164-1203 行

```cpp
1164: TEST(Q8GEMM_8x4c1x4__SSE2, packedA_k_lt_8_nozp) {
1165:   TEST_REQUIRES_X86_SSE2;
1166:   GemmBlockSparseMicrokernelTester tester = GemmBlockSparseMicrokernelTester()
1167:       .mr(8)
1168:       .nr(4)
1169:       .m(8)
1170:       .n(4)
1171:       .k(5)
1172:       .aZeroPoint(0)
1173:       .bZeroPoint(0);
1174:   tester.test_packed<uint32_t>(
1175:       pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
1176:       pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w32__sse2);
1177:   tester.test_packed<uint16_t>(
1178:       pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
1179:       pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w16__sse2);
1180:   tester.test_packed<uint8_t>(
1181:       pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
1182:       pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w8__sse2);
1183: }
1184: 
1185: TEST(Q8GEMM_8x4c1x4__SSE2, packedA_k_eq_8) {
1186:   TEST_REQUIRES_X86_SSE2;
1187:   GemmBlockSparseMicrokernelTester tester = GemmBlockSparseMicrokernelTester()
1188:       .mr(8)
1189:       .nr(4)
1190:       .m(8)
1191:       .n(4)
1192:       .k(8);
1193:   tester.test_packed<uint32_t>(
1194:       pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
1195:       pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w32__sse2);
1196:   tester.test_packed<uint16_t>(
1197:       pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
1198:       pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w16__sse2);
1199:   tester.test_packed<uint8_t>(
1200:       pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
1201:       pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w8__sse2);
1202: }
1203: 
```

- **EN:** This block implements local helper logic for `q8gemm sparse`. Key symbols: no prominent local symbols.
- **CN:** 该代码块实现与 `q8gemm sparse` 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 1204-1243 / 第 1204-1243 行

```cpp
1204: TEST(Q8GEMM_8x4c1x4__SSE2, packedA_k_eq_8_strided_a) {
1205:   TEST_REQUIRES_X86_SSE2;
1206:   GemmBlockSparseMicrokernelTester tester = GemmBlockSparseMicrokernelTester()
1207:       .mr(8)
1208:       .nr(4)
1209:       .m(8)
1210:       .n(4)
1211:       .k(8)
1212:       .aStride(37);
1213:   tester.test_packed<uint32_t>(
1214:       pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
1215:       pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w32__sse2);
1216:   tester.test_packed<uint16_t>(
1217:       pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
1218:       pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w16__sse2);
1219:   tester.test_packed<uint8_t>(
1220:       pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
1221:       pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w8__sse2);
1222: }
1223: 
1224: TEST(Q8GEMM_8x4c1x4__SSE2, packedA_k_eq_8_strided_c) {
1225:   TEST_REQUIRES_X86_SSE2;
1226:   GemmBlockSparseMicrokernelTester tester = GemmBlockSparseMicrokernelTester()
1227:       .mr(8)
1228:       .nr(4)
1229:       .m(8)
1230:       .n(4)
1231:       .k(8)
1232:       .cStride(17);
1233:   tester.test_packed<uint32_t>(
1234:       pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
1235:       pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w32__sse2);
1236:   tester.test_packed<uint16_t>(
1237:       pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
1238:       pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w16__sse2);
1239:   tester.test_packed<uint8_t>(
1240:       pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
1241:       pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w8__sse2);
1242: }
1243: 
```

- **EN:** This block implements local helper logic for `q8gemm sparse`. Key symbols: no prominent local symbols.
- **CN:** 该代码块实现与 `q8gemm sparse` 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 1244-1283 / 第 1244-1283 行

```cpp
1244: TEST(Q8GEMM_8x4c1x4__SSE2, packedA_k_eq_8_qmin128) {
1245:   TEST_REQUIRES_X86_SSE2;
1246:   GemmBlockSparseMicrokernelTester tester = GemmBlockSparseMicrokernelTester()
1247:       .mr(8)
1248:       .nr(4)
1249:       .m(8)
1250:       .n(4)
1251:       .k(8)
1252:       .qmin(128);
1253:   tester.test_packed<uint32_t>(
1254:       pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
1255:       pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w32__sse2);
1256:   tester.test_packed<uint16_t>(
1257:       pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
1258:       pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w16__sse2);
1259:   tester.test_packed<uint8_t>(
1260:       pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
1261:       pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w8__sse2);
1262: }
1263: 
1264: TEST(Q8GEMM_8x4c1x4__SSE2, packedA_k_eq_8_qmax128) {
1265:   TEST_REQUIRES_X86_SSE2;
1266:   GemmBlockSparseMicrokernelTester tester = GemmBlockSparseMicrokernelTester()
1267:       .mr(8)
1268:       .nr(4)
1269:       .m(8)
1270:       .n(4)
1271:       .k(8)
1272:       .qmax(128);
1273:   tester.test_packed<uint32_t>(
1274:       pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
1275:       pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w32__sse2);
1276:   tester.test_packed<uint16_t>(
1277:       pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
1278:       pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w16__sse2);
1279:   tester.test_packed<uint8_t>(
1280:       pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
1281:       pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w8__sse2);
1282: }
1283: 
```

- **EN:** This block implements local helper logic for `q8gemm sparse`. Key symbols: no prominent local symbols.
- **CN:** 该代码块实现与 `q8gemm sparse` 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 1284-1323 / 第 1284-1323 行

```cpp
1284: TEST(Q8GEMM_8x4c1x4__SSE2, packedA_k_eq_8_azp0) {
1285:   TEST_REQUIRES_X86_SSE2;
1286:   GemmBlockSparseMicrokernelTester tester = GemmBlockSparseMicrokernelTester()
1287:       .mr(8)
1288:       .nr(4)
1289:       .m(8)
1290:       .n(4)
1291:       .k(8)
1292:       .aZeroPoint(0);
1293:   tester.test_packed<uint32_t>(
1294:       pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
1295:       pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w32__sse2);
1296:   tester.test_packed<uint16_t>(
1297:       pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
1298:       pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w16__sse2);
1299:   tester.test_packed<uint8_t>(
1300:       pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
1301:       pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w8__sse2);
1302: }
1303: 
1304: TEST(Q8GEMM_8x4c1x4__SSE2, packedA_k_eq_8_bzp0) {
1305:   TEST_REQUIRES_X86_SSE2;
1306:   GemmBlockSparseMicrokernelTester tester = GemmBlockSparseMicrokernelTester()
1307:       .mr(8)
1308:       .nr(4)
1309:       .m(8)
1310:       .n(4)
1311:       .k(8)
1312:       .bZeroPoint(0);
1313:   tester.test_packed<uint32_t>(
1314:       pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
1315:       pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w32__sse2);
1316:   tester.test_packed<uint16_t>(
1317:       pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
1318:       pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w16__sse2);
1319:   tester.test_packed<uint8_t>(
1320:       pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
1321:       pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w8__sse2);
1322: }
1323: 
```

- **EN:** This block implements local helper logic for `q8gemm sparse`. Key symbols: no prominent local symbols.
- **CN:** 该代码块实现与 `q8gemm sparse` 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 1324-1365 / 第 1324-1365 行

```cpp
1324: TEST(Q8GEMM_8x4c1x4__SSE2, packedA_k_eq_8_nozp) {
1325:   TEST_REQUIRES_X86_SSE2;
1326:   GemmBlockSparseMicrokernelTester tester = GemmBlockSparseMicrokernelTester()
1327:       .mr(8)
1328:       .nr(4)
1329:       .m(8)
1330:       .n(4)
1331:       .k(8)
1332:       .aZeroPoint(0)
1333:       .bZeroPoint(0);
1334:   tester.test_packed<uint32_t>(
1335:       pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
1336:       pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w32__sse2);
1337:   tester.test_packed<uint16_t>(
1338:       pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
1339:       pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w16__sse2);
1340:   tester.test_packed<uint8_t>(
1341:       pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
1342:       pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w8__sse2);
1343: }
1344: 
1345: TEST(Q8GEMM_8x4c1x4__SSE2, packedA_k_gt_8) {
1346:   TEST_REQUIRES_X86_SSE2;
1347:   for (size_t k = 9; k < 16; k++) {
1348:     GemmBlockSparseMicrokernelTester tester = GemmBlockSparseMicrokernelTester()
1349:         .mr(8)
1350:         .nr(4)
1351:         .m(8)
1352:         .n(4)
1353:         .k(k);
1354:     tester.test_packed<uint32_t>(
1355:         pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
1356:         pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w32__sse2);
1357:     tester.test_packed<uint16_t>(
1358:         pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
1359:         pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w16__sse2);
1360:     tester.test_packed<uint8_t>(
1361:         pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
1362:         pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w8__sse2);
1363:   }
1364: }
1365: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: no prominent local symbols.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：无明显局部符号。

### Lines 1366-1409 / 第 1366-1409 行

```cpp
1366: TEST(Q8GEMM_8x4c1x4__SSE2, packedA_k_gt_8_strided_a) {
1367:   TEST_REQUIRES_X86_SSE2;
1368:   for (size_t k = 9; k < 16; k++) {
1369:     GemmBlockSparseMicrokernelTester tester = GemmBlockSparseMicrokernelTester()
1370:         .mr(8)
1371:         .nr(4)
1372:         .m(8)
1373:         .n(4)
1374:         .k(k)
1375:         .aStride(37);
1376:     tester.test_packed<uint32_t>(
1377:         pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
1378:         pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w32__sse2);
1379:     tester.test_packed<uint16_t>(
1380:         pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
1381:         pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w16__sse2);
1382:     tester.test_packed<uint8_t>(
1383:         pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
1384:         pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w8__sse2);
1385:   }
1386: }
1387: 
1388: TEST(Q8GEMM_8x4c1x4__SSE2, packedA_k_gt_8_strided_c) {
1389:   TEST_REQUIRES_X86_SSE2;
1390:   for (size_t k = 9; k < 16; k++) {
1391:     GemmBlockSparseMicrokernelTester tester = GemmBlockSparseMicrokernelTester()
1392:         .mr(8)
1393:         .nr(4)
1394:         .m(8)
1395:         .n(4)
1396:         .k(k)
1397:         .cStride(17);
1398:     tester.test_packed<uint32_t>(
1399:         pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
1400:         pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w32__sse2);
1401:     tester.test_packed<uint16_t>(
1402:         pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
1403:         pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w16__sse2);
1404:     tester.test_packed<uint8_t>(
1405:         pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
1406:         pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w8__sse2);
1407:   }
1408: }
1409: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: no prominent local symbols.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：无明显局部符号。

### Lines 1410-1453 / 第 1410-1453 行

```cpp
1410: TEST(Q8GEMM_8x4c1x4__SSE2, packedA_k_gt_8_azp0) {
1411:   TEST_REQUIRES_X86_SSE2;
1412:   for (size_t k = 9; k < 16; k++) {
1413:     GemmBlockSparseMicrokernelTester tester = GemmBlockSparseMicrokernelTester()
1414:         .mr(8)
1415:         .nr(4)
1416:         .m(8)
1417:         .n(4)
1418:         .k(k)
1419:         .aZeroPoint(0);
1420:     tester.test_packed<uint32_t>(
1421:         pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
1422:         pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w32__sse2);
1423:     tester.test_packed<uint16_t>(
1424:         pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
1425:         pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w16__sse2);
1426:     tester.test_packed<uint8_t>(
1427:         pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
1428:         pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w8__sse2);
1429:   }
1430: }
1431: 
1432: TEST(Q8GEMM_8x4c1x4__SSE2, packedA_k_gt_8_bzp0) {
1433:   TEST_REQUIRES_X86_SSE2;
1434:   for (size_t k = 9; k < 16; k++) {
1435:     GemmBlockSparseMicrokernelTester tester = GemmBlockSparseMicrokernelTester()
1436:         .mr(8)
1437:         .nr(4)
1438:         .m(8)
1439:         .n(4)
1440:         .k(k)
1441:         .bZeroPoint(0);
1442:     tester.test_packed<uint32_t>(
1443:         pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
1444:         pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w32__sse2);
1445:     tester.test_packed<uint16_t>(
1446:         pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
1447:         pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w16__sse2);
1448:     tester.test_packed<uint8_t>(
1449:         pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
1450:         pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w8__sse2);
1451:   }
1452: }
1453: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: no prominent local symbols.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：无明显局部符号。

### Lines 1454-1497 / 第 1454-1497 行

```cpp
1454: TEST(Q8GEMM_8x4c1x4__SSE2, packedA_k_gt_8_nozp) {
1455:   TEST_REQUIRES_X86_SSE2;
1456:   for (size_t k = 9; k < 16; k++) {
1457:     GemmBlockSparseMicrokernelTester tester = GemmBlockSparseMicrokernelTester()
1458:         .mr(8)
1459:         .nr(4)
1460:         .m(8)
1461:         .n(4)
1462:         .k(k)
1463:         .aZeroPoint(0)
1464:         .bZeroPoint(0);
1465:     tester.test_packed<uint32_t>(
1466:         pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
1467:         pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w32__sse2);
1468:     tester.test_packed<uint16_t>(
1469:         pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
1470:         pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w16__sse2);
1471:     tester.test_packed<uint8_t>(
1472:         pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
1473:         pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w8__sse2);
1474:   }
1475: }
1476: 
1477: TEST(Q8GEMM_8x4c1x4__SSE2, packedA_k_div_8) {
1478:   TEST_REQUIRES_X86_SSE2;
1479:   for (size_t k = 16; k < 128; k += 8) {
1480:     GemmBlockSparseMicrokernelTester tester = GemmBlockSparseMicrokernelTester()
1481:         .mr(8)
1482:         .nr(4)
1483:         .m(8)
1484:         .n(4)
1485:         .k(k);
1486:     tester.test_packed<uint32_t>(
1487:         pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
1488:         pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w32__sse2);
1489:     tester.test_packed<uint16_t>(
1490:         pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
1491:         pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w16__sse2);
1492:     tester.test_packed<uint8_t>(
1493:         pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
1494:         pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w8__sse2);
1495:   }
1496: }
1497: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: no prominent local symbols.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：无明显局部符号。

### Lines 1498-1541 / 第 1498-1541 行

```cpp
1498: TEST(Q8GEMM_8x4c1x4__SSE2, packedA_k_div_8_strided_a) {
1499:   TEST_REQUIRES_X86_SSE2;
1500:   for (size_t k = 16; k < 128; k += 8) {
1501:     GemmBlockSparseMicrokernelTester tester = GemmBlockSparseMicrokernelTester()
1502:         .mr(8)
1503:         .nr(4)
1504:         .m(8)
1505:         .n(4)
1506:         .k(k)
1507:         .aStride(171);
1508:     tester.test_packed<uint32_t>(
1509:         pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
1510:         pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w32__sse2);
1511:     tester.test_packed<uint16_t>(
1512:         pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
1513:         pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w16__sse2);
1514:     tester.test_packed<uint8_t>(
1515:         pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
1516:         pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w8__sse2);
1517:   }
1518: }
1519: 
1520: TEST(Q8GEMM_8x4c1x4__SSE2, packedA_k_div_8_strided_c) {
1521:   TEST_REQUIRES_X86_SSE2;
1522:   for (size_t k = 16; k < 128; k += 8) {
1523:     GemmBlockSparseMicrokernelTester tester = GemmBlockSparseMicrokernelTester()
1524:         .mr(8)
1525:         .nr(4)
1526:         .m(8)
1527:         .n(4)
1528:         .k(k)
1529:         .cStride(17);
1530:     tester.test_packed<uint32_t>(
1531:         pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
1532:         pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w32__sse2);
1533:     tester.test_packed<uint16_t>(
1534:         pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
1535:         pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w16__sse2);
1536:     tester.test_packed<uint8_t>(
1537:         pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
1538:         pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w8__sse2);
1539:   }
1540: }
1541: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: no prominent local symbols.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：无明显局部符号。

### Lines 1542-1568 / 第 1542-1568 行

```cpp
1542: TEST(Q8GEMM_8x4c1x4__SSE2, packedA_k_div_8_subtile) {
1543:   TEST_REQUIRES_X86_SSE2;
1544:   for (size_t k = 16; k < 128; k += 24) {
1545:     for (uint32_t m = 1; m <= 8; m++) {
1546:       for (uint32_t n = 1; n <= 4; n++) {
1547:         GemmBlockSparseMicrokernelTester tester = GemmBlockSparseMicrokernelTester()
1548:             .mr(8)
1549:             .nr(4)
1550:             .m(m)
1551:             .n(n)
1552:             .k(k)
1553:             .iterations(3);
1554:         tester.test_packed<uint32_t>(
1555:             pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
1556:             pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w32__sse2);
1557:         tester.test_packed<uint16_t>(
1558:             pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
1559:             pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w16__sse2);
1560:         tester.test_packed<uint8_t>(
1561:             pytorch_q8gemm_sparse_packA_ukernel_8x4__sse2,
1562:             pytorch_q8gemm_dq_sparse_1x4_ukernel_8x4_packedA_w8__sse2);
1563:       }
1564:     }
1565:   }
1566: }
1567: 
1568: #endif
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends; iterates over tensors, descriptors, options, or runtime state. Key symbols: no prominent local symbols.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端；遍历张量、描述符、配置项或运行时状态。关键符号：无明显局部符号。


## Key Concepts / 关键概念
- **ATen native operator logic** — ATen 原生算子逻辑
- **GPU kernel/runtime interaction** — GPU 内核/运行时交互
- **Linear algebra backend integration** — 线性代数后端集成
- **Runtime validation and invariants** — 运行时校验与不变量

## Dependencies / 依赖关系
- **Internal includes / 内部头文件**: 无
- **External includes / 外部头文件**: `cpuinfo.h`, `gtest/gtest.h`, `qnnpack/isa-checks.h`, `qnnpack/q8gemm_sparse.h`, `gemm-block-sparse-microkernel-tester.h`
- **Namespaces / 命名空间**: 无
- **Representative symbols / 代表性符号**: 无明显局部符号
