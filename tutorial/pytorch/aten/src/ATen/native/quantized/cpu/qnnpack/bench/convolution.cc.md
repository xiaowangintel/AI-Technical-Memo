# convolution.cc — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `aten/src/ATen/native/quantized/cpu/qnnpack/bench/convolution.cc`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements ATen native operators, checks, helper kernels, and backend dispatch behavior. This specific file centers on `convolution.cc`. The file header highlights: "All rights reserved. This source code is licensed under the BSD-style license found in the LICENSE file in the root directory of this source tree.." Random-number generation or reproducibility semantics are explicitly encoded.
- **Purpose (CN)**: 实现 ATen 原生算子、运行时检查、辅助内核以及后端分发行为。 该文件具体围绕 `convolution.cc` 展开。 文件头部注释也概括了其核心职责。 该文件显式编码了随机数生成或可复现性语义。

## Line-by-Line Analysis / 逐行分析
### Lines 1-33 / 第 1-33 行

```cpp
0001: /*
0002:  * Copyright (c) Facebook, Inc. and its affiliates.
0003:  * All rights reserved.
0004:  *
0005:  * This source code is licensed under the BSD-style license found in the
0006:  * LICENSE file in the root directory of this source tree.
0007:  */
0008: 
0009: #include <algorithm>
0010: #include <cfloat>
0011: #include <chrono>
0012: #include <cmath>
0013: #include <functional>
0014: #include <iostream>
0015: #include <random>
0016: #include <vector>
0017: 
0018: #include <pytorch_qnnpack.h>
0019: 
0020: #include <benchmark/benchmark.h>
0021: 
0022: static void convolution_q8(benchmark::State& state, const char* net, bool per_channel=false) {
0023:   const size_t batchSize = state.range(0);
0024:   const size_t inputHeight = state.range(1);
0025:   const size_t inputWidth = state.range(2);
0026:   const size_t kernelHeight = state.range(3);
0027:   const size_t kernelWidth = state.range(4);
0028:   const size_t subsampling = state.range(5);
0029:   const size_t dilation = state.range(6);
0030:   const size_t groups = state.range(7);
0031:   const size_t groupInputChannels = state.range(8);
0032:   const size_t groupOutputChannels = state.range(9);
0033: 
```

- **EN:** This block interacts with accelerator runtime state or GPU execution details; encodes random-number generation or reproducibility semantics. Key symbols: `convolution_q8`.
- **CN:** 该代码块与加速器运行时状态或 GPU 执行细节交互；编码随机数生成或可复现性语义。关键符号：`convolution_q8`。

### Lines 34-63 / 第 34-63 行

```cpp
0034:   std::random_device randomDevice;
0035:   auto rng = std::mt19937(randomDevice());
0036:   auto s32rng =
0037:       std::bind(std::uniform_int_distribution<int32_t>(-10000, 10000), rng);
0038:   auto u8rng = std::bind(std::uniform_int_distribution<uint8_t>(), rng);
0039: 
0040:   const size_t outputPixelStride = groups * groupOutputChannels;
0041:   const size_t inputPixelStride = groups * groupInputChannels;
0042:   const size_t effectiveKernelHeight = (kernelHeight - 1) * dilation + 1;
0043:   const size_t effectiveKernelWidth = (kernelWidth - 1) * dilation + 1;
0044:   const size_t paddingWidth = effectiveKernelWidth / 2;
0045:   const size_t paddingHeight = effectiveKernelHeight / 2;
0046:   const size_t outputHeight =
0047:       (inputHeight + paddingHeight * 2 - effectiveKernelHeight) / subsampling +
0048:       1;
0049:   const size_t outputWidth =
0050:       (inputWidth + paddingWidth * 2 - effectiveKernelWidth) / subsampling + 1;
0051: 
0052:   std::vector<uint8_t> input(
0053:       batchSize * inputHeight * inputWidth * inputPixelStride);
0054:   std::generate(input.begin(), input.end(), std::ref(u8rng));
0055:   std::vector<uint8_t> kernel(
0056:       groups * groupOutputChannels * kernelHeight * kernelWidth *
0057:       groupInputChannels);
0058:   std::generate(kernel.begin(), kernel.end(), std::ref(u8rng));
0059:   std::vector<int32_t> bias(groups * groupOutputChannels);
0060:   std::generate(bias.begin(), bias.end(), std::ref(s32rng));
0061:   std::vector<uint8_t> output(
0062:       batchSize * outputHeight * outputWidth * outputPixelStride);
0063: 
```

- **EN:** This block encodes random-number generation or reproducibility semantics. Key symbols: `bind`, `input`, `generate`, `bias`, `output`.
- **CN:** 该代码块编码随机数生成或可复现性语义。关键符号：`bind`, `input`, `generate`, `bias`, `output`。

### Lines 64-98 / 第 64-98 行

```cpp
0064:   pytorch_qnnp_status status = pytorch_qnnp_initialize();
0065:   if (status != pytorch_qnnp_status_success) {
0066:     state.SkipWithError("failed to initialize QNNPACK");
0067:   }
0068: 
0069:   pytorch_qnnp_operator_t convolutionObject = nullptr;
0070:   size_t num_zero_points_padded =
0071:     ((groups * groupOutputChannels + 7) / 8) * 8;
0072:   std::vector<uint8_t> kernel_zero_points(num_zero_points_padded, 127);
0073:   std::vector<float> requantization_scale(
0074:       num_zero_points_padded, 0.5 * 0.5 / 0.5);
0075:   status = pytorch_qnnp_create_convolution2d_nhwc_q8(
0076:       paddingHeight,
0077:       paddingWidth,
0078:       kernelHeight,
0079:       kernelWidth,
0080:       subsampling,
0081:       subsampling,
0082:       dilation,
0083:       dilation,
0084:       groups,
0085:       groupInputChannels,
0086:       groupOutputChannels,
0087:       127,
0088:       kernel_zero_points.data(),
0089:       kernel.data(),
0090:       bias.data(),
0091:       127,
0092:       0,
0093:       255,
0094:       0 /* flags */,
0095:       requantization_scale.data(),
0096:       per_channel,
0097:       &convolutionObject);
0098:   if (status != pytorch_qnnp_status_success) {
```

- **EN:** This block handles conditional branches and special cases. Key symbols: `kernel_zero_points`, `requantization_scale`.
- **CN:** 该代码块处理条件分支与特殊情况。关键符号：`kernel_zero_points`, `requantization_scale`。

### Lines 99-131 / 第 99-131 行

```cpp
0099:     state.SkipWithError("failed to create Convolution operator");
0100:   }
0101: 
0102:   status = pytorch_qnnp_setup_convolution2d_nhwc_q8(
0103:       convolutionObject,
0104:       batchSize,
0105:       inputHeight,
0106:       inputWidth,
0107:       input.data(),
0108:       inputPixelStride,
0109:       output.data(),
0110:       outputPixelStride,
0111:       nullptr /* thread pool */);
0112:   if (status != pytorch_qnnp_status_success) {
0113:     state.SkipWithError("failed to setup Convolution operator");
0114:   }
0115: 
0116:   for (auto _ : state) {
0117:     pytorch_qnnp_run_operator(convolutionObject, nullptr /* thread pool */);
0118:   }
0119: 
0120:   status = pytorch_qnnp_delete_operator(convolutionObject);
0121:   if (status != pytorch_qnnp_status_success) {
0122:     state.SkipWithError("failed to delete Convolution operator");
0123:   }
0124:   convolutionObject = nullptr;
0125: 
0126:   state.SetItemsProcessed(
0127:       uint64_t(state.iterations()) * 2 * batchSize * outputHeight *
0128:       outputWidth * groups * groupInputChannels * groupOutputChannels *
0129:       kernelHeight * kernelWidth);
0130: }
0131: 
```

- **EN:** This block handles conditional branches and special cases; iterates over tensors, descriptors, options, or runtime state. Key symbols: `pytorch_qnnp_run_operator`, `uint64_t`.
- **CN:** 该代码块处理条件分支与特殊情况；遍历张量、描述符、配置项或运行时状态。关键符号：`pytorch_qnnp_run_operator`, `uint64_t`。

### Lines 132-170 / 第 132-170 行

```cpp
0132: /* ShuffleNet v1 with 1 group */
0133: static void ShuffleNetV1G1(benchmark::internal::Benchmark* b) {
0134:   b->ArgNames({"N", "H", "W", "KH", "KW", "S", "D", "G", "GCin", "GCout"});
0135: 
0136:   /*********************** Conv 1 **********************/
0137:   /*       N   H    W   KH  KW  S  D   G   GCin  GCout */
0138:   b->Args({1, 224, 224, 3, 3, 2, 1, 1, 3, 24});
0139:   /*************** Stage 2: stride-2 unit **************/
0140:   /*       N   H    W   KH  KW  S  D   G   GCin  GCout */
0141:   b->Args({1, 56, 56, 1, 1, 1, 1, 1, 24, 36});
0142:   b->Args({1, 56, 56, 3, 3, 2, 1, 36, 1, 1});
0143:   b->Args({1, 28, 28, 1, 1, 1, 1, 1, 36, 120});
0144:   /*************** Stage 2: stride-1 units *************/
0145:   /*       N   H    W   KH  KW  S  D   G   GCin  GCout */
0146:   b->Args({1, 28, 28, 1, 1, 1, 1, 1, 144, 36});
0147:   b->Args({1, 28, 28, 3, 3, 2, 1, 36, 1, 1});
0148:   b->Args({1, 28, 28, 1, 1, 1, 1, 1, 36, 144});
0149:   /*************** Stage 3: stride-2 unit **************/
0150:   /*       N   H    W   KH  KW  S  D   G   GCin  GCout */
0151:   b->Args({1, 28, 28, 1, 1, 1, 1, 1, 144, 72});
0152:   b->Args({1, 28, 28, 3, 3, 2, 1, 72, 1, 1});
0153:   b->Args({1, 14, 14, 1, 1, 1, 1, 1, 72, 144});
0154:   /*************** Stage 3: stride-1 units *************/
0155:   /*       N   H    W   KH  KW  S  D   G   GCin  GCout */
0156:   b->Args({1, 14, 14, 1, 1, 1, 1, 1, 288, 72});
0157:   b->Args({1, 14, 14, 3, 3, 2, 1, 72, 1, 1});
0158:   b->Args({1, 14, 14, 1, 1, 1, 1, 1, 72, 288});
0159:   /*************** Stage 4: stride-2 unit **************/
0160:   /*       N   H    W   KH  KW  S  D   G   GCin  GCout */
0161:   b->Args({1, 14, 14, 1, 1, 1, 1, 1, 288, 144});
0162:   b->Args({1, 14, 14, 3, 3, 2, 1, 144, 1, 1});
0163:   b->Args({1, 7, 7, 1, 1, 1, 1, 1, 144, 288});
0164:   /*************** Stage 4: stride-1 units *************/
0165:   /*       N   H    W   KH  KW  S  D   G   GCin  GCout */
0166:   b->Args({1, 7, 7, 1, 1, 1, 1, 1, 576, 144});
0167:   b->Args({1, 7, 7, 3, 3, 2, 1, 144, 1, 1});
0168:   b->Args({1, 7, 7, 1, 1, 1, 1, 1, 144, 576});
0169: }
0170: 
```

- **EN:** This block implements local helper logic for `convolution`. Key symbols: `ShuffleNetV1G1`.
- **CN:** 该代码块实现与 `convolution` 相关的局部辅助逻辑。关键符号：`ShuffleNetV1G1`。

### Lines 171-209 / 第 171-209 行

```cpp
0171: /* ShuffleNet v1 with 2 groups */
0172: static void ShuffleNetV1G2(benchmark::internal::Benchmark* b) {
0173:   b->ArgNames({"N", "H", "W", "KH", "KW", "S", "D", "G", "GCin", "GCout"});
0174: 
0175:   /*********************** Conv 1 **********************/
0176:   /*       N   H    W   KH  KW  S  D   G   GCin  GCout */
0177:   b->Args({1, 224, 224, 3, 3, 2, 1, 1, 3, 24});
0178:   /*************** Stage 2: stride-2 unit **************/
0179:   /*       N   H    W   KH  KW  S  D   G   GCin  GCout */
0180:   b->Args({1, 56, 56, 1, 1, 1, 1, 1, 24, 50});
0181:   b->Args({1, 56, 56, 3, 3, 2, 1, 50, 1, 1});
0182:   b->Args({1, 28, 28, 1, 1, 1, 1, 2, 25, 88});
0183:   /*************** Stage 2: stride-1 units *************/
0184:   /*       N   H    W   KH  KW  S  D   G   GCin  GCout */
0185:   b->Args({1, 28, 28, 1, 1, 1, 1, 2, 100, 25});
0186:   b->Args({1, 28, 28, 3, 3, 2, 1, 50, 1, 1});
0187:   b->Args({1, 28, 28, 1, 1, 1, 1, 2, 25, 100});
0188:   /*************** Stage 3: stride-2 unit **************/
0189:   /*       N   H    W   KH  KW  S  D   G   GCin  GCout */
0190:   b->Args({1, 28, 28, 1, 1, 1, 1, 2, 100, 50});
0191:   b->Args({1, 28, 28, 3, 3, 2, 1, 100, 1, 1});
0192:   b->Args({1, 14, 14, 1, 1, 1, 1, 2, 50, 100});
0193:   /*************** Stage 3: stride-1 units *************/
0194:   /*       N   H    W   KH  KW  S  D   G   GCin  GCout */
0195:   b->Args({1, 14, 14, 1, 1, 1, 1, 2, 200, 50});
0196:   b->Args({1, 14, 14, 3, 3, 2, 1, 100, 1, 1});
0197:   b->Args({1, 14, 14, 1, 1, 1, 1, 2, 50, 200});
0198:   /*************** Stage 4: stride-2 unit **************/
0199:   /*       N   H    W   KH  KW  S  D   G   GCin  GCout */
0200:   b->Args({1, 14, 14, 1, 1, 1, 1, 2, 200, 100});
0201:   b->Args({1, 14, 14, 3, 3, 2, 1, 200, 1, 1});
0202:   b->Args({1, 7, 7, 1, 1, 1, 1, 2, 100, 200});
0203:   /*************** Stage 4: stride-1 units *************/
0204:   /*       N   H    W   KH  KW  S  D   G   GCin  GCout */
0205:   b->Args({1, 7, 7, 1, 1, 1, 1, 2, 400, 100});
0206:   b->Args({1, 7, 7, 3, 3, 2, 1, 200, 1, 1});
0207:   b->Args({1, 7, 7, 1, 1, 1, 1, 2, 100, 400});
0208: }
0209: 
```

- **EN:** This block implements local helper logic for `convolution`. Key symbols: `ShuffleNetV1G2`.
- **CN:** 该代码块实现与 `convolution` 相关的局部辅助逻辑。关键符号：`ShuffleNetV1G2`。

### Lines 210-248 / 第 210-248 行

```cpp
0210: /* ShuffleNet v1 with 3 groups */
0211: static void ShuffleNetV1G3(benchmark::internal::Benchmark* b) {
0212:   b->ArgNames({"N", "H", "W", "KH", "KW", "S", "D", "G", "GCin", "GCout"});
0213: 
0214:   /*********************** Conv 1 **********************/
0215:   /*       N   H    W   KH  KW  S  D   G   GCin  GCout */
0216:   b->Args({1, 224, 224, 3, 3, 2, 1, 1, 3, 24});
0217:   /*************** Stage 2: stride-2 unit **************/
0218:   /*       N   H    W   KH  KW  S  D   G   GCin  GCout */
0219:   b->Args({1, 56, 56, 1, 1, 1, 1, 1, 24, 60});
0220:   b->Args({1, 56, 56, 3, 3, 2, 1, 60, 1, 1});
0221:   b->Args({1, 28, 28, 1, 1, 1, 1, 3, 20, 72});
0222:   /*************** Stage 2: stride-1 units *************/
0223:   /*       N   H    W   KH  KW  S  D   G   GCin  GCout */
0224:   b->Args({1, 28, 28, 1, 1, 1, 1, 3, 80, 20});
0225:   b->Args({1, 28, 28, 3, 3, 2, 1, 60, 1, 1});
0226:   b->Args({1, 28, 28, 1, 1, 1, 1, 3, 20, 80});
0227:   /*************** Stage 3: stride-2 unit **************/
0228:   /*       N   H    W   KH  KW  S  D   G   GCin  GCout */
0229:   b->Args({1, 28, 28, 1, 1, 1, 1, 3, 80, 40});
0230:   b->Args({1, 28, 28, 3, 3, 2, 1, 120, 1, 1});
0231:   b->Args({1, 14, 14, 1, 1, 1, 1, 3, 40, 80});
0232:   /*************** Stage 3: stride-1 units *************/
0233:   /*       N   H    W   KH  KW  S  D   G   GCin  GCout */
0234:   b->Args({1, 14, 14, 1, 1, 1, 1, 3, 160, 40});
0235:   b->Args({1, 14, 14, 3, 3, 2, 1, 120, 1, 1});
0236:   b->Args({1, 14, 14, 1, 1, 1, 1, 3, 40, 160});
0237:   /*************** Stage 4: stride-2 unit **************/
0238:   /*       N   H    W   KH  KW  S  D   G   GCin  GCout */
0239:   b->Args({1, 14, 14, 1, 1, 1, 1, 3, 160, 80});
0240:   b->Args({1, 14, 14, 3, 3, 2, 1, 240, 1, 1});
0241:   b->Args({1, 7, 7, 1, 1, 1, 1, 3, 80, 160});
0242:   /*************** Stage 4: stride-1 units *************/
0243:   /*       N   H    W   KH  KW  S  D   G   GCin  GCout */
0244:   b->Args({1, 7, 7, 1, 1, 1, 1, 3, 320, 80});
0245:   b->Args({1, 7, 7, 3, 3, 2, 1, 240, 1, 1});
0246:   b->Args({1, 7, 7, 1, 1, 1, 1, 3, 80, 320});
0247: }
0248: 
```

- **EN:** This block implements local helper logic for `convolution`. Key symbols: `ShuffleNetV1G3`.
- **CN:** 该代码块实现与 `convolution` 相关的局部辅助逻辑。关键符号：`ShuffleNetV1G3`。

### Lines 249-287 / 第 249-287 行

```cpp
0249: /* ShuffleNet v1 with 4 groups */
0250: static void ShuffleNetV1G4(benchmark::internal::Benchmark* b) {
0251:   b->ArgNames({"N", "H", "W", "KH", "KW", "S", "D", "G", "GCin", "GCout"});
0252: 
0253:   /*********************** Conv 1 **********************/
0254:   /*       N   H    W   KH  KW  S  D   G   GCin  GCout */
0255:   b->Args({1, 224, 224, 3, 3, 2, 1, 1, 3, 24});
0256:   /*************** Stage 2: stride-2 unit **************/
0257:   /*       N   H    W   KH  KW  S  D   G   GCin  GCout */
0258:   b->Args({1, 56, 56, 1, 1, 1, 1, 1, 24, 68});
0259:   b->Args({1, 56, 56, 3, 3, 2, 1, 68, 1, 1});
0260:   b->Args({1, 28, 28, 1, 1, 1, 1, 4, 17, 62});
0261:   /*************** Stage 2: stride-1 units *************/
0262:   /*       N   H    W   KH  KW  S  D   G   GCin  GCout */
0263:   b->Args({1, 28, 28, 1, 1, 1, 1, 4, 68, 17});
0264:   b->Args({1, 28, 28, 3, 3, 2, 1, 68, 1, 1});
0265:   b->Args({1, 28, 28, 1, 1, 1, 1, 4, 17, 68});
0266:   /*************** Stage 3: stride-2 unit **************/
0267:   /*       N   H    W   KH  KW  S  D   G   GCin  GCout */
0268:   b->Args({1, 28, 28, 1, 1, 1, 1, 4, 68, 34});
0269:   b->Args({1, 28, 28, 3, 3, 2, 1, 136, 1, 1});
0270:   b->Args({1, 14, 14, 1, 1, 1, 1, 4, 34, 68});
0271:   /*************** Stage 3: stride-1 units *************/
0272:   /*       N   H    W   KH  KW  S  D   G   GCin  GCout */
0273:   b->Args({1, 14, 14, 1, 1, 1, 1, 4, 136, 34});
0274:   b->Args({1, 14, 14, 3, 3, 2, 1, 136, 1, 1});
0275:   b->Args({1, 14, 14, 1, 1, 1, 1, 4, 34, 136});
0276:   /*************** Stage 4: stride-2 unit **************/
0277:   /*       N   H    W   KH  KW  S  D   G   GCin  GCout */
0278:   b->Args({1, 14, 14, 1, 1, 1, 1, 4, 136, 68});
0279:   b->Args({1, 14, 14, 3, 3, 2, 1, 272, 1, 1});
0280:   b->Args({1, 7, 7, 1, 1, 1, 1, 4, 68, 136});
0281:   /*************** Stage 4: stride-1 units *************/
0282:   /*       N   H    W   KH  KW  S  D   G   GCin  GCout */
0283:   b->Args({1, 7, 7, 1, 1, 1, 1, 4, 272, 68});
0284:   b->Args({1, 7, 7, 3, 3, 2, 1, 272, 1, 1});
0285:   b->Args({1, 7, 7, 1, 1, 1, 1, 4, 68, 272});
0286: }
0287: 
```

- **EN:** This block implements local helper logic for `convolution`. Key symbols: `ShuffleNetV1G4`.
- **CN:** 该代码块实现与 `convolution` 相关的局部辅助逻辑。关键符号：`ShuffleNetV1G4`。

### Lines 288-326 / 第 288-326 行

```cpp
0288: /* ShuffleNet v1 with 8 groups */
0289: static void ShuffleNetV1G8(benchmark::internal::Benchmark* b) {
0290:   b->ArgNames({"N", "H", "W", "KH", "KW", "S", "D", "G", "GCin", "GCout"});
0291: 
0292:   /*********************** Conv 1 **********************/
0293:   /*       N   H    W   KH  KW  S  D   G   GCin  GCout */
0294:   b->Args({1, 224, 224, 3, 3, 2, 1, 1, 3, 24});
0295:   /*************** Stage 2: stride-2 unit **************/
0296:   /*       N   H    W   KH  KW  S  D   G   GCin  GCout */
0297:   b->Args({1, 56, 56, 1, 1, 1, 1, 1, 24, 96});
0298:   b->Args({1, 56, 56, 3, 3, 2, 1, 96, 1, 1});
0299:   b->Args({1, 28, 28, 1, 1, 1, 1, 8, 12, 45});
0300:   /*************** Stage 2: stride-1 units *************/
0301:   /*       N   H    W   KH  KW  S  D   G   GCin  GCout */
0302:   b->Args({1, 28, 28, 1, 1, 1, 1, 8, 48, 12});
0303:   b->Args({1, 28, 28, 3, 3, 2, 1, 96, 1, 1});
0304:   b->Args({1, 28, 28, 1, 1, 1, 1, 8, 12, 48});
0305:   /*************** Stage 3: stride-2 unit **************/
0306:   /*       N   H    W   KH  KW  S  D   G   GCin  GCout */
0307:   b->Args({1, 28, 28, 1, 1, 1, 1, 8, 48, 24});
0308:   b->Args({1, 28, 28, 3, 3, 2, 1, 192, 1, 1});
0309:   b->Args({1, 14, 14, 1, 1, 1, 1, 8, 24, 48});
0310:   /*************** Stage 3: stride-1 units *************/
0311:   /*       N   H    W   KH  KW  S  D   G   GCin  GCout */
0312:   b->Args({1, 14, 14, 1, 1, 1, 1, 8, 96, 24});
0313:   b->Args({1, 14, 14, 3, 3, 2, 1, 192, 1, 1});
0314:   b->Args({1, 14, 14, 1, 1, 1, 1, 8, 24, 96});
0315:   /*************** Stage 4: stride-2 unit **************/
0316:   /*       N   H    W   KH  KW  S  D   G   GCin  GCout */
0317:   b->Args({1, 14, 14, 1, 1, 1, 1, 8, 96, 48});
0318:   b->Args({1, 14, 14, 3, 3, 2, 1, 384, 1, 1});
0319:   b->Args({1, 7, 7, 1, 1, 1, 1, 8, 48, 96});
0320:   /*************** Stage 4: stride-1 units *************/
0321:   /*       N   H    W   KH  KW  S  D   G   GCin  GCout */
0322:   b->Args({1, 7, 7, 1, 1, 1, 1, 8, 192, 48});
0323:   b->Args({1, 7, 7, 3, 3, 2, 1, 384, 1, 1});
0324:   b->Args({1, 7, 7, 1, 1, 1, 1, 8, 48, 192});
0325: }
0326: 
```

- **EN:** This block implements local helper logic for `convolution`. Key symbols: `ShuffleNetV1G8`.
- **CN:** 该代码块实现与 `convolution` 相关的局部辅助逻辑。关键符号：`ShuffleNetV1G8`。

### Lines 327-356 / 第 327-356 行

```cpp
0327: /* ShuffleNet v2 (0.5X scale) */
0328: static void ShuffleNetV2X05(benchmark::internal::Benchmark* b) {
0329:   b->ArgNames({"N", "H", "W", "KH", "KW", "S", "D", "G", "GCin", "GCout"});
0330: 
0331:   /*********************** Conv 1 **********************/
0332:   /*       N   H    W   KH  KW  S  D   G   GCin  GCout */
0333:   b->Args({1, 224, 224, 3, 3, 2, 1, 1, 3, 24});
0334:   /********************** Stage 2 **********************/
0335:   /*       N   H    W   KH  KW  S  D   G   GCin  GCout */
0336:   b->Args({1, 56, 56, 3, 3, 2, 1, 24, 1, 1});
0337:   b->Args({1, 28, 28, 1, 1, 1, 1, 1, 24, 24});
0338:   b->Args({1, 56, 56, 1, 1, 1, 1, 1, 24, 24});
0339:   b->Args({1, 28, 28, 3, 3, 1, 1, 24, 1, 1});
0340:   /********************** Stage 3 **********************/
0341:   /*       N   H    W   KH  KW  S  D   G   GCin  GCout */
0342:   b->Args({1, 28, 28, 3, 3, 2, 1, 48, 1, 1});
0343:   b->Args({1, 14, 14, 1, 1, 1, 1, 1, 48, 48});
0344:   b->Args({1, 28, 28, 1, 1, 1, 1, 1, 48, 48});
0345:   b->Args({1, 14, 14, 3, 3, 1, 1, 48, 1, 1});
0346:   /********************** Stage 4 **********************/
0347:   /*       N   H    W   KH  KW  S  D   G   GCin  GCout */
0348:   b->Args({1, 14, 14, 3, 3, 2, 1, 96, 1, 1});
0349:   b->Args({1, 7, 7, 1, 1, 1, 1, 1, 96, 96});
0350:   b->Args({1, 14, 14, 1, 1, 1, 1, 1, 96, 96});
0351:   b->Args({1, 7, 7, 3, 3, 1, 1, 96, 1, 1});
0352:   /*********************** Conv 5 **********************/
0353:   /*       N   H    W   KH  KW  S  D   G   GCin  GCout */
0354:   b->Args({1, 7, 7, 1, 1, 1, 1, 1, 192, 1024});
0355: }
0356: 
```

- **EN:** This block implements local helper logic for `convolution`. Key symbols: `ShuffleNetV2X05`.
- **CN:** 该代码块实现与 `convolution` 相关的局部辅助逻辑。关键符号：`ShuffleNetV2X05`。

### Lines 357-388 / 第 357-388 行

```cpp
0357: /* ShuffleNet v2 (1.0X scale) */
0358: static void ShuffleNetV2X10(benchmark::internal::Benchmark* b) {
0359:   b->ArgNames({"N", "H", "W", "KH", "KW", "S", "D", "G", "GCin", "GCout"});
0360: 
0361:   /*********************** Conv 1 **********************/
0362:   /*       N   H    W   KH  KW  S  D   G   GCin  GCout */
0363:   b->Args({1, 224, 224, 3, 3, 2, 1, 1, 3, 24});
0364:   /********************** Stage 2 **********************/
0365:   /*       N   H    W   KH  KW  S  D   G   GCin  GCout */
0366:   b->Args({1, 56, 56, 3, 3, 2, 1, 24, 1, 1});
0367:   b->Args({1, 28, 28, 1, 1, 1, 1, 1, 24, 58});
0368:   b->Args({1, 56, 56, 1, 1, 1, 1, 1, 24, 58});
0369:   b->Args({1, 56, 56, 3, 3, 2, 1, 58, 1, 1});
0370:   b->Args({1, 28, 28, 1, 1, 1, 1, 1, 58, 58});
0371:   b->Args({1, 28, 28, 3, 3, 1, 1, 58, 1, 1});
0372:   /********************** Stage 3 **********************/
0373:   /*       N   H    W   KH  KW  S  D   G   GCin  GCout */
0374:   b->Args({1, 28, 28, 3, 3, 2, 1, 116, 1, 1});
0375:   b->Args({1, 14, 14, 1, 1, 1, 1, 1, 116, 116});
0376:   b->Args({1, 28, 28, 1, 1, 1, 1, 1, 116, 116});
0377:   b->Args({1, 14, 14, 3, 3, 1, 1, 116, 1, 1});
0378:   /********************** Stage 4 **********************/
0379:   /*       N   H    W   KH  KW  S  D   G   GCin  GCout */
0380:   b->Args({1, 14, 14, 3, 3, 2, 1, 232, 1, 1});
0381:   b->Args({1, 7, 7, 1, 1, 1, 1, 1, 232, 232});
0382:   b->Args({1, 14, 14, 1, 1, 1, 1, 1, 232, 232});
0383:   b->Args({1, 7, 7, 3, 3, 1, 1, 232, 1, 1});
0384:   /*********************** Conv 5 **********************/
0385:   /*       N   H    W   KH  KW  S  D   G   GCin  GCout */
0386:   b->Args({1, 7, 7, 1, 1, 1, 1, 1, 464, 1024});
0387: }
0388: 
```

- **EN:** This block implements local helper logic for `convolution`. Key symbols: `ShuffleNetV2X10`.
- **CN:** 该代码块实现与 `convolution` 相关的局部辅助逻辑。关键符号：`ShuffleNetV2X10`。

### Lines 389-420 / 第 389-420 行

```cpp
0389: /* ShuffleNet v2 (1.5X scale) */
0390: static void ShuffleNetV2X15(benchmark::internal::Benchmark* b) {
0391:   b->ArgNames({"N", "H", "W", "KH", "KW", "S", "D", "G", "GCin", "GCout"});
0392: 
0393:   /*********************** Conv 1 **********************/
0394:   /*       N   H    W   KH  KW  S  D   G   GCin  GCout */
0395:   b->Args({1, 224, 224, 3, 3, 2, 1, 1, 3, 24});
0396:   /********************** Stage 2 **********************/
0397:   /*       N   H    W   KH  KW  S  D   G   GCin  GCout */
0398:   b->Args({1, 56, 56, 3, 3, 2, 1, 24, 1, 1});
0399:   b->Args({1, 28, 28, 1, 1, 1, 1, 1, 24, 88});
0400:   b->Args({1, 56, 56, 1, 1, 1, 1, 1, 24, 88});
0401:   b->Args({1, 56, 56, 3, 3, 2, 1, 88, 1, 1});
0402:   b->Args({1, 28, 28, 1, 1, 1, 1, 1, 88, 88});
0403:   b->Args({1, 28, 28, 3, 3, 1, 1, 88, 1, 1});
0404:   /********************** Stage 3 **********************/
0405:   /*       N   H    W   KH  KW  S  D   G   GCin  GCout */
0406:   b->Args({1, 28, 28, 3, 3, 2, 1, 176, 1, 1});
0407:   b->Args({1, 14, 14, 1, 1, 1, 1, 1, 176, 176});
0408:   b->Args({1, 28, 28, 1, 1, 1, 1, 1, 176, 176});
0409:   b->Args({1, 14, 14, 3, 3, 1, 1, 176, 1, 1});
0410:   /********************** Stage 4 **********************/
0411:   /*       N   H    W   KH  KW  S  D   G   GCin  GCout */
0412:   b->Args({1, 14, 14, 3, 3, 2, 1, 352, 1, 1});
0413:   b->Args({1, 7, 7, 1, 1, 1, 1, 1, 352, 352});
0414:   b->Args({1, 14, 14, 1, 1, 1, 1, 1, 352, 352});
0415:   b->Args({1, 7, 7, 3, 3, 1, 1, 352, 1, 1});
0416:   /*********************** Conv 5 **********************/
0417:   /*       N   H    W   KH  KW  S  D   G   GCin  GCout */
0418:   b->Args({1, 7, 7, 1, 1, 1, 1, 1, 704, 1024});
0419: }
0420: 
```

- **EN:** This block implements local helper logic for `convolution`. Key symbols: `ShuffleNetV2X15`.
- **CN:** 该代码块实现与 `convolution` 相关的局部辅助逻辑。关键符号：`ShuffleNetV2X15`。

### Lines 421-452 / 第 421-452 行

```cpp
0421: /* ShuffleNet v2 (2.0X scale) */
0422: static void ShuffleNetV2X20(benchmark::internal::Benchmark* b) {
0423:   b->ArgNames({"N", "H", "W", "KH", "KW", "S", "D", "G", "GCin", "GCout"});
0424: 
0425:   /*********************** Conv 1 **********************/
0426:   /*       N   H    W   KH  KW  S  D   G   GCin  GCout */
0427:   b->Args({1, 224, 224, 3, 3, 2, 1, 1, 3, 24});
0428:   /********************** Stage 2 **********************/
0429:   /*       N   H    W   KH  KW  S  D   G   GCin  GCout */
0430:   b->Args({1, 56, 56, 3, 3, 2, 1, 24, 1, 1});
0431:   b->Args({1, 28, 28, 1, 1, 1, 1, 1, 24, 122});
0432:   b->Args({1, 56, 56, 1, 1, 1, 1, 1, 24, 122});
0433:   b->Args({1, 56, 56, 3, 3, 2, 1, 122, 1, 1});
0434:   b->Args({1, 28, 28, 1, 1, 1, 1, 1, 122, 122});
0435:   b->Args({1, 28, 28, 3, 3, 1, 1, 122, 1, 1});
0436:   /********************** Stage 3 **********************/
0437:   /*       N   H    W   KH  KW  S  D   G   GCin  GCout */
0438:   b->Args({1, 28, 28, 3, 3, 2, 1, 244, 1, 1});
0439:   b->Args({1, 14, 14, 1, 1, 1, 1, 1, 244, 244});
0440:   b->Args({1, 28, 28, 1, 1, 1, 1, 1, 244, 244});
0441:   b->Args({1, 14, 14, 3, 3, 1, 1, 244, 1, 1});
0442:   /********************** Stage 4 **********************/
0443:   /*       N   H    W   KH  KW  S  D   G   GCin  GCout */
0444:   b->Args({1, 14, 14, 3, 3, 2, 1, 488, 1, 1});
0445:   b->Args({1, 7, 7, 1, 1, 1, 1, 1, 488, 488});
0446:   b->Args({1, 14, 14, 1, 1, 1, 1, 1, 488, 488});
0447:   b->Args({1, 7, 7, 3, 3, 1, 1, 488, 1, 1});
0448:   /*********************** Conv 5 **********************/
0449:   /*       N   H    W   KH  KW  S  D   G   GCin  GCout */
0450:   b->Args({1, 7, 7, 1, 1, 1, 1, 1, 976, 2048});
0451: }
0452: 
```

- **EN:** This block implements local helper logic for `convolution`. Key symbols: `ShuffleNetV2X20`.
- **CN:** 该代码块实现与 `convolution` 相关的局部辅助逻辑。关键符号：`ShuffleNetV2X20`。

### Lines 453-483 / 第 453-483 行

```cpp
0453: static void MobileNetV1(benchmark::internal::Benchmark* b) {
0454:   b->ArgNames({"N", "H", "W", "KH", "KW", "S", "D", "G", "GCin", "GCout"});
0455: 
0456:   /*       N   H    W   KH  KW  S  D    G   GCin  GCout */
0457:   b->Args({1, 224, 224, 3, 3, 2, 1, 1, 3, 32});
0458:   b->Args({1, 112, 112, 3, 3, 1, 1, 32, 1, 1});
0459:   b->Args({1, 112, 112, 1, 1, 1, 1, 1, 32, 64});
0460:   b->Args({1, 112, 112, 3, 3, 2, 1, 64, 1, 1});
0461:   b->Args({1, 56, 56, 1, 1, 1, 1, 1, 64, 128});
0462:   b->Args({1, 56, 56, 3, 3, 1, 1, 128, 1, 1});
0463:   b->Args({1, 56, 56, 1, 1, 1, 1, 1, 128, 128});
0464:   b->Args({1, 56, 56, 3, 3, 2, 1, 128, 1, 1});
0465:   b->Args({1, 28, 28, 1, 1, 1, 1, 1, 128, 256});
0466:   b->Args({1, 28, 28, 3, 3, 1, 1, 256, 1, 1});
0467:   b->Args({1, 28, 28, 1, 1, 1, 1, 1, 256, 256});
0468:   b->Args({1, 28, 28, 3, 3, 2, 1, 256, 1, 1});
0469:   b->Args({1, 14, 14, 1, 1, 1, 1, 1, 256, 512});
0470:   b->Args({1, 14, 14, 3, 3, 1, 1, 512, 1, 1});
0471:   b->Args({1, 14, 14, 1, 1, 1, 1, 1, 512, 512});
0472:   b->Args({1, 14, 14, 3, 3, 2, 1, 512, 1, 1});
0473:   b->Args({1, 7, 7, 1, 1, 1, 1, 1, 512, 1024});
0474:   b->Args({1, 7, 7, 3, 3, 1, 1, 1024, 1, 1});
0475:   b->Args({1, 7, 7, 1, 1, 1, 1, 1, 1024, 1024});
0476: }
0477: 
0478: static void MobileNetV2(benchmark::internal::Benchmark* b) {
0479:   b->ArgNames({"N", "H", "W", "KH", "KW", "S", "D", "G", "GCin", "GCout"});
0480: 
0481:   /*       N   H    W   KH  KW  S  D    G  GCin  GCout */
0482:   b->Args({1, 224, 224, 3, 3, 2, 1, 1, 3, 32});
0483: 
```

- **EN:** This block implements local helper logic for `convolution`. Key symbols: `MobileNetV1`, `MobileNetV2`.
- **CN:** 该代码块实现与 `convolution` 相关的局部辅助逻辑。关键符号：`MobileNetV1`, `MobileNetV2`。

### Lines 484-524 / 第 484-524 行

```cpp
0484:   /******************** Bottleneck 1 *******************/
0485:   /*       N   H    W   KH  KW  S  D    G  GCin  GCout */
0486:   b->Args({1, 112, 112, 3, 3, 1, 1, 32, 1, 1});
0487:   b->Args({1, 112, 112, 1, 1, 1, 1, 1, 32, 16});
0488: 
0489:   /******************** Bottleneck 2 *******************/
0490:   /*       N   H    W   KH  KW  S  D    G  GCin  GCout */
0491:   b->Args({1, 112, 112, 1, 1, 1, 1, 1, 16, 96});
0492:   b->Args({1, 112, 112, 3, 3, 2, 1, 96, 1, 1});
0493:   b->Args({1, 56, 56, 1, 1, 1, 1, 1, 96, 24});
0494:   b->Args({1, 56, 56, 1, 1, 1, 1, 1, 24, 144});
0495:   b->Args({1, 56, 56, 3, 3, 1, 1, 144, 1, 1});
0496:   b->Args({1, 56, 56, 1, 1, 1, 1, 1, 144, 24});
0497: 
0498:   /******************** Bottleneck 3 *******************/
0499:   /*       N   H    W   KH  KW  S  D    G  GCin  GCout */
0500:   // b->Args({1,  56,  56,  1,  1, 1, 1,   1,   24,  144});
0501:   b->Args({1, 56, 56, 3, 3, 2, 1, 144, 1, 1});
0502:   b->Args({1, 28, 28, 1, 1, 1, 1, 1, 144, 32});
0503:   b->Args({1, 28, 28, 1, 1, 1, 1, 1, 32, 192});
0504:   b->Args({1, 28, 28, 3, 3, 1, 1, 192, 1, 1});
0505:   b->Args({1, 28, 28, 1, 1, 1, 1, 1, 192, 32});
0506:   // b->Args({1,  28,  28,  1,  1, 1, 1,   1,   32,  192});
0507:   // b->Args({1,  28,  28,  3,  3, 1, 1, 192,    1,    1});
0508:   // b->Args({1,  28,  28,  1,  1, 1, 1,   1,  192,   32});
0509: 
0510:   /******************** Bottleneck 4 *******************/
0511:   /*       N   H    W   KH  KW  S  D    G  GCin  GCout */
0512:   // b->Args({1,  28,  28,  1,  1, 1, 1,   1,   32,  192});
0513:   b->Args({1, 28, 28, 3, 3, 2, 1, 192, 1, 1});
0514:   b->Args({1, 14, 14, 1, 1, 1, 1, 1, 192, 64});
0515:   b->Args({1, 14, 14, 1, 1, 1, 1, 1, 64, 384});
0516:   b->Args({1, 14, 14, 3, 3, 1, 1, 384, 1, 1});
0517:   b->Args({1, 14, 14, 1, 1, 1, 1, 1, 384, 64});
0518:   // b->Args({1,  14,  14,  1,  1, 1, 1,   1,   64,  384});
0519:   // b->Args({1,  14,  14,  3,  3, 1, 1, 384,    1,    1});
0520:   // b->Args({1,  14,  14,  1,  1, 1, 1,   1,  384,   64});
0521:   // b->Args({1,  14,  14,  1,  1, 1, 1,   1,   64,  384});
0522:   // b->Args({1,  14,  14,  3,  3, 1, 1, 384,    1,    1});
0523:   // b->Args({1,  14,  14,  1,  1, 1, 1,   1,  384,   64});
0524: 
```

- **EN:** This block implements local helper logic for `convolution`. Key symbols: no prominent local symbols.
- **CN:** 该代码块实现与 `convolution` 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 525-554 / 第 525-554 行

```cpp
0525:   /******************** Bottleneck 5 *******************/
0526:   /*       N   H    W   KH  KW  S  D    G  GCin  GCout */
0527:   // b->Args({1,  14,  14,  1,  1, 1, 1,   1,   64,  384});
0528:   // b->Args({1,  14,  14,  3,  3, 1, 1, 384,    1,    1});
0529:   b->Args({1, 14, 14, 1, 1, 1, 1, 1, 384, 96});
0530:   b->Args({1, 14, 14, 1, 1, 1, 1, 1, 96, 576});
0531:   b->Args({1, 14, 14, 3, 3, 1, 1, 576, 1, 1});
0532:   b->Args({1, 14, 14, 1, 1, 1, 1, 1, 576, 96});
0533:   // b->Args({1,  14,  14,  1,  1, 1, 1,   1,   96,  576});
0534:   // b->Args({1,  14,  14,  3,  3, 1, 1, 576,    1,    1});
0535:   // b->Args({1,  14,  14,  1,  1, 1, 1,   1,  576,   96});
0536: 
0537:   /******************** Bottleneck 6 *******************/
0538:   /*       N   H    W   KH  KW  S  D    G  GCin  GCout */
0539:   // b->Args({1,  14,  14,  1,  1, 1, 1,   1,   96,  576});
0540:   b->Args({1, 14, 14, 3, 3, 2, 1, 576, 1, 1});
0541:   b->Args({1, 7, 7, 1, 1, 1, 1, 1, 576, 160});
0542:   b->Args({1, 7, 7, 1, 1, 1, 1, 1, 160, 960});
0543:   b->Args({1, 7, 7, 3, 3, 1, 1, 960, 1, 1});
0544:   b->Args({1, 7, 7, 1, 1, 1, 1, 1, 960, 160});
0545:   // b->Args({1,   7,   7,  1,  1, 1, 1,   1,  160,  960});
0546:   // b->Args({1,   7,   7,  3,  3, 1, 1, 960,    1,    1});
0547:   // b->Args({1,   7,   7,  1,  1, 1, 1,   1,  960,  160});
0548: 
0549:   /******************** Bottleneck 7 *******************/
0550:   /*       N   H    W   KH  KW  S  D    G  GCin  GCout */
0551:   // b->Args({1,   7,   7,  1,  1, 1, 1,   1,  160,  960});
0552:   // b->Args({1,   7,   7,  3,  3, 1, 1, 960,    1,    1});
0553:   b->Args({1, 7, 7, 1, 1, 1, 1, 1, 960, 320});
0554: 
```

- **EN:** This block implements local helper logic for `convolution`. Key symbols: no prominent local symbols.
- **CN:** 该代码块实现与 `convolution` 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 555-614 / 第 555-614 行

```cpp
0555:   /**************** Pre-pooling Conv2D *****************/
0556:   /*       N   H    W   KH  KW  S  D    G  GCin  GCout */
0557:   b->Args({1, 7, 7, 1, 1, 1, 1, 1, 320, 1280});
0558:   /**************** Post-pooling Conv2D ****************/
0559:   /*       N   H    W   KH  KW  S  D    G  GCin  GCout */
0560:   b->Args({1, 1, 1, 1, 1, 1, 1, 1, 1280, 1000});
0561: }
0562: 
0563: /* SqueezeNet 1.0 */
0564: static void SqueezeNetV10(benchmark::internal::Benchmark* b) {
0565:   b->ArgNames({"N", "H", "W", "KH", "KW", "S", "D", "G", "GCin", "GCout"});
0566: 
0567:   /********************** Conv 1 *********************/
0568:   /*       N   H    W   KH  KW  S  D  G  GCin  GCout */
0569:   b->Args({1, 224, 224, 7, 7, 2, 1, 1, 3, 96});
0570:   /********************** Fire 2 *********************/
0571:   /*       N   H    W   KH  KW  S  D  G  GCin  GCout */
0572:   b->Args({1, 55, 55, 1, 1, 1, 1, 1, 96, 16});
0573:   b->Args({1, 55, 55, 1, 1, 1, 1, 1, 16, 64});
0574:   b->Args({1, 55, 55, 3, 3, 1, 1, 1, 16, 64});
0575:   /********************** Fire 3 *********************/
0576:   /*       N   H    W   KH  KW  S  D  G  GCin  GCout */
0577:   b->Args({1, 56, 55, 1, 1, 1, 1, 1, 128, 16});
0578:   /*b->Args({1,  55,  55,  1,  1, 1, 1, 1,   16,   64});*/
0579:   /*b->Args({1,  55,  55,  3,  3, 1, 1, 1,   16,   64});*/
0580:   /********************** Fire 4 *********************/
0581:   /*       N   H    W   KH  KW  S  D  G  GCin  GCout */
0582:   b->Args({1, 55, 55, 1, 1, 1, 1, 1, 128, 32});
0583:   b->Args({1, 55, 55, 1, 1, 1, 1, 1, 32, 128});
0584:   b->Args({1, 55, 55, 3, 3, 1, 1, 1, 32, 128});
0585:   /********************** Fire 5 *********************/
0586:   /*       N   H    W   KH  KW  S  D  G  GCin  GCout */
0587:   b->Args({1, 27, 27, 1, 1, 1, 1, 1, 256, 32});
0588:   b->Args({1, 27, 27, 1, 1, 1, 1, 1, 32, 128});
0589:   b->Args({1, 27, 27, 3, 3, 1, 1, 1, 32, 128});
0590:   /********************** Fire 6 *********************/
0591:   /*       N   H    W   KH  KW  S  D  G  GCin  GCout */
0592:   b->Args({1, 27, 27, 1, 1, 1, 1, 1, 256, 48});
0593:   b->Args({1, 27, 27, 1, 1, 1, 1, 1, 48, 192});
0594:   b->Args({1, 27, 27, 3, 3, 1, 1, 1, 48, 192});
0595:   /********************** Fire 7 *********************/
0596:   /*       N   H    W   KH  KW  S  D  G  GCin  GCout */
0597:   b->Args({1, 27, 27, 1, 1, 1, 1, 1, 384, 48});
0598:   /*b->Args({1,  27,  27,  1,  1, 1, 1, 1,   48,  192});*/
0599:   /*b->Args({1,  27,  27,  3,  3, 1, 1, 1,   48,  192});*/
0600:   /********************** Fire 8 *********************/
0601:   /*       N   H    W   KH  KW  S  D  G  GCin  GCout */
0602:   b->Args({1, 27, 27, 1, 1, 1, 1, 1, 384, 64});
0603:   b->Args({1, 27, 27, 1, 1, 1, 1, 1, 64, 256});
0604:   b->Args({1, 27, 27, 3, 3, 1, 1, 1, 64, 256});
0605:   /********************** Fire 9 *********************/
0606:   /*       N   H    W   KH  KW  S  D  G  GCin  GCout */
0607:   b->Args({1, 13, 13, 1, 1, 1, 1, 1, 512, 64});
0608:   b->Args({1, 13, 13, 1, 1, 1, 1, 1, 64, 256});
0609:   b->Args({1, 13, 13, 3, 3, 1, 1, 1, 64, 256});
0610:   /********************* Conv 10 *********************/
0611:   /*       N   H    W   KH  KW  S  D  G  GCin  GCout */
0612:   b->Args({1, 13, 13, 1, 1, 1, 1, 1, 512, 1000});
0613: }
0614: 
```

- **EN:** This block implements local helper logic for `convolution`. Key symbols: `SqueezeNetV10`.
- **CN:** 该代码块实现与 `convolution` 相关的局部辅助逻辑。关键符号：`SqueezeNetV10`。

### Lines 615-666 / 第 615-666 行

```cpp
0615: /* SqueezeNet 1.1 */
0616: static void SqueezeNetV11(benchmark::internal::Benchmark* b) {
0617:   b->ArgNames({"N", "H", "W", "KH", "KW", "S", "D", "G", "GCin", "GCout"});
0618: 
0619:   /********************** Conv 1 *********************/
0620:   /*       N   H    W   KH  KW  S  D  G  GCin  GCout */
0621:   b->Args({1, 224, 224, 3, 3, 2, 1, 1, 3, 64});
0622:   /********************** Fire 2 *********************/
0623:   /*       N   H    W   KH  KW  S  D  G  GCin  GCout */
0624:   b->Args({1, 55, 55, 1, 1, 1, 1, 1, 64, 16});
0625:   b->Args({1, 55, 55, 1, 1, 1, 1, 1, 16, 64});
0626:   b->Args({1, 55, 55, 3, 3, 1, 1, 1, 16, 64});
0627:   /********************** Fire 3 *********************/
0628:   /*       N   H    W   KH  KW  S  D  G  GCin  GCout */
0629:   b->Args({1, 55, 55, 1, 1, 1, 1, 1, 128, 16});
0630:   /*b->Args({1,  55,  55,  1,  1, 1, 1, 1,   16,   64});*/
0631:   /*b->Args({1,  55,  55,  3,  3, 1, 1, 1,   16,   64});*/
0632:   /********************** Fire 4 *********************/
0633:   /*       N   H    W   KH  KW  S  D  G  GCin  GCout */
0634:   b->Args({1, 27, 27, 1, 1, 1, 1, 1, 128, 32});
0635:   b->Args({1, 27, 27, 1, 1, 1, 1, 1, 32, 128});
0636:   b->Args({1, 27, 27, 3, 3, 1, 1, 1, 32, 128});
0637:   /********************** Fire 5 *********************/
0638:   /*       N   H    W   KH  KW  S  D  G  GCin  GCout */
0639:   b->Args({1, 27, 27, 1, 1, 1, 1, 1, 256, 32});
0640:   /*b->Args({1,  27,  27,  1,  1, 1, 1, 1,   32,  128});*/
0641:   /*b->Args({1,  27,  27,  3,  3, 1, 1, 1,   32,  128});*/
0642:   /********************** Fire 6 *********************/
0643:   /*       N   H    W   KH  KW  S  D  G  GCin  GCout */
0644:   b->Args({1, 13, 13, 1, 1, 1, 1, 1, 256, 48});
0645:   b->Args({1, 13, 13, 1, 1, 1, 1, 1, 48, 192});
0646:   b->Args({1, 13, 13, 3, 3, 1, 1, 1, 48, 192});
0647:   /********************** Fire 7 *********************/
0648:   /*       N   H    W   KH  KW  S  D  G  GCin  GCout */
0649:   b->Args({1, 13, 13, 1, 1, 1, 1, 1, 384, 48});
0650:   /*b->Args({1,  13,  13,  1,  1, 1, 1, 1,   48,  192});*/
0651:   /*b->Args({1,  13,  13,  3,  3, 1, 1, 1,   48,  192});*/
0652:   /********************** Fire 8 *********************/
0653:   /*       N   H    W   KH  KW  S  D  G  GCin  GCout */
0654:   b->Args({1, 13, 13, 1, 1, 1, 1, 1, 384, 64});
0655:   b->Args({1, 13, 13, 1, 1, 1, 1, 1, 64, 256});
0656:   b->Args({1, 13, 13, 3, 3, 1, 1, 1, 64, 256});
0657:   /********************** Fire 9 *********************/
0658:   /*       N   H    W   KH  KW  S  D  G  GCin  GCout */
0659:   b->Args({1, 13, 13, 1, 1, 1, 1, 1, 512, 64});
0660:   /*b->Args({1,  13,  13,  1,  1, 1, 1, 1,   64,  256});*/
0661:   /*b->Args({1,  13,  13,  3,  3, 1, 1, 1,   64,  256});*/
0662:   /********************* Conv 10 *********************/
0663:   /*       N   H    W   KH  KW  S  D  G  GCin  GCout */
0664:   b->Args({1, 13, 13, 1, 1, 1, 1, 1, 512, 1000});
0665: }
0666: 
```

- **EN:** This block implements local helper logic for `convolution`. Key symbols: `SqueezeNetV11`.
- **CN:** 该代码块实现与 `convolution` 相关的局部辅助逻辑。关键符号：`SqueezeNetV11`。

### Lines 667-726 / 第 667-726 行

```cpp
0667: static void ResNet18(benchmark::internal::Benchmark* b) {
0668:   b->ArgNames({"N", "H", "W", "KH", "KW", "S", "D", "G", "GCin", "GCout"});
0669: 
0670:   /********************* Conv 1 *********************/
0671:   /*       N   H    W   KH  KW  S  D  G GCin  GCout */
0672:   b->Args({1, 224, 224, 7, 7, 2, 1, 1, 3, 64});
0673:   /******************** Conv 2.X ********************/
0674:   /*       N   H    W   KH  KW  S  D  G GCin  GCout */
0675:   b->Args({1, 56, 56, 3, 3, 1, 1, 1, 64, 64});
0676:   /******************** Conv 3.X ********************/
0677:   /*       N   H    W   KH  KW  S  D  G GCin  GCout */
0678:   b->Args({1, 56, 56, 3, 3, 2, 1, 1, 64, 128});
0679:   b->Args({1, 28, 28, 3, 3, 1, 1, 1, 128, 128});
0680:   b->Args({1, 56, 56, 1, 1, 2, 1, 1, 64, 128});
0681:   /******************** Conv 4.X ********************/
0682:   /*       N   H    W   KH  KW  S  D  G GCin  GCout */
0683:   b->Args({1, 28, 28, 3, 3, 2, 1, 1, 128, 256});
0684:   b->Args({1, 14, 14, 3, 3, 1, 1, 1, 256, 256});
0685:   b->Args({1, 28, 28, 1, 1, 2, 1, 1, 128, 256});
0686:   /******************** Conv 5.X ********************/
0687:   /*       N   H    W   KH  KW  S  D  G GCin  GCout */
0688:   b->Args({1, 14, 14, 3, 3, 2, 1, 1, 256, 512});
0689:   b->Args({1, 7, 7, 3, 3, 1, 1, 1, 512, 512});
0690:   b->Args({1, 14, 14, 1, 1, 2, 1, 1, 256, 512});
0691: }
0692: 
0693: static void ResNet50(benchmark::internal::Benchmark* b) {
0694:   b->ArgNames({"N", "H", "W", "KH", "KW", "S", "D", "G", "GCin", "GCout"});
0695: 
0696:   /********************* Conv 1 *********************/
0697:   /*       N   H    W   KH  KW  S  D  G GCin  GCout */
0698:   b->Args({1, 224, 224, 7, 7, 2, 1, 1, 3, 64});
0699:   /******************** Conv 2.1 ********************/
0700:   /*       N   H    W   KH  KW  S  D  G GCin  GCout */
0701:   b->Args({1, 56, 56, 1, 1, 1, 1, 1, 64, 64});
0702:   b->Args({1, 56, 56, 3, 3, 1, 1, 1, 64, 64});
0703:   b->Args({1, 56, 56, 1, 1, 1, 1, 1, 64, 256});
0704:   /*b->Args({1,  56,  56,  1,  1, 1, 1, 1,   64,  256});*/
0705:   /******************** Conv 2.X ********************/
0706:   /*       N   H    W   KH  KW  S  D  G GCin  GCout */
0707:   b->Args({1, 56, 56, 1, 1, 1, 1, 1, 256, 64});
0708:   /*b->Args({1,  56,  56,  3,  3, 1, 1, 1,   64,   64});*/
0709:   /*b->Args({1,  56,  56,  1,  1, 1, 1, 1,   64,  256});*/
0710:   /******************** Conv 3.1 ********************/
0711:   /*       N   H    W   KH  KW  S  D  G GCin  GCout */
0712:   b->Args({1, 56, 56, 1, 1, 1, 1, 1, 256, 128});
0713:   b->Args({1, 56, 56, 3, 3, 2, 1, 1, 128, 128});
0714:   b->Args({1, 28, 28, 1, 1, 1, 1, 1, 128, 512});
0715:   b->Args({1, 56, 56, 1, 1, 2, 1, 1, 256, 512});
0716:   /******************** Conv 3.X ********************/
0717:   /*       N   H    W   KH  KW  S  D  G GCin  GCout */
0718:   b->Args({1, 28, 28, 1, 1, 1, 1, 1, 512, 128});
0719:   b->Args({1, 28, 28, 3, 3, 1, 1, 1, 128, 128});
0720:   /*b->Args({1,  28,  28,  1,  1, 1, 1, 1,  128,  512});*/
0721:   /******************** Conv 4.1 ********************/
0722:   /*       N   H    W   KH  KW  S  D  G GCin  GCout */
0723:   b->Args({1, 28, 28, 1, 1, 1, 1, 1, 512, 256});
0724:   b->Args({1, 28, 28, 3, 3, 2, 1, 1, 256, 256});
0725:   b->Args({1, 14, 14, 1, 1, 1, 1, 1, 256, 1024});
0726:   b->Args({1, 28, 28, 1, 1, 2, 1, 1, 512, 1024});
```

- **EN:** This block implements local helper logic for `convolution`. Key symbols: `ResNet18`, `ResNet50`.
- **CN:** 该代码块实现与 `convolution` 相关的局部辅助逻辑。关键符号：`ResNet18`, `ResNet50`。

### Lines 727-761 / 第 727-761 行

```cpp
0727:   /******************** Conv 4.X ********************/
0728:   /*       N   H    W   KH  KW  S  D  G GCin  GCout */
0729:   b->Args({1, 14, 14, 1, 1, 1, 1, 1, 1024, 256});
0730:   b->Args({1, 14, 14, 3, 3, 1, 1, 1, 256, 256});
0731:   /*b->Args({1,  14,  14,  1,  1, 1, 1, 1,  256, 1024});*/
0732:   /******************** Conv 5.1 ********************/
0733:   /*       N   H    W   KH  KW  S  D  G GCin  GCout */
0734:   b->Args({1, 14, 14, 1, 1, 1, 1, 1, 1024, 512});
0735:   b->Args({1, 14, 14, 3, 3, 2, 1, 1, 512, 512});
0736:   b->Args({1, 7, 7, 1, 1, 1, 1, 1, 512, 2048});
0737:   b->Args({1, 14, 14, 1, 1, 2, 1, 1, 1024, 2048});
0738:   /******************** Conv 5.X ********************/
0739:   /*       N   H    W   KH  KW  S  D  G GCin  GCout */
0740:   b->Args({1, 7, 7, 1, 1, 1, 1, 1, 2048, 512});
0741:   b->Args({1, 7, 7, 3, 3, 1, 1, 1, 512, 512});
0742:   /*b->Args({1,   7,   7,  1,  1, 1, 1, 1,  512, 2048});*/
0743: }
0744: 
0745: static void VGG(benchmark::internal::Benchmark* b) {
0746:   b->ArgNames({"N", "H", "W", "KH", "KW", "S", "D", "G", "GCin", "GCout"});
0747: 
0748:   /********************* Conv 1.1 ********************/
0749:   /*       N   H    W   KH  KW  S  D  G  GCin  GCout */
0750:   b->Args({1, 224, 224, 3, 3, 1, 1, 1, 3, 64});
0751:   /********************* Conv 1.2 ********************/
0752:   /*       N   H    W   KH  KW  S  D  G  GCin  GCout */
0753:   b->Args({1, 224, 224, 3, 3, 1, 1, 1, 64, 64});
0754: 
0755:   /********************* Conv 2.1 ********************/
0756:   /*       N   H    W   KH  KW  S  D  G  GCin  GCout */
0757:   b->Args({1, 112, 112, 3, 3, 1, 1, 1, 64, 128});
0758:   /********************* Conv 2.2 ********************/
0759:   /*       N   H    W   KH  KW  S  D  G  GCin  GCout */
0760:   b->Args({1, 112, 112, 3, 3, 1, 1, 1, 128, 128});
0761: 
```

- **EN:** This block implements local helper logic for `convolution`. Key symbols: no prominent local symbols.
- **CN:** 该代码块实现与 `convolution` 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 762-792 / 第 762-792 行

```cpp
0762:   /********************* Conv 3.1 ********************/
0763:   /*       N   H    W   KH  KW  S  D  G  GCin  GCout */
0764:   b->Args({1, 56, 56, 3, 3, 1, 1, 1, 128, 256});
0765:   /********************* Conv 3.2 ********************/
0766:   /*       N   H    W   KH  KW  S  D  G  GCin  GCout */
0767:   b->Args({1, 56, 56, 3, 3, 1, 1, 1, 256, 256});
0768:   /********************* Conv 3.3 ********************/
0769:   /*       N   H    W   KH  KW  S  D  G  GCin  GCout */
0770:   b->Args({1, 56, 56, 1, 1, 1, 1, 1, 256, 256});
0771: 
0772:   /********************* Conv 4.1 ********************/
0773:   /*       N   H    W   KH  KW  S  D  G  GCin  GCout */
0774:   b->Args({1, 28, 28, 3, 3, 1, 1, 1, 256, 512});
0775:   /********************* Conv 4.2 ********************/
0776:   /*       N   H    W   KH  KW  S  D  G  GCin  GCout */
0777:   b->Args({1, 28, 28, 3, 3, 1, 1, 1, 512, 512});
0778:   /********************* Conv 4.3 ********************/
0779:   /*       N   H    W   KH  KW  S  D  G  GCin  GCout */
0780:   b->Args({1, 28, 28, 1, 1, 1, 1, 1, 512, 512});
0781: 
0782:   /********************* Conv 5.X ********************/
0783:   /*       N   H    W   KH  KW  S  D  G  GCin  GCout */
0784:   b->Args({1, 14, 14, 3, 3, 1, 1, 1, 512, 512});
0785:   /********************* Conv 5.3 ********************/
0786:   /*       N   H    W   KH  KW  S  D  G  GCin  GCout */
0787:   b->Args({1, 14, 14, 1, 1, 1, 1, 1, 512, 512});
0788: }
0789: 
0790: static void DWConv3x3(benchmark::internal::Benchmark* b) {
0791:   b->ArgNames({"N", "H", "W", "KH", "KW", "S", "D", "G", "GCin", "GCout"});
0792: 
```

- **EN:** This block implements local helper logic for `convolution`. Key symbols: `DWConv3x3`.
- **CN:** 该代码块实现与 `convolution` 相关的局部辅助逻辑。关键符号：`DWConv3x3`。

### Lines 793-849 / 第 793-849 行

```cpp
0793:   /********************** 96 x 96 *********************/
0794:   /*       N   H   W  KH  KW  S  D    G   GCin  GCout */
0795:   b->Args({1, 96, 96, 3, 3, 1, 1, 512, 1, 1});
0796:   b->Args({1, 96, 96, 3, 3, 1, 1, 256, 1, 1});
0797:   b->Args({1, 96, 96, 3, 3, 1, 1, 128, 1, 1});
0798:   b->Args({1, 96, 96, 3, 3, 1, 1, 64, 1, 1});
0799:   b->Args({1, 96, 96, 3, 3, 1, 1, 48, 1, 1});
0800:   b->Args({1, 96, 96, 3, 3, 1, 1, 32, 1, 1});
0801:   b->Args({1, 96, 96, 3, 3, 1, 1, 24, 1, 1});
0802:   b->Args({1, 96, 96, 3, 3, 1, 1, 16, 1, 1});
0803:   /********************** 32 x 32 *********************/
0804:   /*       N   H   W  KH  KW  S  D    G   GCin  GCout */
0805:   b->Args({1, 32, 32, 3, 3, 1, 1, 768, 1, 1});
0806:   b->Args({1, 32, 32, 3, 3, 1, 1, 512, 1, 1});
0807:   b->Args({1, 32, 32, 3, 3, 1, 1, 256, 1, 1});
0808:   b->Args({1, 32, 32, 3, 3, 1, 1, 128, 1, 1});
0809:   b->Args({1, 32, 32, 3, 3, 1, 1, 64, 1, 1});
0810:   b->Args({1, 32, 32, 3, 3, 1, 1, 48, 1, 1});
0811:   b->Args({1, 32, 32, 3, 3, 1, 1, 32, 1, 1});
0812:   b->Args({1, 32, 32, 3, 3, 1, 1, 24, 1, 1});
0813:   b->Args({1, 32, 32, 3, 3, 1, 1, 16, 1, 1});
0814:   /********************** 17 x 17 *********************/
0815:   /*       N   H   W  KH  KW  S  D    G   GCin  GCout */
0816:   b->Args({1, 17, 17, 3, 3, 1, 1, 1024, 1, 1});
0817:   b->Args({1, 17, 17, 3, 3, 1, 1, 768, 1, 1});
0818:   b->Args({1, 17, 17, 3, 3, 1, 1, 512, 1, 1});
0819:   b->Args({1, 17, 17, 3, 3, 1, 1, 384, 1, 1});
0820:   b->Args({1, 17, 17, 3, 3, 1, 1, 256, 1, 1});
0821:   b->Args({1, 17, 17, 3, 3, 1, 1, 128, 1, 1});
0822:   b->Args({1, 17, 17, 3, 3, 1, 1, 64, 1, 1});
0823:   b->Args({1, 17, 17, 3, 3, 1, 1, 32, 1, 1});
0824:   b->Args({1, 17, 17, 3, 3, 1, 1, 16, 1, 1});
0825:   /********************** 11 x 11 *********************/
0826:   /*       N   H   W  KH  KW  S  D    G   GCin  GCout */
0827:   b->Args({1, 11, 11, 3, 3, 1, 1, 1024, 1, 1});
0828:   b->Args({1, 11, 11, 3, 3, 1, 1, 768, 1, 1});
0829:   b->Args({1, 11, 11, 3, 3, 1, 1, 512, 1, 1});
0830:   b->Args({1, 11, 11, 3, 3, 1, 1, 384, 1, 1});
0831:   b->Args({1, 11, 11, 3, 3, 1, 1, 256, 1, 1});
0832:   b->Args({1, 11, 11, 3, 3, 1, 1, 192, 1, 1});
0833:   b->Args({1, 11, 11, 3, 3, 1, 1, 128, 1, 1});
0834:   b->Args({1, 11, 11, 3, 3, 1, 1, 64, 1, 1});
0835:   b->Args({1, 11, 11, 3, 3, 1, 1, 32, 1, 1});
0836:   b->Args({1, 11, 11, 3, 3, 1, 1, 16, 1, 1});
0837:   /*********************** 7 x 7 **********************/
0838:   /*       N   H   W  KH  KW  S  D    G   GCin  GCout */
0839:   b->Args({1, 7, 7, 3, 3, 1, 1, 1024, 1, 1});
0840:   b->Args({1, 7, 7, 3, 3, 1, 1, 768, 1, 1});
0841:   b->Args({1, 7, 7, 3, 3, 1, 1, 512, 1, 1});
0842:   b->Args({1, 7, 7, 3, 3, 1, 1, 384, 1, 1});
0843:   b->Args({1, 7, 7, 3, 3, 1, 1, 256, 1, 1});
0844:   b->Args({1, 7, 7, 3, 3, 1, 1, 128, 1, 1});
0845:   b->Args({1, 7, 7, 3, 3, 1, 1, 64, 1, 1});
0846:   b->Args({1, 7, 7, 3, 3, 1, 1, 32, 1, 1});
0847:   b->Args({1, 7, 7, 3, 3, 1, 1, 16, 1, 1});
0848: }
0849: 
```

- **EN:** This block implements local helper logic for `convolution`. Key symbols: no prominent local symbols.
- **CN:** 该代码块实现与 `convolution` 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 850-909 / 第 850-909 行

```cpp
0850: static void DWConv3x3d2(benchmark::internal::Benchmark* b) {
0851:   b->ArgNames({"N", "H", "W", "KH", "KW", "S", "D", "G", "GCin", "GCout"});
0852: 
0853:   /********************** 96 x 96 *********************/
0854:   /*       N   H   W  KH  KW  S  D    G   GCin  GCout */
0855:   b->Args({1, 96, 96, 3, 3, 1, 2, 512, 1, 1});
0856:   b->Args({1, 96, 96, 3, 3, 1, 2, 256, 1, 1});
0857:   b->Args({1, 96, 96, 3, 3, 1, 2, 128, 1, 1});
0858:   b->Args({1, 96, 96, 3, 3, 1, 2, 64, 1, 1});
0859:   b->Args({1, 96, 96, 3, 3, 1, 2, 48, 1, 1});
0860:   b->Args({1, 96, 96, 3, 3, 1, 2, 32, 1, 1});
0861:   b->Args({1, 96, 96, 3, 3, 1, 2, 24, 1, 1});
0862:   b->Args({1, 96, 96, 3, 3, 1, 2, 16, 1, 1});
0863:   /********************** 32 x 32 *********************/
0864:   /*       N   H   W  KH  KW  S  D    G   GCin  GCout */
0865:   b->Args({1, 32, 32, 3, 3, 1, 2, 768, 1, 1});
0866:   b->Args({1, 32, 32, 3, 3, 1, 2, 512, 1, 1});
0867:   b->Args({1, 32, 32, 3, 3, 1, 2, 256, 1, 1});
0868:   b->Args({1, 32, 32, 3, 3, 1, 2, 128, 1, 1});
0869:   b->Args({1, 32, 32, 3, 3, 1, 2, 64, 1, 1});
0870:   b->Args({1, 32, 32, 3, 3, 1, 2, 48, 1, 1});
0871:   b->Args({1, 32, 32, 3, 3, 1, 2, 32, 1, 1});
0872:   b->Args({1, 32, 32, 3, 3, 1, 2, 24, 1, 1});
0873:   b->Args({1, 32, 32, 3, 3, 1, 2, 16, 1, 1});
0874:   /********************** 17 x 17 *********************/
0875:   /*       N   H   W  KH  KW  S  D    G   GCin  GCout */
0876:   b->Args({1, 17, 17, 3, 3, 1, 2, 1024, 1, 1});
0877:   b->Args({1, 17, 17, 3, 3, 1, 2, 768, 1, 1});
0878:   b->Args({1, 17, 17, 3, 3, 1, 2, 512, 1, 1});
0879:   b->Args({1, 17, 17, 3, 3, 1, 2, 384, 1, 1});
0880:   b->Args({1, 17, 17, 3, 3, 1, 2, 256, 1, 1});
0881:   b->Args({1, 17, 17, 3, 3, 1, 2, 128, 1, 1});
0882:   b->Args({1, 17, 17, 3, 3, 1, 2, 64, 1, 1});
0883:   b->Args({1, 17, 17, 3, 3, 1, 2, 32, 1, 1});
0884:   b->Args({1, 17, 17, 3, 3, 1, 2, 16, 1, 1});
0885:   /********************** 11 x 11 *********************/
0886:   /*       N   H   W  KH  KW  S  D    G   GCin  GCout */
0887:   b->Args({1, 11, 11, 3, 3, 1, 2, 1024, 1, 1});
0888:   b->Args({1, 11, 11, 3, 3, 1, 2, 768, 1, 1});
0889:   b->Args({1, 11, 11, 3, 3, 1, 2, 512, 1, 1});
0890:   b->Args({1, 11, 11, 3, 3, 1, 2, 384, 1, 1});
0891:   b->Args({1, 11, 11, 3, 3, 1, 2, 256, 1, 1});
0892:   b->Args({1, 11, 11, 3, 3, 1, 2, 192, 1, 1});
0893:   b->Args({1, 11, 11, 3, 3, 1, 2, 128, 1, 1});
0894:   b->Args({1, 11, 11, 3, 3, 1, 2, 64, 1, 1});
0895:   b->Args({1, 11, 11, 3, 3, 1, 2, 32, 1, 1});
0896:   b->Args({1, 11, 11, 3, 3, 1, 2, 16, 1, 1});
0897:   /*********************** 7 x 7 **********************/
0898:   /*       N   H   W  KH  KW  S  D    G   GCin  GCout */
0899:   b->Args({1, 7, 7, 3, 3, 1, 2, 1024, 1, 1});
0900:   b->Args({1, 7, 7, 3, 3, 1, 2, 768, 1, 1});
0901:   b->Args({1, 7, 7, 3, 3, 1, 2, 512, 1, 1});
0902:   b->Args({1, 7, 7, 3, 3, 1, 2, 384, 1, 1});
0903:   b->Args({1, 7, 7, 3, 3, 1, 2, 256, 1, 1});
0904:   b->Args({1, 7, 7, 3, 3, 1, 2, 128, 1, 1});
0905:   b->Args({1, 7, 7, 3, 3, 1, 2, 64, 1, 1});
0906:   b->Args({1, 7, 7, 3, 3, 1, 2, 32, 1, 1});
0907:   b->Args({1, 7, 7, 3, 3, 1, 2, 16, 1, 1});
0908: }
0909: 
```

- **EN:** This block implements local helper logic for `convolution`. Key symbols: `DWConv3x3d2`.
- **CN:** 该代码块实现与 `convolution` 相关的局部辅助逻辑。关键符号：`DWConv3x3d2`。

### Lines 910-968 / 第 910-968 行

```cpp
0910: static void DWConv5x5(benchmark::internal::Benchmark* b) {
0911:   b->ArgNames({"N", "H", "W", "KH", "KW", "S", "D", "G", "GCin", "GCout"});
0912: 
0913:   /********************** 96 x 96 *********************/
0914:   /*       N   H   W  KH  KW  S  D    G   GCin  GCout */
0915:   b->Args({1, 96, 96, 5, 5, 1, 1, 512, 1, 1});
0916:   b->Args({1, 96, 96, 5, 5, 1, 1, 256, 1, 1});
0917:   b->Args({1, 96, 96, 5, 5, 1, 1, 128, 1, 1});
0918:   b->Args({1, 96, 96, 5, 5, 1, 1, 64, 1, 1});
0919:   b->Args({1, 96, 96, 5, 5, 1, 1, 48, 1, 1});
0920:   b->Args({1, 96, 96, 5, 5, 1, 1, 32, 1, 1});
0921:   b->Args({1, 96, 96, 5, 5, 1, 1, 24, 1, 1});
0922:   b->Args({1, 96, 96, 5, 5, 1, 1, 16, 1, 1});
0923:   /********************** 32 x 32 *********************/
0924:   /*       N   H   W  KH  KW  S  D    G   GCin  GCout */
0925:   b->Args({1, 32, 32, 5, 5, 1, 1, 768, 1, 1});
0926:   b->Args({1, 32, 32, 5, 5, 1, 1, 512, 1, 1});
0927:   b->Args({1, 32, 32, 5, 5, 1, 1, 256, 1, 1});
0928:   b->Args({1, 32, 32, 5, 5, 1, 1, 128, 1, 1});
0929:   b->Args({1, 32, 32, 5, 5, 1, 1, 64, 1, 1});
0930:   b->Args({1, 32, 32, 5, 5, 1, 1, 48, 1, 1});
0931:   b->Args({1, 32, 32, 5, 5, 1, 1, 32, 1, 1});
0932:   b->Args({1, 32, 32, 5, 5, 1, 1, 24, 1, 1});
0933:   b->Args({1, 32, 32, 5, 5, 1, 1, 16, 1, 1});
0934:   /********************** 17 x 17 *********************/
0935:   /*       N   H   W  KH  KW  S  D    G   GCin  GCout */
0936:   b->Args({1, 17, 17, 5, 5, 1, 1, 1024, 1, 1});
0937:   b->Args({1, 17, 17, 5, 5, 1, 1, 768, 1, 1});
0938:   b->Args({1, 17, 17, 5, 5, 1, 1, 512, 1, 1});
0939:   b->Args({1, 17, 17, 5, 5, 1, 1, 384, 1, 1});
0940:   b->Args({1, 17, 17, 5, 5, 1, 1, 256, 1, 1});
0941:   b->Args({1, 17, 17, 5, 5, 1, 1, 128, 1, 1});
0942:   b->Args({1, 17, 17, 5, 5, 1, 1, 64, 1, 1});
0943:   b->Args({1, 17, 17, 5, 5, 1, 1, 32, 1, 1});
0944:   b->Args({1, 17, 17, 5, 5, 1, 1, 16, 1, 1});
0945:   /********************** 11 x 11 *********************/
0946:   /*       N   H   W  KH  KW  S  D    G   GCin  GCout */
0947:   b->Args({1, 11, 11, 5, 5, 1, 1, 1024, 1, 1});
0948:   b->Args({1, 11, 11, 5, 5, 1, 1, 768, 1, 1});
0949:   b->Args({1, 11, 11, 5, 5, 1, 1, 512, 1, 1});
0950:   b->Args({1, 11, 11, 5, 5, 1, 1, 384, 1, 1});
0951:   b->Args({1, 11, 11, 5, 5, 1, 1, 256, 1, 1});
0952:   b->Args({1, 11, 11, 5, 5, 1, 1, 128, 1, 1});
0953:   b->Args({1, 11, 11, 5, 5, 1, 1, 64, 1, 1});
0954:   b->Args({1, 11, 11, 5, 5, 1, 1, 32, 1, 1});
0955:   b->Args({1, 11, 11, 5, 5, 1, 1, 16, 1, 1});
0956:   /*********************** 7 x 7 **********************/
0957:   /*       N   H   W  KH  KW  S  D    G   GCin  GCout */
0958:   b->Args({1, 7, 7, 5, 5, 1, 1, 1024, 1, 1});
0959:   b->Args({1, 7, 7, 5, 5, 1, 1, 768, 1, 1});
0960:   b->Args({1, 7, 7, 5, 5, 1, 1, 512, 1, 1});
0961:   b->Args({1, 7, 7, 5, 5, 1, 1, 384, 1, 1});
0962:   b->Args({1, 7, 7, 5, 5, 1, 1, 256, 1, 1});
0963:   b->Args({1, 7, 7, 5, 5, 1, 1, 128, 1, 1});
0964:   b->Args({1, 7, 7, 5, 5, 1, 1, 64, 1, 1});
0965:   b->Args({1, 7, 7, 5, 5, 1, 1, 32, 1, 1});
0966:   b->Args({1, 7, 7, 5, 5, 1, 1, 16, 1, 1});
0967: }
0968: 
```

- **EN:** This block implements local helper logic for `convolution`. Key symbols: `DWConv5x5`.
- **CN:** 该代码块实现与 `convolution` 相关的局部辅助逻辑。关键符号：`DWConv5x5`。

### Lines 969-1016 / 第 969-1016 行

```cpp
0969: BENCHMARK_CAPTURE(convolution_q8, mobilenet_v1, "MobileNet v1")
0970:     ->Apply(MobileNetV1);
0971: BENCHMARK_CAPTURE(convolution_q8, mobilenet_v2, "MobileNet v2")
0972:     ->Apply(MobileNetV2);
0973: BENCHMARK_CAPTURE(convolution_q8, shufflenet_v1_g1, "ShuffleNet v1 (1 group)")
0974:     ->Apply(ShuffleNetV1G1);
0975: BENCHMARK_CAPTURE(convolution_q8, shufflenet_v1_g2, "ShuffleNet v1 (2 groups)")
0976:     ->Apply(ShuffleNetV1G2);
0977: BENCHMARK_CAPTURE(convolution_q8, shufflenet_v1_g3, "ShuffleNet v1 (3 groups)")
0978:     ->Apply(ShuffleNetV1G3);
0979: BENCHMARK_CAPTURE(convolution_q8, shufflenet_v1_g4, "ShuffleNet v1 (4 groups)")
0980:     ->Apply(ShuffleNetV1G4);
0981: BENCHMARK_CAPTURE(convolution_q8, shufflenet_v1_g8, "ShuffleNet v1 (8 groups)")
0982:     ->Apply(ShuffleNetV1G8);
0983: BENCHMARK_CAPTURE(convolution_q8, shufflenet_v2_x05, "ShuffleNet v2 0.5X")
0984:     ->Apply(ShuffleNetV2X05);
0985: BENCHMARK_CAPTURE(convolution_q8, shufflenet_v2_x10, "ShuffleNet v2 1.0X")
0986:     ->Apply(ShuffleNetV2X10);
0987: BENCHMARK_CAPTURE(convolution_q8, shufflenet_v2_x15, "ShuffleNet v2 1.5X")
0988:     ->Apply(ShuffleNetV2X15);
0989: BENCHMARK_CAPTURE(convolution_q8, shufflenet_v2_x20, "ShuffleNet v2 2.0X")
0990:     ->Apply(ShuffleNetV2X20);
0991: BENCHMARK_CAPTURE(convolution_q8, squeezenet_v10, "SqueezeNet 1.0")
0992:     ->Apply(SqueezeNetV10);
0993: BENCHMARK_CAPTURE(convolution_q8, squeezenet_v11, "SqueezeNet 1.1")
0994:     ->Apply(SqueezeNetV11);
0995: BENCHMARK_CAPTURE(convolution_q8, resnet18, "ResNet-18")->Apply(ResNet18);
0996: BENCHMARK_CAPTURE(convolution_q8, resnet50, "ResNet-50")->Apply(ResNet50);
0997: BENCHMARK_CAPTURE(convolution_q8, vgg, "VGG")->Apply(VGG);
0998: BENCHMARK_CAPTURE(convolution_q8, dwconv3x3, "3x3 DW Convolutions")
0999:     ->Apply(DWConv3x3);
1000: BENCHMARK_CAPTURE(
1001:     convolution_q8,
1002:     dwconv3x3d2,
1003:     "3x3 DW Convolutions (dilation 2)")
1004:     ->Apply(DWConv3x3d2);
1005: BENCHMARK_CAPTURE(convolution_q8, dwconv5x5, "5x5 DW Convolutions")
1006:     ->Apply(DWConv5x5);
1007: BENCHMARK_CAPTURE(convolution_q8, dwconv3x3_per_channel, "3x3 DW Convolutions", true)
1008:     ->Apply(DWConv3x3);
1009: BENCHMARK_CAPTURE(
1010:     convolution_q8,
1011:     dwconv3x3d2_per_channel,
1012:     "3x3 DW Convolutions (dilation 2)", true)
1013:     ->Apply(DWConv3x3d2);
1014: BENCHMARK_CAPTURE(convolution_q8, dwconv5x5_per_channel, "5x5 DW Convolutions", true)
1015:     ->Apply(DWConv5x5);
1016: 
```

- **EN:** This block implements local helper logic for `convolution`. Key symbols: no prominent local symbols.
- **CN:** 该代码块实现与 `convolution` 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 1017-1019 / 第 1017-1019 行

```cpp
1017: #ifndef PYTORCH_QNNPACK_BENCHMARK_NO_MAIN
1018: BENCHMARK_MAIN();
1019: #endif
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends. Key symbols: no prominent local symbols.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端。关键符号：无明显局部符号。


## Key Concepts / 关键概念
- **ATen native operator logic** — ATen 原生算子逻辑
- **GPU kernel/runtime interaction** — GPU 内核/运行时交互
- **Device and stream coordination** — 设备与流协调
- **Random-number generation** — 随机数生成
- **Mixed-precision/autocast logic** — 混合精度/自动混精逻辑
- **Core symbols: convolution_q8, bind, input, generate, bias, output, kernel_zero_points, requantization_scale** — 核心符号：convolution_q8、bind、input、generate、bias、output、kernel_zero_points、requantization_scale

## Dependencies / 依赖关系
- **Internal includes / 内部头文件**: 无
- **External includes / 外部头文件**: `algorithm`, `cfloat`, `chrono`, `cmath`, `functional`, `iostream`, `random`, `vector`, `pytorch_qnnpack.h`, `benchmark/benchmark.h`
- **Namespaces / 命名空间**: 无
- **Representative symbols / 代表性符号**: `convolution_q8`, `bind`, `input`, `generate`, `bias`, `output`, `kernel_zero_points`, `requantization_scale`, `pytorch_qnnp_run_operator`, `uint64_t`, `ShuffleNetV1G1`, `ShuffleNetV1G2`, `...`
