# fully-connected-sparse-operator-tester.h — Code Analysis / 代码分析

## Source / 来源
- File: `aten/src/ATen/native/quantized/cpu/qnnpack/test/fully-connected-sparse-operator-tester.h`
- Repository: `https://github.com/pytorch/pytorch.git` (`/root/xw/pytorch`, commit `27a4844d7fb`)
- Purpose (EN): This file implements or declares quantized CPU operator implementations, scale/zero-point handling, and packed weights in the ATen native quantized CPU path.
- 用途 (CN): 该文件在 ATen 原生量化 CPU 路径中实现或声明了量化 CPU 算子实现、scale/zero-point 处理与打包权重逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-23
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
13: #include <cstddef>
14: #include <cstdlib>
15: #include <functional>
16: #include <random>
17: #include <vector>
18: #include <memory>
19:
20: #include <pack_block_sparse.h>
21: #include <pytorch_qnnpack.h>
22: #include <qnnpack_func.h>
23: #include <qnnpack/AlignedAllocator.h>
```
- EN: This range pulls in required headers, including `algorithm`, `cmath`, `cstddef`. It also sets up compile-time guards, feature switches, or platform-specific paths. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段引入了所需头文件，例如 `algorithm`, `cmath`, `cstddef`。 它还建立了编译期开关、特性选择或平台相关分支。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 25-50
```cpp
25: #define MAYBE_UNUSED __attribute__((unused))
26:
27: namespace {
28:   void fillBlockSparseWeights(
29:       uint8_t* b,
30:       size_t N,
31:       size_t K,
32:       size_t row_block_size,
33:       size_t col_block_size,
34:       float sparsity,
35:       const uint8_t* zero_points) {
36:     std::random_device randomDevice;
37:     auto rng = std::mt19937(randomDevice());
38:     std::bernoulli_distribution dist{sparsity};
39:     for (uint32_t n = 0; n < N ; n += row_block_size) {
40:       for (uint32_t k = 0; k < K; k += col_block_size) {
41:         if (dist(rng)) {
42:           for (uint32_t nb = 0; (nb < row_block_size) && (n + nb < N); ++nb) {
43:             for (uint32_t kb = 0; (kb < col_block_size) && (k + kb < K); ++kb) {
44:               *(b + (n + nb) * K + k + kb) = zero_points[n + nb];
45:             }
46:           }
47:         }
48:       }
49:     }
50:   }
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The code enters or documents the namespace scope used by ATen native CPU operators. The main symbol in this range is `fillBlockSparseWeights`, which contributes directly to this file's operator logic.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。 这一段的主要符号是 `fillBlockSparseWeights`，它们直接构成本文件的算子逻辑。

### Lines 52-73
```cpp
52:   // Temp Debug utils that will be removed later
53:   MAYBE_UNUSED void printMatrix(const char* name, const uint8_t* a, const size_t M, const size_t N) {
54:     std::cout << "Matrix START:" << name << "...\n";
55:     for (uint32_t m = 0; m < M ; ++m) {
56:       for (uint32_t n = 0; n < N; n++) {
57:         std::cout << (const uint32_t)(*(a + m * N + n)) << ", ";
58:       }
59:       std::cout << std::endl;
60:     }
61:     std::cout << "Matrix END...\n\n";
62:   }
63:
64:   MAYBE_UNUSED void printMatrix(const char* name, const float* a, const size_t M, const size_t N) {
65:     std::cout << "Matrix START:" << name << "...\n";
66:     for (uint32_t m = 0; m < M ; ++m) {
67:       for (uint32_t n = 0; n < N; n++) {
68:         std::cout << (*(a + m * N + n)) << ", ";
69:       }
70:       std::cout << std::endl;
71:     }
72:     std::cout << "Matrix END...\n\n";
73:   }
```
- EN: The main symbol in this range is `printMatrix`, which contributes directly to this file's operator logic. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 这一段的主要符号是 `printMatrix`，它们直接构成本文件的算子逻辑。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 75-93
```cpp
75: }
76:
77: class FullyConnectedSparseOperatorTester {
78:  public:
79:   inline FullyConnectedSparseOperatorTester& inputChannels(size_t inputChannels) {
80:     assert(inputChannels >= 1);
81:     this->inputChannels_ = inputChannels;
82:     return *this;
83:   }
84:
85:   inline size_t inputChannels() const {
86:     return this->inputChannels_;
87:   }
88:
89:   inline FullyConnectedSparseOperatorTester& outputChannels(size_t outputChannels) {
90:     assert(outputChannels >= 1);
91:     this->outputChannels_ = outputChannels;
92:     return *this;
93:   }
```
- EN: The main symbol in this range is `inputChannels`, `outputChannels`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `inputChannels`, `outputChannels`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 95-121
```cpp
 95:   inline size_t outputChannels() const {
 96:     return this->outputChannels_;
 97:   }
 98:
 99:   inline FullyConnectedSparseOperatorTester& batchSize(size_t batchSize) {
100:     this->batchSize_ = batchSize;
101:     return *this;
102:   }
103:
104:   inline size_t batchSize() const {
105:     return this->batchSize_;
106:   }
107:
108:   inline FullyConnectedSparseOperatorTester& inputStride(size_t inputStride) {
109:     assert(inputStride >= 1);
110:     this->inputStride_ = inputStride;
111:     return *this;
112:   }
113:
114:   inline size_t inputStride() const {
115:     if (this->inputStride_ == 0) {
116:       return inputChannels();
117:     } else {
118:       assert(this->inputStride_ >= inputChannels());
119:       return this->inputStride_;
120:     }
121:   }
```
- EN: The main symbol in this range is `outputChannels`, `batchSize`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `outputChannels`, `batchSize`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 123-141
```cpp
123:   inline FullyConnectedSparseOperatorTester& outputStride(size_t outputStride) {
124:     assert(outputStride >= 1);
125:     this->outputStride_ = outputStride;
126:     return *this;
127:   }
128:
129:   inline size_t outputStride() const {
130:     if (this->outputStride_ == 0) {
131:       return outputChannels();
132:     } else {
133:       assert(this->outputStride_ >= outputChannels());
134:       return this->outputStride_;
135:     }
136:   }
137:
138:   inline FullyConnectedSparseOperatorTester& qmin(uint8_t qmin) {
139:     this->qmin_ = qmin;
140:     return *this;
141:   }
```
- EN: The main symbol in this range is `outputStride`, `qmin`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `outputStride`, `qmin`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 143-163
```cpp
143:   inline uint8_t qmin() const {
144:     return this->qmin_;
145:   }
146:
147:   inline FullyConnectedSparseOperatorTester& qmax(uint8_t qmax) {
148:     this->qmax_ = qmax;
149:     return *this;
150:   }
151:
152:   inline uint8_t qmax() const {
153:     return this->qmax_;
154:   }
155:
156:   inline FullyConnectedSparseOperatorTester& iterations(size_t iterations) {
157:     this->iterations_ = iterations;
158:     return *this;
159:   }
160:
161:   inline size_t iterations() const {
162:     return this->iterations_;
163:   }
```
- EN: The main symbol in this range is `qmin`, `qmax`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `qmin`, `qmax`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 165-186
```cpp
165:   inline FullyConnectedSparseOperatorTester& rowBlockSize(size_t block_size) {
166:     this->rowBlockSize_ = block_size;
167:     return *this;
168:   }
169:
170:   inline FullyConnectedSparseOperatorTester& colBlockSize(size_t block_size) {
171:     this->colBlockSize_ = block_size;
172:     return *this;
173:   }
174:
175:   inline FullyConnectedSparseOperatorTester& sparsity(float s) {
176:     this->sparsity_ = s;
177:     return *this;
178:   }
179:
180:   inline size_t rowBlockSize() const {
181:     return this->rowBlockSize_;
182:   }
183:
184:   inline size_t colBlockSize() const {
185:     return this->colBlockSize_;
186:   }
```
- EN: The main symbol in this range is `rowBlockSize`, `colBlockSize`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `rowBlockSize`, `colBlockSize`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 188-214
```cpp
188:   inline float sparsity() const {
189:     return this->sparsity_;
190:   }
191:
192:   enum class Mode {
193:     Dynamic,
194:     Runtime,
195:   };
196:
197:   void testQ8(const Mode mode) const {
198:     std::random_device randomDevice;
199:     auto rng = std::mt19937(randomDevice());
200:     auto s32rng =
201:         std::bind(std::uniform_int_distribution<int32_t>(-10000, 10000), rng);
202:     auto u8rng = std::bind(std::uniform_int_distribution<uint8_t>(), rng);
203:     auto f32rng =
204:         std::bind(std::uniform_real_distribution<float>(1, 5), rng);
205:
206:     std::vector<uint8_t> input(
207:         (batchSize() - 1) * inputStride() + inputChannels() + 8);
208:     std::vector<uint8_t> kernel(outputChannels() * inputChannels());
209:     std::vector<int32_t> bias(outputChannels());
210:     std::vector<uint8_t> output(
211:         (batchSize() - 1) * outputStride() + outputChannels());
212:     std::vector<float> output_dynamic(output.size());
213:     std::vector<int32_t> accumulators(batchSize() * outputChannels());
214:     std::vector<float> accumulators_float(batchSize() * outputChannels());
```
- EN: The main symbol in this range is `sparsity`, `testQ8`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `sparsity`, `testQ8`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 216-241
```cpp
216:     const uint8_t* const inputPtr = input.data();
217:     const uint8_t inputZeroPoint = 127;
218:     // Make number of output channels multiple of 8.
219:     // This is the least common denominator for SSE/ARM kernels we have.
220:     size_t num_zero_points_padded = outputChannels() + 8;
221:     std::vector<uint8_t> kernelZeroPoints(num_zero_points_padded, 127);
222:
223:     for (size_t iteration = 0; iteration < iterations(); iteration++) {
224:       std::generate(input.begin(), input.end(), std::ref(u8rng));
225:       std::generate(bias.begin(), bias.end(), std::ref(s32rng));
226:       std::generate(kernelZeroPoints.begin(), kernelZeroPoints.end(), std::ref(u8rng));
227:
228:       uint8_t max_elem, min_elem;
229:       do {
230:         std::generate(kernel.begin(), kernel.end(), std::ref(u8rng));
231:         fillBlockSparseWeights(
232:             kernel.data(),
233:             outputChannels(),
234:             inputChannels(),
235:             rowBlockSize(),
236:             colBlockSize(),
237:             sparsity(),
238:             kernelZeroPoints.data());
239:         max_elem = *std::max_element(kernel.cbegin(), kernel.cend());
240:         min_elem = *std::min_element(kernel.cbegin(), kernel.cend());
241:       } while (max_elem == min_elem);
```
- EN: Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 243-261
```cpp
243:       std::unique_ptr<qnnpack::BCSRMatrix> bcsr_matrix =
244:           qnnpack::generateBlockCSRMatrix<uint32_t>(
245:               kernel.data(),
246:               outputChannels(),
247:               inputChannels(),
248:               rowBlockSize(),
249:               colBlockSize(),
250:               kernelZeroPoints.data());
251:
252:       std::fill(output.begin(), output.end(), 0xA5);
253:       std::fill(output_dynamic.begin(), output_dynamic.end(), 0.0f);
254:       std::fill(accumulators.begin(), accumulators.end(), 0);
255:
256:       for (size_t i = 0; i < batchSize(); i++) {
257:         for (size_t oc = 0; oc < outputChannels(); oc++) {
258:           accumulators[i * outputChannels() + oc] = bias[oc];
259:         }
260:       }
261:       for (size_t i = 0; i < batchSize(); i++) {
```
- EN: Looping logic performs the repeated per-element, per-channel, or per-kernel computation. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 262-281
```cpp
262:         for (size_t oc = 0; oc < outputChannels(); oc++) {
263:           for (size_t ic = 0; ic < inputChannels(); ic++) {
264:             accumulators[i * outputChannels() + oc] +=
265:                 (int32_t(inputPtr[i * inputStride() + ic]) -
266:                  int32_t(inputZeroPoint)) *
267:                 (int32_t(kernel[oc * inputChannels() + ic]) -
268:                  int32_t(kernelZeroPoints[oc]));
269:           }
270:         }
271:       }
272:
273:       // Create dummy min/max for empty inputs.
274:       // These are only used to compute scale and zero point,
275:       // and real callers will just pull those values from the model.
276:       const int32_t accumulatorsMin = accumulators.empty()
277:           ? 0
278:           : *std::min_element(accumulators.cbegin(), accumulators.cend());
279:       const int32_t accumulatorsMax = accumulators.empty()
280:           ? 900
281:           : *std::max_element(accumulators.cbegin(), accumulators.cend());
```
- EN: Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 283-302
```cpp
283:       const double outputScale =
284:           double(uint32_t(accumulatorsMax - accumulatorsMin)) / 255.0;
285:       const uint8_t outputZeroPoint = uint8_t(std::max(
286:           std::min(
287:               lrint(
288:                   127.5 -
289:                   0.5 * double(accumulatorsMin + accumulatorsMax) /
290:                       outputScale),
291:               long(std::numeric_limits<uint8_t>::max())),
292:           long(std::numeric_limits<uint8_t>::min())));
293:
294:       ASSERT_EQ(pytorch_qnnp_status_success, pytorch_qnnp_initialize());
295:       // 1 bcz input_scale and kernel_scale are both 1.
296:       std::vector<float>
297:         requantization_scales(num_zero_points_padded, 1.0 * 1.0 / outputScale);
298:       auto scale_generator = [&]() -> float {return (f32rng()/outputScale);};
299:       std::generate(
300:           requantization_scales.begin(),
301:           requantization_scales.end(),
302:           std::ref(scale_generator));
```
- EN: The block also assembles or returns the result expected by the surrounding operator code. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 该代码块还会组装或返回外围算子代码所需的结果。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 304-322
```cpp
304:       switch(mode) {
305:         case Mode::Runtime:
306:           break;
307:         case Mode::Dynamic: {
308:             // Attention! Bias size must be a multiple of 8.
309:             constexpr size_t kBiasSizeMultiple = 8u;
310:             std::vector<float, AlignedAllocator<float, 32>> bias_float(
311:               (bias.size() + (kBiasSizeMultiple - 1)) & -kBiasSizeMultiple);
312:             std::copy(bias.cbegin(), bias.cend(), bias_float.begin());
313:
314:             pytorch_qnnp_operator_t sparse_gemm = nullptr;
315:
316:             ASSERT_EQ(
317:                 pytorch_qnnp_status_success,
318:                 pytorch_qnnp_create_fully_connected_sparse_dq_nc_q8(
319:                     inputChannels(),
320:                     outputChannels(),
321:                     inputZeroPoint,
322:                     kernelZeroPoints.data(),
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 323-346
```cpp
323:                     bcsr_matrix->col_indices_data_ptr(),
324:                     bcsr_matrix->row_values_data_ptr(),
325:                     bcsr_matrix->values.data(),
326:                     bcsr_matrix->row_block_size,
327:                     bcsr_matrix->col_block_size,
328:                     pytorch_qnnp_sparse_matrix_indices_dtype_uint32_t,
329:                     outputZeroPoint,
330:                     qmin(),
331:                     qmax(),
332:                     0,
333:                     requantization_scales.data(),
334:                     false,
335:                     &sparse_gemm));
336:
337:             ASSERT_EQ(
338:                 pytorch_qnnp_status_success,
339:                 pytorch_qnnp_setup_fully_connected_sparse_dq_nc_q8(
340:                     sparse_gemm,
341:                     batchSize(),
342:                     inputPtr,
343:                     inputStride(),
344:                     bias_float.data(),
345:                     output_dynamic.data(),
346:                     outputStride()));
```
- EN: Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段涉及量化数据处理或打包参数逻辑。

### Lines 348-366
```cpp
348:             ASSERT_EQ(
349:                 pytorch_qnnp_status_success,
350:                 pytorch_qnnp_run_operator(sparse_gemm, nullptr /* thread pool */));
351:
352:             ASSERT_EQ(
353:                 pytorch_qnnp_status_success,
354:                 pytorch_qnnp_delete_operator(sparse_gemm));
355:             sparse_gemm = nullptr;
356:
357:             break;
358:           }
359:         default:
360:           // Undefined!
361:           ASSERT_TRUE(false);
362:       }
363:
364:       switch (mode) {
365:         case Mode::Runtime:
366:           break;
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The code participates in pooling window traversal or pooled-output shape computation.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 这段代码参与池化窗口遍历或池化输出形状计算。

### Lines 367-390
```cpp
367:         case Mode::Dynamic:
368:         {
369:           // Bias is added post scaling, as float.
370:           for (size_t i = 0; i < batchSize(); i++) {
371:             for (size_t oc = 0; oc < outputChannels(); oc++) {
372:               accumulators[i * outputChannels() + oc] -= bias[oc];
373:               accumulators_float[i * outputChannels() + oc] =
374:                 (float)accumulators[i * outputChannels() + oc] *
375:                   requantization_scales[oc] + float(bias[oc]);
376:             }
377:           }
378:           for (size_t i = 0; i < batchSize(); i++) {
379:             for (size_t c = 0; c < outputChannels(); c++) {
380:               ASSERT_EQ(
381:                   output_dynamic[i * outputChannels() + c],
382:                   accumulators_float[i * outputChannels() + c])
383:                   << "at " << i << ", " << c
384:                   << ": reference = " <<
385:                   accumulators_float[i * outputChannels() + c]
386:                   << ", optimized = " << output_dynamic[i * outputChannels() + c];
387:             }
388:           }
389:         }
390:         break;
```
- EN: Looping logic performs the repeated per-element, per-channel, or per-kernel computation. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 392-416
```cpp
392:         default:
393:           // Undefined!
394:           ASSERT_TRUE(false);
395:       }
396:     }
397:   }
398:
399:   void testQ8_prepacked(const Mode mode) const {
400:     std::random_device randomDevice;
401:     auto rng = std::mt19937(randomDevice());
402:     auto s32rng =
403:         std::bind(std::uniform_int_distribution<int32_t>(-10000, 10000), rng);
404:     auto u8rng = std::bind(std::uniform_int_distribution<uint8_t>(), rng);
405:     auto f32rng =
406:         std::bind(std::uniform_real_distribution<float>(1, 5), rng);
407:
408:     std::vector<uint8_t> input(
409:         (batchSize() - 1) * inputStride() + inputChannels() + 8);
410:     std::vector<uint8_t> kernel(outputChannels() * inputChannels());
411:     std::vector<int32_t> bias(outputChannels());
412:     std::vector<uint8_t> output(
413:         (batchSize() - 1) * outputStride() + outputChannels());
414:     std::vector<float> output_dynamic(output.size());
415:     std::vector<int32_t> accumulators(batchSize() * outputChannels());
416:     std::vector<float> accumulators_float(batchSize() * outputChannels());
```
- EN: The main symbol in this range is `testQ8_prepacked`, which contributes directly to this file's operator logic. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段的主要符号是 `testQ8_prepacked`，它们直接构成本文件的算子逻辑。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 418-436
```cpp
418:     const uint8_t* const inputPtr = input.data();
419:     const uint8_t inputZeroPoint = 127;
420:     // Make number of output channels multiple of 8.
421:     // This is the least common denominator for SSE/ARM kernels we have.
422:     size_t num_zero_points_padded = outputChannels() + 8;
423:     std::vector<uint8_t> kernelZeroPoints(num_zero_points_padded, 127);
424:
425:     for (size_t iteration = 0; iteration < iterations(); iteration++) {
426:       std::generate(input.begin(), input.end(), std::ref(u8rng));
427:       std::generate(bias.begin(), bias.end(), std::ref(s32rng));
428:       std::generate(kernelZeroPoints.begin(), kernelZeroPoints.end(), std::ref(u8rng));
429:
430:       uint8_t max_elem, min_elem;
431:       do {
432:         std::generate(kernel.begin(), kernel.end(), std::ref(u8rng));
433:         fillBlockSparseWeights(
434:             kernel.data(),
435:             outputChannels(),
436:             inputChannels(),
```
- EN: Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 437-455
```cpp
437:             rowBlockSize(),
438:             colBlockSize(),
439:             sparsity(),
440:             kernelZeroPoints.data());
441:         max_elem = *std::max_element(kernel.cbegin(), kernel.cend());
442:         min_elem = *std::min_element(kernel.cbegin(), kernel.cend());
443:       } while (max_elem == min_elem);
444:       std::unique_ptr<qnnpack::BCSRMatrix> bcsr_matrix =
445:           qnnpack::generateBlockCSRMatrix<uint32_t>(
446:               kernel.data(),
447:               outputChannels(),
448:               inputChannels(),
449:               rowBlockSize(),
450:               colBlockSize(),
451:               kernelZeroPoints.data());
452:
453:       std::fill(output.begin(), output.end(), 0xA5);
454:       std::fill(output_dynamic.begin(), output_dynamic.end(), 0.0f);
455:       std::fill(accumulators.begin(), accumulators.end(), 0);
```
- EN: Looping logic performs the repeated per-element, per-channel, or per-kernel computation. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 457-482
```cpp
457:       for (size_t i = 0; i < batchSize(); i++) {
458:         for (size_t oc = 0; oc < outputChannels(); oc++) {
459:           accumulators[i * outputChannels() + oc] = bias[oc];
460:         }
461:       }
462:       for (size_t i = 0; i < batchSize(); i++) {
463:         for (size_t oc = 0; oc < outputChannels(); oc++) {
464:           for (size_t ic = 0; ic < inputChannels(); ic++) {
465:             accumulators[i * outputChannels() + oc] +=
466:                 (int32_t(inputPtr[i * inputStride() + ic]) -
467:                  int32_t(inputZeroPoint)) *
468:                 (int32_t(kernel[oc * inputChannels() + ic]) -
469:                  int32_t(kernelZeroPoints[oc]));
470:           }
471:         }
472:       }
473:
474:       // Create dummy min/max for empty inputs.
475:       // These are only used to compute scale and zero point,
476:       // and real callers will just pull those values from the model.
477:       const int32_t accumulatorsMin = accumulators.empty()
478:           ? 0
479:           : *std::min_element(accumulators.cbegin(), accumulators.cend());
480:       const int32_t accumulatorsMax = accumulators.empty()
481:           ? 900
482:           : *std::max_element(accumulators.cbegin(), accumulators.cend());
```
- EN: Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 484-503
```cpp
484:       const double outputScale =
485:           double(uint32_t(accumulatorsMax - accumulatorsMin)) / 255.0;
486:       const uint8_t outputZeroPoint = uint8_t(std::max(
487:           std::min(
488:               lrint(
489:                   127.5 -
490:                   0.5 * double(accumulatorsMin + accumulatorsMax) /
491:                       outputScale),
492:               long(std::numeric_limits<uint8_t>::max())),
493:           long(std::numeric_limits<uint8_t>::min())));
494:
495:       ASSERT_EQ(pytorch_qnnp_status_success, pytorch_qnnp_initialize());
496:       // 1 bcz input_scale and kernel_scale are both 1.
497:       std::vector<float>
498:         requantization_scales(num_zero_points_padded, 1.0 * 1.0 / outputScale);
499:       auto scale_generator = [&]() -> float {return (f32rng()/outputScale);};
500:       std::generate(
501:           requantization_scales.begin(),
502:           requantization_scales.end(),
503:           std::ref(scale_generator));
```
- EN: The block also assembles or returns the result expected by the surrounding operator code. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 该代码块还会组装或返回外围算子代码所需的结果。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 505-523
```cpp
505:       switch(mode) {
506:         case Mode::Runtime:
507:           break;
508:         case Mode::Dynamic: {
509:             // Attention! Bias size must be a multiple of 8.
510:             constexpr size_t kBiasSizeMultiple = 8u;
511:             std::vector<float, AlignedAllocator<float, 32>> bias_float(
512:               (bias.size() + (kBiasSizeMultiple - 1)) & -kBiasSizeMultiple);
513:             std::copy(bias.cbegin(), bias.cend(), bias_float.begin());
514:
515:             pytorch_qnnp_operator_t sparse_gemm = nullptr;
516:
517:             ASSERT_EQ(
518:                 pytorch_qnnp_status_success,
519:                 pytorch_qnnp_create_fully_connected_sparse_dq_nc_q8(
520:                     inputChannels(),
521:                     outputChannels(),
522:                     inputZeroPoint,
523:                     kernelZeroPoints.data(),
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 524-547
```cpp
524:                     bcsr_matrix->col_indices_data_ptr(),
525:                     bcsr_matrix->row_values_data_ptr(),
526:                     bcsr_matrix->values.data(),
527:                     bcsr_matrix->row_block_size,
528:                     bcsr_matrix->col_block_size,
529:                     pytorch_qnnp_sparse_matrix_indices_dtype_uint32_t,
530:                     outputZeroPoint,
531:                     qmin(),
532:                     qmax(),
533:                     0,
534:                     requantization_scales.data(),
535:                     true,
536:                     &sparse_gemm));
537:
538:             ASSERT_EQ(
539:                 pytorch_qnnp_status_success,
540:                 pytorch_qnnp_setup_fully_connected_sparse_dq_nc_q8(
541:                     sparse_gemm,
542:                     batchSize(),
543:                     inputPtr,
544:                     inputStride(),
545:                     bias_float.data(),
546:                     output_dynamic.data(),
547:                     outputStride()));
```
- EN: Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段涉及量化数据处理或打包参数逻辑。

### Lines 549-567
```cpp
549:             ASSERT_EQ(
550:                 pytorch_qnnp_status_success,
551:                 pytorch_qnnp_run_operator(sparse_gemm, nullptr /* thread pool */));
552:
553:             ASSERT_EQ(
554:                 pytorch_qnnp_status_success,
555:                 pytorch_qnnp_delete_operator(sparse_gemm));
556:             sparse_gemm = nullptr;
557:
558:             break;
559:           }
560:         default:
561:           // Undefined!
562:           ASSERT_TRUE(false);
563:       }
564:
565:       switch (mode) {
566:         case Mode::Runtime:
567:           break;
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The code participates in pooling window traversal or pooled-output shape computation.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 这段代码参与池化窗口遍历或池化输出形状计算。

### Lines 568-592
```cpp
568:         case Mode::Dynamic:
569:         {
570:           // Bias is added post scaling, as float.
571:           for (size_t i = 0; i < batchSize(); i++) {
572:             for (size_t oc = 0; oc < outputChannels(); oc++) {
573:               accumulators[i * outputChannels() + oc] -= bias[oc];
574:               accumulators_float[i * outputChannels() + oc] =
575:                 (float)accumulators[i * outputChannels() + oc] *
576:                   requantization_scales[oc] + float(bias[oc]);
577:             }
578:           }
579:
580:           for (size_t i = 0; i < batchSize(); i++) {
581:             for (size_t c = 0; c < outputChannels(); c++) {
582:               ASSERT_NEAR(
583:                   output_dynamic[i * outputChannels() + c],
584:                   accumulators_float[i * outputChannels() + c], 1e-3)
585:                   << "at " << i << ", " << c
586:                   << ": reference = " <<
587:                   accumulators_float[i * outputChannels() + c]
588:                   << ", optimized = " << output_dynamic[i * outputChannels() + c];
589:             }
590:           }
591:         }
592:         break;
```
- EN: Looping logic performs the repeated per-element, per-channel, or per-kernel computation. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 594-612
```cpp
594:         default:
595:           // Undefined!
596:           ASSERT_TRUE(false);
597:       }
598:     }
599:   }
600:
601:  private:
602:   size_t inputChannels_{1};
603:   size_t inputStride_{0};
604:   size_t outputChannels_{1};
605:   size_t outputStride_{0};
606:   size_t batchSize_{1};
607:   uint8_t qmin_{0};
608:   uint8_t qmax_{255};
609:   size_t iterations_{1};
610:   float sparsity_{0.7f};
611:   size_t rowBlockSize_{1};
612:   size_t colBlockSize_{4};
```
- EN: This range continues the file's low-level CPU implementation details and operator glue code.
- CN: 这一段继续展开本文件的底层 CPU 实现细节和算子胶水代码。

### Lines 613-613
```cpp
613: };
```
- EN: This range continues the file's low-level CPU implementation details and operator glue code.
- CN: 这一段继续展开本文件的底层 CPU 实现细节和算子胶水代码。

## Key Concepts / 关键概念

- Quantized arithmetic and packed parameters / 量化算术与打包参数
- Pooling reductions / 池化归约
- Low-level memory access / 底层内存访问
- Shape/container bookkeeping / 形状与容器管理

## Dependencies / 依赖关系

- Standard or third-party headers / 标准库或第三方头文件: `algorithm`, `cmath`, `cstddef`, `cstdlib`, `functional`, `random`, `vector`, `memory`, `pack_block_sparse.h`, `pytorch_qnnpack.h`
- Key helper symbols / 关键辅助符号: `qnnpack`
