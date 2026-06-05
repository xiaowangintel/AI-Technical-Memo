# fully-connected-operator-tester.h — Code Analysis / 代码分析

## Source / 来源
- File: `aten/src/ATen/native/quantized/cpu/qnnpack/test/fully-connected-operator-tester.h`
- Repository: `https://github.com/pytorch/pytorch.git` (`/root/xw/pytorch`, commit `27a4844d7fb`)
- Purpose (EN): This file implements or declares quantized CPU operator implementations, scale/zero-point handling, and packed weights in the ATen native quantized CPU path.
- 用途 (CN): 该文件在 ATen 原生量化 CPU 路径中实现或声明了量化 CPU 算子实现、scale/zero-point 处理与打包权重逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12
```cpp
 1: /*
 2:  * Copyright (c) Facebook, Inc. and its affiliates.
 3:  * All rights reserved.
 4:  *
 5:  * This source code is licensed under the BSD-style license found in the
 6:  * LICENSE file in the root directory of this source tree.
 7:  */
 8:
 9: #pragma once
10:
11: #include <algorithm>
12: #include <cmath>
```
- EN: This range pulls in required headers, including `algorithm`, `cmath`. It also sets up compile-time guards, feature switches, or platform-specific paths.
- CN: 这一段引入了所需头文件，例如 `algorithm`, `cmath`。 它还建立了编译期开关、特性选择或平台相关分支。

### Lines 13-24
```cpp
13: #include <cstddef>
14: #include <cstdlib>
15: #include <functional>
16: #include <random>
17: #include <vector>
18: #include <memory>
19:
20: #include <pytorch_qnnpack.h>
21: #include <qnnpack_func.h>
22: #include <qnnpack/AlignedAllocator.h>
23:
24: class FullyConnectedOperatorTester {
```
- EN: This range pulls in required headers, including `cstddef`, `cstdlib`, `functional`. The main symbol in this range is `FullyConnectedOperatorTester`, which contributes directly to this file's operator logic. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段引入了所需头文件，例如 `cstddef`, `cstdlib`, `functional`。 这一段的主要符号是 `FullyConnectedOperatorTester`，它们直接构成本文件的算子逻辑。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 25-40
```cpp
25:  public:
26:   inline FullyConnectedOperatorTester& inputChannels(size_t inputChannels) {
27:     assert(inputChannels >= 1);
28:     this->inputChannels_ = inputChannels;
29:     return *this;
30:   }
31:
32:   inline size_t inputChannels() const {
33:     return this->inputChannels_;
34:   }
35:
36:   inline FullyConnectedOperatorTester& outputChannels(size_t outputChannels) {
37:     assert(outputChannels >= 1);
38:     this->outputChannels_ = outputChannels;
39:     return *this;
40:   }
```
- EN: The main symbol in this range is `inputChannels`, `outputChannels`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `inputChannels`, `outputChannels`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 42-53
```cpp
42:   inline size_t outputChannels() const {
43:     return this->outputChannels_;
44:   }
45:
46:   inline FullyConnectedOperatorTester& batchSize(size_t batchSize) {
47:     this->batchSize_ = batchSize;
48:     return *this;
49:   }
50:
51:   inline size_t batchSize() const {
52:     return this->batchSize_;
53:   }
```
- EN: The main symbol in this range is `outputChannels`, `batchSize`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `outputChannels`, `batchSize`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 55-68
```cpp
55:   inline FullyConnectedOperatorTester& inputStride(size_t inputStride) {
56:     assert(inputStride >= 1);
57:     this->inputStride_ = inputStride;
58:     return *this;
59:   }
60:
61:   inline size_t inputStride() const {
62:     if (this->inputStride_ == 0) {
63:       return inputChannels();
64:     } else {
65:       assert(this->inputStride_ >= inputChannels());
66:       return this->inputStride_;
67:     }
68:   }
```
- EN: The main symbol in this range is `inputStride`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `inputStride`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 70-83
```cpp
70:   inline FullyConnectedOperatorTester& outputStride(size_t outputStride) {
71:     assert(outputStride >= 1);
72:     this->outputStride_ = outputStride;
73:     return *this;
74:   }
75:
76:   inline size_t outputStride() const {
77:     if (this->outputStride_ == 0) {
78:       return outputChannels();
79:     } else {
80:       assert(this->outputStride_ >= outputChannels());
81:       return this->outputStride_;
82:     }
83:   }
```
- EN: The main symbol in this range is `outputStride`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `outputStride`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 85-97
```cpp
85:   inline FullyConnectedOperatorTester& per_channel(bool per_channel) {
86:     this->per_channel_ = per_channel;
87:     return *this;
88:   }
89:
90:   inline bool per_channel() const {
91:     return this->per_channel_;
92:   }
93:
94:   inline FullyConnectedOperatorTester& qmin(uint8_t qmin) {
95:     this->qmin_ = qmin;
96:     return *this;
97:   }
```
- EN: The main symbol in this range is `per_channel`, `qmin`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `per_channel`, `qmin`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 99-110
```cpp
 99:   inline uint8_t qmin() const {
100:     return this->qmin_;
101:   }
102:
103:   inline FullyConnectedOperatorTester& qmax(uint8_t qmax) {
104:     this->qmax_ = qmax;
105:     return *this;
106:   }
107:
108:   inline uint8_t qmax() const {
109:     return this->qmax_;
110:   }
```
- EN: The main symbol in this range is `qmin`, `qmax`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `qmin`, `qmax`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 112-125
```cpp
112:   inline FullyConnectedOperatorTester& iterations(size_t iterations) {
113:     this->iterations_ = iterations;
114:     return *this;
115:   }
116:
117:   inline size_t iterations() const {
118:     return this->iterations_;
119:   }
120:
121:   enum class Mode {
122:     Static,
123:     Dynamic,
124:     Runtime,
125:   };
```
- EN: The main symbol in this range is `iterations`, `Mode`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `iterations`, `Mode`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 127-137
```cpp
127:   void testQ8(const Mode mode) const {
128:     std::random_device randomDevice;
129:     auto rng = std::mt19937(randomDevice());
130:     auto s32rng =
131:         std::bind(std::uniform_int_distribution<int32_t>(-10000, 10000), rng);
132:     auto u8rng = std::bind(std::uniform_int_distribution<uint8_t>(), rng);
133:     auto f32rng =
134:         std::bind(std::uniform_real_distribution<float>(1, 5), rng);
135:
136:     std::vector<uint8_t> input(
137:         (batchSize() - 1) * inputStride() + inputChannels() + 8);
```
- EN: The main symbol in this range is `testQ8`, which contributes directly to this file's operator logic.
- CN: 这一段的主要符号是 `testQ8`，它们直接构成本文件的算子逻辑。

### Lines 138-150
```cpp
138:     std::vector<uint8_t> kernel(outputChannels() * inputChannels());
139:     std::vector<int32_t> bias(outputChannels());
140:     std::vector<uint8_t> output(
141:         (batchSize() - 1) * outputStride() + outputChannels());
142:     std::vector<float> output_dynamic(output.size());
143:     std::vector<int32_t> accumulators(batchSize() * outputChannels());
144:
145:     const uint8_t* const inputPtr = input.data() + 8;
146:     const uint8_t inputZeroPoint = 127;
147:     // Make number of output channels multiple of 8.
148:     // This is the least common denominator for SSE/ARM kernels we have.
149:     size_t num_zero_points_padded = outputChannels() + 8;
150:     std::vector<uint8_t> kernelZeroPoints(num_zero_points_padded, 127);
```
- EN: This range continues the file's low-level CPU implementation details and operator glue code.
- CN: 这一段继续展开本文件的底层 CPU 实现细节和算子胶水代码。

### Lines 152-161
```cpp
152:     for (size_t iteration = 0; iteration < iterations(); iteration++) {
153:       std::generate(input.begin(), input.end(), std::ref(u8rng));
154:       std::generate(kernel.begin(), kernel.end(), std::ref(u8rng));
155:       std::generate(bias.begin(), bias.end(), std::ref(s32rng));
156:       if (per_channel()) {
157:         std::generate(kernelZeroPoints.begin(), kernelZeroPoints.end(), std::ref(u8rng));
158:       }
159:       std::fill(output.begin(), output.end(), 0xA5);
160:       std::fill(output_dynamic.begin(), output_dynamic.end(), 0.0f);
161:       std::fill(accumulators.begin(), accumulators.end(), 0);
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 163-178
```cpp
163:       for (size_t i = 0; i < batchSize(); i++) {
164:         for (size_t oc = 0; oc < outputChannels(); oc++) {
165:           accumulators[i * outputChannels() + oc] = bias[oc];
166:         }
167:       }
168:       for (size_t i = 0; i < batchSize(); i++) {
169:         for (size_t oc = 0; oc < outputChannels(); oc++) {
170:           for (size_t ic = 0; ic < inputChannels(); ic++) {
171:             accumulators[i * outputChannels() + oc] +=
172:                 (int32_t(inputPtr[i * inputStride() + ic]) -
173:                  int32_t(inputZeroPoint)) *
174:                 (int32_t(kernel[oc * inputChannels() + ic]) -
175:                  int32_t(kernelZeroPoints[oc]));
176:           }
177:         }
178:       }
```
- EN: Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 180-190
```cpp
180:       // Create dummy min/max for empty inputs.
181:       // These are only used to compute scale and zero point,
182:       // and real callers will just pull those values from the model.
183:       const int32_t accumulatorsMin = accumulators.empty()
184:           ? 0
185:           : *std::min_element(accumulators.cbegin(), accumulators.cend());
186:       const int32_t accumulatorsMax = accumulators.empty()
187:           ? 900
188:           : *std::max_element(accumulators.cbegin(), accumulators.cend());
189:
190:       const double outputScale =
```
- EN: This range continues the file's low-level CPU implementation details and operator glue code.
- CN: 这一段继续展开本文件的底层 CPU 实现细节和算子胶水代码。

### Lines 191-202
```cpp
191:           double(uint32_t(accumulatorsMax - accumulatorsMin)) / 255.0;
192:       const uint8_t outputZeroPoint = uint8_t(std::max(
193:           std::min(
194:               lrint(
195:                   127.5 -
196:                   0.5 * double(accumulatorsMin + accumulatorsMax) /
197:                       outputScale),
198:               long(std::numeric_limits<uint8_t>::max())),
199:           long(std::numeric_limits<uint8_t>::min())));
200:
201:       ASSERT_EQ(pytorch_qnnp_status_success, pytorch_qnnp_initialize());
202:       // 1 bcz input_scale and kernel_scale are both 1.
```
- EN: This range continues the file's low-level CPU implementation details and operator glue code.
- CN: 这一段继续展开本文件的底层 CPU 实现细节和算子胶水代码。

### Lines 203-216
```cpp
203:       std::vector<float>
204:         requantization_scales(num_zero_points_padded, 1.0 * 1.0 / outputScale);
205:       if (per_channel()) {
206:         auto scale_generator = [&]() -> float {return (f32rng()/outputScale);};
207:         std::generate(
208:             requantization_scales.begin(),
209:             requantization_scales.end(),
210:             std::ref(scale_generator));
211:       }
212:
213:       switch(mode) {
214:         case Mode::Static:
215:         {
216:           pytorch_qnnp_operator_t convolution = nullptr;
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 218-232
```cpp
218:           ASSERT_EQ(
219:               pytorch_qnnp_status_success,
220:               pytorch_qnnp_create_fully_connected_nc_q8(
221:                   inputChannels(),
222:                   outputChannels(),
223:                   inputZeroPoint,
224:                   kernelZeroPoints.data(),
225:                   kernel.data(),
226:                   bias.data(),
227:                   outputZeroPoint,
228:                   qmin(),
229:                   qmax(),
230:                   0,
231:                   requantization_scales.data(),
232:                   &convolution));
```
- EN: Quantization-specific data handling or packed-parameter logic is active in this range. The math and shape handling relate to convolution-style operators.
- CN: 这一段涉及量化数据处理或打包参数逻辑。 这里的计算与形状处理与卷积类算子相关。

### Lines 234-246
```cpp
234:           ASSERT_EQ(
235:               pytorch_qnnp_status_success,
236:               pytorch_qnnp_setup_fully_connected_nc_q8(
237:                   convolution,
238:                   batchSize(),
239:                   inputPtr,
240:                   inputStride(),
241:                   output.data(),
242:                   outputStride()));
243:
244:           ASSERT_EQ(
245:               pytorch_qnnp_status_success,
246:               pytorch_qnnp_run_operator(convolution, nullptr /* thread pool */));
```
- EN: The math and shape handling relate to convolution-style operators. The code participates in pooling window traversal or pooled-output shape computation.
- CN: 这里的计算与形状处理与卷积类算子相关。 这段代码参与池化窗口遍历或池化输出形状计算。

### Lines 248-258
```cpp
248:           ASSERT_EQ(
249:               pytorch_qnnp_status_success,
250:               pytorch_qnnp_delete_operator(convolution));
251:           convolution = nullptr;
252:         }
253:         break;
254:
255:         case Mode::Dynamic:
256:         {
257:           auto packW = std::unique_ptr<qnnpack::PackBMatrix>(
258:               new qnnpack::PackBMatrix(
```
- EN: Quantization-specific data handling or packed-parameter logic is active in this range. The math and shape handling relate to convolution-style operators.
- CN: 这一段涉及量化数据处理或打包参数逻辑。 这里的计算与形状处理与卷积类算子相关。

### Lines 259-270
```cpp
259:                   inputChannels(),
260:                   outputChannels(),
261:                   kernelZeroPoints.data(),
262:                   requantization_scales.data(),
263:                   kernel.data(),
264:                   nullptr));
265:
266:           // Attention! Bias size must be a multiple of 8.
267:           constexpr size_t kBiasSizeMultiple = 8u;
268:           std::vector<float, AlignedAllocator<float, 32>> bias_float(
269:               (bias.size() + (kBiasSizeMultiple - 1)) & -kBiasSizeMultiple);
270:           std::copy(bias.cbegin(), bias.cend(), bias_float.begin());
```
- EN: Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段涉及量化数据处理或打包参数逻辑。

### Lines 272-282
```cpp
272:           const pytorch_qnnp_status runStatus = qnnpack::qnnpackLinearDynamic(
273:               batchSize() /* batch_size */,
274:               inputChannels() /* input_channels */,
275:               outputChannels() /* output_channels */,
276:               inputZeroPoint,
277:               kernelZeroPoints.data(),
278:               requantization_scales.data(), /* Dequantization scale */
279:               inputPtr,
280:               inputChannels() /* input_stride */,
281:               packW->getPackedWeights(),
282:               bias_float.data(),
```
- EN: Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段涉及量化数据处理或打包参数逻辑。

### Lines 283-299
```cpp
283:               output_dynamic.data(),
284:               outputStride() /* output_stride */,
285:               nullptr /* threadpool */);
286:           ASSERT_EQ(pytorch_qnnp_status_success, runStatus);
287:         }
288:         break;
289:
290:         case Mode::Runtime:
291:         {
292:           auto packW = std::unique_ptr<qnnpack::PackBMatrix>(
293:               new qnnpack::PackBMatrix(
294:                   inputChannels(),
295:                   outputChannels(),
296:                   kernelZeroPoints.data(),
297:                   requantization_scales.data(),
298:                   kernel.data(),
299:                   bias.data()));
```
- EN: Quantization-specific data handling or packed-parameter logic is active in this range. The code participates in pooling window traversal or pooled-output shape computation.
- CN: 这一段涉及量化数据处理或打包参数逻辑。 这段代码参与池化窗口遍历或池化输出形状计算。

### Lines 301-311
```cpp
301:           const pytorch_qnnp_status runStatus = qnnpack::qnnpackLinear(
302:               batchSize() /* batch_size */,
303:               inputChannels() /* input_channels */,
304:               outputChannels() /* output_channels */,
305:               inputZeroPoint,
306:               kernelZeroPoints.data(),
307:               requantization_scales.data(),
308:               outputZeroPoint,
309:               qmin(),
310:               qmax(),
311:               inputPtr,
```
- EN: Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段涉及量化数据处理或打包参数逻辑。

### Lines 312-324
```cpp
312:               inputChannels() /* input_stride */,
313:               packW->getPackedWeights(),
314:               output.data(),
315:               outputStride() /* output_stride */,
316:               nullptr /* threadpool */);
317:           ASSERT_EQ(pytorch_qnnp_status_success, runStatus);
318:         }
319:         break;
320:
321:         default:
322:           // Undefined!
323:           ASSERT_TRUE(false);
324:       }
```
- EN: Quantization-specific data handling or packed-parameter logic is active in this range. The code participates in pooling window traversal or pooled-output shape computation.
- CN: 这一段涉及量化数据处理或打包参数逻辑。 这段代码参与池化窗口遍历或池化输出形状计算。

### Lines 326-336
```cpp
326:       switch (mode) {
327:         case Mode::Static:
328:         case Mode::Runtime:
329:         {
330:           for (size_t i = 0; i < batchSize(); i++) {
331:             for (size_t c = 0; c < outputChannels(); c++) {
332:               const double scaledAccumulator =
333:                   accumulators[i * outputChannels() + c] *
334:                   requantization_scales[c];
335:               const double clampedAccumulator = std::max(
336:                   std::min(
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 337-347
```cpp
337:                       scaledAccumulator, double(qmax()) - double(outputZeroPoint)),
338:                   double(qmin()) - double(outputZeroPoint));
339:               ASSERT_NEAR(
340:                   clampedAccumulator,
341:                   (int32_t(output[i * outputStride() + c]) - outputZeroPoint),
342:                   0.9)
343:                   << "batch index = " << i << ", channel = " << c;
344:             }
345:           }
346:         }
347:         break;
```
- EN: This range continues the file's low-level CPU implementation details and operator glue code.
- CN: 这一段继续展开本文件的底层 CPU 实现细节和算子胶水代码。

### Lines 349-359
```cpp
349:         case Mode::Dynamic:
350:         {
351:           // Bias is added post scaling, as float.
352:           for (size_t i = 0; i < batchSize(); i++) {
353:             for (size_t oc = 0; oc < outputChannels(); oc++) {
354:               accumulators[i * outputChannels() + oc] -= bias[oc];
355:             }
356:           }
357:           for (size_t i = 0; i < batchSize(); i++) {
358:             for (size_t c = 0; c < outputChannels(); c++) {
359:               const float ref = ((float)accumulators[i * outputChannels() + c] *
```
- EN: Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 360-372
```cpp
360:                            requantization_scales[c]) +
361:                   float(bias[c]);
362:               ASSERT_NEAR(
363:                   output_dynamic[i * outputChannels() + c],
364:                   ref,
365:                   std::abs(ref) * 1.0e-4)
366:                   << "at " << i << ", " << c << ": reference = " << ref
367:                   << ", optimized = "
368:                   << output_dynamic[i * outputChannels() + c];
369:             }
370:           }
371:         }
372:         break;
```
- EN: Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段涉及量化数据处理或打包参数逻辑。

### Lines 374-384
```cpp
374:         default:
375:           // Undefined!
376:           ASSERT_TRUE(false);
377:       }
378:     }
379:   }
380:
381:  private:
382:   size_t inputChannels_{1};
383:   size_t inputStride_{0};
384:   size_t outputChannels_{1};
```
- EN: This range continues the file's low-level CPU implementation details and operator glue code.
- CN: 这一段继续展开本文件的底层 CPU 实现细节和算子胶水代码。

### Lines 385-391
```cpp
385:   size_t outputStride_{0};
386:   size_t batchSize_{1};
387:   uint8_t qmin_{0};
388:   uint8_t qmax_{255};
389:   size_t iterations_{1};
390:   bool per_channel_{false};
391: };
```
- EN: This range continues the file's low-level CPU implementation details and operator glue code.
- CN: 这一段继续展开本文件的底层 CPU 实现细节和算子胶水代码。

## Key Concepts / 关键概念

- Quantized arithmetic and packed parameters / 量化算术与打包参数
- Convolution shape/kernel handling / 卷积形状与 kernel 处理
- Pooling reductions / 池化归约
- Shape/container bookkeeping / 形状与容器管理

## Dependencies / 依赖关系

- Standard or third-party headers / 标准库或第三方头文件: `algorithm`, `cmath`, `cstddef`, `cstdlib`, `functional`, `random`, `vector`, `memory`, `pytorch_qnnpack.h`, `qnnpack_func.h`
- Key helper symbols / 关键辅助符号: `qnnpack`
