# hardswish-operator-tester.h — Code Analysis / 代码分析

## Source / 来源
- File: `aten/src/ATen/native/quantized/cpu/qnnpack/test/hardswish-operator-tester.h`
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
22: class HardswishOperatorTester {
23:  public:
24:   inline HardswishOperatorTester& channels(size_t channels) {
25:     assert(channels != 0);
26:     this->channels_ = channels;
27:     return *this;
28:   }
```
- EN: This range pulls in required headers, including `cmath`, `cstddef`, `cstdlib`. The main symbol in this range is `channels`, `HardswishOperatorTester`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段引入了所需头文件，例如 `cmath`, `cstddef`, `cstdlib`。 这一段的主要符号是 `channels`, `HardswishOperatorTester`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 30-40
```cpp
30:   inline size_t channels() const {
31:     return this->channels_;
32:   }
33:
34:   inline HardswishOperatorTester& inputStride(size_t inputStride) {
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
49:   inline HardswishOperatorTester& outputStride(size_t outputStride) {
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
64:   inline HardswishOperatorTester& batchSize(size_t batchSize) {
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
73:   inline HardswishOperatorTester& inputScale(float inputScale) {
74:     assert(inputScale > 0.0f);
75:     assert(std::isnormal(inputScale));
76:     this->inputScale_ = inputScale;
77:     return *this;
78:   }
79:
80:   inline float inputScale() const {
81:     return this->inputScale_;
82:   }
```
- EN: The main symbol in this range is `batchSize`, `inputScale`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code. Normalization-related state, scaling, or statistics are handled here.
- CN: 这一段的主要符号是 `batchSize`, `inputScale`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。 这里处理归一化相关的状态、缩放或统计量。

### Lines 84-98
```cpp
84:   inline HardswishOperatorTester& inputZeroPoint(uint8_t inputZeroPoint) {
85:     this->inputZeroPoint_ = inputZeroPoint;
86:     return *this;
87:   }
88:
89:   inline uint8_t inputZeroPoint() const {
90:     return this->inputZeroPoint_;
91:   }
92:
93:   inline HardswishOperatorTester& outputScale(float outputScale) {
94:     assert(outputScale > 0.0f);
95:     assert(std::isnormal(outputScale));
96:     this->outputScale_ = outputScale;
97:     return *this;
98:   }
```
- EN: The main symbol in this range is `inputZeroPoint`, `outputScale`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code. Normalization-related state, scaling, or statistics are handled here.
- CN: 这一段的主要符号是 `inputZeroPoint`, `outputScale`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。 这里处理归一化相关的状态、缩放或统计量。

### Lines 100-111
```cpp
100:   inline float outputScale() const {
101:     return this->outputScale_;
102:   }
103:
104:   inline HardswishOperatorTester& outputZeroPoint(uint8_t outputZeroPoint) {
105:     this->outputZeroPoint_ = outputZeroPoint;
106:     return *this;
107:   }
108:
109:   inline uint8_t outputZeroPoint() const {
110:     return this->outputZeroPoint_;
111:   }
```
- EN: The main symbol in this range is `outputScale`, `outputZeroPoint`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `outputScale`, `outputZeroPoint`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 113-125
```cpp
113:   inline HardswishOperatorTester& qmin(uint8_t qmin) {
114:     this->qmin_ = qmin;
115:     return *this;
116:   }
117:
118:   inline uint8_t qmin() const {
119:     return this->qmin_;
120:   }
121:
122:   inline HardswishOperatorTester& qmax(uint8_t qmax) {
123:     this->qmax_ = qmax;
124:     return *this;
125:   }
```
- EN: The main symbol in this range is `qmin`, `qmax`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `qmin`, `qmax`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 127-138
```cpp
127:   inline uint8_t qmax() const {
128:     return this->qmax_;
129:   }
130:
131:   inline HardswishOperatorTester& iterations(size_t iterations) {
132:     this->iterations_ = iterations;
133:     return *this;
134:   }
135:
136:   inline size_t iterations() const {
137:     return this->iterations_;
138:   }
```
- EN: The main symbol in this range is `qmax`, `iterations`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `qmax`, `iterations`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 140-151
```cpp
140:   void testQ8() const {
141:     std::random_device randomDevice;
142:     auto rng = std::mt19937(randomDevice());
143:     auto u8rng = std::bind(std::uniform_int_distribution<uint8_t>(), rng);
144:
145:     std::vector<uint8_t> input((batchSize() - 1) * inputStride() + channels());
146:     std::vector<uint8_t> output(
147:         (batchSize() - 1) * outputStride() + channels());
148:     std::vector<float> outputRef(batchSize() * channels());
149:     for (size_t iteration = 0; iteration < iterations(); iteration++) {
150:       std::generate(input.begin(), input.end(), std::ref(u8rng));
151:       std::fill(output.begin(), output.end(), 0xA5);
```
- EN: The main symbol in this range is `testQ8`, which contributes directly to this file's operator logic. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 这一段的主要符号是 `testQ8`，它们直接构成本文件的算子逻辑。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 153-167
```cpp
153:       /* Compute reference results */
154:       for (size_t i = 0; i < batchSize(); i++) {
155:         for (size_t c = 0; c < channels(); c++) {
156:           const float x = inputScale() *
157:               (int32_t(input[i * inputStride() + c]) -
158:                int32_t(inputZeroPoint()));
159:           const float hardswishX =
160:             x * std::min(std::max(x + 3.0f, 0.0f), 6.0f) / 6.0f;
161:           const float scaledHardswishX = hardswishX / outputScale();
162:           float y = scaledHardswishX;
163:           y = std::min<float>(y, int32_t(qmax()) - int32_t(outputZeroPoint()));
164:           y = std::max<float>(y, int32_t(qmin()) - int32_t(outputZeroPoint()));
165:           outputRef[i * channels() + c] = y + int32_t(outputZeroPoint());
166:         }
167:       }
```
- EN: Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 169-179
```cpp
169:       /* Create, setup, run, and destroy Hardswish operator */
170:       ASSERT_EQ(pytorch_qnnp_status_success, pytorch_qnnp_initialize());
171:       pytorch_qnnp_operator_t hardswishOp = nullptr;
172:
173:       ASSERT_EQ(
174:           pytorch_qnnp_status_success,
175:           pytorch_qnnp_create_hardswish_nc_q8(
176:               channels(),
177:               inputZeroPoint(),
178:               inputScale(),
179:               outputZeroPoint(),
```
- EN: This range continues the file's low-level CPU implementation details and operator glue code.
- CN: 这一段继续展开本文件的底层 CPU 实现细节和算子胶水代码。

### Lines 180-195
```cpp
180:               outputScale(),
181:               qmin(),
182:               qmax(),
183:               0,
184:               &hardswishOp));
185:       ASSERT_NE(nullptr, hardswishOp);
186:
187:       ASSERT_EQ(
188:           pytorch_qnnp_status_success,
189:           pytorch_qnnp_setup_hardswish_nc_q8(
190:               hardswishOp,
191:               batchSize(),
192:               input.data(),
193:               inputStride(),
194:               output.data(),
195:               outputStride()));
```
- EN: This range continues the file's low-level CPU implementation details and operator glue code.
- CN: 这一段继续展开本文件的底层 CPU 实现细节和算子胶水代码。

### Lines 197-207
```cpp
197:       ASSERT_EQ(
198:           pytorch_qnnp_status_success,
199:           pytorch_qnnp_run_operator(hardswishOp, nullptr /* thread pool */));
200:
201:       ASSERT_EQ(
202:           pytorch_qnnp_status_success, pytorch_qnnp_delete_operator(hardswishOp));
203:       hardswishOp = nullptr;
204:
205:       /* Verify results */
206:       for (size_t i = 0; i < batchSize(); i++) {
207:         for (size_t c = 0; c < channels(); c++) {
```
- EN: Looping logic performs the repeated per-element, per-channel, or per-kernel computation. The code participates in pooling window traversal or pooled-output shape computation.
- CN: 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。 这段代码参与池化窗口遍历或池化输出形状计算。

### Lines 208-219
```cpp
208:           ASSERT_NEAR(
209:               float(int32_t(output[i * outputStride() + c])),
210:               outputRef[i * channels() + c],
211:               0.6f);
212:         }
213:       }
214:     }
215:   }
216:
217:  private:
218:   size_t batchSize_{1};
219:   size_t channels_{1};
```
- EN: This range continues the file's low-level CPU implementation details and operator glue code.
- CN: 这一段继续展开本文件的底层 CPU 实现细节和算子胶水代码。

### Lines 220-229
```cpp
220:   size_t inputStride_{0};
221:   size_t outputStride_{0};
222:   float inputScale_{0.75f};
223:   uint8_t inputZeroPoint_{121};
224:   float outputScale_{0.75f};
225:   uint8_t outputZeroPoint_{121};
226:   uint8_t qmin_{0};
227:   uint8_t qmax_{255};
228:   size_t iterations_{15};
229: };
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
