# print_error.hpp — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `tools/util/include/cutlass/util/print_error.hpp`
- **Purpose (EN):** This file declares print error for general CUTLASS utility support.
- **目的 (CN):** 该文件声明了面向CUTLASS 通用工具支持的print error逻辑。

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

### Lines 32-32
```cpp
32: #pragma once
```
- **EN:** Uses include guards or pragmas to ensure the declarations are processed only once.
- **CN:** 使用 include guard 或 pragma，确保这些声明只被处理一次。

### Lines 34-38
```cpp
34: #include <array>
35: #include <cassert>
36: #include <cmath>
37: #include <iostream>
38: #include <type_traits>
```
- **EN:** Includes the main dependencies required by this module; notable headers are `array`, `cassert`, `cmath`, `iostream`, `type_traits`.
- **CN:** 引入该模块所需的主要依赖；较重要的头文件有 `array`, `cassert`, `cmath`, `iostream`, `type_traits`。

### Lines 40-41
```cpp
40: #include <cute/util/type_traits.hpp>
41: #include <cute/tensor.hpp>
```
- **EN:** Includes the main dependencies required by this module; notable headers are `cute/util/type_traits.hpp`, `cute/tensor.hpp`.
- **CN:** 引入该模块所需的主要依赖；较重要的头文件有 `cute/util/type_traits.hpp`, `cute/tensor.hpp`。

### Lines 43-44
```cpp
43: #include <cute/numeric/numeric_types.hpp>
44: #include <cute/numeric/complex.hpp>
```
- **EN:** Includes the main dependencies required by this module; notable headers are `cute/numeric/numeric_types.hpp`, `cute/numeric/complex.hpp`.
- **CN:** 引入该模块所需的主要依赖；较重要的头文件有 `cute/numeric/numeric_types.hpp`, `cute/numeric/complex.hpp`。

### Lines 46-46
```cpp
46: #include <cutlass/layout/layout.h>
```
- **EN:** Includes the main dependencies required by this module; notable headers are `cutlass/layout/layout.h`.
- **CN:** 引入该模块所需的主要依赖；较重要的头文件有 `cutlass/layout/layout.h`。

### Lines 48-49
```cpp
48: // The computed infinity norm does not include
49: // any NaN column absolute-value sums.
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 50-50
```cpp
50: struct matrix_inf_norm_result {
```
- **EN:** Introduces `matrix_inf_norm_result`, a type used to support print error.
- **CN:** 引入 `matrix_inf_norm_result`，即一个用于支持print error的类型。

### Lines 51-52
```cpp
51:   // Accumulate errors in double, as this is generally
52:   // the highest precision that the examples use.
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 53-55
```cpp
53:   double inf_norm = 0.0;
54:   bool found_nan = false;
55: };
```
- **EN:** Declares or updates local/member state such as `inf_norm`, `found_nan`, `false`.
- **CN:** 声明或更新局部/成员状态，例如 `inf_norm`, `found_nan`, `false`。

### Lines 57-60
```cpp
57: // In theory, cute::Tensor<ViewEngine<T*>, T> could be treated as a view type,
58: // and thus passed by value (as std::span or std::string_view would be).
59: // However, generic cute::Tensor are more like containers
60: // and thus are best passed by reference or const reference.
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 61-66
```cpp
61: template <typename EngineType, typename LayoutType>
62: matrix_inf_norm_result
63: matrix_inf_norm(cute::Tensor<EngineType, LayoutType> const& host_matrix)
64: {
65:   using error_type = decltype(std::declval<matrix_inf_norm_result>().inf_norm);
66:   using element_type = typename EngineType::value_type;
```
- **EN:** Implements `matrix_inf_norm` for this file's main component.
- **CN:** 为该文件的核心组件实现 `matrix_inf_norm`。

### Lines 68-69
```cpp
68:   error_type inf_norm = 0.0;
69:   bool found_nan = false;
```
- **EN:** Declares or updates local/member state such as `inf_norm`, `found_nan`, `false`.
- **CN:** 声明或更新局部/成员状态，例如 `inf_norm`, `found_nan`, `false`。

### Lines 71-72
```cpp
71:   // Computing the infinity norm requires that we be able
72:   // to treat the input as a matrix, with rows and columns.
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 73-74
```cpp
73:   const int64_t num_rows = cute::size<0>(host_matrix);
74:   const int64_t num_cols = cute::size<1>(host_matrix);
```
- **EN:** Declares or updates local/member state such as `num_rows`, `num_cols`.
- **CN:** 声明或更新局部/成员状态，例如 `num_rows`, `num_cols`。

### Lines 76-84
```cpp
76:   auto abs_fn = [] (element_type A_ij) {
77:     if constexpr (not std::is_unsigned_v<element_type>) {
78:       using std::abs;
79:       return abs(A_ij);
80:     }
81:     else {
82:       return A_ij;
83:     }
84:   };
```
- **EN:** Implements `constexpr` and coordinates helper calls such as `abs`.
- **CN:** 实现 `constexpr`，并协调调用 `abs` 等辅助逻辑。

### Lines 86-97
```cpp
86:   for (int64_t i = 0; i < num_rows; ++i) {
87:     error_type row_abs_sum = 0.0;
88:     for(int64_t j = 0; j < num_cols; ++j) {
89:       row_abs_sum += abs_fn(host_matrix(i, j));
90:     }
91:     if (std::isnan(row_abs_sum)) {
92:       found_nan = true;
93:     }
94:     else {
95:       inf_norm = row_abs_sum > inf_norm ? row_abs_sum : inf_norm;
96:     }
97:   }
```
- **EN:** Declares or updates local/member state such as `i`, `num_rows`, `row_abs_sum`, `j`.
- **CN:** 声明或更新局部/成员状态，例如 `i`, `num_rows`, `row_abs_sum`, `j`。

### Lines 99-100
```cpp
99:   return {inf_norm, found_nan};
100: }
```
- **EN:** Returns the computed value, status, or object to the caller.
- **CN:** 向调用者返回计算得到的值、状态或对象。

### Lines 102-102
```cpp
102: // Infinity norm of (X - Y).
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 103-109
```cpp
103: template <typename EngineType, typename LayoutType>
104: matrix_inf_norm_result
105: matrix_diff_inf_norm(cute::Tensor<EngineType, LayoutType> const& X,
106:                      cute::Tensor<EngineType, LayoutType> const& Y)
107: {
108:   using error_type = decltype(std::declval<matrix_inf_norm_result>().inf_norm);
109:   using element_type = typename EngineType::value_type;
```
- **EN:** Implements `matrix_diff_inf_norm` for this file's main component.
- **CN:** 为该文件的核心组件实现 `matrix_diff_inf_norm`。

### Lines 111-119
```cpp
111:   auto abs_fn = [] (element_type A_ij) {
112:     if constexpr (not std::is_unsigned_v<element_type>) {
113:       using std::abs;
114:       return abs(A_ij);
115:     }
116:     else {
117:       return A_ij;
118:     }
119:   };
```
- **EN:** Implements `constexpr` and coordinates helper calls such as `abs`.
- **CN:** 实现 `constexpr`，并协调调用 `abs` 等辅助逻辑。

### Lines 121-122
```cpp
121:   assert(cute::size<0>(X) == cute::size<0>(Y));
122:   assert(cute::size<1>(X) == cute::size<1>(Y));
```
- **EN:** Supporting logic for the print error implementation.
- **CN:** print error实现的辅助逻辑。

### Lines 124-125
```cpp
124:   // Computing the infinity norm requires that we be able
125:   // to treat the input as a matrix, with rows and columns.
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 126-127
```cpp
126:   const int64_t num_rows = cute::size<0>(X);
127:   const int64_t num_cols = cute::size<1>(X);
```
- **EN:** Declares or updates local/member state such as `num_rows`, `num_cols`.
- **CN:** 声明或更新局部/成员状态，例如 `num_rows`, `num_cols`。

### Lines 129-130
```cpp
129:   error_type inf_norm = 0.0;
130:   bool found_nan = false;
```
- **EN:** Declares or updates local/member state such as `inf_norm`, `found_nan`, `false`.
- **CN:** 声明或更新局部/成员状态，例如 `inf_norm`, `found_nan`, `false`。

### Lines 132-144
```cpp
132:   for (int64_t i = 0; i < num_rows; ++i) {
133:     error_type row_abs_sum = 0.0;
134:     for (int64_t j = 0; j < num_cols; ++j) {
135:       row_abs_sum += error_type(abs_fn(element_type(X(i,j)) -
136:                                        element_type(Y(i,j))));
137:     }
138:     if (std::isnan(row_abs_sum)) {
139:       found_nan = true;
140:     }
141:     else {
142:       inf_norm = row_abs_sum > inf_norm ? row_abs_sum : inf_norm;
143:     }
144:   }
```
- **EN:** Declares or updates local/member state such as `i`, `num_rows`, `row_abs_sum`, `j`.
- **CN:** 声明或更新局部/成员状态，例如 `i`, `num_rows`, `row_abs_sum`, `j`。

### Lines 146-147
```cpp
146:   return {inf_norm, found_nan};
147: }
```
- **EN:** Returns the computed value, status, or object to the caller.
- **CN:** 向调用者返回计算得到的值、状态或对象。

### Lines 149-162
```cpp
149: template <typename EngineType_A, typename LayoutType_A,
150:           typename EngineType_B, typename LayoutType_B,
151:           typename EngineType_C, typename LayoutType_C,
152:           typename EngineType_C_ref, typename LayoutType_C_ref>
153: auto
154: print_matrix_multiply_mollified_relative_error(
155:   char const A_value_type_name[],
156:   cute::Tensor<EngineType_A, LayoutType_A> const& A,
157:   char const B_value_type_name[],
158:   cute::Tensor<EngineType_B, LayoutType_B> const& B,
159:   char const C_value_type_name[],
160:   cute::Tensor<EngineType_C, LayoutType_C> const& C,
161:   cute::Tensor<EngineType_C_ref, LayoutType_C_ref> const& C_ref)
162: {
```
- **EN:** Supporting logic for the print error implementation.
- **CN:** print error实现的辅助逻辑。

### Lines 163-166
```cpp
163:   const auto [A_norm, A_has_nan] = matrix_inf_norm(A);
164:   const auto [B_norm, B_has_nan] = matrix_inf_norm(B);
165:   const auto [C_norm, C_has_nan] = matrix_inf_norm(C_ref);
166:   const auto [diff_norm, diff_has_nan] = matrix_diff_inf_norm(C, C_ref);
```
- **EN:** Implements `matrix_inf_norm` and coordinates helper calls such as `matrix_diff_inf_norm`.
- **CN:** 实现 `matrix_inf_norm`，并协调调用 `matrix_diff_inf_norm` 等辅助逻辑。

### Lines 168-170
```cpp
168:   const auto A_norm_times_B_norm = A_norm * B_norm;
169:   const auto relative_error = A_norm_times_B_norm == 0.0 ?
170:     diff_norm : (diff_norm / A_norm_times_B_norm);
```
- **EN:** Declares or updates local/member state such as `A_norm_times_B_norm`, `B_norm`, `relative_error`.
- **CN:** 声明或更新局部/成员状态，例如 `A_norm_times_B_norm`, `B_norm`, `relative_error`。

### Lines 172-177
```cpp
172:   // For expected error bounds, please refer to the LAPACK Users' Guide,
173:   // in particular https://netlib.org/lapack/lug/node108.html .
174:   // Printing the infinity norm of C is a way to check
175:   // that both the function being tested (C)
176:   // and the reference implementation (C_ref)
177:   // don't just do nothing (or fill with zeros).
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 178-187
```cpp
178:   using std::cout;
179:   using cute::shape;
180:   cout << "Matrix A: " << shape<0>(A) << "x" << shape<1>(A) << " of " << A_value_type_name << '\n'
181:       << "Matrix B: " << shape<0>(B) << "x" << shape<1>(B) << " of " << B_value_type_name << '\n'
182:       << "Matrix C: " << shape<0>(C) << "x" << shape<1>(C) << " of " << C_value_type_name << '\n'
183:       << std::scientific
184:       << "Infinity norm of A: " << A_norm << '\n'
185:       << "Infinity norm of B: " << B_norm << '\n'
186:       << "Infinity norm of C: " << C_norm << '\n'
187:       << "Infinity norm of (C - C_ref): " << diff_norm << '\n';
```
- **EN:** Introduces namespace imports, aliases, or shorthand names used by later code.
- **CN:** 引入后续代码会使用的命名空间、别名或简写名称。

### Lines 189-193
```cpp
189:   if(A_norm_times_B_norm == 0.0) {
190:     cout << "Mollified relative error: " << relative_error << '\n';
191:   } else {
192:     cout << "Relative error: " << relative_error << '\n';
193:   }
```
- **EN:** Declares or updates local/member state such as `A_norm_times_B_norm`.
- **CN:** 声明或更新局部/成员状态，例如 `A_norm_times_B_norm`。

### Lines 195-202
```cpp
195:   if (A_has_nan || B_has_nan || C_has_nan || diff_has_nan) {
196:     cout << "Did we encounter NaN in A? " << (A_has_nan ? "yes" : "no") << '\n'
197:         << "Did we encounter NaN in B? " << (B_has_nan ? "yes" : "no") << '\n'
198:         << "Did we encounter NaN in C? " << (C_has_nan ? "yes" : "no") << '\n'
199:         << "Did we encounter NaN in (C - C_ref)? " << (diff_has_nan ? "yes" : "no") << '\n';
200:   }
201:   return relative_error;
202: }
```
- **EN:** Returns the computed value, status, or object to the caller.
- **CN:** 向调用者返回计算得到的值、状态或对象。

### Lines 204-215
```cpp
204: template <typename EngineType, typename LayoutType>
205: auto
206: print_matrix_multiply_mollified_relative_error(
207:   const char value_type_name[],
208:   const cute::Tensor<EngineType, LayoutType>& A,
209:   const cute::Tensor<EngineType, LayoutType>& B,
210:   const cute::Tensor<EngineType, LayoutType>& C_computed,
211:   const cute::Tensor<EngineType, LayoutType>& C_expected)
212: {
213:   return print_matrix_multiply_mollified_relative_error(value_type_name, A, value_type_name, B,
214:                                                  value_type_name, C_computed, C_expected);
215: }
```
- **EN:** Returns the computed value, status, or object to the caller.
- **CN:** 向调用者返回计算得到的值、状态或对象。

### Lines 217-223
```cpp
217: // Take a CUTLASS HostTensor (or the like) as input,
218: // and return a const CuTe Tensor.
219: // This is useful for use with the above error printing functions.
220: // This implicitly "transposes" if the layout is RowMajor.
221: // Note that the HostTensor must be captured by nonconst reference
222: // in order for X.host_ref().data() to compile.
223: // (CUTLASS is a bit more container-y than CuTe.)
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 224-226
```cpp
224: template<class CutlassHostTensorType>
225: auto host_matrix_to_const_cute_tensor(CutlassHostTensorType& X)
226: {
```
- **EN:** Introduces `CutlassHostTensorType`, a type used to support print error.
- **CN:** 引入 `CutlassHostTensorType`，即一个用于支持print error的类型。

### Lines 227-227
```cpp
227:   // The tensors were created with post-transposed extents.
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 228-229
```cpp
228:   const auto extents = X.extent();
229:   const auto shape = cute::Shape<int, int>{extents[0], extents[1]};
```
- **EN:** Implements `extent` for this file's main component.
- **CN:** 为该文件的核心组件实现 `extent`。

### Lines 230-230
```cpp
230:   // Both RowMajor and ColumnMajor only store one stride.
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 231-244
```cpp
231:   const int LDX = X.stride(0);
232:   const auto strides = [&]() {
233:       using input_layout_type = typename std::decay_t<decltype(X)>::Layout;
234:       if constexpr (std::is_same_v<input_layout_type, cutlass::layout::ColumnMajor>) {
235:         return cute::Stride<int, int>{1, LDX};
236:       }
237:       else {
238:         static_assert(std::is_same_v<input_layout_type, cutlass::layout::RowMajor>);
239:         return cute::Stride<int, int>{LDX, 1};
240:       }
241:     }();
242:   const auto layout = cute::make_layout(shape, strides);
243:   auto X_data = X.host_ref().data();
244:   auto X_data_const = const_cast<std::add_const_t< decltype(X_data)> >(X_data);
```
- **EN:** Implements `stride` and coordinates helper calls such as `constexpr`, `make_layout`, `host_ref`.
- **CN:** 实现 `stride`，并协调调用 `constexpr`, `make_layout`, `host_ref` 等辅助逻辑。

### Lines 245-246
```cpp
245:   return cute::make_tensor(X_data_const, layout);
246: };
```
- **EN:** Returns the computed value, status, or object to the caller.
- **CN:** 向调用者返回计算得到的值、状态或对象。

### Lines 249-250
```cpp
249: // Returns EXIT_SUCCESS if the 2-norm relative error is exactly zero, else returns EXIT_FAILURE.
250: // This makes the return value suitable as the return value of main().
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 251-260
```cpp
251: template <typename T1, typename T2>
252: int
253: print_relative_error(
254:     std::size_t n,
255:     T1 const& data,
256:     T2 const& reference,
257:     bool print_verbose = false,
258:     bool print_error = true,
259:     double error_margin = 0.00001) {
260:   using std::abs; using std::sqrt;
```
- **EN:** Declares or updates local/member state such as `print_verbose`, `print_error`, `error_margin`, `abs`.
- **CN:** 声明或更新局部/成员状态，例如 `print_verbose`, `print_error`, `error_margin`, `abs`。

### Lines 262-262
```cpp
262:   // Use either double or complex<double> for error computation
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 263-266
```cpp
263:   using value_type = cute::remove_cvref_t<decltype(reference[0])>;
264:   using error_type = std::conditional_t<cute::is_complex<value_type>::value,
265:                                         cute::complex<double>,
266:                                         double>;
```
- **EN:** Introduces namespace imports, aliases, or shorthand names used by later code.
- **CN:** 引入后续代码会使用的命名空间、别名或简写名称。

### Lines 268-270
```cpp
268:   if (print_verbose) {
269:     std::cout << "Idx:\t"<< "Val\t" << "RefVal\t" << "RelError" << std::endl;
270:   }
```
- **EN:** Emits debug or diagnostic output so intermediate state can be inspected.
- **CN:** 输出调试或诊断信息，便于检查中间状态。

### Lines 272-272
```cpp
272:   double eps = 1e-200;
```
- **EN:** Declares or updates local/member state such as `eps`.
- **CN:** 声明或更新局部/成员状态，例如 `eps`。

### Lines 274-281
```cpp
274:   double tot_error_sq = 0;
275:   double tot_norm_sq = 0;
276:   double tot_ind_rel_err = 0;
277:   double max_ind_rel_err = 0;
278:   double max_diff = 0;
279:   for (std::size_t i = 0; i < n; ++i) {
280:     error_type val = data[i];
281:     error_type ref = reference[i];
```
- **EN:** Declares or updates local/member state such as `tot_error_sq`, `tot_norm_sq`, `tot_ind_rel_err`, `max_ind_rel_err`.
- **CN:** 声明或更新局部/成员状态，例如 `tot_error_sq`, `tot_norm_sq`, `tot_ind_rel_err`, `max_ind_rel_err`。

### Lines 283-285
```cpp
283:     double aref = abs(ref);
284:     double diff = abs(ref - val);
285:     double rel_error = diff / (aref + eps);
```
- **EN:** Declares or updates local/member state such as `aref`, `diff`, `rel_error`.
- **CN:** 声明或更新局部/成员状态，例如 `aref`, `diff`, `rel_error`。

### Lines 287-287
```cpp
287:     // Individual relative error
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 288-288
```cpp
288:     tot_ind_rel_err += rel_error;
```
- **EN:** Declares or updates local/member state such as `rel_error`.
- **CN:** 声明或更新局部/成员状态，例如 `rel_error`。

### Lines 290-290
```cpp
290:     // Maximum relative error
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 291-291
```cpp
291:     max_ind_rel_err  = std::max(max_ind_rel_err, rel_error);
```
- **EN:** Implements `max` for this file's main component.
- **CN:** 为该文件的核心组件实现 `max`。

### Lines 293-293
```cpp
293:     // Maximum delta in value error
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 294-294
```cpp
294:     max_diff = std::max(max_diff, diff);
```
- **EN:** Implements `max` for this file's main component.
- **CN:** 为该文件的核心组件实现 `max`。

### Lines 296-296
```cpp
296:     // Total relative error
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 297-298
```cpp
297:     tot_error_sq += diff * diff;
298:     tot_norm_sq  += aref * aref;
```
- **EN:** Declares or updates local/member state such as `diff`, `aref`.
- **CN:** 声明或更新局部/成员状态，例如 `diff`, `aref`。

### Lines 300-303
```cpp
300:     if (print_verbose) {
301:       std::cout << i << ":\t" << val << "\t" << ref << "\t" << rel_error << std::endl;
302:     }
303:   }
```
- **EN:** Emits debug or diagnostic output so intermediate state can be inspected.
- **CN:** 输出调试或诊断信息，便于检查中间状态。

### Lines 305-308
```cpp
305:   double ave_rel_err = tot_ind_rel_err / double(n);
306:   if (print_error) {
307:     printf("Average relative error: %.3e\n", ave_rel_err);
308:   }
```
- **EN:** Emits debug or diagnostic output so intermediate state can be inspected.
- **CN:** 输出调试或诊断信息，便于检查中间状态。

### Lines 310-312
```cpp
310:   if (print_error) {
311:     printf("Maximum relative error: %.3e\n", max_ind_rel_err);
312:   }
```
- **EN:** Emits debug or diagnostic output so intermediate state can be inspected.
- **CN:** 输出调试或诊断信息，便于检查中间状态。

### Lines 314-316
```cpp
314:   if (print_error) {
315:     printf("Maximum difference    : %.3e\n", max_diff);
316:   }
```
- **EN:** Emits debug or diagnostic output so intermediate state can be inspected.
- **CN:** 输出调试或诊断信息，便于检查中间状态。

### Lines 318-321
```cpp
318:   double tot_rel_err = sqrt(tot_error_sq/(tot_norm_sq+eps));
319:   if (print_error) {
320:     printf("Vector relative error:  %.3e\n", tot_rel_err);
321:   }
```
- **EN:** Emits debug or diagnostic output so intermediate state can be inspected.
- **CN:** 输出调试或诊断信息，便于检查中间状态。

### Lines 323-323
```cpp
323:   printf("Vector reference  norm: %.3e\n", sqrt(tot_norm_sq));
```
- **EN:** Emits debug or diagnostic output so intermediate state can be inspected.
- **CN:** 输出调试或诊断信息，便于检查中间状态。

### Lines 325-326
```cpp
325:   return (tot_rel_err <= error_margin) ? EXIT_SUCCESS : EXIT_FAILURE;
326: }
```
- **EN:** Returns the computed value, status, or object to the caller.
- **CN:** 向调用者返回计算得到的值、状态或对象。

### Lines 328-328
```cpp
328: // Overload for cute::Tensor<>
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 329-341
```cpp
329: template <class Engine, class Layout>
330: int
331: print_relative_error(
332:     cute::Tensor<Engine, Layout> data,
333:     cute::Tensor<Engine, Layout> reference,
334:     bool print_verbose = false,
335:     bool print_error = true,
336:     double error_margin = 0.00001) {
337:   assert(size(data) == size(reference));
338:   return print_relative_error(static_cast<std::size_t>(size(data)),
339:                               data, reference,
340:                               print_verbose, print_error, error_margin);
341: }
```
- **EN:** Introduces `Engine`, a type used to support print error.
- **CN:** 引入 `Engine`，即一个用于支持print error的类型。

## Key Concepts / 关键概念
- **Template metaprogramming / 模板元编程**
- **Structured type design / 结构化类型设计**
- **Reference implementation / 参考实现**
- **Tensor manipulation / 张量处理**

## Dependencies / 依赖关系
- **Internal headers / 内部头文件:** `cute/util/type_traits.hpp`, `cute/tensor.hpp`, `cute/numeric/numeric_types.hpp`, `cute/numeric/complex.hpp`, `cutlass/layout/layout.h`
- **External headers / 外部头文件:** `array`, `cassert`, `cmath`, `iostream`, `type_traits`
- **Runtime/backends / 运行时与后端:** `CuTe`
