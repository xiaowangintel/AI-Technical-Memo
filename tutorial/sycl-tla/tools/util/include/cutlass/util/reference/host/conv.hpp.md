# conv.hpp — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `tools/util/include/cutlass/util/reference/host/conv.hpp`
- **Purpose (EN):** This file declares conv for host-side reference utilities.
- **目的 (CN):** 该文件声明了面向主机端参考工具的conv逻辑。
- **Brief / 简述:** Reference implementation for CONV in host-side code.

## Line-by-Line Analysis / 逐行分析
### Lines 1-20
```cpp
1: /***************************************************************************************************
2:  * Copyright (c) 2023 - 2025 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
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
31: /*! \file
32:     \brief Reference implementation for CONV in host-side code.
33: */
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 34-34
```cpp
34: #pragma once
```
- **EN:** Uses include guards or pragmas to ensure the declarations are processed only once.
- **CN:** 使用 include guard 或 pragma，确保这些声明只被处理一次。

### Lines 36-36
```cpp
36: /////////////////////////////////////////////////////////////////////////////////////////////////
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 38-40
```cpp
38: #include "cutlass/complex.h"
39: #include "cutlass/numeric_conversion.h"
40: #include "cutlass/epilogue/thread/activation.h"
```
- **EN:** Includes the main dependencies required by this module; notable headers are `cutlass/complex.h`, `cutlass/numeric_conversion.h`, `cutlass/epilogue/thread/activation.h`.
- **CN:** 引入该模块所需的主要依赖；较重要的头文件有 `cutlass/complex.h`, `cutlass/numeric_conversion.h`, `cutlass/epilogue/thread/activation.h`。

### Lines 42-42
```cpp
42: #include "cute/tensor.hpp"
```
- **EN:** Includes the main dependencies required by this module; notable headers are `cute/tensor.hpp`.
- **CN:** 引入该模块所需的主要依赖；较重要的头文件有 `cute/tensor.hpp`。

### Lines 44-44
```cpp
44: #include <cuda_runtime.h>
```
- **EN:** Includes the main dependencies required by this module; notable headers are `cuda_runtime.h`.
- **CN:** 引入该模块所需的主要依赖；较重要的头文件有 `cuda_runtime.h`。

### Lines 46-46
```cpp
46: /////////////////////////////////////////////////////////////////////////////////////////////////
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 48-48
```cpp
48: namespace cutlass::reference::host {
```
- **EN:** Supporting logic for the conv implementation.
- **CN:** conv实现的辅助逻辑。

### Lines 50-50
```cpp
50: /////////////////////////////////////////////////////////////////////////////////////////////////
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 52-52
```cpp
52: namespace detail {
```
- **EN:** Supporting logic for the conv implementation.
- **CN:** conv实现的辅助逻辑。

### Lines 54-65
```cpp
54: template<class EngineAct, class LayoutAct>
55: bool
56: is_activation_in_bounds(
57:     cute::Tensor<EngineAct, LayoutAct> const& activation,
58:     int32_t n_, int32_t d_, int32_t h_, int32_t w_, int32_t c_, int32_t g_) {
59:   return ((g_ >= 0 && g_ < size<5>(activation)) &&
60:           (n_ >= 0 && n_ < size<4>(activation)) &&
61:           (d_ >= 0 && d_ < size<3>(activation)) &&
62:           (h_ >= 0 && h_ < size<2>(activation)) &&
63:           (w_ >= 0 && w_ < size<1>(activation)) &&
64:           (c_ >= 0 && c_ < size<0>(activation)));
65: }
```
- **EN:** Introduces `EngineAct`, a type used to support conv.
- **CN:** 引入 `EngineAct`，即一个用于支持conv的类型。

### Lines 67-77
```cpp
67: template<class EngineAct, class LayoutAct>
68: bool
69: is_activation_in_bounds(
70:     cute::Tensor<EngineAct, LayoutAct> const& activation,
71:     int32_t n_, int32_t h_, int32_t w_, int32_t c_, int32_t g_) {
72:   return ((g_ >= 0 && g_ < size<4>(activation)) &&
73:           (n_ >= 0 && n_ < size<3>(activation)) &&
74:           (h_ >= 0 && h_ < size<2>(activation)) &&
75:           (w_ >= 0 && w_ < size<1>(activation)) &&
76:           (c_ >= 0 && c_ < size<0>(activation)));
77: }
```
- **EN:** Introduces `EngineAct`, a type used to support conv.
- **CN:** 引入 `EngineAct`，即一个用于支持conv的类型。

### Lines 79-88
```cpp
79: template<class EngineAct, class LayoutAct>
80: bool
81: is_activation_in_bounds(
82:     cute::Tensor<EngineAct, LayoutAct> const& activation,
83:     int32_t n_, int32_t w_, int32_t c_, int32_t g_) {
84:   return ((g_ >= 0 && g_ < size<3>(activation)) &&
85:           (n_ >= 0 && n_ < size<2>(activation)) &&
86:           (w_ >= 0 && w_ < size<1>(activation)) &&
87:           (c_ >= 0 && c_ < size<0>(activation)));
88: }
```
- **EN:** Introduces `EngineAct`, a type used to support conv.
- **CN:** 引入 `EngineAct`，即一个用于支持conv的类型。

### Lines 90-90
```cpp
90: } // namespace detail
```
- **EN:** Supporting logic for the conv implementation.
- **CN:** conv实现的辅助逻辑。

### Lines 92-105
```cpp
92: template<
93:   class ElementAcc_,
94:   class ElementScalar_,
95:   class ElementCompute_,
96:   class ElementC_,
97:   class ElementOut_,
98:   bool ResidualAdd_,
99:   class TensorAlpha_,
100:   class TensorBeta_,
101:   class TensorBias_,
102:   class ActivationFunctor_ = cutlass::epilogue::thread::Identity<ElementCompute_>
103: >
104: struct ConvEpilogueFusionParams {
105:   using ElementAcc = ElementAcc_;
```
- **EN:** Introduces `ElementAcc_`, a type used to support conv.
- **CN:** 引入 `ElementAcc_`，即一个用于支持conv的类型。

### Lines 106-114
```cpp
106:   using ElementScalar = ElementScalar_;
107:   using ElementCompute = ElementCompute_;
108:   using ElementC = ElementC_;
109:   using ElementOut = ElementOut_;
110:   using TensorAlpha = TensorAlpha_;
111:   using TensorBeta = TensorBeta_;
112:   using TensorBias = TensorBias_;
113:   using ActivationFunctor = ActivationFunctor_;
114:   static constexpr bool ResidualAdd = ResidualAdd_; // Source added after activation
```
- **EN:** Introduces namespace imports, aliases, or shorthand names used by later code.
- **CN:** 引入后续代码会使用的命名空间、别名或简写名称。

### Lines 116-117
```cpp
116:   ElementScalar alpha = ElementScalar(1);
117:   ElementScalar beta = ElementScalar(0);
```
- **EN:** Implements `ElementScalar` for this file's main component.
- **CN:** 为该文件的核心组件实现 `ElementScalar`。

### Lines 119-122
```cpp
119:   TensorAlpha tensor_alpha{};
120:   TensorBeta tensor_beta{};
121:   TensorBias tensor_bias{};
122: };
```
- **EN:** Supporting logic for the conv implementation.
- **CN:** conv实现的辅助逻辑。

### Lines 124-136
```cpp
124: template<
125:   cutlass::conv::Operator ConvOp,
126:   int NumSpatialDims,
127:   class TensorA,
128:   class TensorB,
129:   class TensorC,
130:   class TensorD,
131:   class ShapePadding,
132:   class StrideTraversal,
133:   class ShapeDilation,
134:   class EpilogueFusionParams
135: >
136: struct ConvReferenceImpl {
```
- **EN:** Introduces `TensorA`, a type used to support conv.
- **CN:** 引入 `TensorA`，即一个用于支持conv的类型。

### Lines 137-137
```cpp
137:   // Hard code accumlulator type to float to avoid data lost in accumulating add.
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 138-144
```cpp
138:   using ElementAcc = cutlass::platform::conditional_t<cutlass::platform::is_same_v<typename EpilogueFusionParams::ElementAcc, double>, double, float>;
139:   using ElementC = typename EpilogueFusionParams::ElementC;
140:   using ElementOut = typename EpilogueFusionParams::ElementOut;
141:   using ElementScalar = typename EpilogueFusionParams::ElementScalar;
142:   using ElementCompute = typename EpilogueFusionParams::ElementCompute;
143:   using ElementBias = typename EpilogueFusionParams::TensorBias::value_type;
144:   using ActivationFunctor = typename EpilogueFusionParams::ActivationFunctor;
```
- **EN:** Introduces namespace imports, aliases, or shorthand names used by later code.
- **CN:** 引入后续代码会使用的命名空间、别名或简写名称。

### Lines 146-146
```cpp
146:   // Input related converter
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 147-149
```cpp
147:   NumericConverter<ElementCompute, ElementAcc> acc_converter;
148:   NumericConverter<ElementCompute, ElementC> residual_converter;
149:   NumericConverter<ElementCompute, ElementBias> bias_converter;
```
- **EN:** Declares or updates local/member state such as `acc_converter`, `residual_converter`, `bias_converter`.
- **CN:** 声明或更新局部/成员状态，例如 `acc_converter`, `residual_converter`, `bias_converter`。

### Lines 150-150
```cpp
150:   // Scale related converter
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 151-151
```cpp
151:   NumericConverter<ElementCompute, ElementScalar> scale_converter;
```
- **EN:** Declares or updates local/member state such as `scale_converter`.
- **CN:** 声明或更新局部/成员状态，例如 `scale_converter`。

### Lines 152-152
```cpp
152:   // Output related converter
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 153-153
```cpp
153:   NumericConverter<ElementOut, ElementCompute> output_converter;
```
- **EN:** Declares or updates local/member state such as `output_converter`.
- **CN:** 声明或更新局部/成员状态，例如 `output_converter`。

### Lines 155-159
```cpp
155:   EpilogueFusionParams& epi_fusion_params_;
156:   TensorA const& tensor_a_;
157:   TensorB const& tensor_b_;
158:   TensorC const& tensor_c_;
159:   TensorD& tensor_d_;
```
- **EN:** Declares or updates local/member state such as `epi_fusion_params_`, `tensor_a_`, `tensor_b_`, `tensor_c_`.
- **CN:** 声明或更新局部/成员状态，例如 `epi_fusion_params_`, `tensor_a_`, `tensor_b_`, `tensor_c_`。

### Lines 161-163
```cpp
161:   ShapePadding const& padding_;
162:   StrideTraversal const& tstride_;
163:   ShapeDilation const& dilation_;
```
- **EN:** Declares or updates local/member state such as `padding_`, `tstride_`, `dilation_`.
- **CN:** 声明或更新局部/成员状态，例如 `padding_`, `tstride_`, `dilation_`。

### Lines 165-165
```cpp
165:   // Epilogue activation operation
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 166-166
```cpp
166:   ActivationFunctor epi_activation;
```
- **EN:** Declares or updates local/member state such as `epi_activation`.
- **CN:** 声明或更新局部/成员状态，例如 `epi_activation`。

### Lines 168-181
```cpp
168:   ConvReferenceImpl(
169:     TensorA const& tensor_a,
170:     TensorB const& tensor_b,
171:     TensorC const& tensor_c,
172:     TensorD& tensor_d,
173:     ShapePadding const& padding,
174:     StrideTraversal const& tstride,
175:     ShapeDilation const& dilation,
176:     EpilogueFusionParams& epi_fusion_params)
177:   : tensor_a_(tensor_a),
178:     tensor_b_(tensor_b),
179:     tensor_c_(tensor_c),
180:     tensor_d_(tensor_d),
181:     padding_(padding),
```
- **EN:** Supporting logic for the conv implementation.
- **CN:** conv实现的辅助逻辑。

### Lines 182-185
```cpp
182:     tstride_(tstride),
183:     dilation_(dilation),
184:     epi_fusion_params_(epi_fusion_params)
185:   {
```
- **EN:** Supporting logic for the conv implementation.
- **CN:** conv实现的辅助逻辑。

### Lines 186-188
```cpp
186:     static_assert(rank(ShapePadding{}) == rank(ShapeDilation{}));
187:     static_assert(rank(ShapePadding{}) == rank(StrideTraversal{}));
188:   }
```
- **EN:** Supporting logic for the conv implementation.
- **CN:** conv实现的辅助逻辑。

### Lines 190-200
```cpp
190:   void compute_reference() {
191:     if constexpr (ConvOp == cutlass::conv::Operator::kFprop) {
192:       fprop_reference(cute::Int<NumSpatialDims>{});
193:     }
194:     else if constexpr (ConvOp == cutlass::conv::Operator::kDgrad) {
195:       dgrad_reference(cute::Int<NumSpatialDims>{});
196:     }
197:     else {
198:       wgrad_reference(cute::Int<NumSpatialDims>{});
199:     }
200:   }
```
- **EN:** Implements `compute_reference` and coordinates helper calls such as `constexpr`, `fprop_reference`, `dgrad_reference`.
- **CN:** 实现 `compute_reference`，并协调调用 `constexpr`, `fprop_reference`, `dgrad_reference` 等辅助逻辑。

### Lines 202-202
```cpp
202: private:
```
- **EN:** Supporting logic for the conv implementation.
- **CN:** conv实现的辅助逻辑。

### Lines 203-203
```cpp
203:   // Specialization for 1D fprop kernel
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 204-210
```cpp
204:   void fprop_reference(cute::Int<1> spatial_dims) {
205:     int32_t G = size<3>(tensor_d_);
206:     int32_t N = size<2>(tensor_d_);
207:     int32_t Q = size<1>(tensor_d_);
208:     int32_t K = size<0>(tensor_d_);
209:     int32_t S = size<1>(tensor_b_);
210:     int32_t C = size<0>(tensor_b_);
```
- **EN:** Implements `fprop_reference` for this file's main component.
- **CN:** 为该文件的核心组件实现 `fprop_reference`。

### Lines 212-214
```cpp
212: #if defined(_OPENMP)
213:   #pragma omp parallel for collapse(2)
214: #endif
```
- **EN:** Conditional-compilation or macro block keyed on `defined(_OPENMP)`.
- **CN:** 以 `defined(_OPENMP)` 为条件的条件编译或宏定义代码块。

### Lines 215-228
```cpp
215:     for (int32_t g = 0; g < G; ++g) {
216:       for (int32_t n = 0; n < N; ++n) {
217:         for (int32_t q = 0; q < Q; ++q) {
218:           for (int32_t k = 0; k < K; ++k) {
219:             auto accumulator = ElementAcc(0);
220:             for (int32_t s = 0; s < S; ++s) {
221:               for (int32_t c = 0; c < C; ++c) {
222:                 int32_t w =  q * cute::get<0>(tstride_) - cute::get<0>(padding_) + s * cute::get<0>(dilation_);
223:                 if (detail::is_activation_in_bounds(tensor_a_, n, w, c, g)) {
224:                   auto a = tensor_a_(c, w, n, g);
225:                   auto b = tensor_b_(c, s, k, g);
226:                   accumulator += ElementAcc(a * b);
227:                 }
228:               }
```
- **EN:** Declares or updates local/member state such as `g`, `G`, `n`, `N`.
- **CN:** 声明或更新局部/成员状态，例如 `g`, `G`, `n`, `N`。

### Lines 229-235
```cpp
229:             }
230:             ElementScalar alpha = raw_pointer_cast(epi_fusion_params_.tensor_alpha.data()) ?
231:               epi_fusion_params_.tensor_alpha[k] : epi_fusion_params_.alpha;
232:             ElementScalar beta = raw_pointer_cast(epi_fusion_params_.tensor_beta.data()) ?
233:               epi_fusion_params_.tensor_beta[k] : epi_fusion_params_.beta;
234:             ElementCompute output = scale_converter(alpha) * acc_converter(accumulator);
235:             if (not EpilogueFusionParams::ResidualAdd) {
```
- **EN:** Declares or updates local/member state such as `alpha`, `beta`, `output`.
- **CN:** 声明或更新局部/成员状态，例如 `alpha`, `beta`, `output`。

### Lines 236-249
```cpp
236:               output += scale_converter(beta) * residual_converter(tensor_c_(k, q, n, g));
237:             }
238:             if (raw_pointer_cast(epi_fusion_params_.tensor_bias.data())) {
239:               output += bias_converter(epi_fusion_params_.tensor_bias[k]);
240:             }
241:             output = epi_activation(output);
242:             if (EpilogueFusionParams::ResidualAdd) {
243:               output += scale_converter(beta) * residual_converter(tensor_c_(k, q, n, g));
244:             }
245:             tensor_d_(k, q, n, g) = output_converter(output);
246:           }
247:         }
248:       }
249:     }
```
- **EN:** Implements `scale_converter` and coordinates helper calls such as `residual_converter`, `tensor_c_`, `raw_pointer_cast`.
- **CN:** 实现 `scale_converter`，并协调调用 `residual_converter`, `tensor_c_`, `raw_pointer_cast` 等辅助逻辑。

### Lines 251-251
```cpp
251:   }
```
- **EN:** Supporting logic for the conv implementation.
- **CN:** conv实现的辅助逻辑。

### Lines 253-253
```cpp
253:   // Specialization for 2D fprop kernel
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 254-262
```cpp
254:   void fprop_reference(cute::Int<2> spatial_dims) {
255:     int32_t G = size<4>(tensor_d_);
256:     int32_t N = size<3>(tensor_d_);
257:     int32_t P = size<2>(tensor_d_);
258:     int32_t Q = size<1>(tensor_d_);
259:     int32_t K = size<0>(tensor_d_);
260:     int32_t R = size<2>(tensor_b_);
261:     int32_t S = size<1>(tensor_b_);
262:     int32_t C = size<0>(tensor_b_);
```
- **EN:** Implements `fprop_reference` for this file's main component.
- **CN:** 为该文件的核心组件实现 `fprop_reference`。

### Lines 264-266
```cpp
264: #if defined(_OPENMP)
265:     #pragma omp parallel for collapse(3)
266: #endif
```
- **EN:** Conditional-compilation or macro block keyed on `defined(_OPENMP)`.
- **CN:** 以 `defined(_OPENMP)` 为条件的条件编译或宏定义代码块。

### Lines 267-280
```cpp
267:     for (int32_t g = 0; g < G; ++g) {
268:       for (int32_t n = 0; n < N; ++n) {
269:         for (int32_t p = 0; p < P; ++p) {
270:           for (int32_t q = 0; q < Q; ++q) {
271:             for (int32_t k = 0; k < K; ++k) {
272:               auto accumulator = ElementAcc(0);
273:               for (int32_t r = 0; r < R; ++r) {
274:                 for (int32_t s = 0; s < S; ++s) {
275:                   for (int32_t c = 0; c < C; ++c) {
276:                     int32_t w =  q * cute::get<0>(tstride_) - cute::get<0>(padding_) + s * cute::get<0>(dilation_);
277:                     int32_t h =  p * cute::get<1>(tstride_) - cute::get<1>(padding_) + r * cute::get<1>(dilation_);
278:                     if (detail::is_activation_in_bounds(tensor_a_, n, h, w, c, g)) {
279:                       auto a = tensor_a_(c, w, h, n, g);
280:                       auto b = tensor_b_(c, s, r, k, g);
```
- **EN:** Declares or updates local/member state such as `g`, `G`, `n`, `N`.
- **CN:** 声明或更新局部/成员状态，例如 `g`, `G`, `n`, `N`。

### Lines 281-284
```cpp
281:                       accumulator += ElementAcc(a * b);
282:                     }
283:                   }
284:                 }
```
- **EN:** Implements `ElementAcc` for this file's main component.
- **CN:** 为该文件的核心组件实现 `ElementAcc`。

### Lines 285-298
```cpp
285:               }
286:               ElementScalar alpha = raw_pointer_cast(epi_fusion_params_.tensor_alpha.data()) ?
287:                 epi_fusion_params_.tensor_alpha[k] : epi_fusion_params_.alpha;
288:               ElementScalar beta = raw_pointer_cast(epi_fusion_params_.tensor_beta.data()) ?
289:                 epi_fusion_params_.tensor_beta[k] : epi_fusion_params_.beta;
290:               ElementCompute output = scale_converter(alpha) * acc_converter(accumulator);
291:               if (not EpilogueFusionParams::ResidualAdd) {
292:                 output += scale_converter(beta) * residual_converter(tensor_c_(k, q, p, n, g));
293:               }
294:               if (raw_pointer_cast(epi_fusion_params_.tensor_bias.data())) {
295:                 output += bias_converter(epi_fusion_params_.tensor_bias[k]);
296:               }
297:               output = epi_activation(output);
298:               if (EpilogueFusionParams::ResidualAdd) {
```
- **EN:** Declares or updates local/member state such as `alpha`, `beta`, `output`.
- **CN:** 声明或更新局部/成员状态，例如 `alpha`, `beta`, `output`。

### Lines 299-302
```cpp
299:                 output += scale_converter(beta) * residual_converter(tensor_c_(k, q, p, n, g));
300:               }
301:               tensor_d_(k, q, p, n, g) = output_converter(output);
302:             }
```
- **EN:** Implements `scale_converter` and coordinates helper calls such as `residual_converter`, `tensor_c_`, `tensor_d_`.
- **CN:** 实现 `scale_converter`，并协调调用 `residual_converter`, `tensor_c_`, `tensor_d_` 等辅助逻辑。

### Lines 303-306
```cpp
303:           }
304:         }
305:       }
306:     }
```
- **EN:** Supporting logic for the conv implementation.
- **CN:** conv实现的辅助逻辑。

### Lines 308-308
```cpp
308:   }
```
- **EN:** Supporting logic for the conv implementation.
- **CN:** conv实现的辅助逻辑。

### Lines 310-310
```cpp
310:   // Specialization for 3D fprop kernel
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 311-321
```cpp
311:   void fprop_reference(cute::Int<3> spatial_dims) {
312:     int32_t G = size<5>(tensor_d_);
313:     int32_t N = size<4>(tensor_d_);
314:     int32_t Z = size<3>(tensor_d_);
315:     int32_t P = size<2>(tensor_d_);
316:     int32_t Q = size<1>(tensor_d_);
317:     int32_t K = size<0>(tensor_d_);
318:     int32_t T = size<3>(tensor_b_);
319:     int32_t R = size<2>(tensor_b_);
320:     int32_t S = size<1>(tensor_b_);
321:     int32_t C = size<0>(tensor_b_);
```
- **EN:** Implements `fprop_reference` for this file's main component.
- **CN:** 为该文件的核心组件实现 `fprop_reference`。

### Lines 323-325
```cpp
323: #if defined(_OPENMP)
324:     #pragma omp parallel for collapse(3)
325: #endif
```
- **EN:** Conditional-compilation or macro block keyed on `defined(_OPENMP)`.
- **CN:** 以 `defined(_OPENMP)` 为条件的条件编译或宏定义代码块。

### Lines 326-339
```cpp
326:     for (int32_t g = 0; g < G; ++g) {
327:       for (int32_t n = 0; n < N; ++n) {
328:         for (int32_t z = 0; z < Z; ++z) {
329:           for (int32_t p = 0; p < P; ++p) {
330:             for (int32_t q = 0; q < Q; ++q) {
331:               for (int32_t k = 0; k < K; ++k) {
332:                 auto accumulator = ElementAcc(0);
333:                 for (int32_t t = 0; t < T; ++t) {
334:                   for (int32_t r = 0; r < R; ++r) {
335:                     for (int32_t s = 0; s < S; ++s) {
336:                       for (int32_t c = 0; c < C; ++c) {
337:                         int32_t w =  q * cute::get<0>(tstride_) - cute::get<0>(padding_) + s * cute::get<0>(dilation_);
338:                         int32_t h =  p * cute::get<1>(tstride_) - cute::get<1>(padding_) + r * cute::get<1>(dilation_);
339:                         int32_t d =  z * cute::get<2>(tstride_) - cute::get<2>(padding_) + t * cute::get<2>(dilation_);
```
- **EN:** Declares or updates local/member state such as `g`, `G`, `n`, `N`.
- **CN:** 声明或更新局部/成员状态，例如 `g`, `G`, `n`, `N`。

### Lines 340-344
```cpp
340:                         if (detail::is_activation_in_bounds(tensor_a_, n, d, h, w, c, g)) {
341:                           auto a = tensor_a_(c, w, h, d, n, g);
342:                           auto b = tensor_b_(c, s, r, t, k, g);
343:                           accumulator += ElementAcc(a * b);
344:                         }
```
- **EN:** Declares or updates local/member state such as `a`, `b`.
- **CN:** 声明或更新局部/成员状态，例如 `a`, `b`。

### Lines 345-358
```cpp
345:                       }
346:                     }
347:                   }
348:                 }
349:                 ElementScalar alpha = raw_pointer_cast(epi_fusion_params_.tensor_alpha.data()) ?
350:                   epi_fusion_params_.tensor_alpha[k] : epi_fusion_params_.alpha;
351:                 ElementScalar beta = raw_pointer_cast(epi_fusion_params_.tensor_beta.data()) ?
352:                   epi_fusion_params_.tensor_beta[k] : epi_fusion_params_.beta;
353:                 ElementCompute output = scale_converter(alpha) * acc_converter(accumulator);
354:                 if (not EpilogueFusionParams::ResidualAdd) {
355:                   output += scale_converter(beta) * residual_converter(tensor_c_(k, q, p, z, n, g));
356:                 }
357:                 if (raw_pointer_cast(epi_fusion_params_.tensor_bias.data())) {
358:                   output += bias_converter(epi_fusion_params_.tensor_bias[k]);
```
- **EN:** Declares or updates local/member state such as `alpha`, `beta`, `output`.
- **CN:** 声明或更新局部/成员状态，例如 `alpha`, `beta`, `output`。

### Lines 359-363
```cpp
359:                 }
360:                 output = epi_activation(output);
361:                 if (EpilogueFusionParams::ResidualAdd) {
362:                   output += scale_converter(beta) * residual_converter(tensor_c_(k, q, p, z, n, g));
363:                 }
```
- **EN:** Implements `epi_activation` and coordinates helper calls such as `scale_converter`, `residual_converter`, `tensor_c_`.
- **CN:** 实现 `epi_activation`，并协调调用 `scale_converter`, `residual_converter`, `tensor_c_` 等辅助逻辑。

### Lines 364-370
```cpp
364:                 tensor_d_(k, q, p, z, n, g) = output_converter(output);
365:               }
366:             }
367:           }
368:         }
369:       }
370:     }
```
- **EN:** Supporting logic for the conv implementation.
- **CN:** conv实现的辅助逻辑。

### Lines 372-372
```cpp
372:   }
```
- **EN:** Supporting logic for the conv implementation.
- **CN:** conv实现的辅助逻辑。

### Lines 374-374
```cpp
374:   // Specialization for 1D dgrad kernel
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 375-381
```cpp
375:   void dgrad_reference(cute::Int<1> spatial_dims) {
376:     int32_t G = size<3>(tensor_d_);
377:     int32_t N = size<2>(tensor_d_);
378:     int32_t W = size<1>(tensor_d_);
379:     int32_t C = size<0>(tensor_d_);
380:     int32_t K = size<2>(tensor_b_);
381:     int32_t S = size<1>(tensor_b_);
```
- **EN:** Implements `dgrad_reference` for this file's main component.
- **CN:** 为该文件的核心组件实现 `dgrad_reference`。

### Lines 383-385
```cpp
383: #if defined(_OPENMP)
384:    #pragma omp parallel for collapse(2)
385: #endif
```
- **EN:** Conditional-compilation or macro block keyed on `defined(_OPENMP)`.
- **CN:** 以 `defined(_OPENMP)` 为条件的条件编译或宏定义代码块。

### Lines 386-393
```cpp
386:     for (int32_t g = 0; g < G; ++g) {
387:       for (int32_t n = 0; n < N; ++n) {
388:         for (int32_t w = 0; w < W; ++w) {
389:           for (int32_t c = 0; c < C; ++c) {
390:             auto accumulator = ElementAcc(0);
391:             for (int32_t k = 0; k < K; ++k) {
392:               for (int32_t s = 0; s < S; ++s) {
393:                 int32_t q = w + cute::get<0>(padding_) - s * cute::get<0>(dilation_);
```
- **EN:** Declares or updates local/member state such as `g`, `G`, `n`, `N`.
- **CN:** 声明或更新局部/成员状态，例如 `g`, `G`, `n`, `N`。

### Lines 395-399
```cpp
395:                 if (q % cute::get<0>(tstride_) == 0) {
396:                   q /= cute::get<0>(tstride_);
397:                 } else {
398:                   continue;
399:                 }
```
- **EN:** Declares or updates local/member state such as `continue`.
- **CN:** 声明或更新局部/成员状态，例如 `continue`。

### Lines 401-414
```cpp
401:                 if (detail::is_activation_in_bounds(tensor_a_, n, q, k, g)) {
402:                   accumulator += ElementAcc(tensor_a_(k, q, n, g) * tensor_b_(c, s, k, g));
403:                 }
404:               }
405:             }
406:             ElementScalar alpha = raw_pointer_cast(epi_fusion_params_.tensor_alpha.data())
407:               ? epi_fusion_params_.tensor_alpha[c] : epi_fusion_params_.alpha;
408:             ElementScalar beta = raw_pointer_cast(epi_fusion_params_.tensor_beta.data())
409:               ? epi_fusion_params_.tensor_beta[c] : epi_fusion_params_.beta;
410:             ElementCompute output = scale_converter(alpha) * acc_converter(accumulator);
411:             if (not EpilogueFusionParams::ResidualAdd) {
412:               output += scale_converter(beta) * residual_converter(tensor_c_(c, w, n, g));
413:             }
414:             if (raw_pointer_cast(epi_fusion_params_.tensor_bias.data())) {
```
- **EN:** Declares or updates local/member state such as `alpha`, `beta`, `output`.
- **CN:** 声明或更新局部/成员状态，例如 `alpha`, `beta`, `output`。

### Lines 415-418
```cpp
415:               output += bias_converter(epi_fusion_params_.tensor_bias[c]);
416:             }
417:             output = epi_activation(output);
418:             if (EpilogueFusionParams::ResidualAdd) {
```
- **EN:** Implements `bias_converter` and coordinates helper calls such as `epi_activation`.
- **CN:** 实现 `bias_converter`，并协调调用 `epi_activation` 等辅助逻辑。

### Lines 419-425
```cpp
419:               output += scale_converter(beta) * residual_converter(tensor_c_(c, w, n, g));
420:             }
421:             tensor_d_(c, w, n, g) = output_converter(output);
422:           }
423:         }
424:       }
425:     }
```
- **EN:** Implements `scale_converter` and coordinates helper calls such as `residual_converter`, `tensor_c_`, `tensor_d_`.
- **CN:** 实现 `scale_converter`，并协调调用 `residual_converter`, `tensor_c_`, `tensor_d_` 等辅助逻辑。

### Lines 427-427
```cpp
427:   }
```
- **EN:** Supporting logic for the conv implementation.
- **CN:** conv实现的辅助逻辑。

### Lines 429-429
```cpp
429:   // Specialization for 2D dgrad kernel
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 430-438
```cpp
430:   void dgrad_reference(cute::Int<2> spatial_dims) {
431:     int32_t G = size<4>(tensor_d_);
432:     int32_t N = size<3>(tensor_d_);
433:     int32_t H = size<2>(tensor_d_);
434:     int32_t W = size<1>(tensor_d_);
435:     int32_t C = size<0>(tensor_d_);
436:     int32_t K = size<3>(tensor_b_);
437:     int32_t R = size<2>(tensor_b_);
438:     int32_t S = size<1>(tensor_b_);
```
- **EN:** Implements `dgrad_reference` for this file's main component.
- **CN:** 为该文件的核心组件实现 `dgrad_reference`。

### Lines 440-442
```cpp
440: #if defined(_OPENMP)
441:     #pragma omp parallel for collapse(3)
442: #endif
```
- **EN:** Conditional-compilation or macro block keyed on `defined(_OPENMP)`.
- **CN:** 以 `defined(_OPENMP)` 为条件的条件编译或宏定义代码块。

### Lines 443-453
```cpp
443:     for (int32_t g = 0; g < G; ++g) {
444:       for (int32_t n = 0; n < N; ++n) {
445:         for (int32_t h = 0; h < H; ++h) {
446:           for (int32_t w = 0; w < W; ++w) {
447:             for (int32_t c = 0; c < C; ++c) {
448:               auto accumulator = ElementAcc(0);
449:               for (int32_t k = 0; k < K; ++k) {
450:                 for (int32_t r = 0; r < R; ++r) {
451:                   for (int32_t s = 0; s < S; ++s) {
452:                     int32_t q = w + cute::get<0>(padding_) - s * cute::get<0>(dilation_);
453:                     int32_t p = h + cute::get<1>(padding_) - r * cute::get<1>(dilation_);
```
- **EN:** Declares or updates local/member state such as `g`, `G`, `n`, `N`.
- **CN:** 声明或更新局部/成员状态，例如 `g`, `G`, `n`, `N`。

### Lines 455-459
```cpp
455:                     if (q % cute::get<0>(tstride_) == 0) {
456:                       q /= cute::get<0>(tstride_);
457:                     } else {
458:                       continue;
459:                     }
```
- **EN:** Declares or updates local/member state such as `continue`.
- **CN:** 声明或更新局部/成员状态，例如 `continue`。

### Lines 461-465
```cpp
461:                     if (p % cute::get<1>(tstride_) == 0) {
462:                       p /= cute::get<1>(tstride_);
463:                     } else {
464:                       continue;
465:                     }
```
- **EN:** Declares or updates local/member state such as `continue`.
- **CN:** 声明或更新局部/成员状态，例如 `continue`。

### Lines 467-480
```cpp
467:                     if (detail::is_activation_in_bounds(tensor_a_, n, p, q, k, g)) {
468:                       accumulator += ElementAcc(tensor_a_(k, q, p, n, g) * tensor_b_(c, s, r, k, g));
469:                     }
470:                   }
471:                 }
472:               }
473:               ElementScalar alpha = raw_pointer_cast(epi_fusion_params_.tensor_alpha.data())
474:                 ? epi_fusion_params_.tensor_alpha[c] : epi_fusion_params_.alpha;
475:               ElementScalar beta = raw_pointer_cast(epi_fusion_params_.tensor_beta.data())
476:                 ? epi_fusion_params_.tensor_beta[c] : epi_fusion_params_.beta;
477:               ElementCompute output = scale_converter(alpha) * acc_converter(accumulator);
478:               if (not EpilogueFusionParams::ResidualAdd) {
479:                 output += scale_converter(beta) * residual_converter(tensor_c_(c, w, h, n, g));
480:               }
```
- **EN:** Declares or updates local/member state such as `alpha`, `beta`, `output`.
- **CN:** 声明或更新局部/成员状态，例如 `alpha`, `beta`, `output`。

### Lines 481-485
```cpp
481:               if (raw_pointer_cast(epi_fusion_params_.tensor_bias.data())) {
482:                 output += bias_converter(epi_fusion_params_.tensor_bias[c]);
483:               }
484:               output = epi_activation(output);
485:               if (EpilogueFusionParams::ResidualAdd) {
```
- **EN:** Declares or updates local/member state such as `output`.
- **CN:** 声明或更新局部/成员状态，例如 `output`。

### Lines 486-487
```cpp
486:                 output += scale_converter(beta) * residual_converter(tensor_c_(c, w, h, n, g));
487:               }
```
- **EN:** Implements `scale_converter` and coordinates helper calls such as `residual_converter`, `tensor_c_`.
- **CN:** 实现 `scale_converter`，并协调调用 `residual_converter`, `tensor_c_` 等辅助逻辑。

### Lines 489-494
```cpp
489:               tensor_d_(c, w, h, n, g) = output_converter(output);
490:             }
491:           }
492:         }
493:       }
494:     }
```
- **EN:** Supporting logic for the conv implementation.
- **CN:** conv实现的辅助逻辑。

### Lines 496-496
```cpp
496:   }
```
- **EN:** Supporting logic for the conv implementation.
- **CN:** conv实现的辅助逻辑。

### Lines 498-498
```cpp
498:   // Specialization for 3D dgrad kernel
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 499-509
```cpp
499:   void dgrad_reference(cute::Int<3> spatial_dims) {
500:     int32_t G = size<5>(tensor_d_);
501:     int32_t N = size<4>(tensor_d_);
502:     int32_t D = size<3>(tensor_d_);
503:     int32_t H = size<2>(tensor_d_);
504:     int32_t W = size<1>(tensor_d_);
505:     int32_t C = size<0>(tensor_d_);
506:     int32_t K = size<4>(tensor_b_);
507:     int32_t T = size<3>(tensor_b_);
508:     int32_t R = size<2>(tensor_b_);
509:     int32_t S = size<1>(tensor_b_);
```
- **EN:** Implements `dgrad_reference` for this file's main component.
- **CN:** 为该文件的核心组件实现 `dgrad_reference`。

### Lines 511-513
```cpp
511: #if defined(_OPENMP)
512:     #pragma omp parallel for collapse(3)
513: #endif
```
- **EN:** Conditional-compilation or macro block keyed on `defined(_OPENMP)`.
- **CN:** 以 `defined(_OPENMP)` 为条件的条件编译或宏定义代码块。

### Lines 514-527
```cpp
514:     for (int32_t g = 0; g < G; ++g) {
515:       for (int32_t n = 0; n < N; ++n) {
516:         for (int32_t d = 0; d < D; ++d) {
517:           for (int32_t h = 0; h < H; ++h) {
518:             for (int32_t w = 0; w < W; ++w) {
519:               for (int32_t c = 0; c < C; ++c) {
520:                 auto accumulator = ElementAcc(0);
521:                 for (int32_t k = 0; k < K; ++k) {
522:                   for (int32_t t = 0; t < T; ++t) {
523:                     for (int32_t r = 0; r < R; ++r) {
524:                       for (int32_t s = 0; s < S; ++s) {
525:                         int32_t q = w + cute::get<0>(padding_) - s * cute::get<0>(dilation_);
526:                         int32_t p = h + cute::get<1>(padding_) - r * cute::get<1>(dilation_);
527:                         int32_t z = d + cute::get<2>(padding_) - t * cute::get<2>(dilation_);
```
- **EN:** Declares or updates local/member state such as `g`, `G`, `n`, `N`.
- **CN:** 声明或更新局部/成员状态，例如 `g`, `G`, `n`, `N`。

### Lines 529-533
```cpp
529:                         if (q % cute::get<0>(tstride_) == 0) {
530:                           q /= cute::get<0>(tstride_);
531:                         } else {
532:                           continue;
533:                         }
```
- **EN:** Declares or updates local/member state such as `continue`.
- **CN:** 声明或更新局部/成员状态，例如 `continue`。

### Lines 535-539
```cpp
535:                         if (p % cute::get<1>(tstride_) == 0) {
536:                           p /= cute::get<1>(tstride_);
537:                         } else {
538:                           continue;
539:                         }
```
- **EN:** Declares or updates local/member state such as `continue`.
- **CN:** 声明或更新局部/成员状态，例如 `continue`。

### Lines 541-545
```cpp
541:                         if (z % cute::get<2>(tstride_) == 0) {
542:                           z /= cute::get<2>(tstride_);
543:                         } else {
544:                           continue;
545:                         }
```
- **EN:** Declares or updates local/member state such as `continue`.
- **CN:** 声明或更新局部/成员状态，例如 `continue`。

### Lines 547-560
```cpp
547:                         if (detail::is_activation_in_bounds(tensor_a_, n, z, p, q, k, g)) {
548:                           accumulator += ElementAcc(tensor_a_(k, q, p, z, n, g) * tensor_b_(c, s, r, t, k, g));
549:                         }
550:                       }
551:                     }
552:                   }
553:                 }
554:                 ElementScalar alpha = raw_pointer_cast(epi_fusion_params_.tensor_alpha.data())
555:                   ? epi_fusion_params_.tensor_alpha[c] : epi_fusion_params_.alpha;
556:                 ElementScalar beta = raw_pointer_cast(epi_fusion_params_.tensor_beta.data())
557:                   ? epi_fusion_params_.tensor_beta[c] : epi_fusion_params_.beta;
558:                 ElementCompute output = scale_converter(alpha) * acc_converter(accumulator);
559:                 if (not EpilogueFusionParams::ResidualAdd) {
560:                   output += scale_converter(beta) * residual_converter(tensor_c_(c, w, h, d, n, g));
```
- **EN:** Declares or updates local/member state such as `alpha`, `beta`, `output`.
- **CN:** 声明或更新局部/成员状态，例如 `alpha`, `beta`, `output`。

### Lines 561-564
```cpp
561:                 }
562:                 if (raw_pointer_cast(epi_fusion_params_.tensor_bias.data())) {
563:                   output += bias_converter(epi_fusion_params_.tensor_bias[c]);
564:                 }
```
- **EN:** Supporting logic for the conv implementation.
- **CN:** conv实现的辅助逻辑。

### Lines 565-575
```cpp
565:                 output = epi_activation(output);
566:                 if (EpilogueFusionParams::ResidualAdd) {
567:                   output += scale_converter(beta) * residual_converter(tensor_c_(c, w, h, d, n, g));
568:                 }
569:                 tensor_d_(c, w, h, d, n, g) = output_converter(output);
570:               }
571:             }
572:           }
573:         }
574:       }
575:     }
```
- **EN:** Implements `epi_activation` and coordinates helper calls such as `scale_converter`, `residual_converter`, `tensor_c_`.
- **CN:** 实现 `epi_activation`，并协调调用 `scale_converter`, `residual_converter`, `tensor_c_` 等辅助逻辑。

### Lines 577-577
```cpp
577:   }
```
- **EN:** Supporting logic for the conv implementation.
- **CN:** conv实现的辅助逻辑。

### Lines 579-579
```cpp
579:   // Specialization for 1D wgrad kernel
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 580-589
```cpp
580:   void wgrad_reference(cute::Int<1> spatial_dims) {
581:     int32_t G = size<3>(tensor_d_);
582:     int32_t N =
583:         size<2>(tensor_a_);
584:     int32_t Q =
585:         size<1>(tensor_a_);
586:     int32_t K =
587:         size<0>(tensor_a_);
588:     int32_t S = size<1>(tensor_d_);
589:     int32_t C = size<0>(tensor_d_);
```
- **EN:** Implements `wgrad_reference` for this file's main component.
- **CN:** 为该文件的核心组件实现 `wgrad_reference`。

### Lines 591-593
```cpp
591: #if defined(_OPENMP)
592:     #pragma omp parallel for collapse(2)
593: #endif
```
- **EN:** Conditional-compilation or macro block keyed on `defined(_OPENMP)`.
- **CN:** 以 `defined(_OPENMP)` 为条件的条件编译或宏定义代码块。

### Lines 594-607
```cpp
594:     for (int32_t g = 0; g < G; ++g) {
595:       for (int32_t k = 0; k < K; ++k) {
596:         for (int32_t s = 0; s < S; ++s) {
597:           for (int32_t c = 0; c < C; ++c) {
598:             auto accumulator = ElementAcc(0);
599:             for (int32_t n = 0; n < N; ++n) {
600:               for (int32_t q = 0; q < Q; ++q) {
601:                 int32_t w =  q * cute::get<0>(tstride_) - cute::get<0>(padding_) + s * cute::get<0>(dilation_);
602:                 bool is_in_bounds =
603:                     detail::is_activation_in_bounds(tensor_b_, n, w, c, g);
604:                 if (is_in_bounds) {
605:                   auto act =
606:                       tensor_b_(c, w, n, g);
607:                   auto xformed_act =
```
- **EN:** Declares or updates local/member state such as `g`, `G`, `k`, `K`.
- **CN:** 声明或更新局部/成员状态，例如 `g`, `G`, `k`, `K`。

### Lines 608-611
```cpp
608:                       tensor_a_(k, q, n, g);
609:                   accumulator += ElementAcc(act * xformed_act);
610:                 }
611:               }
```
- **EN:** Supporting logic for the conv implementation.
- **CN:** conv实现的辅助逻辑。

### Lines 612-612
```cpp
612:             }
```
- **EN:** Supporting logic for the conv implementation.
- **CN:** conv实现的辅助逻辑。

### Lines 614-617
```cpp
614:             ElementScalar alpha = raw_pointer_cast(epi_fusion_params_.tensor_alpha.data()) ?
615:               epi_fusion_params_.tensor_alpha[c] : epi_fusion_params_.alpha;
616:             ElementScalar beta = raw_pointer_cast(epi_fusion_params_.tensor_beta.data()) ?
617:               epi_fusion_params_.tensor_beta[c] : epi_fusion_params_.beta;
```
- **EN:** Declares or updates local/member state such as `alpha`, `beta`.
- **CN:** 声明或更新局部/成员状态，例如 `alpha`, `beta`。

### Lines 619-632
```cpp
619:             ElementCompute output = scale_converter(alpha) * acc_converter(accumulator);
620:             if (not EpilogueFusionParams::ResidualAdd) {
621:               output += scale_converter(beta) * residual_converter(tensor_c_(c, s, k, g));
622:             }
623:             if (raw_pointer_cast(epi_fusion_params_.tensor_bias.data())) {
624:               output += bias_converter(epi_fusion_params_.tensor_bias[c]);
625:             }
626:             output = epi_activation(output);
627:             if (EpilogueFusionParams::ResidualAdd) {
628:               output += scale_converter(beta) * residual_converter(tensor_c_(c, s, k, g));
629:             }
630:             tensor_d_(c, s, k, g) = output_converter(output);
631:           }
632:         }
```
- **EN:** Implements `scale_converter` and coordinates helper calls such as `acc_converter`, `residual_converter`, `tensor_c_`.
- **CN:** 实现 `scale_converter`，并协调调用 `acc_converter`, `residual_converter`, `tensor_c_` 等辅助逻辑。

### Lines 633-635
```cpp
633:       }
634:     }
635:   }
```
- **EN:** Supporting logic for the conv implementation.
- **CN:** conv实现的辅助逻辑。

### Lines 637-637
```cpp
637:   // Specialization for 2D wgrad kernel
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 638-650
```cpp
638:   void wgrad_reference(cute::Int<2> spatial_dims) {
639:     int32_t G = size<4>(tensor_d_);
640:     int32_t N =
641:         size<3>(tensor_a_);
642:     int32_t P =
643:         size<2>(tensor_a_);
644:     int32_t Q =
645:         size<1>(tensor_a_);
646:     int32_t K =
647:         size<0>(tensor_a_);
648:     int32_t R = size<2>(tensor_d_);
649:     int32_t S = size<1>(tensor_d_);
650:     int32_t C = size<0>(tensor_d_);
```
- **EN:** Implements `wgrad_reference` for this file's main component.
- **CN:** 为该文件的核心组件实现 `wgrad_reference`。

### Lines 652-654
```cpp
652: #if defined(_OPENMP)
653:     #pragma omp parallel for collapse(3)
654: #endif
```
- **EN:** Conditional-compilation or macro block keyed on `defined(_OPENMP)`.
- **CN:** 以 `defined(_OPENMP)` 为条件的条件编译或宏定义代码块。

### Lines 655-668
```cpp
655:     for (int32_t g = 0; g < G; ++g) {
656:       for (int32_t k = 0; k < K; ++k) {
657:         for (int32_t r = 0; r < R; ++r) {
658:           for (int32_t s = 0; s < S; ++s) {
659:             for (int32_t c = 0; c < C; ++c) {
660:               auto accumulator = ElementAcc(0);
661:               for (int32_t n = 0; n < N; ++n) {
662:                 for (int32_t p = 0; p < P; ++p) {
663:                   for (int32_t q = 0; q < Q; ++q) {
664:                     int32_t w =  q * cute::get<0>(tstride_) - cute::get<0>(padding_) + s * cute::get<0>(dilation_);
665:                     int32_t h =  p * cute::get<1>(tstride_) - cute::get<1>(padding_) + r * cute::get<1>(dilation_);
666:                     bool is_in_bounds =
667:                         detail::is_activation_in_bounds(tensor_b_, n, h, w, c, g);
668:                     if (is_in_bounds) {
```
- **EN:** Declares or updates local/member state such as `g`, `G`, `k`, `K`.
- **CN:** 声明或更新局部/成员状态，例如 `g`, `G`, `k`, `K`。

### Lines 669-674
```cpp
669:                       auto act =
670:                           tensor_b_(c, w, h, n, g);
671:                       auto xformed_act =
672:                           tensor_a_(k, q, p, n, g);
673:                       accumulator += ElementAcc(act * xformed_act);
674:                     }
```
- **EN:** Implements `tensor_b_` and coordinates helper calls such as `tensor_a_`, `ElementAcc`.
- **CN:** 实现 `tensor_b_`，并协调调用 `tensor_a_`, `ElementAcc` 等辅助逻辑。

### Lines 675-677
```cpp
675:                   }
676:                 }
677:               }
```
- **EN:** Supporting logic for the conv implementation.
- **CN:** conv实现的辅助逻辑。

### Lines 679-682
```cpp
679:               ElementScalar alpha = raw_pointer_cast(epi_fusion_params_.tensor_alpha.data()) ?
680:                 epi_fusion_params_.tensor_alpha[c] : epi_fusion_params_.alpha;
681:               ElementScalar beta = raw_pointer_cast(epi_fusion_params_.tensor_beta.data()) ?
682:                 epi_fusion_params_.tensor_beta[c] : epi_fusion_params_.beta;
```
- **EN:** Declares or updates local/member state such as `alpha`, `beta`.
- **CN:** 声明或更新局部/成员状态，例如 `alpha`, `beta`。

### Lines 684-697
```cpp
684:               ElementCompute output = scale_converter(alpha) * acc_converter(accumulator);
685:               if (not EpilogueFusionParams::ResidualAdd) {
686:                 output += scale_converter(beta) * residual_converter(tensor_c_(c, s, r, k, g));
687:               }
688:               if (raw_pointer_cast(epi_fusion_params_.tensor_bias.data())) {
689:                 output += bias_converter(epi_fusion_params_.tensor_bias[c]);
690:               }
691:               output = epi_activation(output);
692:               if (EpilogueFusionParams::ResidualAdd) {
693:                 output += scale_converter(beta) * residual_converter(tensor_c_(c, s, r, k, g));
694:               }
695:               tensor_d_(c, s, r, k, g) = output_converter(output);
696:             }
697:           }
```
- **EN:** Implements `scale_converter` and coordinates helper calls such as `acc_converter`, `residual_converter`, `tensor_c_`.
- **CN:** 实现 `scale_converter`，并协调调用 `acc_converter`, `residual_converter`, `tensor_c_` 等辅助逻辑。

### Lines 698-701
```cpp
698:         }
699:       }
700:     }
701:   }
```
- **EN:** Supporting logic for the conv implementation.
- **CN:** conv实现的辅助逻辑。

### Lines 703-703
```cpp
703:   // Specialization for 3D wgrad kernel
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 704-717
```cpp
704:   void wgrad_reference(cute::Int<3> spatial_dims) {
705:     int32_t G = size<5>(tensor_d_);
706:     int32_t N =
707:         size<4>(tensor_a_);
708:     int32_t Z =
709:         size<3>(tensor_a_);
710:     int32_t P =
711:         size<2>(tensor_a_);
712:     int32_t Q =
713:         size<1>(tensor_a_);
714:     int32_t K =
715:         size<0>(tensor_a_);
716:     int32_t T = size<3>(tensor_d_);
717:     int32_t R = size<2>(tensor_d_);
```
- **EN:** Implements `wgrad_reference` for this file's main component.
- **CN:** 为该文件的核心组件实现 `wgrad_reference`。

### Lines 718-719
```cpp
718:     int32_t S = size<1>(tensor_d_);
719:     int32_t C = size<0>(tensor_d_);
```
- **EN:** Declares or updates local/member state such as `S`, `C`.
- **CN:** 声明或更新局部/成员状态，例如 `S`, `C`。

### Lines 721-723
```cpp
721: #if defined(_OPENMP)
722:     #pragma omp parallel for collapse(3)
723: #endif
```
- **EN:** Conditional-compilation or macro block keyed on `defined(_OPENMP)`.
- **CN:** 以 `defined(_OPENMP)` 为条件的条件编译或宏定义代码块。

### Lines 724-737
```cpp
724:     for (int32_t g = 0 ; g < G; ++g) {
725:       for (int32_t k = 0; k < K; ++k) {
726:         for (int32_t t = 0; t < T; ++t) {
727:           for (int32_t r = 0; r < R; ++r) {
728:             for (int32_t s = 0; s < S; ++s) {
729:               for (int32_t c = 0; c < C; ++c) {
730:                 auto accumulator = ElementAcc(0);
731:                 for (int32_t n = 0; n < N; ++n) {
732:                   for (int32_t z = 0; z < Z; ++z) {
733:                     for (int32_t p = 0; p < P; ++p) {
734:                       for (int32_t q = 0; q < Q; ++q) {
735:                         int32_t w =  q * cute::get<0>(tstride_) - cute::get<0>(padding_) + s * cute::get<0>(dilation_);
736:                         int32_t h =  p * cute::get<1>(tstride_) - cute::get<1>(padding_) + r * cute::get<1>(dilation_);
737:                         int32_t d =  z * cute::get<2>(tstride_) - cute::get<2>(padding_) + t * cute::get<2>(dilation_);
```
- **EN:** Declares or updates local/member state such as `g`, `G`, `k`, `K`.
- **CN:** 声明或更新局部/成员状态，例如 `g`, `G`, `k`, `K`。

### Lines 738-746
```cpp
738:                         bool is_in_bounds =
739:                             detail::is_activation_in_bounds(tensor_b_, n, d, h, w, c, g);
740:                         if (is_in_bounds) {
741:                           auto act =
742:                               tensor_b_(c, w, h, d, n, g);
743:                           auto xformed_act =
744:                               tensor_a_(k, q, p, z, n, g);
745:                           accumulator += ElementAcc(act * xformed_act);
746:                         }
```
- **EN:** Implements `is_activation_in_bounds` and coordinates helper calls such as `tensor_b_`, `tensor_a_`, `ElementAcc`.
- **CN:** 实现 `is_activation_in_bounds`，并协调调用 `tensor_b_`, `tensor_a_`, `ElementAcc` 等辅助逻辑。

### Lines 747-750
```cpp
747:                       }
748:                     }
749:                   }
750:                 }
```
- **EN:** Supporting logic for the conv implementation.
- **CN:** conv实现的辅助逻辑。

### Lines 752-755
```cpp
752:                 ElementScalar alpha = raw_pointer_cast(epi_fusion_params_.tensor_alpha.data()) ?
753:                   epi_fusion_params_.tensor_alpha[c] : epi_fusion_params_.alpha;
754:                 ElementScalar beta = raw_pointer_cast(epi_fusion_params_.tensor_beta.data()) ?
755:                   epi_fusion_params_.tensor_beta[c] : epi_fusion_params_.beta;
```
- **EN:** Declares or updates local/member state such as `alpha`, `beta`.
- **CN:** 声明或更新局部/成员状态，例如 `alpha`, `beta`。

### Lines 757-770
```cpp
757:                 ElementCompute output = scale_converter(alpha) * acc_converter(accumulator);
758:                 if (not EpilogueFusionParams::ResidualAdd) {
759:                   output += scale_converter(beta) * residual_converter(tensor_c_(c, s, r, t, k, g));
760:                 }
761:                 if (raw_pointer_cast(epi_fusion_params_.tensor_bias.data())) {
762:                   output += bias_converter(epi_fusion_params_.tensor_bias[c]);
763:                 }
764:                 output = epi_activation(output);
765:                 if (EpilogueFusionParams::ResidualAdd) {
766:                   output += scale_converter(beta) * residual_converter(tensor_c_(c, s, r, t, k, g));
767:                 }
768:                 tensor_d_(c, s, r, t, k, g) = output_converter(output);
769:               }
770:             }
```
- **EN:** Implements `scale_converter` and coordinates helper calls such as `acc_converter`, `residual_converter`, `tensor_c_`.
- **CN:** 实现 `scale_converter`，并协调调用 `acc_converter`, `residual_converter`, `tensor_c_` 等辅助逻辑。

### Lines 771-774
```cpp
771:           }
772:         }
773:       }
774:     }
```
- **EN:** Supporting logic for the conv implementation.
- **CN:** conv实现的辅助逻辑。

### Lines 775-776
```cpp
775:   }
776: };
```
- **EN:** Supporting logic for the conv implementation.
- **CN:** conv实现的辅助逻辑。

### Lines 778-778
```cpp
778: /////////////////////////////////////////////////////////////////////////////////////////////////
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 780-780
```cpp
780: } // cutlass::reference::host
```
- **EN:** Supporting logic for the conv implementation.
- **CN:** conv实现的辅助逻辑。

### Lines 782-782
```cpp
782: /////////////////////////////////////////////////////////////////////////////////////////////////
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

## Key Concepts / 关键概念
- **Structured type design / 结构化类型设计**
- **Conditional compilation / 条件编译**
- **CUDA ecosystem integration / CUDA 生态集成**
- **Reference implementation / 参考实现**
- **Tensor manipulation / 张量处理**
- **Convolution support / 卷积支持**

## Dependencies / 依赖关系
- **Internal headers / 内部头文件:** `cutlass/complex.h`, `cutlass/numeric_conversion.h`, `cutlass/epilogue/thread/activation.h`, `cute/tensor.hpp`
- **External headers / 外部头文件:** `cuda_runtime.h`
- **Runtime/backends / 运行时与后端:** `CUDA`, `CuTe`
