# tensor_fill.h — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `tools/util/include/cutlass/util/reference/host/tensor_fill.h`
- **Purpose (EN):** This file declares tensor utilities for host-side reference utilities.
- **目的 (CN):** 该文件声明了面向主机端参考工具的张量工具逻辑。
- **Brief / 简述:** Provides several functions for filling tensors with data.

## Line-by-Line Analysis / 逐行分析
### Lines 1-20
```cpp
1: /***************************************************************************************************
2:  * Copyright (c) 2017 - 2025 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
3:  * SPDX-License-Identifier: BSD-3-Clause
4:  *
5:  * Redistribution and use in source and binary forms, with or without
6:  * modification, are permitted provided that the following conditions are met:
7:  *
8:  * 1. Redistributions of source code must retain the above copyright notice, this
9:  * list of conditions and the following disclaimer.
10:  *
11:  * 2. Redistributions in binary form must reproduce the above copyright notice,
12:  * this list of conditions and the following disclaimer in the documentation
13:  * and/or other materials provided with the distribution.
14:  *
15:  * 3. Neither the name of the copyright holder nor the names of its
16:  * contributors may be used to endorse or promote products derived from
17:  * this software without specific prior written permission.
18:  *
19:  * THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
20:  * AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
```
- **EN:** License header and copyright terms for the file.
- **CN:** 文件的许可证头和版权条款。

### Lines 21-30
```cpp
21:  * IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
22:  * DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE
23:  * FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
24:  * DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
25:  * SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
26:  * CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
27:  * OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
28:  * OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
29:  *
30:  **************************************************************************************************/
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 31-33
```cpp
31: /* \file
32:   \brief Provides several functions for filling tensors with data.
33: */
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 35-35
```cpp
35: #pragma once
```
- **EN:** Uses include guards or pragmas to ensure the declarations are processed only once.
- **CN:** 使用 include guard 或 pragma，确保这些声明只被处理一次。

### Lines 37-37
```cpp
37: // Standard Library includes
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 38-42
```cpp
38: #include <utility>
39: #include <cstdlib>
40: #include <cmath>
41: #include <random>
42: #include <stdexcept>
```
- **EN:** Includes the main dependencies required by this module; notable headers are `utility`, `cstdlib`, `cmath`, `random`, `stdexcept`.
- **CN:** 引入该模块所需的主要依赖；较重要的头文件有 `utility`, `cstdlib`, `cmath`, `random`, `stdexcept`。

### Lines 44-44
```cpp
44: // Cutlass includes
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 45-53
```cpp
45: #include "cutlass/cutlass.h"
46: #include "cutlass/complex.h"
47: #include "cutlass/quaternion.h"
48: #include "cutlass/array.h"
49: #include "cutlass/numeric_types.h"
50: #include "cutlass/subbyte_reference.h"
51: #include "cutlass/tensor_view.h"
52: #include "cutlass/tensor_view_planar_complex.h"
53: #include "cutlass/blas3.h"
```
- **EN:** Includes the main dependencies required by this module; notable headers are `cutlass/cutlass.h`, `cutlass/complex.h`, `cutlass/quaternion.h`, `cutlass/array.h`, `cutlass/numeric_types.h`.
- **CN:** 引入该模块所需的主要依赖；较重要的头文件有 `cutlass/cutlass.h`, `cutlass/complex.h`, `cutlass/quaternion.h`, `cutlass/array.h`, `cutlass/numeric_types.h`。

### Lines 55-56
```cpp
55: #include "cutlass/util/distribution.h"
56: #include "tensor_foreach.h"
```
- **EN:** Includes the main dependencies required by this module; notable headers are `cutlass/util/distribution.h`, `tensor_foreach.h`.
- **CN:** 引入该模块所需的主要依赖；较重要的头文件有 `cutlass/util/distribution.h`, `tensor_foreach.h`。

### Lines 58-58
```cpp
58: ///////////////////////////////////////////////////////////////////////////////////////////////////
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 60-62
```cpp
60: namespace cutlass {
61: namespace reference {
62: namespace host {
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 64-65
```cpp
64: ///////////////////////////////////////////////////////////////////////////////////////////////////
65: ///////////////////////////////////////////////////////////////////////////////////////////////////
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 67-67
```cpp
67: namespace detail {
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 69-72
```cpp
69: template <
70:   typename Element,               ///< Element type
71:   typename Layout>                ///< Layout function
72: struct TensorFillFunc {
```
- **EN:** Introduces `TensorFillFunc`, a type used to support tensor utilities.
- **CN:** 引入 `TensorFillFunc`，即一个用于支持张量工具的类型。

### Lines 74-74
```cpp
74:   using TensorView = TensorView<Element, Layout>;
```
- **EN:** Introduces namespace imports, aliases, or shorthand names used by later code.
- **CN:** 引入后续代码会使用的命名空间、别名或简写名称。

### Lines 76-78
```cpp
76:   //
77:   // Data members
78:   //
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 80-81
```cpp
80:   TensorView view;
81:   Element value;
```
- **EN:** Declares or updates local/member state such as `view`, `value`.
- **CN:** 声明或更新局部/成员状态，例如 `view`, `value`。

### Lines 83-85
```cpp
83:   //
84:   // Methods
85:   //
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 87-90
```cpp
87:   TensorFillFunc(
88:     TensorView const &view_ = TensorView(), 
89:     Element value_ = Element(0)
90:   ): view(view_), value(value_) { }
```
- **EN:** Declares or updates local/member state such as `view_`, `value_`.
- **CN:** 声明或更新局部/成员状态，例如 `view_`, `value_`。

### Lines 92-95
```cpp
92:   void operator()(Coord<Layout::kRank> const & coord) const {
93:     view.at(coord) = value;
94:   }
95: };
```
- **EN:** Implements `operator` and coordinates helper calls such as `at`.
- **CN:** 实现 `operator`，并协调调用 `at` 等辅助逻辑。

### Lines 97-97
```cpp
97: /// Returns a pair of values of the Gaussian distribution generated by the Box Muller method 
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 98-98
```cpp
98: struct BoxMullerFunc {
```
- **EN:** Introduces `BoxMullerFunc`, a type used to support tensor utilities.
- **CN:** 引入 `BoxMullerFunc`，即一个用于支持张量工具的类型。

### Lines 100-100
```cpp
100:   BoxMullerFunc() {}
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 102-106
```cpp
102:   void operator()(
103:     double* rnd,                     ///< Size-2 vector to be filled with random values
104:     double  mean = 0,                ///< Mean of the Gaussian distribution
105:     double  stddev = 1,              ///< Standard deviation of the Gaussian distribution
106:     double  pi = std::acos(-1)) const {
```
- **EN:** Declares or updates local/member state such as `mean`, `stddev`, `pi`.
- **CN:** 声明或更新局部/成员状态，例如 `mean`, `stddev`, `pi`。

### Lines 108-116
```cpp
108:     double u1 = double(std::rand()) / double(RAND_MAX);
109:     double u2 = double(std::rand()) / double(RAND_MAX);
110:     rnd[0] = std::sqrt(-2 * std::log(u1)) * std::cos(2 * pi * u2);
111:     rnd[1] = std::sqrt(-2 * std::log(u1)) * std::sin(2 * pi * u2);
112:     rnd[0] = mean + stddev * rnd[0];
113:     rnd[1] = mean + stddev * rnd[1];
114:   }
115: };
116: } // namespace detail
```
- **EN:** Declares or updates local/member state such as `u1`, `u2`.
- **CN:** 声明或更新局部/成员状态，例如 `u1`, `u2`。

### Lines 118-118
```cpp
118: ///////////////////////////////////////////////////////////////////////////////////////////////////
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 120-120
```cpp
120: /// Fills a tensor with a uniform value
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 121-126
```cpp
121: template <
122:   typename Element,               ///< Element type
123:   typename Layout>                ///< Layout function
124: void TensorFill(
125:   TensorView<Element, Layout> dst,    ///< destination tensor 
126:   Element val = Element(0)) {               ///< value to uniformly fill it with
```
- **EN:** Declares or updates local/member state such as `val`.
- **CN:** 声明或更新局部/成员状态，例如 `val`。

### Lines 128-128
```cpp
128:   detail::TensorFillFunc<Element, Layout> func(dst, val);
```
- **EN:** Implements `func` for this file's main component.
- **CN:** 为该文件的核心组件实现 `func`。

### Lines 130-134
```cpp
130:   TensorForEach(
131:     dst.extent(),
132:     func
133:   );
134: }
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 136-136
```cpp
136: /// Fills a tensor with a uniform value
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 137-142
```cpp
137: template <
138:   typename Element,                                                   ///< Element type
139:   typename Layout>                                                    ///< Layout function
140: void TensorFill(
141:   TensorViewPlanarComplex<Element, Layout> dst,                       ///< destination tensor 
142:   cutlass::complex<Element> val = cutlass::complex<Element>(0)) {     ///< value to uniformly fill it with
```
- **EN:** Declares or updates local/member state such as `val`.
- **CN:** 声明或更新局部/成员状态，例如 `val`。

### Lines 144-146
```cpp
144:   TensorFill(dst.view_real(), val.real());
145:   TensorFill(dst.view_imag(), val.imag());
146: }
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 148-149
```cpp
148: ///////////////////////////////////////////////////////////////////////////////////////////////////
149: ///////////////////////////////////////////////////////////////////////////////////////////////////
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 151-151
```cpp
151: namespace detail {
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 153-154
```cpp
153: template <typename Element>
154: struct RandomGaussianFunc {
```
- **EN:** Introduces `RandomGaussianFunc`, a type used to support tensor utilities.
- **CN:** 引入 `RandomGaussianFunc`，即一个用于支持张量工具的类型。

### Lines 156-162
```cpp
156:   uint64_t seed;
157:   double mean;
158:   double stddev;
159:   int int_scale;
160:   double pi;
161:   double pnz;
162:   bool exclude_zero;
```
- **EN:** Declares or updates local/member state such as `seed`, `mean`, `stddev`, `int_scale`.
- **CN:** 声明或更新局部/成员状态，例如 `seed`, `mean`, `stddev`, `int_scale`。

### Lines 164-166
```cpp
164:   //
165:   // Methods
166:   //
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 167-177
```cpp
167:   RandomGaussianFunc(
168:     uint64_t seed_ = 0, 
169:     double mean_ = 0, 
170:     double stddev_ = 1,
171:     int int_scale_ = -1,
172:     double pnz_ = 1.0,
173:     bool exclude_zero_ = false
174:   ):
175:     seed(seed_), mean(mean_), stddev(stddev_), int_scale(int_scale_), pi(std::acos(-1)), pnz(pnz_), exclude_zero(exclude_zero_) {
176:       std::srand((unsigned)seed);
177:   }
```
- **EN:** Declares or updates local/member state such as `seed_`, `mean_`, `stddev_`, `int_scale_`.
- **CN:** 声明或更新局部/成员状态，例如 `seed_`, `mean_`, `stddev_`, `int_scale_`。

### Lines 179-179
```cpp
179:   /// Compute random value and update RNG state
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 180-180
```cpp
180:   Element operator()() const {
```
- **EN:** Implements `operator` for this file's main component.
- **CN:** 为该文件的核心组件实现 `operator`。

### Lines 182-182
```cpp
182:     // Box-Muller transform to generate random numbers with Normal distribution
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 183-184
```cpp
183:     double u1 = double(std::rand()) / double(RAND_MAX);
184:     double u2 = double(std::rand()) / double(RAND_MAX);
```
- **EN:** Declares or updates local/member state such as `u1`, `u2`.
- **CN:** 声明或更新局部/成员状态，例如 `u1`, `u2`。

### Lines 186-186
```cpp
186:     // Compute Gaussian random value
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 187-188
```cpp
187:     double rnd = std::sqrt(-2 * std::log(u1)) * std::cos(2 * pi * u2);
188:     rnd = mean + stddev * rnd;
```
- **EN:** Declares or updates local/member state such as `rnd`.
- **CN:** 声明或更新局部/成员状态，例如 `rnd`。

### Lines 190-190
```cpp
190:     // Scale and convert final result
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 191-191
```cpp
191:     Element result;
```
- **EN:** Declares or updates local/member state such as `result`.
- **CN:** 声明或更新局部/成员状态，例如 `result`。

### Lines 193-193
```cpp
193:     // Sample from the Bernoulli distribution, and use the result to sample from the Gaussian
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 194-197
```cpp
194:     std::random_device rnd_device;
195:     std::mt19937 bernoulli_rnd(rnd_device());
196:     std::bernoulli_distribution bernoulli_dist(pnz);
197:     bool bernoulli_result = bernoulli_dist(bernoulli_rnd);
```
- **EN:** Implements `bernoulli_rnd` and coordinates helper calls such as `rnd_device`, `bernoulli_dist`.
- **CN:** 实现 `bernoulli_rnd`，并协调调用 `rnd_device`, `bernoulli_dist` 等辅助逻辑。

### Lines 199-199
```cpp
199:     // Sample from the Gaussian distribution for a nonzero element
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 200-211
```cpp
200:     if (bernoulli_result) {
201:       if (int_scale >= 0) {
202:         rnd = double(std::llround(rnd * double(1 << int_scale))) / double(1 << int_scale);
203:         result = static_cast<Element>(rnd);
204:       }
205:       else {
206:         result = static_cast<Element>(rnd);
207:       }
208:     }
209:     else {
210:       result = static_cast<Element>(0);
211:     }
```
- **EN:** Declares or updates local/member state such as `rnd`, `result`.
- **CN:** 声明或更新局部/成员状态，例如 `rnd`, `result`。

### Lines 213-213
```cpp
213:     // Note that exclude_zero = true will disable the bernoulli_result above by unsetting zeros
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 214-221
```cpp
214:     if (exclude_zero && result == Element(0)) {
215:       if (rnd > 0) {
216:         rnd += 1;
217:       } else {
218:         rnd -= 1;
219:       }
220:       result = Element(rnd);
221:     }    
```
- **EN:** Declares or updates local/member state such as `result`.
- **CN:** 声明或更新局部/成员状态，例如 `result`。

### Lines 223-225
```cpp
223:     return result;
224:   }
225: };
```
- **EN:** Returns the computed value, status, or object to the caller.
- **CN:** 向调用者返回计算得到的值、状态或对象。

### Lines 227-227
```cpp
227: /// Partial specialization for initializing a complex value.
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 228-229
```cpp
228: template <typename Element>
229: struct RandomGaussianFunc<complex<Element> > {
```
- **EN:** Introduces `RandomGaussianFunc`, a type used to support tensor utilities.
- **CN:** 引入 `RandomGaussianFunc`，即一个用于支持张量工具的类型。

### Lines 231-237
```cpp
231:   uint64_t seed;
232:   double mean;
233:   double stddev;
234:   int int_scale;
235:   double pi;
236:   double pnz;
237:   bool exclude_zero;
```
- **EN:** Declares or updates local/member state such as `seed`, `mean`, `stddev`, `int_scale`.
- **CN:** 声明或更新局部/成员状态，例如 `seed`, `mean`, `stddev`, `int_scale`。

### Lines 239-241
```cpp
239:   //
240:   // Methods
241:   //
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 242-252
```cpp
242:   RandomGaussianFunc(
243:     uint64_t seed_ = 0, 
244:     double mean_ = 0, 
245:     double stddev_ = 1,
246:     int int_scale_ = -1,
247:     double pnz_ = 1.0,
248:     bool exclude_zero_ = false
249:   ):
250:     seed(seed_), mean(mean_), stddev(stddev_), int_scale(int_scale_), pi(std::acos(-1)), pnz(pnz_), exclude_zero(exclude_zero_) {
251:       std::srand((unsigned)seed);
252:   }
```
- **EN:** Declares or updates local/member state such as `seed_`, `mean_`, `stddev_`, `int_scale_`.
- **CN:** 声明或更新局部/成员状态，例如 `seed_`, `mean_`, `stddev_`, `int_scale_`。

### Lines 254-254
```cpp
254:   /// Compute random value and update RNG state
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 255-255
```cpp
255:   complex<Element> operator()() const {
```
- **EN:** Implements `operator` for this file's main component.
- **CN:** 为该文件的核心组件实现 `operator`。

### Lines 257-257
```cpp
257:     Element reals[2];
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 259-261
```cpp
259:     double rnd[2];
260:     detail::BoxMullerFunc func;
261:     func(rnd, mean, stddev, pi);
```
- **EN:** Declares or updates local/member state such as `func`.
- **CN:** 声明或更新局部/成员状态，例如 `func`。

### Lines 263-263
```cpp
263:     // Sample from the Bernoulli distribution, and use the result to sample from the Gaussian
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 264-267
```cpp
264:     std::random_device rnd_device;
265:     std::mt19937 bernoulli_rnd(rnd_device());
266:     std::bernoulli_distribution bernoulli_dist(pnz);
267:     bool bernoulli_result = bernoulli_dist(bernoulli_rnd);
```
- **EN:** Implements `bernoulli_rnd` and coordinates helper calls such as `rnd_device`, `bernoulli_dist`.
- **CN:** 实现 `bernoulli_rnd`，并协调调用 `rnd_device`, `bernoulli_dist` 等辅助逻辑。

### Lines 269-269
```cpp
269:     // Sample from the Gaussian distribution for a nonzero element
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 270-283
```cpp
270:     if (bernoulli_result) {
271:       if (int_scale >= 0) {
272:         rnd[0] = double(std::llround(rnd[0] * double(1 << int_scale)));
273:         rnd[1] = double(std::llround(rnd[1] * double(1 << int_scale)));
274:         reals[0] = from_real<Element>(rnd[0] / double(1 << int_scale));
275:         reals[1] = from_real<Element>(rnd[1] / double(1 << int_scale));
276:       }
277:       else {
278:         reals[0] = from_real<Element>(rnd[0]);
279:         reals[1] = from_real<Element>(rnd[1]);
280:       }
281:     }
282:     else {
283:       reals[0] = from_real<Element>(0);
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 284-285
```cpp
284:       reals[1] = from_real<Element>(0);
285:     }
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 287-287
```cpp
287:     // Note that this will invalidate the above else statement because it unsets zero elements
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 288-290
```cpp
288:     if (exclude_zero &&
289:         reals[0] == from_real<Element>(0.0) &&
290:         reals[1] == from_real<Element>(0.0)) {
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 292-298
```cpp
292:       if (rnd[0] > 0.0) {
293:         rnd[0] += 1.0;
294:       } else {
295:         rnd[0] -= 1.0;
296:       }
297:       reals[0] = from_real<Element>(rnd[0]);
298:     }
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 300-302
```cpp
300:     return complex<Element>(reals[0], reals[1]);
301:   }
302: };
```
- **EN:** Returns the computed value, status, or object to the caller.
- **CN:** 向调用者返回计算得到的值、状态或对象。

### Lines 304-304
```cpp
304: /// Partial specialization for initializing a complex value.
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 305-306
```cpp
305: template <typename Element>
306: struct RandomGaussianFunc<Quaternion<Element> > {
```
- **EN:** Introduces `RandomGaussianFunc`, a type used to support tensor utilities.
- **CN:** 引入 `RandomGaussianFunc`，即一个用于支持张量工具的类型。

### Lines 308-314
```cpp
308:   uint64_t seed;
309:   double mean;
310:   double stddev;
311:   int int_scale;
312:   double pi;
313:   double pnz;
314:   bool exclude_zero;
```
- **EN:** Declares or updates local/member state such as `seed`, `mean`, `stddev`, `int_scale`.
- **CN:** 声明或更新局部/成员状态，例如 `seed`, `mean`, `stddev`, `int_scale`。

### Lines 316-318
```cpp
316:   //
317:   // Methods
318:   //
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 319-329
```cpp
319:   RandomGaussianFunc(
320:     uint64_t seed_ = 0,
321:     double mean_ = 0,
322:     double stddev_ = 1,
323:     int int_scale_ = -1,
324:     double pnz_ = 1.0,
325:     bool exclude_zero_ = false
326:   ):
327:     seed(seed_), mean(mean_), stddev(stddev_), int_scale(int_scale_), pi(std::acos(-1)), pnz(pnz_), exclude_zero(exclude_zero_) {
328:       std::srand((unsigned)seed);
329:   }
```
- **EN:** Declares or updates local/member state such as `seed_`, `mean_`, `stddev_`, `int_scale_`.
- **CN:** 声明或更新局部/成员状态，例如 `seed_`, `mean_`, `stddev_`, `int_scale_`。

### Lines 331-331
```cpp
331:   /// Compute random value and update RNG state
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 332-332
```cpp
332:   Quaternion<Element> operator()() const {
```
- **EN:** Implements `operator` for this file's main component.
- **CN:** 为该文件的核心组件实现 `operator`。

### Lines 334-334
```cpp
334:     Element reals[4];
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 336-340
```cpp
336:     double rnd1[2];
337:     double rnd2[2];
338:     detail::BoxMullerFunc func;
339:     func(rnd1, mean, stddev, pi);
340:     func(rnd2, mean, stddev, pi);
```
- **EN:** Declares or updates local/member state such as `func`.
- **CN:** 声明或更新局部/成员状态，例如 `func`。

### Lines 342-342
```cpp
342:     // Sample from the Bernoulli distribution, and use the result to sample from the Gaussian
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 343-346
```cpp
343:     std::random_device rnd_device;
344:     std::mt19937 bernoulli_rnd(rnd_device());
345:     std::bernoulli_distribution bernoulli_dist(pnz);
346:     bool bernoulli_result = bernoulli_dist(bernoulli_rnd);
```
- **EN:** Implements `bernoulli_rnd` and coordinates helper calls such as `rnd_device`, `bernoulli_dist`.
- **CN:** 实现 `bernoulli_rnd`，并协调调用 `rnd_device`, `bernoulli_dist` 等辅助逻辑。

### Lines 348-348
```cpp
348:     // Sample from the Gaussian distribution for a nonzero element
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 349-354
```cpp
349:     if (bernoulli_result) {
350:       if (int_scale >= 0) {
351:         rnd1[0] = double(std::llround(rnd1[0] * double(1 << int_scale)));
352:         rnd1[1] = double(std::llround(rnd1[1] * double(1 << int_scale)));
353:         rnd2[0] = double(std::llround(rnd2[0] * double(1 << int_scale)));
354:         rnd2[1] = double(std::llround(rnd2[1] * double(1 << int_scale)));
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 356-369
```cpp
356:         reals[0] = from_real<Element>(rnd1[0] / double(1 << int_scale));
357:         reals[1] = from_real<Element>(rnd1[1] / double(1 << int_scale));
358:         reals[2] = from_real<Element>(rnd2[0] / double(1 << int_scale));
359:         reals[3] = from_real<Element>(rnd2[1] / double(1 << int_scale));
360:       }
361:       else {
362:         reals[0] = from_real<Element>(rnd1[0]);
363:         reals[1] = from_real<Element>(rnd1[1]);
364:         reals[2] = from_real<Element>(rnd2[0]);
365:         reals[3] = from_real<Element>(rnd2[1]);
366:       }
367:     }
368:     else {
369:       reals[0] = from_real<Element>(0);
```
- **EN:** Implements `double` for this file's main component.
- **CN:** 为该文件的核心组件实现 `double`。

### Lines 370-373
```cpp
370:       reals[1] = from_real<Element>(0);
371:       reals[2] = from_real<Element>(0);
372:       reals[3] = from_real<Element>(0);
373:     }
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 375-375
```cpp
375:     // Note that this will invalidate the above else statement because it unsets zero elements
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 376-380
```cpp
376:     if (exclude_zero &&
377:         reals[0] == from_real<Element>(0) &&
378:         reals[1] == from_real<Element>(0) &&
379:         reals[2] == from_real<Element>(0) &&
380:         reals[3] == from_real<Element>(0)) {
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 382-388
```cpp
382:       if (rnd1[0] > 0.0) {
383:         rnd1[0] += 1.0;
384:       } else {
385:         rnd1[0] -= 1.0;
386:       }
387:       reals[0] = from_real<Element>(rnd1[0]);
388:     }
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 390-392
```cpp
390:     return Quaternion<Element>(reals[0], reals[1], reals[2], reals[3]);
391:   }
392: };
```
- **EN:** Returns the computed value, status, or object to the caller.
- **CN:** 向调用者返回计算得到的值、状态或对象。

### Lines 394-394
```cpp
394: /// Computes a random Gaussian distribution
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 395-398
```cpp
395: template <
396:   typename Element,               ///< Element type
397:   typename Layout>                ///< Layout function
398: struct TensorFillGaussianFunc {
```
- **EN:** Introduces `TensorFillGaussianFunc`, a type used to support tensor utilities.
- **CN:** 引入 `TensorFillGaussianFunc`，即一个用于支持张量工具的类型。

### Lines 400-400
```cpp
400:   using TensorView = TensorView<Element, Layout>;
```
- **EN:** Introduces namespace imports, aliases, or shorthand names used by later code.
- **CN:** 引入后续代码会使用的命名空间、别名或简写名称。

### Lines 402-404
```cpp
402:   //
403:   // Data members
404:   //
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 406-407
```cpp
406:   TensorView view;
407:   RandomGaussianFunc<Element> func;
```
- **EN:** Declares or updates local/member state such as `view`, `func`.
- **CN:** 声明或更新局部/成员状态，例如 `view`, `func`。

### Lines 409-411
```cpp
409:   //
410:   // Methods
411:   //
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 413-413
```cpp
413:   /// Construction of Gaussian RNG functor.
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 414-418
```cpp
414:   TensorFillGaussianFunc(
415:     TensorView view_ = TensorView(),
416:     RandomGaussianFunc<Element> func_ = RandomGaussianFunc<Element>()
417:   ):
418:     view(view_), func(func_) {
```
- **EN:** Declares or updates local/member state such as `view_`, `func_`.
- **CN:** 声明或更新局部/成员状态，例如 `view_`, `func_`。

### Lines 420-420
```cpp
420:   }
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 422-422
```cpp
422:   /// Compute random value and update RNG state
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 423-426
```cpp
423:   void operator()(Coord<Layout::kRank> const &coord) const {
424:     view.at(coord) = func();
425:   }
426: };
```
- **EN:** Implements `operator` and coordinates helper calls such as `at`, `func`.
- **CN:** 实现 `operator`，并协调调用 `at`, `func` 等辅助逻辑。

### Lines 428-428
```cpp
428: /// Computes a random Gaussian distribution for a rank-2 tensor
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 429-432
```cpp
429: template <
430:   typename Element,               ///< Element type
431:   typename Layout>                ///< Layout function
432: struct TensorFillSymmetricGaussianFunc {
```
- **EN:** Introduces `TensorFillSymmetricGaussianFunc`, a type used to support tensor utilities.
- **CN:** 引入 `TensorFillSymmetricGaussianFunc`，即一个用于支持张量工具的类型。

### Lines 434-434
```cpp
434:   using TensorView = TensorView<Element, Layout>;
```
- **EN:** Introduces namespace imports, aliases, or shorthand names used by later code.
- **CN:** 引入后续代码会使用的命名空间、别名或简写名称。

### Lines 436-438
```cpp
436:   //
437:   // Data members
438:   //
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 440-442
```cpp
440:   TensorView view;
441:   RandomGaussianFunc<Element> func;
442:   cutlass::FillMode fill_mode;
```
- **EN:** Declares or updates local/member state such as `view`, `func`, `fill_mode`.
- **CN:** 声明或更新局部/成员状态，例如 `view`, `func`, `fill_mode`。

### Lines 444-446
```cpp
444:   //
445:   // Methods
446:   //
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 448-448
```cpp
448:   /// Construction of Gaussian RNG functor.
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 449-454
```cpp
449:   TensorFillSymmetricGaussianFunc(
450:     TensorView view_ = TensorView(),
451:     RandomGaussianFunc<Element> func_ = RandomGaussianFunc<Element>(),
452:     cutlass::FillMode fill_mode_ = cutlass::FillMode::kInvalid
453:   ):
454:     view(view_), func(func_), fill_mode(fill_mode_) {
```
- **EN:** Declares or updates local/member state such as `view_`, `func_`, `fill_mode_`.
- **CN:** 声明或更新局部/成员状态，例如 `view_`, `func_`, `fill_mode_`。

### Lines 456-456
```cpp
456:   }
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 458-458
```cpp
458:   /// Compute random value and update RNG state
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 459-459
```cpp
459:   void operator()(Coord<Layout::kRank> const &coord) const {
```
- **EN:** Implements `operator` for this file's main component.
- **CN:** 为该文件的核心组件实现 `operator`。

### Lines 460-460
```cpp
460:     // Fill half of matrix based on FillMode
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 461-471
```cpp
461:     if (Layout::kRank == 2 && 
462:         fill_mode == cutlass::FillMode::kLower &&
463:         coord[0] >= coord[1]) {
464:       view.at(coord) = func();
465:     } else if (Layout::kRank == 2 && 
466:         fill_mode == cutlass::FillMode::kUpper &&
467:         coord[0] <= coord[1]) {
468:       view.at(coord) = func();
469:     }
470:   }
471: };
```
- **EN:** Declares or updates local/member state such as `kRank`, `fill_mode`.
- **CN:** 声明或更新局部/成员状态，例如 `kRank`, `fill_mode`。

### Lines 473-473
```cpp
473: } // namespace detail
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 475-475
```cpp
475: ///////////////////////////////////////////////////////////////////////////////////////////////////
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 477-477
```cpp
477: /// Fills a tensor with random values with a Gaussian distribution.
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 478-487
```cpp
478: template <
479:   typename Element,               ///< Element type
480:   typename Layout>                ///< Layout function
481: void TensorFillRandomGaussian(
482:   TensorView<Element, Layout> dst,        ///< destination tensor
483:   uint64_t seed,                          ///< seed for RNG
484:   double mean = 0,                        ///< Gaussian distribution's mean
485:   double stddev = 1,                      ///< Gaussian distribution's standard deviation
486:   int bits = -1,                          ///< If non-negative, specifies number of fractional bits that 
487:   double pnz = 1.0,                     ///  are not truncated to zero. Permits reducing precision of
```
- **EN:** Declares or updates local/member state such as `mean`, `stddev`, `bits`, `pnz`.
- **CN:** 声明或更新局部/成员状态，例如 `mean`, `stddev`, `bits`, `pnz`。

### Lines 488-488
```cpp
488:                                           ///  data.
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 489-489
```cpp
489:   bool exclude_zero = false) {            ///< Exclude zeros from tensor init.
```
- **EN:** Declares or updates local/member state such as `exclude_zero`.
- **CN:** 声明或更新局部/成员状态，例如 `exclude_zero`。

### Lines 491-491
```cpp
491:   detail::RandomGaussianFunc<Element> random_func(seed, mean, stddev, bits, pnz, exclude_zero);
```
- **EN:** Implements `random_func` for this file's main component.
- **CN:** 为该文件的核心组件实现 `random_func`。

### Lines 493-496
```cpp
493:   detail::TensorFillGaussianFunc<Element, Layout> func(
494:     dst,
495:     random_func
496:   );
```
- **EN:** Implements `func` for this file's main component.
- **CN:** 为该文件的核心组件实现 `func`。

### Lines 498-502
```cpp
498:   TensorForEach(
499:     dst.extent(),
500:     func
501:   );
502: }
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 504-504
```cpp
504: /// Fills a tensor with random values with a Gaussian distribution.
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 505-514
```cpp
505: template <
506:   typename Element,               ///< Element type
507:   typename Layout>                ///< Layout function
508: void TensorFillRandomGaussian(
509:   TensorViewPlanarComplex<Element, Layout> dst,         ///< destination tensor
510:   uint64_t seed,                                        ///< seed for RNG
511:   double mean = 0,                                      ///< Gaussian distribution's mean
512:   double stddev = 1,                                    ///< Gaussian distribution's standard deviation
513:   int bits = -1,                                        ///< If non-negative, specifies number of fractional bits that 
514:   double pnz = 1.0,                                   ///  are not truncated to zero. Permits reducing precision of
```
- **EN:** Declares or updates local/member state such as `mean`, `stddev`, `bits`, `pnz`.
- **CN:** 声明或更新局部/成员状态，例如 `mean`, `stddev`, `bits`, `pnz`。

### Lines 515-515
```cpp
515:                                                         ///  data.
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 516-516
```cpp
516:   bool exclude_zero = false) {                          ///< Exclude zeros from tensor init.
```
- **EN:** Declares or updates local/member state such as `exclude_zero`.
- **CN:** 声明或更新局部/成员状态，例如 `exclude_zero`。

### Lines 518-520
```cpp
518:   TensorFillRandomGaussian(dst.view_real(), seed, mean, stddev, bits, pnz);
519:   TensorFillRandomGaussian(dst.view_imag(), ~seed, mean, stddev, bits, pnz);
520: }
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 522-523
```cpp
522: ///////////////////////////////////////////////////////////////////////////////////////////////////
523: /// Fills the upper or lower part of a symmetric rank-2 tensor with random values of a Gaussian distribution.
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 524-534
```cpp
524: template <
525:   typename Element,               ///< Element type
526:   typename Layout>                ///< Layout function
527: void TensorFillSymmetricRandomGaussian(
528:   TensorView<Element, Layout> dst,        ///< destination tensor
529:   uint64_t seed,                          ///< seed for RNG
530:   cutlass::FillMode fill_mode,            ///< FillMode for symmetric matrices
531:   double mean = 0,                        ///< Gaussian distribution's mean
532:   double stddev = 1,                      ///< Gaussian distribution's standard deviation
533:   int bits = -1,                          ///< If non-negative, specifies number of fractional bits that 
534:   double pnz = 1.0) {                   ///  are not truncated to zero. Permits reducing precision of
```
- **EN:** Declares or updates local/member state such as `mean`, `stddev`, `bits`, `pnz`.
- **CN:** 声明或更新局部/成员状态，例如 `mean`, `stddev`, `bits`, `pnz`。

### Lines 535-535
```cpp
535:                                           ///  data.
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 537-537
```cpp
537:   detail::RandomGaussianFunc<Element> random_func(seed, mean, stddev, bits, pnz);
```
- **EN:** Implements `random_func` for this file's main component.
- **CN:** 为该文件的核心组件实现 `random_func`。

### Lines 539-543
```cpp
539:   detail::TensorFillSymmetricGaussianFunc<Element, Layout> func(
540:     dst,
541:     random_func,
542:     fill_mode
543:   );
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 545-549
```cpp
545:   TensorForEach(
546:     dst.extent(),
547:     func
548:   );
549: }
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 551-551
```cpp
551: ///////////////////////////////////////////////////////////////////////////////////////////////////
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 553-553
```cpp
553: /// Fills a tensor with random values of a Gaussian distribution.
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 554-564
```cpp
554: template <
555:   typename Element                        ///< Element type
556: >
557: void BlockFillRandomGaussian(
558:   Element *ptr,                           ///< destination buffer
559:   size_t capacity,                        ///< number of elements
560:   uint64_t seed,                          ///< seed for RNG
561:   double mean = 0,                        ///< Gaussian distribution's mean
562:   double stddev = 1,                      ///< Gaussian distribution's standard deviation
563:   int bits = -1,                          ///< If non-negative, specifies number of fractional bits that 
564:   double pnz = 1.0) {                   ///  are not truncated to zero. Permits reducing precision of
```
- **EN:** Declares or updates local/member state such as `mean`, `stddev`, `bits`, `pnz`.
- **CN:** 声明或更新局部/成员状态，例如 `mean`, `stddev`, `bits`, `pnz`。

### Lines 565-565
```cpp
565:                                           ///  data.
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 568-568
```cpp
568:   detail::RandomGaussianFunc<Element> random_func(seed, mean, stddev, bits, pnz);
```
- **EN:** Implements `random_func` for this file's main component.
- **CN:** 为该文件的核心组件实现 `random_func`。

### Lines 570-573
```cpp
570:   for (size_t i = 0; i < capacity; ++i) {
571:     ReferenceFactory<Element>::get(ptr, i) = random_func();
572:   }
573: }
```
- **EN:** Declares or updates local/member state such as `i`, `capacity`.
- **CN:** 声明或更新局部/成员状态，例如 `i`, `capacity`。

### Lines 575-575
```cpp
575: ///////////////////////////////////////////////////////////////////////////////////////////////////
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 577-577
```cpp
577: ///////////////////////////////////////////////////////////////////////////////////////////////////
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 579-579
```cpp
579: namespace detail {
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 581-582
```cpp
581: template <typename Element>
582: struct RandomUniformFunc {
```
- **EN:** Introduces `RandomUniformFunc`, a type used to support tensor utilities.
- **CN:** 引入 `RandomUniformFunc`，即一个用于支持张量工具的类型。

### Lines 584-584
```cpp
584:   using Real = typename RealType<Element>::Type;
```
- **EN:** Introduces namespace imports, aliases, or shorthand names used by later code.
- **CN:** 引入后续代码会使用的命名空间、别名或简写名称。

### Lines 586-589
```cpp
586:   uint64_t seed;
587:   double range;
588:   double min;
589:   int int_scale;
```
- **EN:** Declares or updates local/member state such as `seed`, `range`, `min`, `int_scale`.
- **CN:** 声明或更新局部/成员状态，例如 `seed`, `range`, `min`, `int_scale`。

### Lines 591-596
```cpp
591:   double pnan;
592: private:
593:   using engine_type = std::mt19937;
594: public:
595:   engine_type bernoulli_rnd;
596:   std::bernoulli_distribution bernoulli_dist;
```
- **EN:** Declares or updates local/member state such as `pnan`, `engine_type`, `mt19937`, `bernoulli_rnd`.
- **CN:** 声明或更新局部/成员状态，例如 `pnan`, `engine_type`, `mt19937`, `bernoulli_rnd`。

### Lines 598-598
```cpp
598:   bool exclude_zero;
```
- **EN:** Declares or updates local/member state such as `exclude_zero`.
- **CN:** 声明或更新局部/成员状态，例如 `exclude_zero`。

### Lines 600-613
```cpp
600:   RandomUniformFunc(
601:     uint64_t seed_ = 0, 
602:     double max = 1,
603:     double min_ = 0,
604:     int int_scale_ = -1,
605:     double pnan_ = 0,
606:     bool exclude_zero_ = false
607:   ):
608:     seed(seed_), range(max - min_), min(min_), int_scale(int_scale_), pnan(pnan_)
609:     , bernoulli_rnd{static_cast<engine_type::result_type>(seed_)}
610:     , bernoulli_dist(pnan_)
611:     , exclude_zero(exclude_zero_) 
612:     {
613:       std::srand((unsigned)seed);
```
- **EN:** Declares or updates local/member state such as `seed_`, `max`, `min_`, `int_scale_`.
- **CN:** 声明或更新局部/成员状态，例如 `seed_`, `max`, `min_`, `int_scale_`。

### Lines 615-615
```cpp
615:       // Handle cases where min = 0 or max = 0 for excluding zeros
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 616-620
```cpp
616:       if (exclude_zero) {
617:         min = (min == 0.0) ? min + 1: min;
618:         range = (max == 0.0) ? range - 1: range; 
619:       }
620:   }
```
- **EN:** Declares or updates local/member state such as `min`, `range`, `max`.
- **CN:** 声明或更新局部/成员状态，例如 `min`, `range`, `max`。

### Lines 623-623
```cpp
623:   /// Compute random value and update RNG state
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 624-624
```cpp
624:   Element operator()() {
```
- **EN:** Implements `operator` for this file's main component.
- **CN:** 为该文件的核心组件实现 `operator`。

### Lines 626-626
```cpp
626:     // Sample from NaN distribution.
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 627-631
```cpp
627:     if constexpr (std::numeric_limits<Element>::has_quiet_NaN) {
628:       if (pnan > 0 && bernoulli_dist(bernoulli_rnd)) {
629:         return Element(NAN);
630:       }
631:     }
```
- **EN:** Returns the computed value, status, or object to the caller.
- **CN:** 向调用者返回计算得到的值、状态或对象。

### Lines 633-633
```cpp
633:     double rnd = double(std::rand()) / double(RAND_MAX);
```
- **EN:** Declares or updates local/member state such as `rnd`.
- **CN:** 声明或更新局部/成员状态，例如 `rnd`。

### Lines 635-635
```cpp
635:     rnd = min + range * rnd;
```
- **EN:** Declares or updates local/member state such as `rnd`.
- **CN:** 声明或更新局部/成员状态，例如 `rnd`。

### Lines 637-638
```cpp
637:     // Random values are cast to integer after scaling by a power of two to facilitate error
638:     // testing
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 639-646
```cpp
639:     Element result;
640:     if (int_scale >= 0) {
641:       rnd = double(std::llround(rnd * double(1 << int_scale))) / double(1 << int_scale);
642:       result = static_cast<Element>(Real(rnd));
643:     }
644:     else {
645:       result = static_cast<Element>(Real(rnd));
646:     }
```
- **EN:** Declares or updates local/member state such as `result`, `rnd`.
- **CN:** 声明或更新局部/成员状态，例如 `result`, `rnd`。

### Lines 648-655
```cpp
648:     if (exclude_zero && result == Element(0)) {
649:       if (rnd > 0.0) {
650:         rnd = std::min(min + range, rnd + 1.0);
651:       } else {
652:         rnd = std::max(min, rnd - 1.0);
653:       }
654:       result = static_cast<Element>(Real(rnd));
655:     }
```
- **EN:** Declares or updates local/member state such as `result`, `rnd`.
- **CN:** 声明或更新局部/成员状态，例如 `result`, `rnd`。

### Lines 657-659
```cpp
657:     return result;
658:   }
659: };
```
- **EN:** Returns the computed value, status, or object to the caller.
- **CN:** 向调用者返回计算得到的值、状态或对象。

### Lines 661-661
```cpp
661: /// Partial specialization for initializing a complex value.
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 662-663
```cpp
662: template <typename Element>
663: struct RandomUniformFunc<complex<Element> > {
```
- **EN:** Introduces `RandomUniformFunc`, a type used to support tensor utilities.
- **CN:** 引入 `RandomUniformFunc`，即一个用于支持张量工具的类型。

### Lines 665-665
```cpp
665:   using Real = typename RealType<Element>::Type;
```
- **EN:** Introduces namespace imports, aliases, or shorthand names used by later code.
- **CN:** 引入后续代码会使用的命名空间、别名或简写名称。

### Lines 667-670
```cpp
667:   uint64_t seed;
668:   double range;
669:   double min;
670:   int int_scale;
```
- **EN:** Declares or updates local/member state such as `seed`, `range`, `min`, `int_scale`.
- **CN:** 声明或更新局部/成员状态，例如 `seed`, `range`, `min`, `int_scale`。

### Lines 672-677
```cpp
672:   double pnan;
673: private:
674:   using engine_type = std::mt19937;
675: public:
676:   engine_type bernoulli_rnd;
677:   std::bernoulli_distribution bernoulli_dist;
```
- **EN:** Declares or updates local/member state such as `pnan`, `engine_type`, `mt19937`, `bernoulli_rnd`.
- **CN:** 声明或更新局部/成员状态，例如 `pnan`, `engine_type`, `mt19937`, `bernoulli_rnd`。

### Lines 679-679
```cpp
679:   bool exclude_zero;
```
- **EN:** Declares or updates local/member state such as `exclude_zero`.
- **CN:** 声明或更新局部/成员状态，例如 `exclude_zero`。

### Lines 681-683
```cpp
681:   //
682:   // Methods
683:   //
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 685-697
```cpp
685:   RandomUniformFunc(
686:     uint64_t seed_ = 0, 
687:     double max = 1,
688:     double min_ = 0,
689:     int int_scale_ = -1,
690:     double pnan_ = 0,
691:     bool exclude_zero_ = false
692:   ):
693:     seed(seed_), range(max - min_), min(min_), int_scale(int_scale_), pnan(pnan_)
694:     , bernoulli_rnd{static_cast<engine_type::result_type>(seed_)}
695:     , bernoulli_dist(pnan_)
696:     , exclude_zero(exclude_zero_) {
697:       std::srand((unsigned)seed);
```
- **EN:** Declares or updates local/member state such as `seed_`, `max`, `min_`, `int_scale_`.
- **CN:** 声明或更新局部/成员状态，例如 `seed_`, `max`, `min_`, `int_scale_`。

### Lines 699-699
```cpp
699:       // Handle cases where min = 0 or max = 0 for excluding zeros
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 700-704
```cpp
700:       if (exclude_zero) {
701:         min = (min == 0.0) ? min + 1: min;
702:         range = (max == 0.0) ? range - 1: range; 
703:       }
704:   }
```
- **EN:** Declares or updates local/member state such as `min`, `range`, `max`.
- **CN:** 声明或更新局部/成员状态，例如 `min`, `range`, `max`。

### Lines 707-707
```cpp
707:   /// Compute random value and update RNG state
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 708-708
```cpp
708:   complex<Element> operator()() {
```
- **EN:** Implements `operator` for this file's main component.
- **CN:** 为该文件的核心组件实现 `operator`。

### Lines 710-710
```cpp
710:     // Sample from NaN distribution.
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 711-715
```cpp
711:     if constexpr (std::numeric_limits<Element>::has_quiet_NaN) {
712:       if (pnan > 0 && bernoulli_dist(bernoulli_rnd)) {
713:         return Element(NAN);
714:       }
715:     }
```
- **EN:** Returns the computed value, status, or object to the caller.
- **CN:** 向调用者返回计算得到的值、状态或对象。

### Lines 717-717
```cpp
717:     Element reals[2];
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 719-720
```cpp
719:     for (int i = 0; i < 2; ++i) {
720:       double rnd = double(std::rand()) / double(RAND_MAX);
```
- **EN:** Declares or updates local/member state such as `i`, `rnd`.
- **CN:** 声明或更新局部/成员状态，例如 `i`, `rnd`。

### Lines 722-722
```cpp
722:       rnd = min + range * rnd;
```
- **EN:** Declares or updates local/member state such as `rnd`.
- **CN:** 声明或更新局部/成员状态，例如 `rnd`。

### Lines 724-725
```cpp
724:       // Random values are cast to integer after scaling by a power of two to facilitate error
725:       // testing
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 727-733
```cpp
727:       if (int_scale >= 0) {
728:         rnd = double(std::llround(rnd * double(1 << int_scale)));
729:         reals[i] = from_real<Element>(Real(rnd / double(1 << int_scale)));
730:       }
731:       else {
732:         reals[i] = from_real<Element>(Real(rnd));
733:       }
```
- **EN:** Declares or updates local/member state such as `rnd`.
- **CN:** 声明或更新局部/成员状态，例如 `rnd`。

### Lines 735-737
```cpp
735:       if (exclude_zero && 
736:           i == 0 &&
737:           reals[0] == from_real<Element>(0.0)) {
```
- **EN:** Declares or updates local/member state such as `i`.
- **CN:** 声明或更新局部/成员状态，例如 `i`。

### Lines 739-745
```cpp
739:         if (rnd > 0.0) {
740:           rnd = std::min(min + range, rnd + 1.0);
741:         } else {
742:           rnd = std::max(min, rnd - 1.0);
743:         }
744:         reals[0] = from_real<Element>(Real(rnd));
745:       }
```
- **EN:** Declares or updates local/member state such as `rnd`.
- **CN:** 声明或更新局部/成员状态，例如 `rnd`。

### Lines 747-747
```cpp
747:     }
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 749-751
```cpp
749:     return complex<Element>(reals[0], reals[1]);
750:   }
751: };
```
- **EN:** Returns the computed value, status, or object to the caller.
- **CN:** 向调用者返回计算得到的值、状态或对象。

### Lines 753-753
```cpp
753: /// Partial specialization for initializing a Quaternion value.
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 754-755
```cpp
754: template <typename Element>
755: struct RandomUniformFunc<Quaternion<Element> > {
```
- **EN:** Introduces `RandomUniformFunc`, a type used to support tensor utilities.
- **CN:** 引入 `RandomUniformFunc`，即一个用于支持张量工具的类型。

### Lines 757-757
```cpp
757:   using Real = typename RealType<Element>::Type;
```
- **EN:** Introduces namespace imports, aliases, or shorthand names used by later code.
- **CN:** 引入后续代码会使用的命名空间、别名或简写名称。

### Lines 759-762
```cpp
759:   uint64_t seed;
760:   double range;
761:   double min;
762:   int int_scale;
```
- **EN:** Declares or updates local/member state such as `seed`, `range`, `min`, `int_scale`.
- **CN:** 声明或更新局部/成员状态，例如 `seed`, `range`, `min`, `int_scale`。

### Lines 764-769
```cpp
764:   double pnan;
765: private:
766:   using engine_type = std::mt19937;
767: public:
768:   engine_type bernoulli_rnd;
769:   std::bernoulli_distribution bernoulli_dist;
```
- **EN:** Declares or updates local/member state such as `pnan`, `engine_type`, `mt19937`, `bernoulli_rnd`.
- **CN:** 声明或更新局部/成员状态，例如 `pnan`, `engine_type`, `mt19937`, `bernoulli_rnd`。

### Lines 771-773
```cpp
771:   //
772:   // Methods
773:   //
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 775-787
```cpp
775:   RandomUniformFunc(
776:     uint64_t seed_ = 0,
777:     double max = 1,
778:     double min_ = 0,
779:     int int_scale_ = -1,
780:     double pnan_ = 0
781:   ):
782:     seed(seed_), range(max - min_), min(min_), int_scale(int_scale_), pnan(pnan_),
783:     bernoulli_rnd{static_cast<engine_type::result_type>(seed_)},
784:     bernoulli_dist(pnan_)
785:   {
786:     std::srand((unsigned)seed);
787:   }
```
- **EN:** Declares or updates local/member state such as `seed_`, `max`, `min_`, `int_scale_`.
- **CN:** 声明或更新局部/成员状态，例如 `seed_`, `max`, `min_`, `int_scale_`。

### Lines 790-790
```cpp
790:   /// Compute random value and update RNG state
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 791-791
```cpp
791:   Quaternion<Element> operator()() {
```
- **EN:** Implements `operator` for this file's main component.
- **CN:** 为该文件的核心组件实现 `operator`。

### Lines 793-793
```cpp
793:     // Sample from NaN distribution.
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 794-798
```cpp
794:     if constexpr (std::numeric_limits<Element>::has_quiet_NaN) {
795:       if (pnan > 0 && bernoulli_dist(bernoulli_rnd)) {
796:         return Element(NAN);
797:       }
798:     }
```
- **EN:** Returns the computed value, status, or object to the caller.
- **CN:** 向调用者返回计算得到的值、状态或对象。

### Lines 800-800
```cpp
800:     Element reals[4];
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 802-803
```cpp
802:     for (int i = 0; i < 4; ++i) {
803:       double rnd = double(std::rand()) / double(RAND_MAX);
```
- **EN:** Declares or updates local/member state such as `i`, `rnd`.
- **CN:** 声明或更新局部/成员状态，例如 `i`, `rnd`。

### Lines 805-805
```cpp
805:       rnd = min + range * rnd;
```
- **EN:** Declares or updates local/member state such as `rnd`.
- **CN:** 声明或更新局部/成员状态，例如 `rnd`。

### Lines 807-808
```cpp
807:       // Random values are cast to integer after scaling by a power of two to facilitate error
808:       // testing
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 810-817
```cpp
810:       if (int_scale >= 0) {
811:         rnd = double(std::llround(rnd * double(1 << int_scale)));
812:         reals[i] = from_real<Element>(Real(rnd / double(1 << int_scale)));
813:       }
814:       else {
815:         reals[i] = from_real<Element>(Real(rnd));
816:       }
817:     }
```
- **EN:** Declares or updates local/member state such as `rnd`.
- **CN:** 声明或更新局部/成员状态，例如 `rnd`。

### Lines 819-821
```cpp
819:     return make_Quaternion(reals[0], reals[1], reals[2], reals[3]);
820:   }
821: };
```
- **EN:** Returns the computed value, status, or object to the caller.
- **CN:** 向调用者返回计算得到的值、状态或对象。

### Lines 823-823
```cpp
823: /// Computes a random uniform distribution
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 824-827
```cpp
824: template <
825:   typename Element,               ///< Element type
826:   typename Layout>                ///< Layout function
827: struct TensorFillRandomUniformFunc {
```
- **EN:** Introduces `TensorFillRandomUniformFunc`, a type used to support tensor utilities.
- **CN:** 引入 `TensorFillRandomUniformFunc`，即一个用于支持张量工具的类型。

### Lines 829-829
```cpp
829:   using TensorView = TensorView<Element, Layout>;
```
- **EN:** Introduces namespace imports, aliases, or shorthand names used by later code.
- **CN:** 引入后续代码会使用的命名空间、别名或简写名称。

### Lines 831-833
```cpp
831:   //
832:   // Data members
833:   //
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 835-836
```cpp
835:   TensorView view;
836:   RandomUniformFunc<Element> func;
```
- **EN:** Declares or updates local/member state such as `view`, `func`.
- **CN:** 声明或更新局部/成员状态，例如 `view`, `func`。

### Lines 838-840
```cpp
838:   //
839:   // Methods
840:   //
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 842-842
```cpp
842:   /// Construction of uniform RNG functor.
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 843-847
```cpp
843:   TensorFillRandomUniformFunc(
844:     TensorView view_ = TensorView(),
845:     RandomUniformFunc<Element> func_ = RandomUniformFunc<Element>()
846:   ):
847:     view(view_), func(func_) {
```
- **EN:** Declares or updates local/member state such as `view_`, `func_`.
- **CN:** 声明或更新局部/成员状态，例如 `view_`, `func_`。

### Lines 849-849
```cpp
849:   }
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 851-851
```cpp
851:   /// Compute random value and update RNG state
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 852-852
```cpp
852:   void operator()(Coord<Layout::kRank> const &coord) {
```
- **EN:** Implements `operator` for this file's main component.
- **CN:** 为该文件的核心组件实现 `operator`。

### Lines 854-856
```cpp
854:     view.at(coord) = func();
855:   }
856: };
```
- **EN:** Implements `at` and coordinates helper calls such as `func`.
- **CN:** 实现 `at`，并协调调用 `func` 等辅助逻辑。

### Lines 858-858
```cpp
858: /// Fills the upper or lower part of a symmetric rank-2 tensor with random values of a uniform distribution.
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 859-862
```cpp
859: template <
860:   typename Element,               ///< Element type
861:   typename Layout>                ///< Layout function
862: struct TensorFillSymmetricRandomUniformFunc {
```
- **EN:** Introduces `TensorFillSymmetricRandomUniformFunc`, a type used to support tensor utilities.
- **CN:** 引入 `TensorFillSymmetricRandomUniformFunc`，即一个用于支持张量工具的类型。

### Lines 864-864
```cpp
864:   using TensorView = TensorView<Element, Layout>;
```
- **EN:** Introduces namespace imports, aliases, or shorthand names used by later code.
- **CN:** 引入后续代码会使用的命名空间、别名或简写名称。

### Lines 866-868
```cpp
866:   //
867:   // Data members
868:   //
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 870-872
```cpp
870:   TensorView view;
871:   RandomUniformFunc<Element> func;
872:   cutlass::FillMode fill_mode;
```
- **EN:** Declares or updates local/member state such as `view`, `func`, `fill_mode`.
- **CN:** 声明或更新局部/成员状态，例如 `view`, `func`, `fill_mode`。

### Lines 874-876
```cpp
874:   //
875:   // Methods
876:   //
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 878-878
```cpp
878:   /// Construction of uniform RNG functor.
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 879-884
```cpp
879:   TensorFillSymmetricRandomUniformFunc(
880:     TensorView view_ = TensorView(),
881:     RandomUniformFunc<Element> func_ = RandomUniformFunc<Element>(),
882:     cutlass::FillMode fill_mode_ = cutlass::FillMode::kInvalid
883:   ):
884:     view(view_), func(func_), fill_mode(fill_mode_) {
```
- **EN:** Declares or updates local/member state such as `view_`, `func_`, `fill_mode_`.
- **CN:** 声明或更新局部/成员状态，例如 `view_`, `func_`, `fill_mode_`。

### Lines 886-886
```cpp
886:   }
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 888-888
```cpp
888:   /// Compute random value and update RNG state
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 889-889
```cpp
889:   void operator()(Coord<Layout::kRank> const &coord) {
```
- **EN:** Implements `operator` for this file's main component.
- **CN:** 为该文件的核心组件实现 `operator`。

### Lines 890-890
```cpp
890:     // Fill half of matrix based on FillMode
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 891-901
```cpp
891:     if (Layout::kRank == 2 && 
892:         fill_mode == cutlass::FillMode::kLower &&
893:         coord[0] >= coord[1]) {
894:       view.at(coord) = func();
895:     } else if (Layout::kRank == 2 && 
896:         fill_mode == cutlass::FillMode::kUpper &&
897:         coord[0] <= coord[1]) {
898:       view.at(coord) = func();
899:     }
900:   }
901: };
```
- **EN:** Declares or updates local/member state such as `kRank`, `fill_mode`.
- **CN:** 声明或更新局部/成员状态，例如 `kRank`, `fill_mode`。

### Lines 903-903
```cpp
903: /// Computes a random Uniform distribution and pads diagonal with zeros
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 904-907
```cpp
904: template <
905:   typename Element,               ///< Element type
906:   typename Layout>                ///< Layout function
907: struct TensorFillPadDiagonalRandomUniformFunc {
```
- **EN:** Introduces `TensorFillPadDiagonalRandomUniformFunc`, a type used to support tensor utilities.
- **CN:** 引入 `TensorFillPadDiagonalRandomUniformFunc`，即一个用于支持张量工具的类型。

### Lines 909-909
```cpp
909:   using TensorView = TensorView<Element, Layout>;
```
- **EN:** Introduces namespace imports, aliases, or shorthand names used by later code.
- **CN:** 引入后续代码会使用的命名空间、别名或简写名称。

### Lines 911-913
```cpp
911:   //
912:   // Data members
913:   //
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 915-918
```cpp
915:   TensorView view;
916:   RandomUniformFunc<Element> func;
917:   cutlass::FillMode fill_mode;
918:   int alignment;
```
- **EN:** Declares or updates local/member state such as `view`, `func`, `fill_mode`, `alignment`.
- **CN:** 声明或更新局部/成员状态，例如 `view`, `func`, `fill_mode`, `alignment`。

### Lines 920-922
```cpp
920:   //
921:   // Methods
922:   //
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 924-924
```cpp
924:   /// Construction of uniform RNG functor.
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 925-931
```cpp
925:   TensorFillPadDiagonalRandomUniformFunc(
926:     TensorView view_ = TensorView(),
927:     RandomUniformFunc<Element> func_ = RandomUniformFunc<Element>(),
928:     cutlass::FillMode fill_mode_ = cutlass::FillMode::kInvalid,
929:     int alignment_ = 1
930:   ):
931:     view(view_), func(func_), fill_mode(fill_mode_), alignment(alignment_) {
```
- **EN:** Declares or updates local/member state such as `view_`, `func_`, `fill_mode_`, `alignment_`.
- **CN:** 声明或更新局部/成员状态，例如 `view_`, `func_`, `fill_mode_`, `alignment_`。

### Lines 933-933
```cpp
933:   }
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 935-935
```cpp
935:   /// Compute random value and update RNG state
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 936-936
```cpp
936:   void operator()(Coord<Layout::kRank> const &coord) {
```
- **EN:** Implements `operator` for this file's main component.
- **CN:** 为该文件的核心组件实现 `operator`。

### Lines 937-937
```cpp
937:     // Fill half of matrix based on FillMode
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 938-950
```cpp
938:     if (Layout::kRank == 2 && 
939:         (fill_mode == cutlass::FillMode::kLower) &&
940:         (coord[0] >= coord[1]) || 
941:         ((coord[1] - coord[0]) >= alignment)) {
942:       view.at(coord) = func();
943:     } else if (Layout::kRank == 2 && 
944:         fill_mode == cutlass::FillMode::kUpper &&
945:         (coord[0] <= coord[1]) ||
946:         ((coord[0] - coord[1]) >= alignment)) {
947:       view.at(coord) = func();
948:     }
949:   }
950: };
```
- **EN:** Declares or updates local/member state such as `kRank`, `fill_mode`.
- **CN:** 声明或更新局部/成员状态，例如 `kRank`, `fill_mode`。

### Lines 952-952
```cpp
952: } // namespace detail
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 954-954
```cpp
954: ///////////////////////////////////////////////////////////////////////////////////////////////////
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 956-956
```cpp
956: /// Fills a tensor with random values of a uniform random distribution.
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 957-965
```cpp
957: template <
958:   typename Element,               ///< Element type
959:   typename Layout>                ///< Layout function
960: void TensorFillRandomUniform(
961:   TensorView<Element, Layout> dst,        ///< destination tensor
962:   uint64_t seed,                          ///< seed for RNG
963:   double max = 1,                         ///< upper bound of distribution
964:   double min = 0,                         ///< lower bound for distribution
965:   int bits = -1,                          ///< If non-negative, specifies number of fractional bits that 
```
- **EN:** Declares or updates local/member state such as `max`, `min`, `bits`.
- **CN:** 声明或更新局部/成员状态，例如 `max`, `min`, `bits`。

### Lines 966-967
```cpp
966:                                           ///  are not truncated to zero. Permits reducing precision of
967:                                           ///  data.
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 968-970
```cpp
968:   double pnan = 0,                        ///< Percentage of NaN elements.
969:   bool exclude_zero = false) {            ///< Exclude zero from tensor init  
970:   detail::RandomUniformFunc<Element> random_func(seed, max, min, bits, pnan, exclude_zero);
```
- **EN:** Declares or updates local/member state such as `pnan`, `exclude_zero`.
- **CN:** 声明或更新局部/成员状态，例如 `pnan`, `exclude_zero`。

### Lines 972-975
```cpp
972:   detail::TensorFillRandomUniformFunc<Element, Layout> func(
973:     dst,
974:     random_func
975:   );
```
- **EN:** Implements `func` for this file's main component.
- **CN:** 为该文件的核心组件实现 `func`。

### Lines 977-981
```cpp
977:   TensorForEach(
978:     dst.extent(),
979:     func
980:   );
981: }
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 983-983
```cpp
983: /// Fills a tensor with random values of a uniform random distribution.
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 984-992
```cpp
984: template <
985:   typename Element,               ///< Element type
986:   typename Layout>                ///< Layout function
987: void TensorFillRandomUniform(
988:   TensorViewPlanarComplex<Element, Layout> dst,        ///< destination tensor
989:   uint64_t seed,                                       ///< seed for RNG
990:   double max = 1,                                      ///< upper bound of distribution
991:   double min = 0,                                      ///< lower bound for distribution
992:   int bits = -1,                                       ///< If non-negative, specifies number of fractional bits that
```
- **EN:** Declares or updates local/member state such as `max`, `min`, `bits`.
- **CN:** 声明或更新局部/成员状态，例如 `max`, `min`, `bits`。

### Lines 993-994
```cpp
993:                                                        ///  are not truncated to zero. Permits reducing precision of
994:                                                        ///  data.
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 995-996
```cpp
995:   double pnan = 0,                                     ///< Percentage of NaN elements.
996:   bool exclude_zero = false) {                         ///< Exclude zero from tensor init 
```
- **EN:** Declares or updates local/member state such as `pnan`, `exclude_zero`.
- **CN:** 声明或更新局部/成员状态，例如 `pnan`, `exclude_zero`。

### Lines 998-1000
```cpp
998:   TensorFillRandomUniform(dst.view_real(), seed, max, min, bits, pnan, exclude_zero);
999:   TensorFillRandomUniform(dst.view_imag(), ~seed, max, min, bits, pnan, exclude_zero);
1000: }
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 1003-1003
```cpp
1003: /// Fills a tensor with random values with a uniform random distribution.
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1004-1012
```cpp
1004: template <
1005:   typename Element,               ///< Element type
1006:   typename Layout>                ///< Layout function
1007: void TensorFillRandomUniform(
1008:   TensorView<Quaternion<Element>, Layout> dst,        ///< destination tensor
1009:   uint64_t seed,                                      ///< seed for RNG
1010:   double max = 1,                                     ///< upper bound of distribution
1011:   double min = 0,                                     ///< lower bound for distribution
1012:   int bits = -1) {                                    ///< If non-negative, specifies number of fractional bits that 
```
- **EN:** Declares or updates local/member state such as `max`, `min`, `bits`.
- **CN:** 声明或更新局部/成员状态，例如 `max`, `min`, `bits`。

### Lines 1013-1014
```cpp
1013:                                                       ///  are not truncated to zero. Permits reducing precision of
1014:                                                       ///  data.                 
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1015-1015
```cpp
1015:   detail::RandomUniformFunc<Quaternion<Element>> random_func(seed, max, min, bits);
```
- **EN:** Implements `random_func` for this file's main component.
- **CN:** 为该文件的核心组件实现 `random_func`。

### Lines 1017-1020
```cpp
1017:   detail::TensorFillRandomUniformFunc<Quaternion<Element>, Layout> func(
1018:     dst,
1019:     random_func
1020:   );
```
- **EN:** Implements `func` for this file's main component.
- **CN:** 为该文件的核心组件实现 `func`。

### Lines 1022-1026
```cpp
1022:   TensorForEach(
1023:     dst.extent(),
1024:     func
1025:   );
1026: }
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 1028-1028
```cpp
1028: ///////////////////////////////////////////////////////////////////////////////////////////////////
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1030-1030
```cpp
1030: /// Fills a tensor with random values with a uniform random distribution.
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1031-1040
```cpp
1031: template <
1032:   typename Element,               ///< Element type
1033:   typename Layout>                ///< Layout function
1034: void TensorFillSymmetricRandomUniform(
1035:   TensorView<Element, Layout> dst,        ///< destination tensor
1036:   uint64_t seed,                          ///< seed for RNG
1037:   cutlass::FillMode fill_mode,            ///< FillMode for symmetric matrices
1038:   double max = 1,                         ///< upper bound of distribution
1039:   double min = 0,                         ///< lower bound for distribution
1040:   int bits = -1) {                        ///< If non-negative, specifies number of fractional bits that 
```
- **EN:** Declares or updates local/member state such as `max`, `min`, `bits`.
- **CN:** 声明或更新局部/成员状态，例如 `max`, `min`, `bits`。

### Lines 1041-1042
```cpp
1041:                                           ///  are not truncated to zero. Permits reducing precision of
1042:                                           ///  data.
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1044-1044
```cpp
1044:   detail::RandomUniformFunc<Element> random_func(seed, max, min, bits);
```
- **EN:** Implements `random_func` for this file's main component.
- **CN:** 为该文件的核心组件实现 `random_func`。

### Lines 1046-1050
```cpp
1046:   detail::TensorFillSymmetricRandomUniformFunc<Element, Layout> func(
1047:     dst,
1048:     random_func,
1049:     fill_mode
1050:   );
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 1052-1056
```cpp
1052:   TensorForEach(
1053:     dst.extent(),
1054:     func
1055:   );
1056: }
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 1058-1058
```cpp
1058: /// Fills a tensor with random values with a uniform random distribution pads zeros along diagonal
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1059-1068
```cpp
1059: template <
1060:   typename Element,                       ///< Element type
1061:   typename Layout>                        ///< Layout function
1062: void TensorFillPadDiagonalRandomUniform(
1063:   TensorView<Element, Layout> dst,        ///< destination tensor
1064:   uint64_t seed,                          ///< seed for RNG
1065:   cutlass::FillMode fill_mode,            ///< FillMode for symmetric matrices
1066:   double max = 1,                         ///< upper bound of distribution
1067:   double min = 0,                         ///< lower bound for distribution
1068:   int bits = -1,                          ///< If non-negative, specifies number of fractional bits that 
```
- **EN:** Declares or updates local/member state such as `max`, `min`, `bits`.
- **CN:** 声明或更新局部/成员状态，例如 `max`, `min`, `bits`。

### Lines 1069-1070
```cpp
1069:                                           ///  are not truncated to zero. Permits reducing precision of
1070:                                           ///  data.
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1071-1072
```cpp
1071:   int alignment = 1 
1072: ) {
```
- **EN:** Declares or updates local/member state such as `alignment`.
- **CN:** 声明或更新局部/成员状态，例如 `alignment`。

### Lines 1074-1074
```cpp
1074:   detail::RandomUniformFunc<Element> random_func(seed, max, min, bits);
```
- **EN:** Implements `random_func` for this file's main component.
- **CN:** 为该文件的核心组件实现 `random_func`。

### Lines 1076-1081
```cpp
1076:   detail::TensorFillPadDiagonalRandomUniformFunc<Element, Layout> func(
1077:     dst,
1078:     random_func,
1079:     fill_mode,
1080:     alignment
1081:   );
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 1083-1087
```cpp
1083:   TensorForEach(
1084:     dst.extent(),
1085:     func
1086:   );
1087: }
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 1088-1088
```cpp
1088: ///////////////////////////////////////////////////////////////////////////////////////////////////
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1090-1090
```cpp
1090: ///////////////////////////////////////////////////////////////////////////////////////////////////
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1092-1092
```cpp
1092: /// Fills a tensor with a uniform value
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1093-1104
```cpp
1093: template <
1094:   typename Element                        ///< Element type
1095: >
1096: void BlockFill(
1097:   Element *ptr,
1098:   size_t capacity,
1099:   Element val
1100:   ) {                                       
1101:   for (size_t i = 0; i < capacity; ++i) {
1102:     ReferenceFactory<Element>::get(ptr, i) = val;
1103:   }
1104: }
```
- **EN:** Declares or updates local/member state such as `i`, `capacity`, `val`.
- **CN:** 声明或更新局部/成员状态，例如 `i`, `capacity`, `val`。

### Lines 1106-1106
```cpp
1106: /// Fills a tensor with random values with a uniform random distribution.
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1107-1116
```cpp
1107: template <
1108:   typename Element                        ///< Element type
1109: >
1110: void BlockFillRandomUniform(
1111:   Element *ptr,
1112:   size_t capacity,
1113:   uint64_t seed,                          ///< seed for RNG
1114:   double max = 1,                         ///< upper bound of distribution
1115:   double min = 0,                         ///< lower bound for distribution
1116:   int bits = -1,                          ///< If non-negative, specifies number of fractional bits that 
```
- **EN:** Declares or updates local/member state such as `max`, `min`, `bits`.
- **CN:** 声明或更新局部/成员状态，例如 `max`, `min`, `bits`。

### Lines 1117-1118
```cpp
1117:                                           ///  are not truncated to zero. Permits reducing precision of
1118:                                           ///  data.
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1119-1120
```cpp
1119:   double pnan = 0) {                      ///< Percentage of NaN elements.
1120:   detail::RandomUniformFunc<Element> random_func(seed, max, min, bits, pnan);
```
- **EN:** Declares or updates local/member state such as `pnan`.
- **CN:** 声明或更新局部/成员状态，例如 `pnan`。

### Lines 1122-1125
```cpp
1122:   for (size_t i = 0; i < capacity; ++i) {
1123:     ReferenceFactory<Element>::get(ptr, i) = random_func();
1124:   }
1125: }
```
- **EN:** Declares or updates local/member state such as `i`, `capacity`.
- **CN:** 声明或更新局部/成员状态，例如 `i`, `capacity`。

### Lines 1127-1127
```cpp
1127: ///////////////////////////////////////////////////////////////////////////////////////////////////
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1129-1129
```cpp
1129: namespace detail {
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 1131-1134
```cpp
1131: template <
1132:   typename Element,               ///< Element type
1133:   typename Layout>                ///< Layout function
1134: struct TensorFillDiagonalFunc {
```
- **EN:** Introduces `TensorFillDiagonalFunc`, a type used to support tensor utilities.
- **CN:** 引入 `TensorFillDiagonalFunc`，即一个用于支持张量工具的类型。

### Lines 1136-1136
```cpp
1136:   using TensorView = TensorView<Element, Layout>;
```
- **EN:** Introduces namespace imports, aliases, or shorthand names used by later code.
- **CN:** 引入后续代码会使用的命名空间、别名或简写名称。

### Lines 1138-1140
```cpp
1138:   //
1139:   // Data members
1140:   //
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1142-1144
```cpp
1142:   TensorView view;
1143:   Element diag;
1144:   Element other;
```
- **EN:** Declares or updates local/member state such as `view`, `diag`, `other`.
- **CN:** 声明或更新局部/成员状态，例如 `view`, `diag`, `other`。

### Lines 1146-1148
```cpp
1146:   //
1147:   // Methods
1148:   //
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1150-1155
```cpp
1150:   TensorFillDiagonalFunc(
1151:     TensorView const &view_ = TensorView(),
1152:     Element diag_ = Element(1),
1153:     Element other_ = Element(0)
1154:   ):
1155:     view(view_), diag(diag_), other(other_) { }
```
- **EN:** Declares or updates local/member state such as `view_`, `diag_`, `other_`.
- **CN:** 声明或更新局部/成员状态，例如 `view_`, `diag_`, `other_`。

### Lines 1157-1158
```cpp
1157:   void operator()(Coord<Layout::kRank> const & coord) const {
1158:     bool is_diag = true;
```
- **EN:** Implements `operator` for this file's main component.
- **CN:** 为该文件的核心组件实现 `operator`。

### Lines 1160-1166
```cpp
1160:     CUTLASS_PRAGMA_UNROLL
1161:     for (int i = 1; i < Layout::kRank; ++i) {
1162:       if (coord[i] != coord[i - 1]) {
1163:         is_diag = false;
1164:         break;
1165:       }
1166:     }
```
- **EN:** Declares or updates local/member state such as `i`, `kRank`, `is_diag`, `false`.
- **CN:** 声明或更新局部/成员状态，例如 `i`, `kRank`, `is_diag`, `false`。

### Lines 1168-1170
```cpp
1168:     view.at(coord) = (is_diag ? diag : other);
1169:   }
1170: };
```
- **EN:** Implements `at` for this file's main component.
- **CN:** 为该文件的核心组件实现 `at`。

### Lines 1172-1172
```cpp
1172: } // namespace detail
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 1174-1174
```cpp
1174: ///////////////////////////////////////////////////////////////////////////////////////////////////
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1176-1176
```cpp
1176: /// Fills a tensor everywhere with a unique value for its diagonal.
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1177-1183
```cpp
1177: template <
1178:   typename Element,               ///< Element type
1179:   typename Layout>                ///< Layout function
1180: void TensorFillDiagonal(
1181:   TensorView<Element, Layout> dst,        ///< destination tensor
1182:   Element diag = Element(1),              ///< value to write in the diagonal
1183:   Element other = Element(0)) {           ///< value to write off the diagonal
```
- **EN:** Declares or updates local/member state such as `diag`, `other`.
- **CN:** 声明或更新局部/成员状态，例如 `diag`, `other`。

### Lines 1185-1189
```cpp
1185:   detail::TensorFillDiagonalFunc<Element, Layout> func(
1186:     dst,
1187:     diag,
1188:     other
1189:   );
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 1191-1195
```cpp
1191:   TensorForEach(
1192:     dst.extent(),
1193:     func
1194:   );
1195: }
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 1197-1198
```cpp
1197: ///////////////////////////////////////////////////////////////////////////////////////////////////
1198: ///////////////////////////////////////////////////////////////////////////////////////////////////
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1200-1200
```cpp
1200: /// Helper to fill a tensor's diagonal with 1 and 0 everywhere else.
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1201-1205
```cpp
1201: template <
1202:   typename Element,               ///< Element type
1203:   typename Layout>                ///< Layout function
1204: void TensorFillIdentity(
1205:   TensorView<Element, Layout> dst) {               ///< destination tensor
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 1207-1208
```cpp
1207:   TensorFillDiagonal(dst, Element(1), Element(0));
1208: }
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 1210-1211
```cpp
1210: ///////////////////////////////////////////////////////////////////////////////////////////////////
1211: ///////////////////////////////////////////////////////////////////////////////////////////////////
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1213-1213
```cpp
1213: /// Writes a uniform value to the diagonal of a tensor without modifying off-diagonal elements.
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1214-1219
```cpp
1214: template <
1215:   typename Element,               ///< Element type
1216:   typename Layout>                ///< Layout function
1217: void TensorUpdateDiagonal(
1218:   TensorView<Element, Layout> dst,                 ///< destination tensor
1219:   Element val = Element(1)) {
```
- **EN:** Declares or updates local/member state such as `val`.
- **CN:** 声明或更新局部/成员状态，例如 `val`。

### Lines 1221-1221
```cpp
1221:   typename Layout::Index extent = dst.extent().min();
```
- **EN:** Implements `extent` and coordinates helper calls such as `min`.
- **CN:** 实现 `extent`，并协调调用 `min` 等辅助逻辑。

### Lines 1223-1227
```cpp
1223:   for (typename Layout::Index i = 0; i < extent; ++i) {
1224:     Coord<Layout::kRank> coord(i);
1225:     dst.at(coord) = val;
1226:   }
1227: }
```
- **EN:** Declares or updates local/member state such as `i`, `extent`, `val`.
- **CN:** 声明或更新局部/成员状态，例如 `i`, `extent`, `val`。

### Lines 1229-1230
```cpp
1229: ///////////////////////////////////////////////////////////////////////////////////////////////////
1230: ///////////////////////////////////////////////////////////////////////////////////////////////////
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1232-1232
```cpp
1232: namespace detail {
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 1234-1237
```cpp
1234: template <
1235:   typename Element,               ///< Element type
1236:   typename Layout>                ///< Layout function
1237: struct TensorUpdateOffDiagonalFunc {
```
- **EN:** Introduces `TensorUpdateOffDiagonalFunc`, a type used to support tensor utilities.
- **CN:** 引入 `TensorUpdateOffDiagonalFunc`，即一个用于支持张量工具的类型。

### Lines 1239-1239
```cpp
1239:   using TensorView = TensorView<Element, Layout>;
```
- **EN:** Introduces namespace imports, aliases, or shorthand names used by later code.
- **CN:** 引入后续代码会使用的命名空间、别名或简写名称。

### Lines 1241-1243
```cpp
1241:   //
1242:   // Data members
1243:   //
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1245-1246
```cpp
1245:   TensorView view;
1246:   Element other;
```
- **EN:** Declares or updates local/member state such as `view`, `other`.
- **CN:** 声明或更新局部/成员状态，例如 `view`, `other`。

### Lines 1248-1250
```cpp
1248:   //
1249:   // Methods
1250:   //
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1252-1256
```cpp
1252:   TensorUpdateOffDiagonalFunc(
1253:     TensorView const &view_ = TensorView(),
1254:     Element other_ = Element(0)
1255:   ):
1256:     view(view_), other(other_) { }
```
- **EN:** Declares or updates local/member state such as `view_`, `other_`.
- **CN:** 声明或更新局部/成员状态，例如 `view_`, `other_`。

### Lines 1258-1259
```cpp
1258:   void operator()(Coord<Layout::kRank> const & coord) const {
1259:     bool is_diag = true;
```
- **EN:** Implements `operator` for this file's main component.
- **CN:** 为该文件的核心组件实现 `operator`。

### Lines 1261-1267
```cpp
1261:     CUTLASS_PRAGMA_UNROLL
1262:     for (int i = 1; i < Layout::kRank; ++i) {
1263:       if (coord[i] != coord[i - 1]) {
1264:         is_diag = false;
1265:         break;
1266:       }
1267:     }
```
- **EN:** Declares or updates local/member state such as `i`, `kRank`, `is_diag`, `false`.
- **CN:** 声明或更新局部/成员状态，例如 `i`, `kRank`, `is_diag`, `false`。

### Lines 1269-1273
```cpp
1269:     if (!is_diag) {
1270:       view.at(coord) = other;
1271:     }
1272:   }
1273: };
```
- **EN:** Declares or updates local/member state such as `other`.
- **CN:** 声明或更新局部/成员状态，例如 `other`。

### Lines 1275-1275
```cpp
1275: } // namespace detail
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 1277-1277
```cpp
1277: ///////////////////////////////////////////////////////////////////////////////////////////////////
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1279-1279
```cpp
1279: /// Writes a uniform value to all elements in the tensor without modifying diagonal elements.
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1280-1285
```cpp
1280: template <
1281:   typename Element,               ///< Element type
1282:   typename Layout>                ///< Layout function
1283: void TensorUpdateOffDiagonal(
1284:   TensorView<Element, Layout> dst,      ///< destination tensor
1285:   Element other = Element(1)) {
```
- **EN:** Declares or updates local/member state such as `other`.
- **CN:** 声明或更新局部/成员状态，例如 `other`。

### Lines 1287-1290
```cpp
1287:   detail::TensorUpdateOffDiagonalFunc<Element, Layout> func(
1288:     dst,
1289:     other
1290:   );
```
- **EN:** Implements `func` for this file's main component.
- **CN:** 为该文件的核心组件实现 `func`。

### Lines 1292-1296
```cpp
1292:   TensorForEach(
1293:     dst.extent(),
1294:     func
1295:   );
1296: }
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 1299-1300
```cpp
1299: ///////////////////////////////////////////////////////////////////////////////////////////////////
1300: ///////////////////////////////////////////////////////////////////////////////////////////////////
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1302-1302
```cpp
1302: namespace detail {
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 1304-1307
```cpp
1304: template <
1305:   typename Element,               ///< Element type
1306:   typename Layout>                ///< Layout function
1307: struct TensorFillLinearFunc {
```
- **EN:** Introduces `TensorFillLinearFunc`, a type used to support tensor utilities.
- **CN:** 引入 `TensorFillLinearFunc`，即一个用于支持张量工具的类型。

### Lines 1309-1309
```cpp
1309:   using TensorView = TensorView<Element, Layout>;
```
- **EN:** Introduces namespace imports, aliases, or shorthand names used by later code.
- **CN:** 引入后续代码会使用的命名空间、别名或简写名称。

### Lines 1311-1313
```cpp
1311:   //
1312:   // Data members
1313:   //
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1315-1317
```cpp
1315:   TensorView view;
1316:   Array<Element, Layout::kRank> v;
1317:   Element s;
```
- **EN:** Declares or updates local/member state such as `view`, `v`, `s`.
- **CN:** 声明或更新局部/成员状态，例如 `view`, `v`, `s`。

### Lines 1319-1321
```cpp
1319:   //
1320:   // Methods
1321:   //
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1323-1323
```cpp
1323:   TensorFillLinearFunc() { }
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 1325-1325
```cpp
1325:   /// Constructs functor
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1326-1331
```cpp
1326:   TensorFillLinearFunc(
1327:     TensorView const &view_,
1328:     Array<Element, Layout::kRank> const & v_,
1329:     Element s_ = Element(0)
1330:   ):
1331:     view(view_), v(v_), s(s_) { }
```
- **EN:** Declares or updates local/member state such as `s_`.
- **CN:** 声明或更新局部/成员状态，例如 `s_`。

### Lines 1333-1333
```cpp
1333:   /// Updates the tensor
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1334-1334
```cpp
1334:   void operator()(Coord<Layout::kRank> const & coord) const {
```
- **EN:** Implements `operator` for this file's main component.
- **CN:** 为该文件的核心组件实现 `operator`。

### Lines 1336-1336
```cpp
1336:     Element sum(s);
```
- **EN:** Implements `sum` for this file's main component.
- **CN:** 为该文件的核心组件实现 `sum`。

### Lines 1338-1341
```cpp
1338:     CUTLASS_PRAGMA_UNROLL
1339:     for (int i = 0; i < Layout::kRank; ++i) {
1340:       sum += Element(coord[i]) * v[i];
1341:     }
```
- **EN:** Declares or updates local/member state such as `i`, `kRank`.
- **CN:** 声明或更新局部/成员状态，例如 `i`, `kRank`。

### Lines 1343-1345
```cpp
1343:     view.at(coord) = sum;
1344:   }
1345: };
```
- **EN:** Declares or updates local/member state such as `sum`.
- **CN:** 声明或更新局部/成员状态，例如 `sum`。

### Lines 1347-1347
```cpp
1347: } // namespace detail
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 1349-1349
```cpp
1349: ///////////////////////////////////////////////////////////////////////////////////////////////////
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1351-1351
```cpp
1351: /// Fills tensor with a linear combination of its coordinate and another vector
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1352-1358
```cpp
1352: template <
1353:   typename Element,               ///< Element type
1354:   typename Layout>                ///< Layout function
1355: void TensorFillLinear(
1356:   TensorView<Element, Layout> dst,      ///< destination tensor
1357:   Array<Element, Layout::kRank> const & v,
1358:   Element s = Element(0)) {
```
- **EN:** Declares or updates local/member state such as `s`.
- **CN:** 声明或更新局部/成员状态，例如 `s`。

### Lines 1360-1364
```cpp
1360:   detail::TensorFillLinearFunc<Element, Layout> func(
1361:     dst,
1362:     v,
1363:     s
1364:   );
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 1366-1370
```cpp
1366:   TensorForEach(
1367:     dst.extent(),
1368:     func
1369:   );
1370: }
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 1372-1372
```cpp
1372: ///////////////////////////////////////////////////////////////////////////////////////////////////
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1374-1374
```cpp
1374: /// Fills tensor with a linear combination of its coordinate and another vector
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1375-1380
```cpp
1375: template <
1376:   typename Element,               ///< Element type
1377:   typename Layout>                ///< Layout function
1378: void TensorFillSequential(
1379:   TensorView<Element, Layout> dst,     ///< destination tensor
1380:   Element s = Element(0)) {
```
- **EN:** Declares or updates local/member state such as `s`.
- **CN:** 声明或更新局部/成员状态，例如 `s`。

### Lines 1382-1382
```cpp
1382:   Array<Element, Layout::kRank> stride;
```
- **EN:** Declares or updates local/member state such as `stride`.
- **CN:** 声明或更新局部/成员状态，例如 `stride`。

### Lines 1384-1384
```cpp
1384:   stride[0] = Element(1);
```
- **EN:** Implements `Element` for this file's main component.
- **CN:** 为该文件的核心组件实现 `Element`。

### Lines 1386-1389
```cpp
1386:   CUTLASS_PRAGMA_UNROLL
1387:   for (int i = 1; i < Layout::kRank; ++i) {
1388:     stride[i] = stride[i - 1] * Element(dst.extent()[i - 1]);
1389:   }
```
- **EN:** Implements `Element` and coordinates helper calls such as `extent`.
- **CN:** 实现 `Element`，并协调调用 `extent` 等辅助逻辑。

### Lines 1391-1392
```cpp
1391:   TensorFillLinear(dst, stride, s);
1392: }
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 1394-1395
```cpp
1394: ///////////////////////////////////////////////////////////////////////////////////////////////////
1395: ///////////////////////////////////////////////////////////////////////////////////////////////////
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1397-1397
```cpp
1397: /// Fills a tensor with random values from a distribution.
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1398-1405
```cpp
1398: template <
1399:   typename Element,               ///< Element type
1400:   typename Layout>                ///< Layout function
1401: void TensorFillRandom(
1402:   TensorView<Element, Layout> view,       ///< destination tensor
1403:   uint64_t seed,
1404:   Distribution dist,
1405:   bool exclude_zero = false               ///< If true, excludes 0.
```
- **EN:** Declares or updates local/member state such as `exclude_zero`.
- **CN:** 声明或更新局部/成员状态，例如 `exclude_zero`。

### Lines 1406-1407
```cpp
1406:                                           ///  Note that setting this flag will result in more 1's,
1407:                                           ///  as we use a simple mechanism to replace 0's by adding/subtracting 1's.
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1408-1408
```cpp
1408: ) {
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 1410-1410
```cpp
1410:   using Real = typename RealType<Element>::Type;
```
- **EN:** Introduces namespace imports, aliases, or shorthand names used by later code.
- **CN:** 引入后续代码会使用的命名空间、别名或简写名称。

### Lines 1412-1425
```cpp
1412:   if (dist.kind == Distribution::Gaussian) {
1413:     TensorFillRandomGaussian(
1414:       view,
1415:       seed,
1416:       dist.gaussian.mean,
1417:       dist.gaussian.stddev,
1418:       dist.int_scale,
1419:       dist.gaussian.pnz,
1420:       exclude_zero);
1421:   } else if (dist.kind == Distribution::Uniform) {
1422:     TensorFillRandomUniform(
1423:       view,
1424:       seed,
1425:       dist.uniform.max,
```
- **EN:** Declares or updates local/member state such as `kind`.
- **CN:** 声明或更新局部/成员状态，例如 `kind`。

### Lines 1426-1430
```cpp
1426:       dist.uniform.min,
1427:       dist.int_scale,
1428:       dist.uniform.pnan,
1429:       exclude_zero);
1430:   }
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 1431-1431
```cpp
1431: }
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 1433-1434
```cpp
1433: ///////////////////////////////////////////////////////////////////////////////////////////////////
1434: ///////////////////////////////////////////////////////////////////////////////////////////////////
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1436-1436
```cpp
1436: /// Fills a block of data with sequential elements
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1437-1445
```cpp
1437: template <
1438:   typename Element
1439: >
1440: void BlockFillSequential(
1441:   Element *ptr,
1442:   int64_t capacity,
1443:   Element v = Element(1),
1444:   Element s = Element(0)) {
1445:   int i = 0;
```
- **EN:** Declares or updates local/member state such as `v`, `s`, `i`.
- **CN:** 声明或更新局部/成员状态，例如 `v`, `s`, `i`。

### Lines 1447-1449
```cpp
1447:   while (i < capacity) {
1448:     cutlass::ReferenceFactory<Element, (cutlass::sizeof_bits<Element>::value <
1449:                                         8)>::get(ptr, i) = s;
```
- **EN:** Declares or updates local/member state such as `s`.
- **CN:** 声明或更新局部/成员状态，例如 `s`。

### Lines 1451-1454
```cpp
1451:     s = Element(s + v);
1452:     ++i;
1453:   }
1454: }
```
- **EN:** Implements `Element` for this file's main component.
- **CN:** 为该文件的核心组件实现 `Element`。

### Lines 1456-1456
```cpp
1456: /// Fills a block of data with sequential elements
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1457-1466
```cpp
1457: template <
1458:   typename Element
1459: >
1460: void BlockFillSequentialModN(
1461:   Element *ptr,
1462:   int64_t capacity,
1463:   int64_t mod,
1464:   int64_t v = int64_t(1),
1465:   int64_t s = int64_t(0)) {
1466:   int i = 0;
```
- **EN:** Declares or updates local/member state such as `v`, `s`, `i`.
- **CN:** 声明或更新局部/成员状态，例如 `v`, `s`, `i`。

### Lines 1468-1470
```cpp
1468:   while (i < capacity) {
1469:     cutlass::ReferenceFactory<Element, (cutlass::sizeof_bits<Element>::value <
1470:                                         8)>::get(ptr, i) = Element(s);
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 1472-1475
```cpp
1472:     s = int64_t(s + v) % mod;
1473:     ++i;
1474:   }
1475: }
```
- **EN:** Declares or updates local/member state such as `s`, `mod`, `i`.
- **CN:** 声明或更新局部/成员状态，例如 `s`, `mod`, `i`。

### Lines 1477-1478
```cpp
1477: ///////////////////////////////////////////////////////////////////////////////////////////////////
1478: ///////////////////////////////////////////////////////////////////////////////////////////////////
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1480-1480
```cpp
1480: /// Fills a block of data with sequential elements
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1481-1488
```cpp
1481: template <
1482:   typename Element
1483: >
1484: void BlockFillRandom(
1485:   Element *ptr,
1486:   size_t capacity,
1487:   uint64_t seed,
1488:   Distribution dist) {
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 1490-1503
```cpp
1490:   if (dist.kind == Distribution::Gaussian) {
1491:     BlockFillRandomGaussian<Element>(
1492:       ptr, 
1493:       capacity, 
1494:       seed, 
1495:       dist.gaussian.mean, 
1496:       dist.gaussian.stddev, 
1497:       dist.int_scale,
1498:       dist.gaussian.pnz);
1499:   }
1500:   else if (dist.kind == Distribution::Uniform) {
1501:     BlockFillRandomUniform<Element>(
1502:       ptr, 
1503:       capacity, 
```
- **EN:** Declares or updates local/member state such as `kind`.
- **CN:** 声明或更新局部/成员状态，例如 `kind`。

### Lines 1504-1509
```cpp
1504:       seed, 
1505:       dist.uniform.max,
1506:       dist.uniform.min, 
1507:       dist.int_scale,
1508:       dist.uniform.pnan);
1509:   }
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 1510-1510
```cpp
1510: }
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 1512-1513
```cpp
1512: ///////////////////////////////////////////////////////////////////////////////////////////////////
1513: ///////////////////////////////////////////////////////////////////////////////////////////////////
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1515-1515
```cpp
1515: namespace detail {
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 1517-1518
```cpp
1517: template <typename Element>
1518: struct RandomSparseMetaFunc {
```
- **EN:** Introduces `RandomSparseMetaFunc`, a type used to support tensor utilities.
- **CN:** 引入 `RandomSparseMetaFunc`，即一个用于支持张量工具的类型。

### Lines 1520-1522
```cpp
1520:   uint64_t seed;
1521:   int range;
1522:   int MetaSizeInBits;
```
- **EN:** Declares or updates local/member state such as `seed`, `range`, `MetaSizeInBits`.
- **CN:** 声明或更新局部/成员状态，例如 `seed`, `range`, `MetaSizeInBits`。

### Lines 1524-1526
```cpp
1524:   //
1525:   // Methods
1526:   //
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1528-1541
```cpp
1528:   RandomSparseMetaFunc(
1529:     uint64_t seed_ = 0, 
1530:     int MetaSizeInBits_ = 2
1531:   ):
1532:     seed(seed_), MetaSizeInBits(MetaSizeInBits_) {
1533:       std::srand((unsigned)seed);
1534:       if (MetaSizeInBits_ == 2) {
1535:         range = 6;
1536:       }
1537:       else if (MetaSizeInBits_ == 4) {
1538:         range = 2;
1539:       }
1540:       else {
1541:         throw std::invalid_argument("Invalid MetaSizeInBits");
```
- **EN:** Declares or updates local/member state such as `seed_`, `MetaSizeInBits_`, `range`.
- **CN:** 声明或更新局部/成员状态，例如 `seed_`, `MetaSizeInBits_`, `range`。

### Lines 1542-1543
```cpp
1542:       }
1543:     }
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 1545-1545
```cpp
1545:   /// Compute random value and update RNG state
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1546-1548
```cpp
1546:   Element operator()() const {
1547:     Element FourToTwoMeta[6] = {0x4, 0x8, 0x9, 0xc, 0xd, 0xe};
1548:     Element TwoToOneMeta[2] = {0x4, 0xe};
```
- **EN:** Implements `operator` for this file's main component.
- **CN:** 为该文件的核心组件实现 `operator`。

### Lines 1550-1550
```cpp
1550:     Element * MetaArray = (MetaSizeInBits == 2) ? FourToTwoMeta : TwoToOneMeta;
```
- **EN:** Declares or updates local/member state such as `MetaArray`, `MetaSizeInBits`, `TwoToOneMeta`.
- **CN:** 声明或更新局部/成员状态，例如 `MetaArray`, `MetaSizeInBits`, `TwoToOneMeta`。

### Lines 1552-1552
```cpp
1552:     Element result = 0x0;
```
- **EN:** Declares or updates local/member state such as `result`.
- **CN:** 声明或更新局部/成员状态，例如 `result`。

### Lines 1554-1556
```cpp
1554:     for (int i = 0; i < cutlass::sizeof_bits<Element>::value / 4; ++i) {
1555:       int rnd = std::rand() % range;
1556:       Element meta = MetaArray[rnd];
```
- **EN:** Declares or updates local/member state such as `i`, `rnd`, `range`, `meta`.
- **CN:** 声明或更新局部/成员状态，例如 `i`, `rnd`, `range`, `meta`。

### Lines 1558-1559
```cpp
1558:       result = (Element)(result | ((Element)(meta << (i * 4))));
1559:     }
```
- **EN:** Declares or updates local/member state such as `result`.
- **CN:** 声明或更新局部/成员状态，例如 `result`。

### Lines 1561-1563
```cpp
1561:     return result;
1562:   }
1563: };
```
- **EN:** Returns the computed value, status, or object to the caller.
- **CN:** 向调用者返回计算得到的值、状态或对象。

### Lines 1565-1565
```cpp
1565: /// Computes a random sparse meta
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1566-1569
```cpp
1566: template <
1567:   typename Element,               ///< Element type
1568:   typename Layout>                ///< Layout function
1569: struct TensorFillRandomSparseMetaFunc {
```
- **EN:** Introduces `TensorFillRandomSparseMetaFunc`, a type used to support tensor utilities.
- **CN:** 引入 `TensorFillRandomSparseMetaFunc`，即一个用于支持张量工具的类型。

### Lines 1571-1571
```cpp
1571:   using TensorView = TensorView<Element, Layout>;
```
- **EN:** Introduces namespace imports, aliases, or shorthand names used by later code.
- **CN:** 引入后续代码会使用的命名空间、别名或简写名称。

### Lines 1573-1575
```cpp
1573:   //
1574:   // Data members
1575:   //
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1577-1578
```cpp
1577:   TensorView view;
1578:   RandomSparseMetaFunc<Element> func;
```
- **EN:** Declares or updates local/member state such as `view`, `func`.
- **CN:** 声明或更新局部/成员状态，例如 `view`, `func`。

### Lines 1580-1582
```cpp
1580:   //
1581:   // Methods
1582:   //
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1584-1584
```cpp
1584:   /// Construction of Gaussian RNG functor.
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1585-1589
```cpp
1585:   TensorFillRandomSparseMetaFunc(
1586:     TensorView view_ = TensorView(),
1587:     RandomSparseMetaFunc<Element> func_ = RandomSparseMetaFunc<Element>()
1588:   ):
1589:     view(view_), func(func_) {
```
- **EN:** Declares or updates local/member state such as `view_`, `func_`.
- **CN:** 声明或更新局部/成员状态，例如 `view_`, `func_`。

### Lines 1591-1591
```cpp
1591:   }
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 1593-1593
```cpp
1593:   /// Compute random value and update RNG state
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1594-1594
```cpp
1594:   void operator()(Coord<Layout::kRank> const &coord) const {
```
- **EN:** Implements `operator` for this file's main component.
- **CN:** 为该文件的核心组件实现 `operator`。

### Lines 1596-1598
```cpp
1596:     view.at(coord) = func();
1597:   }
1598: };
```
- **EN:** Implements `at` and coordinates helper calls such as `func`.
- **CN:** 实现 `at`，并协调调用 `func` 等辅助逻辑。

### Lines 1600-1600
```cpp
1600: } // namespace detail
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 1602-1602
```cpp
1602: ///////////////////////////////////////////////////////////////////////////////////////////////////
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1604-1604
```cpp
1604: /// Fills a tensor with random values with a uniform random distribution.
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1605-1611
```cpp
1605: template <
1606:   typename Element,                 ///< Element type
1607:   typename Layout>                  ///< Layout function
1608: void TensorFillRandomSparseMeta(
1609:   TensorView<Element, Layout> dst,  ///< destination tensor
1610:   uint64_t seed,                    ///< seed for RNG
1611:   int MetaSizeInBits) {             ///< 2 bit or 4 bit
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 1613-1613
```cpp
1613:   detail::RandomSparseMetaFunc<Element> random_func(seed, MetaSizeInBits);
```
- **EN:** Implements `random_func` for this file's main component.
- **CN:** 为该文件的核心组件实现 `random_func`。

### Lines 1615-1618
```cpp
1615:   detail::TensorFillRandomSparseMetaFunc<Element, Layout> func(
1616:     dst,
1617:     random_func
1618:   );
```
- **EN:** Implements `func` for this file's main component.
- **CN:** 为该文件的核心组件实现 `func`。

### Lines 1620-1624
```cpp
1620:   TensorForEach(
1621:     dst.extent(),
1622:     func
1623:   );
1624: }
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 1626-1626
```cpp
1626: ///////////////////////////////////////////////////////////////////////////////////////////////////
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1628-1628
```cpp
1628: /// Fills a tensor with random values with a uniform random distribution.
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1629-1636
```cpp
1629: template <
1630:   typename Element                        ///< Element type
1631: >
1632: void BlockFillRandomSparseMeta(
1633:   Element *ptr,
1634:   size_t capacity,
1635:   uint64_t seed,                          ///< seed for RNG
1636:   int MetaSizeInBits) {                   ///< 2 bit or 4bit
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 1638-1638
```cpp
1638:   detail::RandomSparseMetaFunc<Element> random_func(seed, MetaSizeInBits);
```
- **EN:** Implements `random_func` for this file's main component.
- **CN:** 为该文件的核心组件实现 `random_func`。

### Lines 1640-1643
```cpp
1640:   for (size_t i = 0; i < capacity; ++i) {
1641:     ptr[i] = random_func();
1642:   }
1643: }
```
- **EN:** Declares or updates local/member state such as `i`, `capacity`.
- **CN:** 声明或更新局部/成员状态，例如 `i`, `capacity`。

### Lines 1645-1646
```cpp
1645: ///////////////////////////////////////////////////////////////////////////////////////////////////
1646: ///////////////////////////////////////////////////////////////////////////////////////////////////
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1648-1648
```cpp
1648: /// Fills a ell block index matrix with random values with a uniform random distribution.
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1649-1655
```cpp
1649: template <
1650:   typename Element,                                ///< Element type
1651:   typename Layout>                                 ///< Layout function
1652: void TensorFillRandomEllIdx(
1653:   TensorView<Element, Layout> dst,                 ///< destination tensor
1654:   uint64_t seed,                                   ///< seed for RNG
1655:   int rows, int ell_cols, int cols) {              ///< dimension of the matrix 
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 1657-1657
```cpp
1657:   std::srand((unsigned)seed);
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 1659-1660
```cpp
1659:   for (int i = 0; i < rows; ++i) {
1660:     int col_idx = std::rand() % cols;
```
- **EN:** Declares or updates local/member state such as `i`, `rows`, `col_idx`, `cols`.
- **CN:** 声明或更新局部/成员状态，例如 `i`, `rows`, `col_idx`, `cols`。

### Lines 1662-1663
```cpp
1662:     for (int j = 0; j < ell_cols; ++j) {
1663:       dst.at({i, j}) = col_idx;
```
- **EN:** Declares or updates local/member state such as `j`, `ell_cols`, `col_idx`.
- **CN:** 声明或更新局部/成员状态，例如 `j`, `ell_cols`, `col_idx`。

### Lines 1665-1674
```cpp
1665:       if (col_idx != -1) {
1666:         if (col_idx == (cols - 1)) {
1667:           col_idx = -1;
1668:         } else {
1669:           col_idx = std::rand() % (cols - col_idx - 1) + col_idx + 1;
1670:         }
1671:       }
1672:     }
1673:   }
1674: }
```
- **EN:** Declares or updates local/member state such as `col_idx`.
- **CN:** 声明或更新局部/成员状态，例如 `col_idx`。

### Lines 1676-1676
```cpp
1676: ///////////////////////////////////////////////////////////////////////////////////////////////////
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1678-1678
```cpp
1678: /// Copies a diagonal in from host memory without modifying off-diagonal elements.
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1679-1684
```cpp
1679: template <
1680:   typename Element,               ///< Element type
1681:   typename Layout>                ///< Layout function
1682: void TensorCopyDiagonalIn(
1683:   TensorView<Element, Layout> dst,          ///< destination tensor
1684:   Element const *ptr) {                     ///< dense buffer of elements
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 1686-1686
```cpp
1686:   typename Layout::Index extent = dst.extent().min();
```
- **EN:** Implements `extent` and coordinates helper calls such as `min`.
- **CN:** 实现 `extent`，并协调调用 `min` 等辅助逻辑。

### Lines 1688-1692
```cpp
1688:   for (typename Layout::Index i = 0; i < extent; ++i) {
1689:     Coord<Layout::kRank> coord(i);
1690:     dst.at(coord) = ReferenceFactory<Element>::get(ptr, i);
1691:   }
1692: }
```
- **EN:** Declares or updates local/member state such as `i`, `extent`.
- **CN:** 声明或更新局部/成员状态，例如 `i`, `extent`。

### Lines 1694-1695
```cpp
1694: ///////////////////////////////////////////////////////////////////////////////////////////////////
1695: ///////////////////////////////////////////////////////////////////////////////////////////////////
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1697-1697
```cpp
1697: /// Copies the diagonal of a tensor into a dense buffer in host memory.
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1698-1703
```cpp
1698: template <
1699:   typename Element,               ///< Element type
1700:   typename Layout>                ///< Layout function
1701: void TensorCopyDiagonalOut(
1702:   Element *ptr,                               ///< dense buffer of elements
1703:   TensorView<Element, Layout> src) {          ///< source tensor
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 1705-1705
```cpp
1705:   typename Layout::Index extent = src.extent().min();
```
- **EN:** Implements `extent` and coordinates helper calls such as `min`.
- **CN:** 实现 `extent`，并协调调用 `min` 等辅助逻辑。

### Lines 1707-1711
```cpp
1707:   for (typename Layout::Index i = 0; i < extent; ++i) {
1708:     Coord<Layout::kRank> coord(i);
1709:     ReferenceFactory<Element>::get(ptr, i) = src.at(coord);
1710:   }
1711: }
```
- **EN:** Declares or updates local/member state such as `i`, `extent`.
- **CN:** 声明或更新局部/成员状态，例如 `i`, `extent`。

### Lines 1713-1714
```cpp
1713: ///////////////////////////////////////////////////////////////////////////////////////////////////
1714: ///////////////////////////////////////////////////////////////////////////////////////////////////
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1716-1718
```cpp
1716: } // namespace host
1717: } // namespace reference
1718: } // namespace cutlass
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

## Key Concepts / 关键概念
- **Template metaprogramming / 模板元编程**
- **Structured type design / 结构化类型设计**
- **Reference implementation / 参考实现**
- **Tensor manipulation / 张量处理**
- **Convolution support / 卷积支持**

## Dependencies / 依赖关系
- **Internal headers / 内部头文件:** `cutlass/cutlass.h`, `cutlass/complex.h`, `cutlass/quaternion.h`, `cutlass/array.h`, `cutlass/numeric_types.h`, `cutlass/subbyte_reference.h`, `cutlass/tensor_view.h`, `cutlass/tensor_view_planar_complex.h`
- **External headers / 外部头文件:** `utility`, `cstdlib`, `cmath`, `random`, `stdexcept`, `tensor_foreach.h`
