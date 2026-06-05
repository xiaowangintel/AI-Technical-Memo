# leaky-relu-operator-tester.h — Code Analysis / 代码分析

## Source / 来源
- File: `aten/src/ATen/native/quantized/cpu/qnnpack/test/leaky-relu-operator-tester.h`
- Repository: `https://github.com/pytorch/pytorch.git` (`/root/xw/pytorch`, commit `27a4844d7fb`)
- Purpose (EN): This file implements or declares native CPU activation kernels, dtype dispatch, and vectorized elementwise execution paths in the ATen native quantized CPU path.
- 用途 (CN): 该文件在 ATen 原生量化 CPU 路径中实现或声明了原生 CPU 激活函数 kernel、数据类型分派以及向量化逐元素执行路径。

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
12: #include <cassert>
```
- EN: This range pulls in required headers, including `algorithm`, `cassert`. It also sets up compile-time guards, feature switches, or platform-specific paths.
- CN: 这一段引入了所需头文件，例如 `algorithm`, `cassert`。 它还建立了编译期开关、特性选择或平台相关分支。

### Lines 13-28
```cpp
13: #include <cmath>
14: #include <cstddef>
15: #include <cstdlib>
16: #include <functional>
17: #include <random>
18: #include <vector>
19:
20: #include <pytorch_qnnpack.h>
21:
22: class LeakyReLUOperatorTester {
23:  public:
24:   inline LeakyReLUOperatorTester& channels(size_t channels) {
25:     assert(channels != 0);
26:     this->channels_ = channels;
27:     return *this;
28:   }
```
- EN: This range pulls in required headers, including `cmath`, `cstddef`, `cstdlib`. The main symbol in this range is `channels`, `LeakyReLUOperatorTester`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段引入了所需头文件，例如 `cmath`, `cstddef`, `cstdlib`。 这一段的主要符号是 `channels`, `LeakyReLUOperatorTester`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 30-40
```cpp
30:   inline size_t channels() const {
31:     return this->channels_;
32:   }
33:
34:   inline LeakyReLUOperatorTester& inputStride(size_t inputStride) {
35:     assert(inputStride != 0);
36:     this->inputStride_ = inputStride;
37:     return *this;
38:   }
39:
40:   inline size_t inputStride() const {
```
- EN: The main symbol in this range is `channels`, `inputStride`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `channels`, `inputStride`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 41-53
```cpp
41:     if (this->inputStride_ == 0) {
42:       return this->channels_;
43:     } else {
44:       assert(this->inputStride_ >= this->channels_);
45:       return this->inputStride_;
46:     }
47:   }
48:
49:   inline LeakyReLUOperatorTester& outputStride(size_t outputStride) {
50:     assert(outputStride != 0);
51:     this->outputStride_ = outputStride;
52:     return *this;
53:   }
```
- EN: The main symbol in this range is `outputStride`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `outputStride`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 55-67
```cpp
55:   inline size_t outputStride() const {
56:     if (this->outputStride_ == 0) {
57:       return this->channels_;
58:     } else {
59:       assert(this->outputStride_ >= this->channels_);
60:       return this->outputStride_;
61:     }
62:   }
63:
64:   inline LeakyReLUOperatorTester& batchSize(size_t batchSize) {
65:     this->batchSize_ = batchSize;
66:     return *this;
67:   }
```
- EN: The main symbol in this range is `outputStride`, `batchSize`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `outputStride`, `batchSize`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 69-82
```cpp
69:   inline size_t batchSize() const {
70:     return this->batchSize_;
71:   }
72:
73:   inline LeakyReLUOperatorTester& negativeSlope(float negativeSlope) {
74:     assert(negativeSlope > 0.0f);
75:     assert(negativeSlope < 1.0f);
76:     this->negativeSlope_ = negativeSlope;
77:     return *this;
78:   }
79:
80:   inline float negativeSlope() const {
81:     return this->negativeSlope_;
82:   }
```
- EN: The main symbol in this range is `batchSize`, `negativeSlope`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `batchSize`, `negativeSlope`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 84-98
```cpp
84:   inline LeakyReLUOperatorTester& inputScale(float inputScale) {
85:     assert(inputScale > 0.0f);
86:     assert(std::isnormal(inputScale));
87:     this->inputScale_ = inputScale;
88:     return *this;
89:   }
90:
91:   inline float inputScale() const {
92:     return this->inputScale_;
93:   }
94:
95:   inline LeakyReLUOperatorTester& inputZeroPoint(uint8_t inputZeroPoint) {
96:     this->inputZeroPoint_ = inputZeroPoint;
97:     return *this;
98:   }
```
- EN: The main symbol in this range is `inputScale`, `inputZeroPoint`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code. Normalization-related state, scaling, or statistics are handled here.
- CN: 这一段的主要符号是 `inputScale`, `inputZeroPoint`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。 这里处理归一化相关的状态、缩放或统计量。

### Lines 100-113
```cpp
100:   inline uint8_t inputZeroPoint() const {
101:     return this->inputZeroPoint_;
102:   }
103:
104:   inline LeakyReLUOperatorTester& outputScale(float outputScale) {
105:     assert(outputScale > 0.0f);
106:     assert(std::isnormal(outputScale));
107:     this->outputScale_ = outputScale;
108:     return *this;
109:   }
110:
111:   inline float outputScale() const {
112:     return this->outputScale_;
113:   }
```
- EN: The main symbol in this range is `inputZeroPoint`, `outputScale`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code. Normalization-related state, scaling, or statistics are handled here.
- CN: 这一段的主要符号是 `inputZeroPoint`, `outputScale`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。 这里处理归一化相关的状态、缩放或统计量。

### Lines 115-127
```cpp
115:   inline LeakyReLUOperatorTester& outputZeroPoint(uint8_t outputZeroPoint) {
116:     this->outputZeroPoint_ = outputZeroPoint;
117:     return *this;
118:   }
119:
120:   inline uint8_t outputZeroPoint() const {
121:     return this->outputZeroPoint_;
122:   }
123:
124:   inline LeakyReLUOperatorTester& qmin(uint8_t qmin) {
125:     this->qmin_ = qmin;
126:     return *this;
127:   }
```
- EN: The main symbol in this range is `outputZeroPoint`, `qmin`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `outputZeroPoint`, `qmin`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 129-140
```cpp
129:   inline uint8_t qmin() const {
130:     return this->qmin_;
131:   }
132:
133:   inline LeakyReLUOperatorTester& qmax(uint8_t qmax) {
134:     this->qmax_ = qmax;
135:     return *this;
136:   }
137:
138:   inline uint8_t qmax() const {
139:     return this->qmax_;
140:   }
```
- EN: The main symbol in this range is `qmin`, `qmax`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `qmin`, `qmax`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 142-154
```cpp
142:   inline LeakyReLUOperatorTester& iterations(size_t iterations) {
143:     this->iterations_ = iterations;
144:     return *this;
145:   }
146:
147:   inline size_t iterations() const {
148:     return this->iterations_;
149:   }
150:
151:   void testQ8() const {
152:     std::random_device randomDevice;
153:     auto rng = std::mt19937(randomDevice());
154:     auto u8rng = std::bind(std::uniform_int_distribution<uint8_t>(), rng);
```
- EN: The main symbol in this range is `iterations`, `testQ8`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `iterations`, `testQ8`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 156-166
```cpp
156:     std::vector<uint8_t> input((batchSize() - 1) * inputStride() + channels());
157:     std::vector<uint8_t> output(
158:         (batchSize() - 1) * outputStride() + channels());
159:     std::vector<float> outputRef(batchSize() * channels());
160:     for (size_t iteration = 0; iteration < iterations(); iteration++) {
161:       std::generate(input.begin(), input.end(), std::ref(u8rng));
162:       std::fill(output.begin(), output.end(), 0xA5);
163:
164:       /* Compute reference results */
165:       for (size_t i = 0; i < batchSize(); i++) {
166:         for (size_t c = 0; c < channels(); c++) {
```
- EN: Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 167-179
```cpp
167:           const float x = inputScale() *
168:               (int32_t(input[i * inputStride() + c]) -
169:                int32_t(inputZeroPoint()));
170:           float y = (x < 0.0f ? x * negativeSlope() : x) / outputScale();
171:           y = std::min<float>(y, int32_t(qmax()) - int32_t(outputZeroPoint()));
172:           y = std::max<float>(y, int32_t(qmin()) - int32_t(outputZeroPoint()));
173:           outputRef[i * channels() + c] = y + float(int32_t(outputZeroPoint()));
174:         }
175:       }
176:
177:       /* Create, setup, run, and destroy LeakyReLU operator */
178:       ASSERT_EQ(pytorch_qnnp_status_success, pytorch_qnnp_initialize());
179:       pytorch_qnnp_operator_t leakyReLUOp = nullptr;
```
- EN: This range continues the file's low-level CPU implementation details and operator glue code.
- CN: 这一段继续展开本文件的底层 CPU 实现细节和算子胶水代码。

### Lines 181-194
```cpp
181:       ASSERT_EQ(
182:           pytorch_qnnp_status_success,
183:           pytorch_qnnp_create_leaky_relu_nc_q8(
184:               channels(),
185:               negativeSlope(),
186:               inputZeroPoint(),
187:               inputScale(),
188:               outputZeroPoint(),
189:               outputScale(),
190:               qmin(),
191:               qmax(),
192:               0,
193:               &leakyReLUOp));
194:       ASSERT_NE(nullptr, leakyReLUOp);
```
- EN: This range continues the file's low-level CPU implementation details and operator glue code.
- CN: 这一段继续展开本文件的底层 CPU 实现细节和算子胶水代码。

### Lines 196-208
```cpp
196:       ASSERT_EQ(
197:           pytorch_qnnp_status_success,
198:           pytorch_qnnp_setup_leaky_relu_nc_q8(
199:               leakyReLUOp,
200:               batchSize(),
201:               input.data(),
202:               inputStride(),
203:               output.data(),
204:               outputStride()));
205:
206:       ASSERT_EQ(
207:           pytorch_qnnp_status_success,
208:           pytorch_qnnp_run_operator(leakyReLUOp, nullptr /* thread pool */));
```
- EN: The code participates in pooling window traversal or pooled-output shape computation.
- CN: 这段代码参与池化窗口遍历或池化输出形状计算。

### Lines 210-225
```cpp
210:       ASSERT_EQ(
211:           pytorch_qnnp_status_success,
212:           pytorch_qnnp_delete_operator(leakyReLUOp));
213:       leakyReLUOp = nullptr;
214:
215:       /* Verify results */
216:       for (size_t i = 0; i < batchSize(); i++) {
217:         for (size_t c = 0; c < channels(); c++) {
218:           ASSERT_NEAR(
219:               float(int32_t(output[i * outputStride() + c])),
220:               outputRef[i * channels() + c],
221:               0.6f);
222:         }
223:       }
224:     }
225:   }
```
- EN: Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 227-237
```cpp
227:  private:
228:   size_t batchSize_{1};
229:   size_t channels_{1};
230:   size_t inputStride_{0};
231:   size_t outputStride_{0};
232:   float negativeSlope_{0.5f};
233:   float outputScale_{0.75f};
234:   uint8_t outputZeroPoint_{133};
235:   float inputScale_{1.25f};
236:   uint8_t inputZeroPoint_{121};
237:   uint8_t qmin_{0};
```
- EN: This range continues the file's low-level CPU implementation details and operator glue code.
- CN: 这一段继续展开本文件的底层 CPU 实现细节和算子胶水代码。

### Lines 238-240
```cpp
238:   uint8_t qmax_{255};
239:   size_t iterations_{15};
240: };
```
- EN: This range continues the file's low-level CPU implementation details and operator glue code.
- CN: 这一段继续展开本文件的底层 CPU 实现细节和算子胶水代码。

## Key Concepts / 关键概念

- Quantized arithmetic and packed parameters / 量化算术与打包参数
- Pooling reductions / 池化归约
- Normalization statistics / 归一化统计
- Shape/container bookkeeping / 形状与容器管理

## Dependencies / 依赖关系

- Standard or third-party headers / 标准库或第三方头文件: `algorithm`, `cassert`, `cmath`, `cstddef`, `cstdlib`, `functional`, `random`, `vector`, `pytorch_qnnpack.h`
- Key helper symbols / 关键辅助符号: `qnnpack`
