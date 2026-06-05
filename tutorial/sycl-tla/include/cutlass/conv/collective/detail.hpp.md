# detail.hpp — Code Analysis / 代码分析

## Source / 来源
- **Path:** `include/cutlass/conv/collective/detail.hpp`
- **Purpose (EN):** Implements collective-level convolution building blocks centered on detail.
- **用途 (CN):** 实现以 细节 为中心的 collective 级卷积构件。

## Line-by-Line Analysis / 逐行分析

### Lines 1-30
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
**EN:** Records the BSD-3-Clause license and redistribution conditions for the file.

**CN:** 记录该文件的 BSD-3-Clause 许可和再分发条件。

### Lines 31-31
```cpp
 31: #pragma once
```
**EN:** Marks this file as a single-inclusion header with `#pragma once`.

**CN:** 通过 `#pragma once` 把该文件标记为单次包含头文件。

### Lines 33-33
```cpp
 33: #include "cutlass/conv/convnd_problem_shape.hpp"
```
**EN:** Imports direct dependencies used later in the file, including `convnd_problem_shape.hpp`.

**CN:** 引入本文件后续实现要用到的直接依赖，例如 `convnd_problem_shape.hpp`。

### Lines 37-37
```cpp
 37: namespace cutlass::conv::collective::detail {
```
**EN:** Opens the namespace scope `cutlass::conv::collective::detail` that contains these definitions.

**CN:** 打开包含这些定义的命名空间作用域 `cutlass::conv::collective::detail`。

### Lines 41-45
```cpp
 41: // Construct the stride types for conv collectives based on the dispatch policy, strides 64b by default
 42: template <class DispatchPolicy>
 43: constexpr auto
 44: sm90_dispatch_policy_to_stride_A() {
 45:   if constexpr (DispatchPolicy::ConvOp == conv::Operator::kFprop) {
```
**EN:** Defines function `sm90_dispatch_policy_to_stride_A` for this stage of the convolution workflow.

**CN:** 定义函数 `sm90_dispatch_policy_to_stride_A`，服务于卷积工作流的这一阶段。

### Lines 46-50
```cpp
 46:     // Maps to modes ((w,n), C)
 47:     if constexpr (DispatchPolicy::NumSpatialDimensions == 1) {
 48:       return cute::Stride<cute::Stride<int64_t, int64_t>,
 49:                           cute::Int<1>>{};
 50:     }
```
**EN:** Computes and returns the value needed by the surrounding template logic.

**CN:** 计算并返回外围模板逻辑所需的值。

### Lines 51-55
```cpp
 51:     // Maps to modes ((w,h,n), C)
 52:     else if constexpr (DispatchPolicy::NumSpatialDimensions == 2) {
 53:       return cute::Stride<cute::Stride<int64_t, int64_t, int64_t>,
 54:                           cute::Int<1>>{};
 55:     }
```
**EN:** Computes and returns the value needed by the surrounding template logic.

**CN:** 计算并返回外围模板逻辑所需的值。

### Lines 56-60
```cpp
 56:     // Maps to modes ((w,h,d,n), C)
 57:     else if constexpr (DispatchPolicy::NumSpatialDimensions == 3) {
 58:       return cute::Stride<cute::Stride<int64_t, int64_t, int64_t, int64_t>,
 59:                           cute::Int<1>>{};
 60:     }
```
**EN:** Computes and returns the value needed by the surrounding template logic.

**CN:** 计算并返回外围模板逻辑所需的值。

### Lines 61-66
```cpp
 61:     // error dims assert
 62:     else {
 63:       static_assert(cutlass::detail::dependent_false<DispatchPolicy>, "Unsupported spatial dim count.");
 64:     }
 65:   }
 66:   else if constexpr (DispatchPolicy::ConvOp == conv::Operator::kWgrad) {
```
**EN:** Defines function `constexpr` for this stage of the convolution workflow.

**CN:** 定义函数 `constexpr`，服务于卷积工作流的这一阶段。

### Lines 67-72
```cpp
 67:     // Maps to modes (k, nq/npq/nzpq)
 68:     if constexpr (DispatchPolicy::NumSpatialDimensions == 1 ||
 69:                   DispatchPolicy::NumSpatialDimensions == 2 ||
 70:                   DispatchPolicy::NumSpatialDimensions == 3) {
 71:       return cute::Stride<cute::Int<1>, int64_t>{};
 72:     }
```
**EN:** Computes and returns the value needed by the surrounding template logic.

**CN:** 计算并返回外围模板逻辑所需的值。

### Lines 73-78
```cpp
 73:     // error dims assert
 74:     else {
 75:       static_assert(cutlass::detail::dependent_false<DispatchPolicy>, "Unsupported spatial dim count.");
 76:     }
 77:   }
 78:   else if constexpr (DispatchPolicy::ConvOp == conv::Operator::kDgrad) {
```
**EN:** Defines function `constexpr` for this stage of the convolution workflow.

**CN:** 定义函数 `constexpr`，服务于卷积工作流的这一阶段。

### Lines 79-83
```cpp
 79:     // Maps to modes ((q,n), K)
 80:     if constexpr (DispatchPolicy::NumSpatialDimensions == 1) {
 81:       return cute::Stride<cute::Stride<int64_t, int64_t>,
 82:                           cute::Int<1>>{};
 83:     }
```
**EN:** Computes and returns the value needed by the surrounding template logic.

**CN:** 计算并返回外围模板逻辑所需的值。

### Lines 84-88
```cpp
 84:     // Maps to modes ((q,p,n), K)
 85:     else if constexpr (DispatchPolicy::NumSpatialDimensions == 2) {
 86:       return cute::Stride<cute::Stride<int64_t, int64_t, int64_t>,
 87:                           cute::Int<1>>{};
 88:     }
```
**EN:** Computes and returns the value needed by the surrounding template logic.

**CN:** 计算并返回外围模板逻辑所需的值。

### Lines 89-93
```cpp
 89:     // Maps to modes ((q,p,z,n), K)
 90:     else if constexpr (DispatchPolicy::NumSpatialDimensions == 3) {
 91:       return cute::Stride<cute::Stride<int64_t, int64_t, int64_t, int64_t>,
 92:                           cute::Int<1>>{};
 93:     }
```
**EN:** Computes and returns the value needed by the surrounding template logic.

**CN:** 计算并返回外围模板逻辑所需的值。

### Lines 94-102
```cpp
 94:     // error dims assert
 95:     else {
 96:       static_assert(cutlass::detail::dependent_false<DispatchPolicy>, "Unsupported spatial dim count.");
 97:     }
 98:   }
 99:   else {
100:     static_assert(cutlass::detail::dependent_false<DispatchPolicy>, "Unsupported ConvOp.");
101:   }
102: }
```
**EN:** Defines function `static_assert` for this stage of the convolution workflow.

**CN:** 定义函数 `static_assert`，服务于卷积工作流的这一阶段。

### Lines 104-108
```cpp
104: // Construct the stirde types for conv collectives based on the dispatch policy, strides 64b by default
105: template <class DispatchPolicy>
106: constexpr auto
107: sm90_dispatch_policy_to_stride_B() {
108:   if constexpr (DispatchPolicy::ConvOp == conv::Operator::kFprop) {
```
**EN:** Defines function `sm90_dispatch_policy_to_stride_B` for this stage of the convolution workflow.

**CN:** 定义函数 `sm90_dispatch_policy_to_stride_B`，服务于卷积工作流的这一阶段。

### Lines 109-112
```cpp
109:     // Maps to modes (k, (C,s))
110:     if constexpr      (DispatchPolicy::NumSpatialDimensions == 1) {
111:       return cute::Stride<int64_t, cute::Stride<cute::Int<1>, int64_t>>{};
112:     }
```
**EN:** Computes and returns the value needed by the surrounding template logic.

**CN:** 计算并返回外围模板逻辑所需的值。

### Lines 113-116
```cpp
113:     // Maps to modes (k, (C,s,r))
114:     else if constexpr (DispatchPolicy::NumSpatialDimensions == 2) {
115:       return cute::Stride<int64_t, cute::Stride<cute::Int<1>, int64_t, int64_t>>{};
116:     }
```
**EN:** Computes and returns the value needed by the surrounding template logic.

**CN:** 计算并返回外围模板逻辑所需的值。

### Lines 117-120
```cpp
117:     // Maps to modes (k, (C,s,r,t))
118:     else if constexpr (DispatchPolicy::NumSpatialDimensions == 3) {
119:       return cute::Stride<int64_t, cute::Stride<cute::Int<1>, int64_t, int64_t, int64_t>>{};
120:     }
```
**EN:** Computes and returns the value needed by the surrounding template logic.

**CN:** 计算并返回外围模板逻辑所需的值。

### Lines 121-126
```cpp
121:     // error dims assert
122:     else {
123:       static_assert(cutlass::detail::dependent_false<DispatchPolicy>, "Unsupported spatial dim count.");
124:     }
125:   }
126:   else if constexpr (DispatchPolicy::ConvOp == conv::Operator::kWgrad) {
```
**EN:** Defines function `constexpr` for this stage of the convolution workflow.

**CN:** 定义函数 `constexpr`，服务于卷积工作流的这一阶段。

### Lines 127-131
```cpp
127:     // Maps to modes (C, (w,n))
128:     if constexpr (DispatchPolicy::NumSpatialDimensions == 1) {
129:       return cute::Stride<cute::Int<1>,
130:                           cute::Stride<int64_t, int64_t>>{};
131:     }
```
**EN:** Computes and returns the value needed by the surrounding template logic.

**CN:** 计算并返回外围模板逻辑所需的值。

### Lines 132-136
```cpp
132:     // Maps to modes (C, (w,h,n))
133:     else if constexpr (DispatchPolicy::NumSpatialDimensions == 2) {
134:       return cute::Stride<cute::Int<1>,
135:                           cute::Stride<int64_t, int64_t, int64_t>>{};
136:     }
```
**EN:** Computes and returns the value needed by the surrounding template logic.

**CN:** 计算并返回外围模板逻辑所需的值。

### Lines 137-141
```cpp
137:     // Maps to modes (C, (w,h,d,n))
138:     else if constexpr (DispatchPolicy::NumSpatialDimensions == 3) {
139:       return cute::Stride<cute::Int<1>,
140:                           cute::Stride<int64_t, int64_t, int64_t, int64_t>>{};
141:     }
```
**EN:** Computes and returns the value needed by the surrounding template logic.

**CN:** 计算并返回外围模板逻辑所需的值。

### Lines 142-147
```cpp
142:     // error dims assert
143:     else {
144:       static_assert(cutlass::detail::dependent_false<DispatchPolicy>, "Unsupported spatial dim count.");
145:     }
146:   }
147:   else if constexpr (DispatchPolicy::ConvOp == conv::Operator::kDgrad) {
```
**EN:** Defines function `constexpr` for this stage of the convolution workflow.

**CN:** 定义函数 `constexpr`，服务于卷积工作流的这一阶段。

### Lines 148-151
```cpp
148:     // Maps to modes (C, (k,s))
149:     if constexpr      (DispatchPolicy::NumSpatialDimensions == 1) {
150:       return cute::Stride<cute::Int<1>, cute::Stride<int64_t, int64_t>>{};
151:     }
```
**EN:** Computes and returns the value needed by the surrounding template logic.

**CN:** 计算并返回外围模板逻辑所需的值。

### Lines 152-155
```cpp
152:     // Maps to modes (C, (k,s,r))
153:     else if constexpr (DispatchPolicy::NumSpatialDimensions == 2) {
154:       return cute::Stride<cute::Int<1>, cute::Stride<int64_t, int64_t, int64_t>>{};
155:     }
```
**EN:** Computes and returns the value needed by the surrounding template logic.

**CN:** 计算并返回外围模板逻辑所需的值。

### Lines 156-159
```cpp
156:     // Maps to modes (C, (k,s,r,t))
157:     else if constexpr (DispatchPolicy::NumSpatialDimensions == 3) {
158:       return cute::Stride<cute::Int<1>, cute::Stride<int64_t, int64_t, int64_t, int64_t>>{};
159:     }
```
**EN:** Computes and returns the value needed by the surrounding template logic.

**CN:** 计算并返回外围模板逻辑所需的值。

### Lines 160-168
```cpp
160:     // error dims assert
161:     else {
162:       static_assert(cutlass::detail::dependent_false<DispatchPolicy>, "Unsupported spatial dim count.");
163:     }
164:   }
165:   else {
166:     static_assert(cutlass::detail::dependent_false<DispatchPolicy>, "Unsupported ConvOp.");
167:   }
168: }
```
**EN:** Defines function `static_assert` for this stage of the convolution workflow.

**CN:** 定义函数 `static_assert`，服务于卷积工作流的这一阶段。

### Lines 171-175
```cpp
171: template <class DispatchPolicy>
172: constexpr auto
173: sm100_dispatch_policy_to_stride_A() {
174:   return sm90_dispatch_policy_to_stride_A<DispatchPolicy>();
175: }
```
**EN:** Computes and returns the value needed by the surrounding template logic.

**CN:** 计算并返回外围模板逻辑所需的值。

### Lines 177-181
```cpp
177: template <class DispatchPolicy>
178: constexpr auto
179: sm100_dispatch_policy_to_stride_B() {
180:   return sm90_dispatch_policy_to_stride_B<DispatchPolicy>();
181: }
```
**EN:** Computes and returns the value needed by the surrounding template logic.

**CN:** 计算并返回外围模板逻辑所需的值。

### Lines 186-192
```cpp
186: // Compute the lower/near corner, returning it as a cute::array in [W,H,D] order
187: template <conv::Operator ConvOp, int NumSpatialDimensions>
188: CUTLASS_HOST_DEVICE
189: constexpr auto
190: compute_lower_corner_whd(ConvProblemShape<ConvOp, NumSpatialDimensions> const& problem_shape) {
191:   using cute::for_each;
192:   using cute::make_seq;
```
**EN:** Defines function `compute_lower_corner_whd` for this stage of the convolution workflow.

**CN:** 定义函数 `compute_lower_corner_whd`，服务于卷积工作流的这一阶段。

### Lines 194-208
```cpp
194:   cute::array<int, NumSpatialDimensions> lower{};
195:   if constexpr (ConvOp == conv::Operator::kFprop ||
196:                 ConvOp == conv::Operator::kWgrad) {
197:     for_each(make_seq<NumSpatialDimensions>{}, [&](auto i) {
198:       lower[NumSpatialDimensions-1-i] = -1 * problem_shape.lower_padding[i];
199:     });
200:   }
201:   else if constexpr (ConvOp == conv::Operator::kDgrad) {
202:     for_each(make_seq<NumSpatialDimensions>{}, [&](auto i) {
203:       lower[NumSpatialDimensions-1-i] = problem_shape.lower_padding[i] -
204:         (problem_shape.shape_B[i+1] - 1) * problem_shape.dilation[i];
205:     });
206:   }
207:   return lower;
208: }
```
**EN:** Computes and returns the value needed by the surrounding template logic.

**CN:** 计算并返回外围模板逻辑所需的值。

### Lines 210-216
```cpp
210: // Computes the upper/far corner, returning it as a cute::array in [W,H,D] order
211: template <conv::Operator ConvOp, int NumSpatialDimensions>
212: CUTLASS_HOST_DEVICE
213: constexpr auto
214: compute_upper_corner_whd(ConvProblemShape<ConvOp, NumSpatialDimensions> const& problem_shape) {
215:   using cute::for_each;
216:   using cute::make_seq;
```
**EN:** Defines function `compute_upper_corner_whd` for this stage of the convolution workflow.

**CN:** 定义函数 `compute_upper_corner_whd`，服务于卷积工作流的这一阶段。

### Lines 218-238
```cpp
218:   cute::array<int, NumSpatialDimensions> upper{};
219:   if constexpr (ConvOp == conv::Operator::kFprop) {
220:     for_each(make_seq<NumSpatialDimensions>{}, [&](auto i) {
221:       upper[NumSpatialDimensions-1-i] = problem_shape.upper_padding[i] -
222:         (problem_shape.shape_B[i+1] - 1) * problem_shape.dilation[i];
223:     });
224:   }
225:   else if constexpr (ConvOp == conv::Operator::kWgrad) {
226:     for_each(make_seq<NumSpatialDimensions>{}, [&](auto i) {
227:       upper[NumSpatialDimensions-1-i] = problem_shape.upper_padding[i] -
228:         (problem_shape.shape_C[i+1] - 1) * problem_shape.dilation[i];
229:     });
230:   }
231:   else if constexpr (ConvOp == conv::Operator::kDgrad) {
232:     for_each(make_seq<NumSpatialDimensions>{}, [&](auto i) {
233:       upper[NumSpatialDimensions-1-i] = problem_shape.lower_padding[i] -
234:         (problem_shape.shape_B[i+1] - 1) * problem_shape.dilation[i] + problem_shape.shape_C[i+1] - problem_shape.shape_A[i+1];
235:     });
236:   }
237:   return upper;
238: }
```
**EN:** Computes and returns the value needed by the surrounding template logic.

**CN:** 计算并返回外围模板逻辑所需的值。

### Lines 240-246
```cpp
240: // Compute the lower/near corner of (t,r,s), returning it as a cute::array in [S,R,T] order
241: template <conv::Operator ConvOp, int NumSpatialDimensions>
242: CUTLASS_HOST_DEVICE
243: constexpr auto
244: compute_lower_srt(ConvProblemShape<ConvOp, NumSpatialDimensions> const& problem_shape) {
245:   using cute::for_each;
246:   using cute::make_seq;
```
**EN:** Defines function `compute_lower_srt` for this stage of the convolution workflow.

**CN:** 定义函数 `compute_lower_srt`，服务于卷积工作流的这一阶段。

### Lines 248-261
```cpp
248:   cute::array<int, NumSpatialDimensions> lower{};
249:   if constexpr (ConvOp == conv::Operator::kFprop ||
250:                 ConvOp == conv::Operator::kWgrad) {
251:     for_each(make_seq<NumSpatialDimensions>{}, [&](auto i) {
252:       lower[NumSpatialDimensions-1-i] = 0;
253:     });
254:   }
255:   else if constexpr (ConvOp == conv::Operator::kDgrad) {
256:     for_each(make_seq<NumSpatialDimensions>{}, [&](auto i) {
257:       lower[NumSpatialDimensions-1-i] = (problem_shape.shape_B[i+1] - 1) * problem_shape.dilation[i];
258:     });
259:   }
260:   return lower;
261: }
```
**EN:** Computes and returns the value needed by the surrounding template logic.

**CN:** 计算并返回外围模板逻辑所需的值。

### Lines 263-267
```cpp
263: template <class CopyOp> struct is_im2col_load { static constexpr bool value = false; };
264: template <> struct is_im2col_load<cute::SM90_TMA_LOAD_IM2COL          > { static constexpr bool value = true; };
265: template <> struct is_im2col_load<cute::SM90_TMA_LOAD_IM2COL_MULTICAST> { static constexpr bool value = true; };
266: template <> struct is_im2col_load<cute::SM100_TMA_2SM_LOAD_IM2COL          > { static constexpr bool value = true; }; 
267: template <> struct is_im2col_load<cute::SM100_TMA_2SM_LOAD_IM2COL_MULTICAST> { static constexpr bool value = true; }; 
```
**EN:** Defines compile-time constants such as `value` that parameterize later logic.

**CN:** 定义 `value` 等编译期常量，用来参数化后续逻辑。

### Lines 271-271
```cpp
271: } // namespace cutlass::conv::collective::detail
```
**EN:** Closes the current scope and finishes the surrounding definition block.

**CN:** 关闭当前作用域，并结束外围定义块。

## Key Concepts / 关键概念
- **EN:** Main role: Implements collective-level convolution building blocks centered on detail. **CN:** 核心作用：实现以 细节 为中心的 collective 级卷积构件。
- **EN:** Key exported symbols include `is_im2col_load`, `DispatchPolicy`, `CopyOp`. **CN:** 关键导出符号包括 `is_im2col_load`, `DispatchPolicy`, `CopyOp`。
- **EN:** Heavy use of C++ templates encodes shapes, layouts, datatypes, and architecture choices. **CN:** 大量使用 C++ 模板来编码形状、布局、数据类型和架构选择。

## Dependencies / 依赖关系
### Direct Includes / 直接包含
- `cutlass/conv/convnd_problem_shape.hpp`

### Internal Relationships / 内部关系
- **EN:** Uses CUTE metaprogramming and shape utilities. **CN:** 使用 CUTE 元编程与形状工具。
