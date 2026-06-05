# math.hpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `include/cute/util/compat/math.hpp`
- Purpose (EN): Implements the compatibility layer that maps CUDA-like concepts onto SYCL/back-end abstractions for atomics, kernels, launch, memory, math, and device queries.
- 作用 (CN): 实现兼容层，把类 CUDA 概念映射到 SYCL/后端抽象，覆盖原子操作、内核、启动、内存、数学和设备查询。

## Line-by-Line Analysis / 逐行分析

### Lines 1-21

```text
    1 | /***************************************************************************
    2 |  *
    3 |  *  Copyright (C) Codeplay Software Ltd.
    4 |  *  Copyright (C) 2025 Intel Corporation, All rights reserved.
    5 |  *
    6 |  *  Part of the LLVM Project, under the Apache License v2.0 with LLVM
    7 |  *  Exceptions. See https://llvm.org/LICENSE.txt for license information.
    8 |  *  SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
    9 |  *
   10 |  *  Unless required by applicable law or agreed to in writing, software
   11 |  *  distributed under the License is distributed on an "AS IS" BASIS,
   12 |  *  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
   13 |  *  See the License for the specific language governing permissions and
   14 |  *  limitations under the License.
   15 |  *
   16 |  *  SYCL compatibility extension
   17 |  *
   18 |  *  math.hpp
   19 |  *
   20 |  *  Description:
   21 |  *    math utilities for the SYCL compatibility extension.
```
**EN:** Provides the license notice, copyright ownership, and redistribution terms for this header.
**CN:** 给出该头文件的许可证声明、版权归属以及再分发条款。

### Lines 22-34

```text
   22 |  **************************************************************************/
   24 | // The original source was under the license below:
   25 | //==---- math.hpp ---------------------------------*- C++ -*----------------==//
   26 | //
   27 | // Copyright (C) Intel Corporation
   28 | // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   29 | // See https://llvm.org/LICENSE.txt for license information.
   30 | //
   31 | //===----------------------------------------------------------------------===//
   33 | #pragma once
   34 | #pragma GCC system_header
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 36-38

```text
   36 | #include <limits>
   37 | #include <sycl/feature_test.hpp>
   38 | #include <type_traits>
```
**EN:** Sets up the header dependencies for this file by importing `limits`, `sycl/feature_test.hpp`, `type_traits`.
**CN:** 通过引入 `limits`, `sycl/feature_test.hpp`, `type_traits` 为该文件建立头文件依赖。

### Lines 40-43

```text
   40 | // TODO(compat-lib-reviewers): this should not be required
   41 | #ifndef SYCL_EXT_ONEAPI_COMPLEX
   42 | #define SYCL_EXT_ONEAPI_COMPLEX
   43 | #endif
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。

### Lines 45-49

```text
   45 | #ifdef SYCL_EXT_ONEAPI_BFLOAT16_MATH_FUNCTIONS
   46 | #include <sycl/ext/oneapi/experimental/bfloat16_math.hpp>
   47 | #endif
   48 | #include <sycl/ext/oneapi/experimental/complex/complex.hpp>
   49 | #include <cute/util/compat/traits.hpp>
```
**EN:** Sets up the header dependencies for this file by importing `sycl/ext/oneapi/experimental/bfloat16_math.hpp`, `sycl/ext/oneapi/experimental/complex/complex.hpp`, `cute/util/compat/traits.hpp`.
**CN:** 通过引入 `sycl/ext/oneapi/experimental/bfloat16_math.hpp`, `sycl/ext/oneapi/experimental/complex/complex.hpp`, `cute/util/compat/traits.hpp` 为该文件建立头文件依赖。

### Lines 51-52

```text
   51 | namespace compat {
   52 | namespace detail {
```
**EN:** Opens or closes namespace scope so the following declarations remain grouped under the intended CuTe or backend namespace hierarchy.
**CN:** 打开或关闭命名空间作用域，使后续声明保持在预期的 CuTe 或后端命名空间层级中。

### Lines 54-54

```text
   54 | namespace complex_namespace = sycl::ext::oneapi::experimental;
```
**EN:** Opens or closes namespace scope so the following declarations remain grouped under the intended CuTe or backend namespace hierarchy.
**CN:** 打开或关闭命名空间作用域，使后续声明保持在预期的 CuTe 或后端命名空间层级中。

### Lines 56-57

```text
   56 | template <typename ValueT>
   57 | using complex_type = detail::complex_namespace::complex<ValueT>;
```
**EN:** Introduces `complex_type` as a shorthand alias so later template-heavy code can refer to this type or mapping more concisely.
**CN:** 引入 `complex_type` 这一简写别名，使后续模板密集代码能够更简洁地引用该类型或映射。

### Lines 59-61

```text
   59 | template <typename T>
   60 | constexpr bool is_int32_type = std::is_same_v<std::decay_t<T>, int32_t> ||
   61 |   std::is_same_v<std::decay_t<T>, uint32_t>;
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还大量依赖编译期特化。

### Lines 63-68

```text
   63 | // Helper constexpr bool to avoid ugly macros where possible
   64 | #ifdef SYCL_EXT_ONEAPI_BFLOAT16_MATH_FUNCTIONS
   65 | constexpr bool support_bfloat16_math = true;
   66 | #else
   67 | constexpr bool support_bfloat16_math = false;
   68 | #endif
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 70-87

```text
   70 | template <typename ValueT>
   71 | inline ValueT clamp(ValueT val, ValueT min_val, ValueT max_val) {
   72 |   return sycl::clamp(val, min_val, max_val);
   73 | }
   74 | #ifdef SYCL_EXT_ONEAPI_BFLOAT16_MATH_FUNCTIONS
   75 | // TODO(compat-lib-reviewers): Follow the process to add this (& other math
   76 | // fns) to the bfloat16 math function extension. If added, remove this
   77 | // functionality from the header.
   78 | template <>
   79 | inline sycl::ext::oneapi::bfloat16 clamp(sycl::ext::oneapi::bfloat16 val,
   80 |                                          sycl::ext::oneapi::bfloat16 min_val,
   81 |                                          sycl::ext::oneapi::bfloat16 max_val) {
   82 |   if (val < min_val)
   83 |     return min_val;
   84 |   if (val > max_val)
   85 |     return max_val;
   86 |   return val;
   87 | }
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 89-98

```text
   89 | template <typename T, int Size>
   90 | inline std::enable_if_t<std::is_same_v<T, sycl::ext::oneapi::bfloat16>,
   91 |                         sycl::vec<T, Size>>
   92 | clamp(sycl::vec<T, Size> val, sycl::vec<T, Size> min_val,
   93 |       sycl::vec<T, Size> max_val) {
   94 |   return [&val, &min_val, &max_val]<int... I>(std::integer_sequence<int, I...>) {
   95 |     return sycl::vec<T, Size>{
   96 |         clamp<sycl::ext::oneapi::bfloat16>(val[I], min_val[I], max_val[I])...};
   97 |   }(std::make_integer_sequence<int, Size>{});
   98 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 100-110

```text
  100 | template <typename T, std::size_t Size>
  101 | inline std::enable_if_t<std::is_same_v<T, sycl::ext::oneapi::bfloat16>,
  102 |                         sycl::marray<T, Size>>
  103 | clamp(sycl::marray<T, Size> val, sycl::marray<T, Size> min_val,
  104 |       sycl::marray<T, Size> max_val) {
  105 |   return [&val, &min_val, &max_val]<std::size_t... I>(std::index_sequence<I...>) {
  106 |     return sycl::marray<T, Size>{
  107 |         clamp<sycl::ext::oneapi::bfloat16>(val[I], min_val[I], max_val[I])...};
  108 |   }(std::make_index_sequence<Size>{});
  109 | }
  110 | #endif
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 112-122

```text
  112 | template <typename VecT, class BinaryOperation, class = void>
  113 | class vectorized_binary {
  114 | public:
  115 |   inline VecT operator()(VecT a, VecT b, const BinaryOperation binary_op) {
  116 |     VecT v4;
  117 |     for (size_t i = 0; i < v4.size(); ++i) {
  118 |       v4[i] = binary_op(a[i], b[i]);
  119 |     }
  120 |     return v4;
  121 |   }
  122 | };
```
**EN:** Defines `BinaryOperation` as a reusable type-level building block in this header. It also implements coordinate mapping or slicing behavior and acts as a factory/helper for constructing derived objects.
**CN:** 将 `BinaryOperation` 定义为本头文件中的可复用类型级构件。 它还实现坐标映射或切片行为并充当构造派生对象的工厂/辅助函数。

### Lines 124-132

```text
  124 | template <typename VecT, class BinaryOperation>
  125 | class vectorized_binary<
  126 |     VecT, BinaryOperation,
  127 |     std::void_t<std::invoke_result_t<BinaryOperation, VecT, VecT>>> {
  128 | public:
  129 |   inline VecT operator()(VecT a, VecT b, const BinaryOperation binary_op) {
  130 |     return binary_op(a, b).template as<VecT>();
  131 |   }
  132 | };
```
**EN:** Defines `BinaryOperation` as a reusable type-level building block in this header. It also implements coordinate mapping or slicing behavior and acts as a factory/helper for constructing derived objects.
**CN:** 将 `BinaryOperation` 定义为本头文件中的可复用类型级构件。 它还实现坐标映射或切片行为并充当构造派生对象的工厂/辅助函数。

### Lines 134-165

```text
  134 | /// Extend the 'val' to 'bit' size, zero extend for unsigned int and signed
  135 | /// extend for signed int. Returns a signed integer type.
  136 | template <typename ValueT>
  137 | inline auto zero_or_signed_extend(ValueT val, unsigned bit) {
  138 |   static_assert(std::is_integral_v<ValueT>);
  139 |   if constexpr (sizeof(ValueT) == 4) {
  140 |     assert(bit < 64 &&
  141 |            "When extending int32 value, bit must be smaller than 64.");
  142 |     if constexpr (std::is_signed_v<ValueT>)
  143 |       return int64_t(val) << (64 - bit) >> (64 - bit);
  144 |     else
  145 |       return int64_t(val);
  146 |   } else if constexpr (sizeof(ValueT) == 2) {
  147 |     assert(bit < 32 &&
  148 |            "When extending int16 value, bit must be smaller than 32.");
  149 |     if constexpr (std::is_signed_v<ValueT>)
  150 |       return int32_t(val) << (32 - bit) >> (32 - bit);
  151 |     else
  152 |       return int32_t(val);
  153 |   } else if constexpr (sizeof(ValueT) == 1) {
  154 |     assert(bit < 16 &&
  155 |            "When extending int8 value, bit must be smaller than 16.");
  156 |     if constexpr (std::is_signed_v<ValueT>)
  157 |       return int16_t(val) << (16 - bit) >> (16 - bit);
  158 |     else
  159 |       return int16_t(val);
  160 |   } else {
  161 |     static_assert(sizeof(ValueT) == 8);
  162 |     assert(bit < 64 && "Cannot extend int64 value.");
  163 |     return static_cast<int64_t>(val);
  164 |   }
  165 | }
```
**EN:** Implements `zero_or_signed_extend`, a helper routine used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 实现 `zero_or_signed_extend`，这是周围 CuTe 抽象所使用的辅助例程。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 167-177

```text
  167 | template <typename RetT, bool needSat, typename AT, typename BT,
  168 |           typename BinaryOperation>
  169 | inline constexpr RetT extend_binary(AT a, BT b, BinaryOperation binary_op) {
  170 |   const int64_t extend_a = zero_or_signed_extend(a, 33);
  171 |   const int64_t extend_b = zero_or_signed_extend(b, 33);
  172 |   const int64_t ret = binary_op(extend_a, extend_b);
  173 |   if constexpr (needSat)
  174 |     return detail::clamp<int64_t>(ret, std::numeric_limits<RetT>::min(),
  175 |                                   std::numeric_limits<RetT>::max());
  176 |   return ret;
  177 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 179-194

```text
  179 | template <typename RetT, bool needSat, typename AT, typename BT, typename CT,
  180 |           typename BinaryOperation1, typename BinaryOperation2>
  181 | inline constexpr RetT extend_binary(AT a, BT b, CT c,
  182 |                                     BinaryOperation1 binary_op,
  183 |                                     BinaryOperation2 second_op) {
  184 |   const int64_t extend_a = zero_or_signed_extend(a, 33);
  185 |   const int64_t extend_b = zero_or_signed_extend(b, 33);
  186 |   int64_t extend_temp =
  187 |       zero_or_signed_extend(binary_op(extend_a, extend_b), 34);
  188 |   if constexpr (needSat)
  189 |     extend_temp =
  190 |         detail::clamp<int64_t>(extend_temp, std::numeric_limits<RetT>::min(),
  191 |                                std::numeric_limits<RetT>::max());
  192 |   const int64_t extend_c = zero_or_signed_extend(c, 33);
  193 |   return second_op(extend_temp, extend_c);
  194 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 196-204

```text
  196 | template <typename T> sycl::vec<int32_t, 2> extract_and_extend2(T a) {
  197 |   sycl::vec<int32_t, 2> ret;
  198 |   sycl::vec<T, 1> va{a};
  199 |   using IntT = std::conditional_t<std::is_signed_v<T>, int16_t, uint16_t>;
  200 |   auto v = va.template as<sycl::vec<IntT, 2>>();
  201 |   ret[0] = zero_or_signed_extend(v[0], 17);
  202 |   ret[1] = zero_or_signed_extend(v[1], 17);
  203 |   return ret;
  204 | }
```
**EN:** Implements `IntT`, a helper routine used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 实现 `IntT`，这是周围 CuTe 抽象所使用的辅助例程。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 206-216

```text
  206 | template <typename T> sycl::vec<int16_t, 4> extract_and_extend4(T a) {
  207 |   sycl::vec<int16_t, 4> ret;
  208 |   sycl::vec<T, 1> va{a};
  209 |   using IntT = std::conditional_t<std::is_signed_v<T>, int8_t, uint8_t>;
  210 |   auto v = va.template as<sycl::vec<IntT, 4>>();
  211 |   ret[0] = zero_or_signed_extend(v[0], 9);
  212 |   ret[1] = zero_or_signed_extend(v[1], 9);
  213 |   ret[2] = zero_or_signed_extend(v[2], 9);
  214 |   ret[3] = zero_or_signed_extend(v[3], 9);
  215 |   return ret;
  216 | }
```
**EN:** Implements `IntT`, a helper routine used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 实现 `IntT`，这是周围 CuTe 抽象所使用的辅助例程。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 218-227

```text
  218 | template <typename RetT, bool NeedSat, bool NeedAdd, typename AT, typename BT,
  219 |           typename BinaryOperation>
  220 | inline constexpr RetT extend_vbinary2(AT a, BT b, RetT c,
  221 |                                       BinaryOperation binary_op) {
  222 |   static_assert(is_int32_type<AT> && is_int32_type<BT> && is_int32_type<RetT>);
  223 |   sycl::vec<int32_t, 2> extend_a = extract_and_extend2(a);
  224 |   sycl::vec<int32_t, 2> extend_b = extract_and_extend2(b);
  225 |   sycl::vec<int32_t, 2> temp{binary_op(extend_a[0], extend_b[0]),
  226 |                              binary_op(extend_a[1], extend_b[1])};
  227 |   using IntT = std::conditional_t<std::is_signed_v<RetT>, int16_t, uint16_t>;
```
**EN:** Implements `IntT`, a helper routine used by the surrounding CuTe abstractions. It also leans heavily on compile-time specialization and bridges to SYCL execution or group abstractions.
**CN:** 实现 `IntT`，这是周围 CuTe 抽象所使用的辅助例程。 它还大量依赖编译期特化并桥接到 SYCL 执行模型或 group 抽象。

### Lines 229-240

```text
  229 |   if constexpr (NeedSat) {
  230 |     int32_t min_val = 0, max_val = 0;
  231 |     min_val = std::numeric_limits<IntT>::min();
  232 |     max_val = std::numeric_limits<IntT>::max();
  233 |     temp = detail::clamp(temp, sycl::vec<int32_t, 2>(min_val),
  234 |                          sycl::vec<int32_t, 2>(max_val));
  235 |   }
  236 |   if constexpr (NeedAdd) {
  237 |     return temp[0] + temp[1] + c;
  238 |   }
  239 |   return sycl::vec<IntT, 2>{temp[0], temp[1]}.template as<sycl::vec<RetT, 1>>();
  240 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 242-252

```text
  242 | template <typename RetT, bool NeedSat, bool NeedAdd, typename AT, typename BT,
  243 |           typename BinaryOperation>
  244 | inline constexpr RetT extend_vbinary4(AT a, BT b, RetT c,
  245 |                                       BinaryOperation binary_op) {
  246 |   static_assert(is_int32_type<AT> && is_int32_type<BT> && is_int32_type<RetT>);
  247 |   sycl::vec<int16_t, 4> extend_a = extract_and_extend4(a);
  248 |   sycl::vec<int16_t, 4> extend_b = extract_and_extend4(b);
  249 |   sycl::vec<int16_t, 4> temp{
  250 |       binary_op(extend_a[0], extend_b[0]), binary_op(extend_a[1], extend_b[1]),
  251 |       binary_op(extend_a[2], extend_b[2]), binary_op(extend_a[3], extend_b[3])};
  252 |   using IntT = std::conditional_t<std::is_signed_v<RetT>, int8_t, uint8_t>;
```
**EN:** Implements `IntT`, a helper routine used by the surrounding CuTe abstractions. It also leans heavily on compile-time specialization and bridges to SYCL execution or group abstractions.
**CN:** 实现 `IntT`，这是周围 CuTe 抽象所使用的辅助例程。 它还大量依赖编译期特化并桥接到 SYCL 执行模型或 group 抽象。

### Lines 254-263

```text
  254 |   if constexpr (NeedSat) {
  255 |     int16_t min_val = 0, max_val = 0;
  256 |     min_val = std::numeric_limits<IntT>::min();
  257 |     max_val = std::numeric_limits<IntT>::max();
  258 |     temp = detail::clamp(temp, sycl::vec<int16_t, 4>(min_val),
  259 |                          sycl::vec<int16_t, 4>(max_val));
  260 |   }
  261 |   if constexpr (NeedAdd) {
  262 |     return temp[0] + temp[1] + temp[2] + temp[3] + c;
  263 |   }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 265-267

```text
  265 |   return sycl::vec<IntT, 4>{temp[0], temp[1], temp[2], temp[3]}
  266 |       .template as<sycl::vec<RetT, 1>>();
  267 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and bridges to SYCL execution or group abstractions.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并桥接到 SYCL 执行模型或 group 抽象。

### Lines 269-276

```text
  269 | template <typename ValueT> inline bool isnan(const ValueT a) {
  270 |   if constexpr (std::is_same_v<ValueT, sycl::ext::oneapi::bfloat16>) {
  271 |     static_assert(detail::support_bfloat16_math);
  272 |     return sycl::ext::oneapi::experimental::isnan(a);
  273 |   } else {
  274 |     return sycl::isnan(a);
  275 |   }
  276 | }
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 278-299

```text
  278 | // FIXME(compat-lib-reviewers): move bfe outside detail once perf is
  279 | // improved & semantics understood
  280 | /// Bitfield-extract.
  281 | ///
  282 | /// \tparam T The type of \param source value, must be an integer.
  283 | /// \param source The source value to extracting.
  284 | /// \param bit_start The position to start extracting.
  285 | /// \param num_bits The number of bits to extracting.
  286 | template <typename T>
  287 | inline T bfe(const T source, const uint32_t bit_start,
  288 |              const uint32_t num_bits) {
  289 |   static_assert(std::is_unsigned_v<T>);
  290 |   // FIXME(compat-lib-reviewers): This ternary was added to catch a case
  291 |   // which may be undefined anyway. Consider that we are losing perf here.
  292 |   const T mask =
  293 |       num_bits >= std::numeric_limits<unsigned char>::digits * sizeof(T)
  294 |           ? static_cast<T>(-1)
  295 |           : ((static_cast<T>(1) << num_bits) - 1);
  296 |   return (source >> bit_start) & mask;
  297 | }
  299 | } // namespace detail
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 301-369

```text
  301 | /// Bitfield-extract with boundary checking.
  302 | ///
  303 | /// Extract bit field from \param source and return the zero or sign-extended
  304 | /// result. Source \param bit_start gives the bit field starting bit position,
  305 | /// and source \param num_bits gives the bit field length in bits.
  306 | ///
  307 | /// The result is padded with the sign bit of the extracted field. If `num_bits`
  308 | /// is zero, the result is zero. If the start position is beyond the msb of the
  309 | /// input, the result is filled with the replicated sign bit of the extracted
  310 | /// field.
  311 | ///
  312 | /// \tparam T The type of \param source value, must be an integer.
  313 | /// \param source The source value to extracting.
  314 | /// \param bit_start The position to start extracting.
  315 | /// \param num_bits The number of bits to extracting.
  316 | template <typename T>
  317 | inline T bfe_safe(const T source, const uint32_t bit_start,
  318 |                   const uint32_t num_bits) {
  319 |   static_assert(std::is_integral_v<T>);
  320 | #if defined(__SYCL_DEVICE_ONLY__) && defined(__NVPTX__)
  321 |   if constexpr (std::is_same_v<T, int8_t> || std::is_same_v<T, int16_t> ||
  322 |                 std::is_same_v<T, int32_t>) {
  323 |     int32_t res{};
  324 |     asm volatile("bfe.s32 %0, %1, %2, %3;"
  325 |                  : "=r"(res)
  326 |                  : "r"((int32_t)source), "r"(bit_start), "r"(num_bits));
  327 |     return res;
  328 |   } else if constexpr (std::is_same_v<T, uint8_t> ||
  329 |                        std::is_same_v<T, uint16_t> ||
  330 |                        std::is_same_v<T, uint32_t>) {
  331 |     uint32_t res{};
  332 |     asm volatile("bfe.u32 %0, %1, %2, %3;"
  333 |                  : "=r"(res)
  334 |                  : "r"((uint32_t)source), "r"(bit_start), "r"(num_bits));
  335 |     return res;
  336 |   } else if constexpr (std::is_same_v<T, int64_t>) {
  337 |     T res{};
  338 |     asm volatile("bfe.s64 %0, %1, %2, %3;"
  339 |                  : "=l"(res)
  340 |                  : "l"(source), "r"(bit_start), "r"(num_bits));
  341 |     return res;
  342 |   } else if constexpr (std::is_same_v<T, uint64_t>) {
  343 |     T res{};
  344 |     asm volatile("bfe.u64 %0, %1, %2, %3;"
  345 |                  : "=l"(res)
  346 |                  : "l"(source), "r"(bit_start), "r"(num_bits));
  347 |     return res;
  348 |   }
  349 | #endif
  350 |   const uint32_t bit_width =
  351 |       std::numeric_limits<unsigned char>::digits * sizeof(T);
  352 |   const uint32_t pos = std::min(bit_start, bit_width);
  353 |   const uint32_t len = std::min(pos + num_bits, bit_width) - pos;
  354 |   if constexpr (std::is_signed_v<T>) {
  355 |     // FIXME(compat-lib-reviewers): As above, catching a case whose result
  356 |     // is undefined and likely losing perf.
  357 |     const T mask = len >= bit_width ? T{-1} : static_cast<T>((T{1} << len) - 1);
  358 | 
  359 |     // Find the sign-bit, the result is padded with the sign bit of the
  360 |     // extracted field.
  361 |     // Note if requested num_bits==0, we return zero via sign_bit=0
  362 |     const uint32_t sign_bit_pos = std::min(pos + len - 1, bit_width - 1);
  363 |     const T sign_bit = num_bits != 0 && ((source >> sign_bit_pos) & 1);
  364 |     const T sign_bit_padding = (-sign_bit & ~mask);
  365 |     return ((source >> pos) & mask) | sign_bit_padding;
  366 |   } else {
  367 |     return compat::detail::bfe(source, pos, len);
  368 |   }
  369 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 371-386

```text
  371 | namespace detail {
  372 | // FIXME(compat-lib-reviewers): move bfi outside detail once perf is
  373 | // improved & semantics understood
  374 | /// Bitfield-insert.
  375 | ///
  376 | /// \tparam T The type of \param x and \param y , must be an unsigned integer.
  377 | /// \param x The source of the bitfield.
  378 | /// \param y The source where bitfield is inserted.
  379 | /// \param bit_start The position to start insertion.
  380 | /// \param num_bits The number of bits to insertion.
  381 | template <typename T>
  382 | inline T bfi(const T x, const T y, const uint32_t bit_start,
  383 |              const uint32_t num_bits) {
  384 |   static_assert(std::is_unsigned_v<T>);
  385 |   constexpr unsigned bit_width =
  386 |       std::numeric_limits<unsigned char>::digits * sizeof(T);
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还大量依赖编译期特化。

### Lines 388-396

```text
  388 |   // if bit_start > bit_width || len == 0, should return y.
  389 |   const T ignore_bfi = static_cast<T>(bit_start > bit_width || num_bits == 0);
  390 |   T extract_bitfield_mask = (static_cast<T>(~T{0}) >> (bit_width - num_bits))
  391 |                             << bit_start;
  392 |   T clean_bitfield_mask = ~extract_bitfield_mask;
  393 |   return (y & (-ignore_bfi | clean_bitfield_mask)) |
  394 |          (~-ignore_bfi & ((x << bit_start) & extract_bitfield_mask));
  395 | }
  396 | } // namespace detail
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数。

### Lines 398-435

```text
  398 | /// Bitfield-insert with boundary checking.
  399 | ///
  400 | /// Align and insert a bit field from \param x into \param y . Source \param
  401 | /// bit_start gives the starting bit position for the insertion, and source
  402 | /// \param num_bits gives the bit field length in bits.
  403 | ///
  404 | /// \tparam T The type of \param x and \param y , must be an unsigned integer.
  405 | /// \param x The source of the bitfield.
  406 | /// \param y The source where bitfield is inserted.
  407 | /// \param bit_start The position to start insertion.
  408 | /// \param num_bits The number of bits to insertion.
  409 | template <typename T>
  410 | inline T bfi_safe(const T x, const T y, const uint32_t bit_start,
  411 |                   const uint32_t num_bits) {
  412 |   static_assert(std::is_unsigned_v<T>);
  413 | #if defined(__SYCL_DEVICE_ONLY__) && defined(__NVPTX__)
  414 |   if constexpr (std::is_same_v<T, uint8_t> || std::is_same_v<T, uint16_t> ||
  415 |                 std::is_same_v<T, uint32_t>) {
  416 |     uint32_t res{};
  417 |     asm volatile("bfi.b32 %0, %1, %2, %3, %4;"
  418 |                  : "=r"(res)
  419 |                  : "r"((uint32_t)x), "r"((uint32_t)y), "r"(bit_start),
  420 |                    "r"(num_bits));
  421 |     return res;
  422 |   } else if constexpr (std::is_same_v<T, uint64_t>) {
  423 |     uint64_t res{};
  424 |     asm volatile("bfi.b64 %0, %1, %2, %3, %4;"
  425 |                  : "=l"(res)
  426 |                  : "l"(x), "l"(y), "r"(bit_start), "r"(num_bits));
  427 |     return res;
  428 |   }
  429 | #endif
  430 |   constexpr unsigned bit_width =
  431 |       std::numeric_limits<unsigned char>::digits * sizeof(T);
  432 |   const uint32_t pos = std::min(bit_start, bit_width);
  433 |   const uint32_t len = std::min(pos + num_bits, bit_width) - pos;
  434 |   return compat::detail::bfi(x, y, pos, len);
  435 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 437-441

```text
  437 | /// Emulated function for __funnelshift_l
  438 | inline unsigned int funnelshift_l(unsigned int low, unsigned int high,
  439 |                                   unsigned int shift) {
  440 |   return (sycl::upsample(high, low) << (shift & 31U)) >> 32;
  441 | }
```
**EN:** Implements `funnelshift_l`, a helper routine used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and bridges to SYCL execution or group abstractions.
**CN:** 实现 `funnelshift_l`，这是周围 CuTe 抽象所使用的辅助例程。 它还充当构造派生对象的工厂/辅助函数并桥接到 SYCL 执行模型或 group 抽象。

### Lines 443-447

```text
  443 | /// Emulated function for __funnelshift_lc
  444 | inline unsigned int funnelshift_lc(unsigned int low, unsigned int high,
  445 |                                    unsigned int shift) {
  446 |   return (sycl::upsample(high, low) << sycl::min(shift, 32U)) >> 32;
  447 | }
```
**EN:** Implements `funnelshift_lc`, a helper routine used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and bridges to SYCL execution or group abstractions.
**CN:** 实现 `funnelshift_lc`，这是周围 CuTe 抽象所使用的辅助例程。 它还充当构造派生对象的工厂/辅助函数并桥接到 SYCL 执行模型或 group 抽象。

### Lines 449-453

```text
  449 | /// Emulated function for __funnelshift_r
  450 | inline unsigned int funnelshift_r(unsigned int low, unsigned int high,
  451 |                                   unsigned int shift) {
  452 |   return (sycl::upsample(high, low) >> (shift & 31U)) & 0xFFFFFFFF;
  453 | }
```
**EN:** Implements `funnelshift_r`, a helper routine used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and bridges to SYCL execution or group abstractions.
**CN:** 实现 `funnelshift_r`，这是周围 CuTe 抽象所使用的辅助例程。 它还充当构造派生对象的工厂/辅助函数并桥接到 SYCL 执行模型或 group 抽象。

### Lines 455-459

```text
  455 | /// Emulated function for __funnelshift_rc
  456 | inline unsigned int funnelshift_rc(unsigned int low, unsigned int high,
  457 |                                    unsigned int shift) {
  458 |   return (sycl::upsample(high, low) >> sycl::min(shift, 32U)) & 0xFFFFFFFF;
  459 | }
```
**EN:** Implements `funnelshift_rc`, a helper routine used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and bridges to SYCL execution or group abstractions.
**CN:** 实现 `funnelshift_rc`，这是周围 CuTe 抽象所使用的辅助例程。 它还充当构造派生对象的工厂/辅助函数并桥接到 SYCL 执行模型或 group 抽象。

### Lines 461-483

```text
  461 | /// Compute fast_length for variable-length array
  462 | /// \param [in] a The array
  463 | /// \param [in] len Length of the array
  464 | /// \returns The computed fast_length
  465 | inline float fast_length(const float *a, int len) {
  466 |   switch (len) {
  467 |   case 1:
  468 |     return sycl::fast_length(a[0]);
  469 |   case 2:
  470 |     return sycl::fast_length(sycl::float2(a[0], a[1]));
  471 |   case 3:
  472 |     return sycl::fast_length(sycl::float3(a[0], a[1], a[2]));
  473 |   case 4:
  474 |     return sycl::fast_length(sycl::float4(a[0], a[1], a[2], a[3]));
  475 |   case 0:
  476 |     return 0;
  477 |   default:
  478 |     float f = 0;
  479 |     for (int i = 0; i < len; ++i)
  480 |       f += a[i] * a[i];
  481 |     return sycl::sqrt(f);
  482 |   }
  483 | }
```
**EN:** Implements `fast_length`, a helper routine used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and bridges to SYCL execution or group abstractions.
**CN:** 实现 `fast_length`，这是周围 CuTe 抽象所使用的辅助例程。 它还充当构造派生对象的工厂/辅助函数并桥接到 SYCL 执行模型或 group 抽象。

### Lines 485-506

```text
  485 | /// Calculate the square root of the input array.
  486 | /// \param [in] a The array pointer
  487 | /// \param [in] len Length of the array
  488 | /// \returns The square root
  489 | template <typename ValueT>
  490 | inline ValueT length(const ValueT *a, const int len) {
  491 |   switch (len) {
  492 |   case 1:
  493 |     return a[0];
  494 |   case 2:
  495 |     return sycl::length(sycl::vec<ValueT, 2>(a[0], a[1]));
  496 |   case 3:
  497 |     return sycl::length(sycl::vec<ValueT, 3>(a[0], a[1], a[2]));
  498 |   case 4:
  499 |     return sycl::length(sycl::vec<ValueT, 4>(a[0], a[1], a[2], a[3]));
  500 |   default:
  501 |     ValueT ret = 0;
  502 |     for (int i = 0; i < len; ++i)
  503 |       ret += a[i] * a[i];
  504 |     return sycl::sqrt(ret);
  505 |   }
  506 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 508-527

```text
  508 | /// Performs comparison.
  509 | /// \param [in] a The first value
  510 | /// \param [in] b The second value
  511 | /// \param [in] binary_op functor that implements the binary operation
  512 | /// \returns the comparison result
  513 | template <typename ValueT, class BinaryOperation>
  514 | inline std::enable_if_t<
  515 |     std::is_same_v<std::invoke_result_t<BinaryOperation, ValueT, ValueT>, bool>,
  516 |     bool>
  517 | compare(const ValueT a, const ValueT b, const BinaryOperation binary_op) {
  518 |   return binary_op(a, b);
  519 | }
  520 | template <typename ValueT>
  521 | inline std::enable_if_t<
  522 |     std::is_same_v<std::invoke_result_t<std::not_equal_to<>, ValueT, ValueT>,
  523 |                    bool>,
  524 |     bool>
  525 | compare(const ValueT a, const ValueT b, const std::not_equal_to<> binary_op) {
  526 |   return !detail::isnan(a) && !detail::isnan(b) && binary_op(a, b);
  527 | }
```
**EN:** Defines `BinaryOperation` as a reusable type-level building block in this header. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 将 `BinaryOperation` 定义为本头文件中的可复用类型级构件。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 529-538

```text
  529 | /// Performs 2 element comparison.
  530 | /// \param [in] a The first value
  531 | /// \param [in] b The second value
  532 | /// \param [in] binary_op functor that implements the binary operation
  533 | /// \returns the comparison result
  534 | template <typename ValueT, class BinaryOperation>
  535 | inline std::enable_if_t<ValueT::size() == 2, ValueT>
  536 | compare(const ValueT a, const ValueT b, const BinaryOperation binary_op) {
  537 |   return {compare(a[0], b[0], binary_op), compare(a[1], b[1], binary_op)};
  538 | }
```
**EN:** Defines `BinaryOperation` as a reusable type-level building block in this header. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 将 `BinaryOperation` 定义为本头文件中的可复用类型级构件。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 540-552

```text
  540 | /// Performs unordered comparison.
  541 | /// \param [in] a The first value
  542 | /// \param [in] b The second value
  543 | /// \param [in] binary_op functor that implements the binary operation
  544 | /// \returns the comparison result
  545 | template <typename ValueT, class BinaryOperation>
  546 | inline std::enable_if_t<
  547 |     std::is_same_v<std::invoke_result_t<BinaryOperation, ValueT, ValueT>, bool>,
  548 |     bool>
  549 | unordered_compare(const ValueT a, const ValueT b,
  550 |                   const BinaryOperation binary_op) {
  551 |   return detail::isnan(a) || detail::isnan(b) || binary_op(a, b);
  552 | }
```
**EN:** Defines `BinaryOperation` as a reusable type-level building block in this header. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 将 `BinaryOperation` 定义为本头文件中的可复用类型级构件。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 554-565

```text
  554 | /// Performs 2 element unordered comparison.
  555 | /// \param [in] a The first value
  556 | /// \param [in] b The second value
  557 | /// \param [in] binary_op functor that implements the binary operation
  558 | /// \returns the comparison result
  559 | template <typename ValueT, class BinaryOperation>
  560 | inline std::enable_if_t<ValueT::size() == 2, ValueT>
  561 | unordered_compare(const ValueT a, const ValueT b,
  562 |                   const BinaryOperation binary_op) {
  563 |   return {unordered_compare(a[0], b[0], binary_op),
  564 |           unordered_compare(a[1], b[1], binary_op)};
  565 | }
```
**EN:** Defines `BinaryOperation` as a reusable type-level building block in this header. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 将 `BinaryOperation` 定义为本头文件中的可复用类型级构件。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 567-576

```text
  567 | /// Performs 2 element comparison and return true if both results are true.
  568 | /// \param [in] a The first value
  569 | /// \param [in] b The second value
  570 | /// \param [in] binary_op functor that implements the binary operation
  571 | /// \returns the comparison result
  572 | template <typename ValueT, class BinaryOperation>
  573 | inline std::enable_if_t<ValueT::size() == 2, bool>
  574 | compare_both(const ValueT a, const ValueT b, const BinaryOperation binary_op) {
  575 |   return compare(a[0], b[0], binary_op) && compare(a[1], b[1], binary_op);
  576 | }
```
**EN:** Defines `BinaryOperation` as a reusable type-level building block in this header. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 将 `BinaryOperation` 定义为本头文件中的可复用类型级构件。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 578-590

```text
  578 | /// Performs 2 element unordered comparison and return true if both results are
  579 | /// true.
  580 | /// \param [in] a The first value
  581 | /// \param [in] b The second value
  582 | /// \param [in] binary_op functor that implements the binary operation
  583 | /// \returns the comparison result
  584 | template <typename ValueT, class BinaryOperation>
  585 | inline std::enable_if_t<ValueT::size() == 2, bool>
  586 | unordered_compare_both(const ValueT a, const ValueT b,
  587 |                        const BinaryOperation binary_op) {
  588 |   return unordered_compare(a[0], b[0], binary_op) &&
  589 |          unordered_compare(a[1], b[1], binary_op);
  590 | }
```
**EN:** Defines `BinaryOperation` as a reusable type-level building block in this header. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 将 `BinaryOperation` 定义为本头文件中的可复用类型级构件。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 592-605

```text
  592 | /// Performs 2 elements comparison, compare result of each element is 0 (false)
  593 | /// or 0xffff (true), returns an unsigned int by composing compare result of two
  594 | /// elements.
  595 | /// \param [in] a The first value
  596 | /// \param [in] b The second value
  597 | /// \param [in] binary_op functor that implements the binary operation
  598 | /// \returns the comparison result
  599 | template <typename ValueT, class BinaryOperation>
  600 | inline std::enable_if_t<ValueT::size() == 2, unsigned>
  601 | compare_mask(const ValueT a, const ValueT b, const BinaryOperation binary_op) {
  602 |   // Since compare returns 0 or 1, -compare will be 0x00000000 or 0xFFFFFFFF
  603 |   return ((-compare(a[0], b[0], binary_op)) & 0xFFFF) |
  604 |          ((-compare(a[1], b[1], binary_op)) << 16u);
  605 | }
```
**EN:** Defines `BinaryOperation` as a reusable type-level building block in this header. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 将 `BinaryOperation` 定义为本头文件中的可复用类型级构件。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 607-620

```text
  607 | /// Performs 2 elements unordered comparison, compare result of each element is
  608 | /// 0 (false) or 0xffff (true), returns an unsigned int by composing compare
  609 | /// result of two elements.
  610 | /// \param [in] a The first value
  611 | /// \param [in] b The second value
  612 | /// \param [in] binary_op functor that implements the binary operation
  613 | /// \returns the comparison result
  614 | template <typename ValueT, class BinaryOperation>
  615 | inline std::enable_if_t<ValueT::size() == 2, unsigned>
  616 | unordered_compare_mask(const ValueT a, const ValueT b,
  617 |                        const BinaryOperation binary_op) {
  618 |   return ((-unordered_compare(a[0], b[0], binary_op)) & 0xFFFF) |
  619 |          ((-unordered_compare(a[1], b[1], binary_op)) << 16);
  620 | }
```
**EN:** Defines `BinaryOperation` as a reusable type-level building block in this header. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 将 `BinaryOperation` 定义为本头文件中的可复用类型级构件。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 622-636

```text
  622 | /// Compute vectorized max for two values, with each value treated as a vector
  623 | /// type \p S
  624 | /// \param [in] S The type of the vector
  625 | /// \param [in] T The type of the original values
  626 | /// \param [in] a The first value
  627 | /// \param [in] b The second value
  628 | /// \returns The vectorized max of the two values
  629 | template <typename S, typename T> inline T vectorized_max(T a, T b) {
  630 |   sycl::vec<T, 1> v0{a}, v1{b};
  631 |   auto v2 = v0.template as<S>();
  632 |   auto v3 = v1.template as<S>();
  633 |   v2 = sycl::max(v2, v3);
  634 |   v0 = v2.template as<sycl::vec<T, 1>>();
  635 |   return v0;
  636 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 638-652

```text
  638 | /// Compute vectorized min for two values, with each value treated as a vector
  639 | /// type \p S
  640 | /// \param [in] S The type of the vector
  641 | /// \param [in] T The type of the original values
  642 | /// \param [in] a The first value
  643 | /// \param [in] b The second value
  644 | /// \returns The vectorized min of the two values
  645 | template <typename S, typename T> inline T vectorized_min(T a, T b) {
  646 |   sycl::vec<T, 1> v0{a}, v1{b};
  647 |   auto v2 = v0.template as<S>();
  648 |   auto v3 = v1.template as<S>();
  649 |   v2 = sycl::min(v2, v3);
  650 |   v0 = v2.template as<sycl::vec<T, 1>>();
  651 |   return v0;
  652 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 654-667

```text
  654 | /// Compute vectorized unary operation for a value, with the value treated as a
  655 | /// vector type \p VecT.
  656 | /// \tparam [in] VecT The type of the vector
  657 | /// \tparam [in] UnaryOperation The unary operation class
  658 | /// \param [in] a The input value
  659 | /// \returns The vectorized unary operation value of the input value
  660 | template <typename VecT, class UnaryOperation>
  661 | inline unsigned vectorized_unary(unsigned a, const UnaryOperation unary_op) {
  662 |   sycl::vec<unsigned, 1> v0{a};
  663 |   auto v1 = v0.as<VecT>();
  664 |   auto v2 = unary_op(v1);
  665 |   v0 = v2.template as<sycl::vec<unsigned, 1>>();
  666 |   return v0;
  667 | }
```
**EN:** Defines `UnaryOperation` as a reusable type-level building block in this header. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 将 `UnaryOperation` 定义为本头文件中的可复用类型级构件。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 669-687

```text
  669 | /// Compute vectorized absolute difference for two values without modulo
  670 | /// overflow, with each value treated as a vector type \p VecT.
  671 | /// \tparam [in] VecT The type of the vector
  672 | /// \param [in] a The first value
  673 | /// \param [in] b The second value
  674 | /// \returns The vectorized absolute difference of the two values
  675 | template <typename VecT>
  676 | inline unsigned vectorized_sum_abs_diff(unsigned a, unsigned b) {
  677 |   sycl::vec<unsigned, 1> v0{a}, v1{b};
  678 |   // Need convert element type to wider signed type to avoid overflow.
  679 |   auto v2 = v0.as<VecT>().template convert<int>();
  680 |   auto v3 = v1.as<VecT>().template convert<int>();
  681 |   auto v4 = sycl::abs_diff(v2, v3);
  682 |   unsigned sum = 0;
  683 |   for (size_t i = 0; i < v4.size(); ++i) {
  684 |     sum += v4[i];
  685 |   }
  686 |   return sum;
  687 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 689-703

```text
  689 | /// Compute vectorized isgreater for two values, with each value treated as a
  690 | /// vector type \p S
  691 | /// \param [in] S The type of the vector
  692 | /// \param [in] T The type of the original values
  693 | /// \param [in] a The first value
  694 | /// \param [in] b The second value
  695 | /// \returns The vectorized greater than of the two values
  696 | template <typename S, typename T> inline T vectorized_isgreater(T a, T b) {
  697 |   sycl::vec<T, 1> v0{a}, v1{b};
  698 |   auto v2 = v0.template as<S>();
  699 |   auto v3 = v1.template as<S>();
  700 |   auto v4 = sycl::isgreater(v2, v3);
  701 |   v0 = v4.template as<sycl::vec<T, 1>>();
  702 |   return v0;
  703 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 705-721

```text
  705 | /// Compute vectorized isgreater for two unsigned int values, with each value
  706 | /// treated as a vector of two unsigned short
  707 | /// \param [in] a The first value
  708 | /// \param [in] b The second value
  709 | /// \returns The vectorized greater than of the two values
  710 | template <>
  711 | inline unsigned vectorized_isgreater<sycl::ushort2, unsigned>(unsigned a,
  712 |                                                               unsigned b) {
  713 |   sycl::vec<unsigned, 1> v0{a}, v1{b};
  714 |   auto v2 = v0.template as<sycl::ushort2>();
  715 |   auto v3 = v1.template as<sycl::ushort2>();
  716 |   sycl::ushort2 v4;
  717 |   v4[0] = v2[0] > v3[0];
  718 |   v4[1] = v2[1] > v3[1];
  719 |   v0 = v4.template as<sycl::vec<unsigned, 1>>();
  720 |   return v0;
  721 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 723-731

```text
  723 | /// Returns min(max(val, min_val), max_val)
  724 | /// \param [in] val The input value
  725 | /// \param [in] min_val The minimum value
  726 | /// \param [in] max_val The maximum value
  727 | /// \returns the value between min_val and max_val
  728 | template <typename ValueT>
  729 | inline ValueT clamp(ValueT val, ValueT min_val, ValueT max_val) {
  730 |   return detail::clamp(val, min_val, max_val);
  731 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 733-739

```text
  733 | /// Determine whether 2 element value is NaN.
  734 | /// \param [in] a The input value
  735 | /// \returns the comparison result
  736 | template <typename ValueT>
  737 | inline std::enable_if_t<ValueT::size() == 2, ValueT> isnan(const ValueT a) {
  738 |   return {detail::isnan(a[0]), detail::isnan(a[1])};
  739 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 741-748

```text
  741 | /// cbrt function wrapper.
  742 | template <typename ValueT>
  743 | inline std::enable_if_t<std::is_floating_point_v<ValueT> ||
  744 |                             std::is_same_v<ValueT, sycl::half>,
  745 |                         ValueT>
  746 | cbrt(ValueT val) {
  747 |   return sycl::cbrt(static_cast<ValueT>(val));
  748 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 750-762

```text
  750 | // min/max function overloads.
  751 | // For floating-point types, `float` or `double` arguments are acceptable.
  752 | // For integer types, `std::uint32_t`, `std::int32_t`, `std::uint64_t` or
  753 | // `std::int64_t` type arguments are acceptable.
  754 | // sycl::half supported as well, and sycl::ext::oneapi::bfloat16 if available.
  755 | template <typename ValueT, typename ValueU>
  756 | inline std::enable_if_t<std::is_integral_v<ValueT> &&
  757 |                             std::is_integral_v<ValueU>,
  758 |                         std::common_type_t<ValueT, ValueU>>
  759 | min(ValueT a, ValueU b) {
  760 |   return sycl::min(static_cast<std::common_type_t<ValueT, ValueU>>(a),
  761 |                    static_cast<std::common_type_t<ValueT, ValueU>>(b));
  762 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 764-779

```text
  764 | template <typename ValueT, typename ValueU>
  765 | inline std::enable_if_t<compat::is_floating_point_v<ValueT> &&
  766 |                             compat::is_floating_point_v<ValueU>,
  767 |                         std::common_type_t<ValueT, ValueU>>
  768 | min(ValueT a, ValueU b) {
  769 |   if constexpr (std::is_same_v<std::common_type_t<ValueT, ValueU>,
  770 |                                sycl::ext::oneapi::bfloat16>) {
  771 |     static_assert(detail::support_bfloat16_math);
  772 |     return sycl::ext::oneapi::experimental::fmin(
  773 |         static_cast<std::common_type_t<ValueT, ValueU>>(a),
  774 |         static_cast<std::common_type_t<ValueT, ValueU>>(b));
  775 |   } else {
  776 |     return sycl::fmin(static_cast<std::common_type_t<ValueT, ValueU>>(a),
  777 |                       static_cast<std::common_type_t<ValueT, ValueU>>(b));
  778 |   }
  779 | }
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 781-804

```text
  781 | template <typename ValueT, typename ValueU>
  782 | inline std::enable_if_t<std::is_integral_v<ValueT> &&
  783 |                             std::is_integral_v<ValueU>,
  784 |                         std::common_type_t<ValueT, ValueU>>
  785 | max(ValueT a, ValueU b) {
  786 |   return sycl::max(static_cast<std::common_type_t<ValueT, ValueU>>(a),
  787 |                    static_cast<std::common_type_t<ValueT, ValueU>>(b));
  788 | }
  789 | template <typename ValueT, typename ValueU>
  790 | inline std::enable_if_t<compat::is_floating_point_v<ValueT> &&
  791 |                             compat::is_floating_point_v<ValueU>,
  792 |                         std::common_type_t<ValueT, ValueU>>
  793 | max(ValueT a, ValueU b) {
  794 |   if constexpr (std::is_same_v<std::common_type_t<ValueT, ValueU>,
  795 |                                sycl::ext::oneapi::bfloat16>) {
  796 |     static_assert(detail::support_bfloat16_math);
  797 |     return sycl::ext::oneapi::experimental::fmax(
  798 |         static_cast<std::common_type_t<ValueT, ValueU>>(a),
  799 |         static_cast<std::common_type_t<ValueT, ValueU>>(b));
  800 |   } else {
  801 |     return sycl::fmax(static_cast<std::common_type_t<ValueT, ValueU>>(a),
  802 |                       static_cast<std::common_type_t<ValueT, ValueU>>(b));
  803 |   }
  804 | }
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 806-817

```text
  806 | /// Performs 2 elements comparison and returns the bigger one. If either of
  807 | /// inputs is NaN, then return NaN.
  808 | /// \param [in] a The first value
  809 | /// \param [in] b The second value
  810 | /// \returns the bigger value
  811 | template <typename ValueT, typename ValueU>
  812 | inline std::common_type_t<ValueT, ValueU> fmax_nan(const ValueT a,
  813 |                                                    const ValueU b) {
  814 |   if (detail::isnan(a) || detail::isnan(b))
  815 |     return NAN;
  816 |   return compat::max(a, b);
  817 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 819-823

```text
  819 | template <typename ValueT, typename ValueU>
  820 | inline sycl::vec<std::common_type_t<ValueT, ValueU>, 2>
  821 | fmax_nan(const sycl::vec<ValueT, 2> a, const sycl::vec<ValueU, 2> b) {
  822 |   return {fmax_nan(a[0], b[0]), fmax_nan(a[1], b[1])};
  823 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 825-829

```text
  825 | template <typename ValueT, typename ValueU>
  826 | inline sycl::marray<std::common_type_t<ValueT, ValueU>, 2>
  827 | fmax_nan(const sycl::marray<ValueT, 2> a, const sycl::marray<ValueU, 2> b) {
  828 |   return {fmax_nan(a[0], b[0]), fmax_nan(a[1], b[1])};
  829 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 831-842

```text
  831 | /// Performs 2 elements comparison and returns the smaller one. If either of
  832 | /// inputs is NaN, then return NaN.
  833 | /// \param [in] a The first value
  834 | /// \param [in] b The second value
  835 | /// \returns the smaller value
  836 | template <typename ValueT, typename ValueU>
  837 | inline std::common_type_t<ValueT, ValueU> fmin_nan(const ValueT a,
  838 |                                                    const ValueU b) {
  839 |   if (detail::isnan(a) || detail::isnan(b))
  840 |     return NAN;
  841 |   return compat::min(a,b);
  842 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 844-848

```text
  844 | template <typename ValueT, typename ValueU>
  845 | inline sycl::vec<std::common_type_t<ValueT, ValueU>, 2>
  846 | fmin_nan(const sycl::vec<ValueT, 2> a, const sycl::vec<ValueU, 2> b) {
  847 |   return {fmin_nan(a[0], b[0]), fmin_nan(a[1], b[1])};
  848 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 850-854

```text
  850 | template <typename ValueT, typename ValueU>
  851 | inline sycl::marray<std::common_type_t<ValueT, ValueU>, 2>
  852 | fmin_nan(const sycl::marray<ValueT, 2> a, const sycl::marray<ValueU, 2> b) {
  853 |   return {fmin_nan(a[0], b[0]), fmin_nan(a[1], b[1])};
  854 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 856-858

```text
  856 | // pow functions overload.
  857 | inline float pow(const float a, const int b) { return sycl::pown(a, b); }
  858 | inline double pow(const double a, const int b) { return sycl::pown(a, b); }
```
**EN:** Implements `pow`, a helper routine used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and bridges to SYCL execution or group abstractions.
**CN:** 实现 `pow`，这是周围 CuTe 抽象所使用的辅助例程。 它还充当构造派生对象的工厂/辅助函数并桥接到 SYCL 执行模型或 group 抽象。

### Lines 860-873

```text
  860 | template <typename ValueT, typename ValueU>
  861 | inline typename std::enable_if_t<std::is_floating_point_v<ValueT>, ValueT>
  862 | pow(const ValueT a, const ValueU b) {
  863 |   return sycl::pow(a, static_cast<ValueT>(b));
  864 | }
  865 | // TODO(compat-lib-reviewers)  calling pow with non-floating point values
  866 | // is currently defaulting to double, which fails on devices without
  867 | // aspect::fp64. This has to be properly documented, and maybe changed to
  868 | // support all devices.
  869 | template <typename ValueT, typename ValueU>
  870 | inline typename std::enable_if_t<!std::is_floating_point_v<ValueT>, double>
  871 | pow(const ValueT a, const ValueU b) {
  872 |   return sycl::pow(static_cast<double>(a), static_cast<double>(b));
  873 | }
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 875-897

```text
  875 | /// Performs relu saturation.
  876 | /// \param [in] a The input value
  877 | /// \returns the relu saturation result
  878 | template <typename ValueT> inline ValueT relu(const ValueT a) {
  879 |   if constexpr (compat::is_floating_point_v<ValueT>)
  880 |     if (detail::isnan(a))
  881 |       return a;
  882 |   if (a < ValueT(0))
  883 |     return ValueT(0);
  884 |   return a;
  885 | }
  886 | template <class ValueT, int NumElements>
  887 | inline sycl::vec<ValueT, NumElements>
  888 | relu(const sycl::vec<ValueT, NumElements> a) {
  889 |   sycl::vec<ValueT, NumElements> ret;
  890 |   for (int i = 0; i < NumElements; ++i)
  891 |     ret[i] = relu(a[i]);
  892 |   return ret;
  893 | }
  894 | template <class ValueT>
  895 | inline sycl::marray<ValueT, 2> relu(const sycl::marray<ValueT, 2> a) {
  896 |   return {relu(a[0]), relu(a[1])};
  897 | }
```
**EN:** Defines `ValueT` as a reusable type-level building block in this header. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 将 `ValueT` 定义为本头文件中的可复用类型级构件。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 899-909

```text
  899 | /// Computes the multiplication of two complex numbers.
  900 | /// \tparam T Complex element type
  901 | /// \param [in] x The first input complex number
  902 | /// \param [in] y The second input complex number
  903 | /// \returns The result
  904 | template <typename T>
  905 | sycl::vec<T, 2> cmul(sycl::vec<T, 2> x, sycl::vec<T, 2> y) {
  906 |   sycl::ext::oneapi::experimental::complex<T> t1(x[0], x[1]), t2(y[0], y[1]);
  907 |   t1 = t1 * t2;
  908 |   return sycl::vec<T, 2>(t1.real(), t1.imag());
  909 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 911-921

```text
  911 | /// Computes the division of two complex numbers.
  912 | /// \tparam T Complex element type
  913 | /// \param [in] x The first input complex number
  914 | /// \param [in] y The second input complex number
  915 | /// \returns The result
  916 | template <typename T>
  917 | sycl::vec<T, 2> cdiv(sycl::vec<T, 2> x, sycl::vec<T, 2> y) {
  918 |   sycl::ext::oneapi::experimental::complex<T> t1(x[0], x[1]), t2(y[0], y[1]);
  919 |   t1 = t1 / t2;
  920 |   return sycl::vec<T, 2>(t1.real(), t1.imag());
  921 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 923-930

```text
  923 | /// Computes the magnitude of a complex number.
  924 | /// \tparam T Complex element type
  925 | /// \param [in] x The input complex number
  926 | /// \returns The result
  927 | template <typename T> T cabs(sycl::vec<T, 2> x) {
  928 |   sycl::ext::oneapi::experimental::complex<T> t(x[0], x[1]);
  929 |   return sycl::ext::oneapi::experimental::abs(t);
  930 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 932-940

```text
  932 | /// Computes the complex conjugate of a complex number.
  933 | /// \tparam T Complex element type
  934 | /// \param [in] x The input complex number
  935 | /// \returns The result
  936 | template <typename T> sycl::vec<T, 2> conj(sycl::vec<T, 2> x) {
  937 |   sycl::ext::oneapi::experimental::complex<T> t(x[0], x[1]);
  938 |   t = conj(t);
  939 |   return sycl::vec<T, 2>(t.real(), t.imag());
  940 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 942-966

```text
  942 | /// Performs complex number multiply addition.
  943 | /// \param [in] a The first value
  944 | /// \param [in] b The second value
  945 | /// \param [in] c The third value
  946 | /// \returns the operation result
  947 | template <typename ValueT>
  948 | inline sycl::vec<ValueT, 2> cmul_add(const sycl::vec<ValueT, 2> a,
  949 |                                      const sycl::vec<ValueT, 2> b,
  950 |                                      const sycl::vec<ValueT, 2> c) {
  951 |   sycl::ext::oneapi::experimental::complex<ValueT> t(a[0], a[1]);
  952 |   sycl::ext::oneapi::experimental::complex<ValueT> u(b[0], b[1]);
  953 |   sycl::ext::oneapi::experimental::complex<ValueT> v(c[0], c[1]);
  954 |   t = t * u + v;
  955 |   return sycl::vec<ValueT, 2>{t.real(), t.imag()};
  956 | }
  957 | template <typename ValueT>
  958 | inline sycl::marray<ValueT, 2> cmul_add(const sycl::marray<ValueT, 2> a,
  959 |                                         const sycl::marray<ValueT, 2> b,
  960 |                                         const sycl::marray<ValueT, 2> c) {
  961 |   sycl::ext::oneapi::experimental::complex<ValueT> t(a[0], a[1]);
  962 |   sycl::ext::oneapi::experimental::complex<ValueT> u(b[0], b[1]);
  963 |   sycl::ext::oneapi::experimental::complex<ValueT> v(c[0], c[1]);
  964 |   t = t * u + v;
  965 |   return sycl::marray<ValueT, 2>{t.real(), t.imag()};
  966 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 968-973

```text
  968 | /// A sycl::abs wrapper functors.
  969 | struct abs {
  970 |   template <typename ValueT> auto operator()(const ValueT x) const {
  971 |     return sycl::abs(x);
  972 |   }
  973 | };
```
**EN:** Defines `abs` as a reusable type-level building block in this header. It also implements coordinate mapping or slicing behavior and acts as a factory/helper for constructing derived objects.
**CN:** 将 `abs` 定义为本头文件中的可复用类型级构件。 它还实现坐标映射或切片行为并充当构造派生对象的工厂/辅助函数。

### Lines 975-981

```text
  975 | /// A sycl::abs_diff wrapper functors.
  976 | struct abs_diff {
  977 |   template <typename ValueT>
  978 |   auto operator()(const ValueT x, const ValueT y) const {
  979 |     return sycl::abs_diff(x, y);
  980 |   }
  981 | };
```
**EN:** Defines `abs_diff` as a reusable type-level building block in this header. It also implements coordinate mapping or slicing behavior and acts as a factory/helper for constructing derived objects.
**CN:** 将 `abs_diff` 定义为本头文件中的可复用类型级构件。 它还实现坐标映射或切片行为并充当构造派生对象的工厂/辅助函数。

### Lines 983-989

```text
  983 | /// A sycl::add_sat wrapper functors.
  984 | struct add_sat {
  985 |   template <typename ValueT>
  986 |   auto operator()(const ValueT x, const ValueT y) const {
  987 |     return sycl::add_sat(x, y);
  988 |   }
  989 | };
```
**EN:** Defines `add_sat` as a reusable type-level building block in this header. It also implements coordinate mapping or slicing behavior and acts as a factory/helper for constructing derived objects.
**CN:** 将 `add_sat` 定义为本头文件中的可复用类型级构件。 它还实现坐标映射或切片行为并充当构造派生对象的工厂/辅助函数。

### Lines 991-997

```text
  991 | /// A sycl::rhadd wrapper functors.
  992 | struct rhadd {
  993 |   template <typename ValueT>
  994 |   auto operator()(const ValueT x, const ValueT y) const {
  995 |     return sycl::rhadd(x, y);
  996 |   }
  997 | };
```
**EN:** Defines `rhadd` as a reusable type-level building block in this header. It also implements coordinate mapping or slicing behavior and acts as a factory/helper for constructing derived objects.
**CN:** 将 `rhadd` 定义为本头文件中的可复用类型级构件。 它还实现坐标映射或切片行为并充当构造派生对象的工厂/辅助函数。

### Lines 999-1005

```text
  999 | /// A sycl::hadd wrapper functors.
 1000 | struct hadd {
 1001 |   template <typename ValueT>
 1002 |   auto operator()(const ValueT x, const ValueT y) const {
 1003 |     return sycl::hadd(x, y);
 1004 |   }
 1005 | };
```
**EN:** Defines `hadd` as a reusable type-level building block in this header. It also implements coordinate mapping or slicing behavior and acts as a factory/helper for constructing derived objects.
**CN:** 将 `hadd` 定义为本头文件中的可复用类型级构件。 它还实现坐标映射或切片行为并充当构造派生对象的工厂/辅助函数。

### Lines 1007-1017

```text
 1007 | /// A sycl::max wrapper functors.
 1008 | struct maximum {
 1009 |   template <typename ValueT>
 1010 |   auto operator()(const ValueT x, const ValueT y) const {
 1011 |     return sycl::max(x, y);
 1012 |   }
 1013 |   template <typename ValueT>
 1014 |   auto operator()(const ValueT x, const ValueT y, bool *pred) const {
 1015 |     return (x >= y) ? ((*pred = true), x) : ((*pred = false), y);
 1016 |   }
 1017 | };
```
**EN:** Defines `maximum` as a reusable type-level building block in this header. It also implements coordinate mapping or slicing behavior and acts as a factory/helper for constructing derived objects.
**CN:** 将 `maximum` 定义为本头文件中的可复用类型级构件。 它还实现坐标映射或切片行为并充当构造派生对象的工厂/辅助函数。

### Lines 1019-1029

```text
 1019 | /// A sycl::min wrapper functors.
 1020 | struct minimum {
 1021 |   template <typename ValueT>
 1022 |   auto operator()(const ValueT x, const ValueT y) const {
 1023 |     return sycl::min(x, y);
 1024 |   }
 1025 |   template <typename ValueT>
 1026 |   auto operator()(const ValueT x, const ValueT y, bool *pred) const {
 1027 |     return (x <= y) ? ((*pred = true), x) : ((*pred = false), y);
 1028 |   }
 1029 | };
```
**EN:** Defines `minimum` as a reusable type-level building block in this header. It also implements coordinate mapping or slicing behavior and acts as a factory/helper for constructing derived objects.
**CN:** 将 `minimum` 定义为本头文件中的可复用类型级构件。 它还实现坐标映射或切片行为并充当构造派生对象的工厂/辅助函数。

### Lines 1031-1037

```text
 1031 | /// A sycl::sub_sat wrapper functors.
 1032 | struct sub_sat {
 1033 |   template <typename ValueT>
 1034 |   auto operator()(const ValueT x, const ValueT y) const {
 1035 |     return sycl::sub_sat(x, y);
 1036 |   }
 1037 | };
```
**EN:** Defines `sub_sat` as a reusable type-level building block in this header. It also implements coordinate mapping or slicing behavior and acts as a factory/helper for constructing derived objects.
**CN:** 将 `sub_sat` 定义为本头文件中的可复用类型级构件。 它还实现坐标映射或切片行为并充当构造派生对象的工厂/辅助函数。

### Lines 1039-1045

```text
 1039 | namespace detail {
 1040 | struct shift_left {
 1041 |   template <typename T>
 1042 |   auto operator()(const T x, const uint32_t offset) const {
 1043 |     return x << offset;
 1044 |   }
 1045 | };
```
**EN:** Defines `shift_left` as a reusable type-level building block in this header. It also implements coordinate mapping or slicing behavior and acts as a factory/helper for constructing derived objects.
**CN:** 将 `shift_left` 定义为本头文件中的可复用类型级构件。 它还实现坐标映射或切片行为并充当构造派生对象的工厂/辅助函数。

### Lines 1047-1052

```text
 1047 | struct shift_right {
 1048 |   template <typename T>
 1049 |   auto operator()(const T x, const uint32_t offset) const {
 1050 |     return x >> offset;
 1051 |   }
 1052 | };
```
**EN:** Defines `shift_right` as a reusable type-level building block in this header. It also implements coordinate mapping or slicing behavior and acts as a factory/helper for constructing derived objects.
**CN:** 将 `shift_right` 定义为本头文件中的可复用类型级构件。 它还实现坐标映射或切片行为并充当构造派生对象的工厂/辅助函数。

### Lines 1054-1060

```text
 1054 | struct average {
 1055 |   template <typename T> auto operator()(const T x, const T y) const {
 1056 |     return (x + y + (x + y >= 0)) >> 1;
 1057 |   }
 1058 | };
 1060 | } // namespace detail
```
**EN:** Defines `average` as a reusable type-level building block in this header. It also implements coordinate mapping or slicing behavior and acts as a factory/helper for constructing derived objects.
**CN:** 将 `average` 定义为本头文件中的可复用类型级构件。 它还实现坐标映射或切片行为并充当构造派生对象的工厂/辅助函数。

### Lines 1062-1084

```text
 1062 | /// Compute vectorized binary operation value for two/four values, with each
 1063 | /// treated as a vector type \p VecT.
 1064 | /// \tparam [in] VecT The type of the vector
 1065 | /// \tparam [in] BinaryOperation The binary operation class
 1066 | /// \param [in] a The first value
 1067 | /// \param [in] b The second value
 1068 | /// \param [in] binary_op The operation to do with the two values
 1069 | /// \param [in] need_relu Whether the result need relu saturation
 1070 | /// \returns The vectorized binary operation value of the two values
 1071 | template <typename VecT, class BinaryOperation>
 1072 | inline unsigned vectorized_binary(unsigned a, unsigned b,
 1073 |                                   const BinaryOperation binary_op,
 1074 |                                   [[maybe_unused]] bool need_relu = false) {
 1075 |   sycl::vec<unsigned, 1> v0{a}, v1{b};
 1076 |   auto v2 = v0.as<VecT>();
 1077 |   auto v3 = v1.as<VecT>();
 1078 |   auto v4 =
 1079 |       detail::vectorized_binary<VecT, BinaryOperation>()(v2, v3, binary_op);
 1080 |   if (need_relu)
 1081 |     v4 = relu(v4);
 1082 |   v0 = v4.template as<sycl::vec<unsigned, 1>>();
 1083 |   return v0;
 1084 | }
```
**EN:** Defines `BinaryOperation` as a reusable type-level building block in this header. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 将 `BinaryOperation` 定义为本头文件中的可复用类型级构件。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 1086-1113

```text
 1086 | /// Compute two vectorized binary operation value with pred for three values,
 1087 | /// with each value treated as a 2 \p T type elements vector type.
 1088 | ///
 1089 | /// \tparam [in] VecT The type of the vector
 1090 | /// \tparam [in] BinaryOperation1 The first binary operation class
 1091 | /// \tparam [in] BinaryOperation2 The second binary operation class
 1092 | /// \param [in] a The first value
 1093 | /// \param [in] b The second value
 1094 | /// \param [in] c The third value
 1095 | /// \param [in] binary_op1 The first operation to do with the first two values
 1096 | /// \param [in] binary_op2 The second operation to do with the third values
 1097 | /// \param [in] need_relu Whether the result need relu saturation
 1098 | /// \returns The two vectorized binary operation value of the three values
 1099 | template <typename VecT, typename BinaryOperation1, typename BinaryOperation2>
 1100 | inline unsigned vectorized_ternary(unsigned a, unsigned b, unsigned c,
 1101 |                                    const BinaryOperation1 binary_op1,
 1102 |                                    const BinaryOperation2 binary_op2,
 1103 |                                    bool need_relu = false) {
 1104 |   const auto v1 = sycl::vec<unsigned, 1>(a).as<VecT>();
 1105 |   const auto v2 = sycl::vec<unsigned, 1>(b).as<VecT>();
 1106 |   const auto v3 = sycl::vec<unsigned, 1>(c).as<VecT>();
 1107 |   auto v4 =
 1108 |       detail::vectorized_binary<VecT, BinaryOperation1>()(v1, v2, binary_op1);
 1109 |   v4 = detail::vectorized_binary<VecT, BinaryOperation2>()(v4, v3, binary_op2);
 1110 |   if (need_relu)
 1111 |     v4 = relu(v4);
 1112 |   return v4.template as<sycl::vec<unsigned, 1>>();
 1113 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 1115-1136

```text
 1115 | /// Compute vectorized binary operation value with pred for two values, with
 1116 | /// each value treated as a 2 \p T type elements vector type.
 1117 | ///
 1118 | /// \tparam [in] VecT The type of the vector
 1119 | /// \tparam [in] BinaryOperation The binary operation class
 1120 | /// \param [in] a The first value
 1121 | /// \param [in] b The second value
 1122 | /// \param [in] binary_op The operation with pred to do with the two values
 1123 | /// \param [out] pred_hi The pred pointer that pass into high halfword operation
 1124 | /// \param [out] pred_lo The pred pointer that pass into low halfword operation
 1125 | /// \returns The vectorized binary operation value of the two values
 1126 | template <typename VecT, typename BinaryOperation>
 1127 | inline unsigned vectorized_binary_with_pred(unsigned a, unsigned b,
 1128 |                                             const BinaryOperation binary_op,
 1129 |                                             bool *pred_hi, bool *pred_lo) {
 1130 |   auto v1 = sycl::vec<unsigned, 1>(a).as<VecT>();
 1131 |   auto v2 = sycl::vec<unsigned, 1>(b).as<VecT>();
 1132 |   VecT ret;
 1133 |   ret[0] = binary_op(v1[0], v2[0], pred_lo);
 1134 |   ret[1] = binary_op(v1[1], v2[1], pred_hi);
 1135 |   return ret.template as<sycl::vec<unsigned, 1>>();
 1136 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 1138-1141

```text
 1138 | template <typename T1, typename T2>
 1139 | using dot_product_acc_t =
 1140 |     std::conditional_t<std::is_unsigned_v<T1> && std::is_unsigned_v<T2>,
 1141 |                        uint32_t, int32_t>;
```
**EN:** Introduces `dot_product_acc_t` as a shorthand alias so later template-heavy code can refer to this type or mapping more concisely.
**CN:** 引入 `dot_product_acc_t` 这一简写别名，使后续模板密集代码能够更简洁地引用该类型或映射。

### Lines 1143-1143

```text
 1143 | namespace detail {
```
**EN:** Opens or closes namespace scope so the following declarations remain grouped under the intended CuTe or backend namespace hierarchy.
**CN:** 打开或关闭命名空间作用域，使后续声明保持在预期的 CuTe 或后端命名空间层级中。

### Lines 1145-1150

```text
 1145 | template <typename T> sycl::vec<T, 4> extract_and_sign_or_zero_extend4(T val) {
 1146 |   return sycl::vec<T, 1>(val)
 1147 |       .template as<sycl::vec<
 1148 |           std::conditional_t<std::is_signed_v<T>, int8_t, uint8_t>, 4>>()
 1149 |       .template convert<T>();
 1150 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 1152-1159

```text
 1152 | template <typename T> sycl::vec<T, 2> extract_and_sign_or_zero_extend2(T val) {
 1153 |   return sycl::vec<T, 1>(val)
 1154 |       .template as<sycl::vec<
 1155 |           std::conditional_t<std::is_signed_v<T>, int16_t, uint16_t>, 2>>()
 1156 |       .template convert<T>();
 1157 | }
 1159 | } // namespace detail
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 1161-1206

```text
 1161 | /// Two-way dot product-accumulate. Calculate and return integer_vector2(
 1162 | /// \param a) dot product integer_vector2(low16_bit( \param b)) + \param c
 1163 | ///
 1164 | /// \tparam [in] T1 The type of first value.
 1165 | /// \tparam [in] T2 The type of second value.
 1166 | /// \param [in] a The first value.
 1167 | /// \param [in] b The second value.
 1168 | /// \param [in] c The third value. It has type uint32_t if both T1 and T1 are
 1169 | /// uint32_t else has type int32_t.
 1170 | /// \return Two-way 16-bit to 8-bit dot product which is accumulated in 32-bit
 1171 | /// result.
 1172 | template <typename T1, typename T2>
 1173 | inline dot_product_acc_t<T1, T2> dp2a_lo(T1 a, T2 b,
 1174 |                                          dot_product_acc_t<T1, T2> c) {
 1175 |   static_assert(detail::is_int32_type<T1> && detail::is_int32_type<T2>,
 1176 |                 "[Compat] dp2a_lo expects 32-bit integers as operands.");
 1177 | #if defined(__SYCL_DEVICE_ONLY__) && defined(__NVPTX__) &&                     \
 1178 |     defined(__SYCL_CUDA_ARCH__) && __SYCL_CUDA_ARCH__ >= 610
 1179 |   dot_product_acc_t<T1, T2> res;
 1180 |   if constexpr (std::is_signed_v<T1> && std::is_signed_v<T2>) {
 1181 |     asm volatile("dp2a.lo.s32.s32 %0, %1, %2, %3;"
 1182 |                  : "=r"(res)
 1183 |                  : "r"(a), "r"(b), "r"(c));
 1184 |   } else if constexpr (std::is_signed_v<T1> && std::is_unsigned_v<T2>) {
 1185 |     asm volatile("dp2a.lo.s32.u32 %0, %1, %2, %3;"
 1186 |                  : "=r"(res)
 1187 |                  : "r"(a), "r"(b), "r"(c));
 1188 |   } else if constexpr (std::is_unsigned_v<T1> && std::is_signed_v<T2>) {
 1189 |     asm volatile("dp2a.lo.u32.s32 %0, %1, %2, %3;"
 1190 |                  : "=r"(res)
 1191 |                  : "r"(a), "r"(b), "r"(c));
 1192 |   } else {
 1193 |     asm volatile("dp2a.lo.u32.u32 %0, %1, %2, %3;"
 1194 |                  : "=r"(res)
 1195 |                  : "r"(a), "r"(b), "r"(c));
 1196 |   }
 1197 |   return res;
 1198 | #else
 1199 |   dot_product_acc_t<T1, T2> res = c;
 1200 |   auto va = detail::extract_and_sign_or_zero_extend2(a);
 1201 |   auto vb = detail::extract_and_sign_or_zero_extend4(b);
 1202 |   res += va[0] * vb[0];
 1203 |   res += va[1] * vb[1];
 1204 |   return res;
 1205 | #endif
 1206 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 1208-1253

```text
 1208 | /// Two-way dot product-accumulate. Calculate and return integer_vector2(
 1209 | /// \param a) dot product integer_vector2(high_16bit( \param b)) + \param c
 1210 | ///
 1211 | /// \tparam [in] T1 The type of first value.
 1212 | /// \tparam [in] T2 The type of second value.
 1213 | /// \param [in] a The first value.
 1214 | /// \param [in] b The second value.
 1215 | /// \param [in] c The third value. uint32_t if both T1 and T1 are
 1216 | /// uint32_t else has type int32_t.
 1217 | /// \return Two-way 16-bit to 8-bit dot product which is accumulated in 32-bit
 1218 | /// result.
 1219 | template <typename T1, typename T2>
 1220 | inline dot_product_acc_t<T1, T2> dp2a_hi(T1 a, T2 b,
 1221 |                                          dot_product_acc_t<T1, T2> c) {
 1222 |   static_assert(detail::is_int32_type<T1> && detail::is_int32_type<T2>,
 1223 |                 "[Compat] dp2a_hi expects 32-bit integers as operands.");
 1224 | #if defined(__SYCL_DEVICE_ONLY__) && defined(__NVPTX__) &&                     \
 1225 |     defined(__SYCL_CUDA_ARCH__) && __SYCL_CUDA_ARCH__ >= 610
 1226 |   dot_product_acc_t<T1, T2> res;
 1227 |   if constexpr (std::is_signed_v<T1> && std::is_signed_v<T2>) {
 1228 |     asm volatile("dp2a.hi.s32.s32 %0, %1, %2, %3;"
 1229 |                  : "=r"(res)
 1230 |                  : "r"(a), "r"(b), "r"(c));
 1231 |   } else if constexpr (std::is_signed_v<T1> && std::is_unsigned_v<T2>) {
 1232 |     asm volatile("dp2a.hi.s32.u32 %0, %1, %2, %3;"
 1233 |                  : "=r"(res)
 1234 |                  : "r"(a), "r"(b), "r"(c));
 1235 |   } else if constexpr (std::is_unsigned_v<T1> && std::is_signed_v<T2>) {
 1236 |     asm volatile("dp2a.hi.u32.s32 %0, %1, %2, %3;"
 1237 |                  : "=r"(res)
 1238 |                  : "r"(a), "r"(b), "r"(c));
 1239 |   } else {
 1240 |     asm volatile("dp2a.hi.u32.u32 %0, %1, %2, %3;"
 1241 |                  : "=r"(res)
 1242 |                  : "r"(a), "r"(b), "r"(c));
 1243 |   }
 1244 |   return res;
 1245 | #else
 1246 |   dot_product_acc_t<T1, T2> res = c;
 1247 |   auto va = detail::extract_and_sign_or_zero_extend2(a);
 1248 |   auto vb = detail::extract_and_sign_or_zero_extend4(b);
 1249 |   res += va[0] * vb[2];
 1250 |   res += va[1] * vb[3];
 1251 |   return res;
 1252 | #endif
 1253 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 1255-1300

```text
 1255 | /// Four-way byte dot product-accumulate. Calculate and return integer_vector4(
 1256 | /// \param a) dot product integer_vector4( \param b)  + \param c
 1257 | ///
 1258 | /// \tparam [in] T1 The type of first value.
 1259 | /// \tparam [in] T2 The type of second value.
 1260 | /// \param [in] a The first value.
 1261 | /// \param [in] b The second value.
 1262 | /// \param [in] c The third value. It has type uint32_t if both T1 and T1 are
 1263 | /// uint32_t else has type int32_t.
 1264 | /// \return Four-way byte dot product which is accumulated in 32-bit result.
 1265 | template <typename T1, typename T2>
 1266 | inline dot_product_acc_t<T1, T2> dp4a(T1 a, T2 b, dot_product_acc_t<T1, T2> c) {
 1267 |   static_assert(detail::is_int32_type<T1> && detail::is_int32_type<T2>,
 1268 |                 "[Compat] dp4a expects 32-bit integers as operands.");
 1269 | #if defined(__SYCL_DEVICE_ONLY__) && defined(__NVPTX__) &&                     \
 1270 |     defined(__SYCL_CUDA_ARCH__) && __SYCL_CUDA_ARCH__ >= 610
 1271 |   dot_product_acc_t<T1, T2> res;
 1272 |   if constexpr (std::is_signed_v<T1> && std::is_signed_v<T2>) {
 1273 |     asm volatile("dp4a.s32.s32 %0, %1, %2, %3;"
 1274 |                  : "=r"(res)
 1275 |                  : "r"(a), "r"(b), "r"(c));
 1276 |   } else if constexpr (std::is_signed_v<T1> && std::is_unsigned_v<T2>) {
 1277 |     asm volatile("dp4a.s32.u32 %0, %1, %2, %3;"
 1278 |                  : "=r"(res)
 1279 |                  : "r"(a), "r"(b), "r"(c));
 1280 |   } else if constexpr (std::is_unsigned_v<T1> && std::is_signed_v<T2>) {
 1281 |     asm volatile("dp4a.u32.s32 %0, %1, %2, %3;"
 1282 |                  : "=r"(res)
 1283 |                  : "r"(a), "r"(b), "r"(c));
 1284 |   } else {
 1285 |     asm volatile("dp4a.u32.u32 %0, %1, %2, %3;"
 1286 |                  : "=r"(res)
 1287 |                  : "r"(a), "r"(b), "r"(c));
 1288 |   }
 1289 |   return res;
 1290 | #else
 1291 |   dot_product_acc_t<T1, T2> res = c;
 1292 |   auto va = detail::extract_and_sign_or_zero_extend4(a);
 1293 |   auto vb = detail::extract_and_sign_or_zero_extend4(b);
 1294 |   res += va[0] * vb[0];
 1295 |   res += va[1] * vb[1];
 1296 |   res += va[2] * vb[2];
 1297 |   res += va[3] * vb[3];
 1298 |   return res;
 1299 | #endif
 1300 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 1302-1312

```text
 1302 | /// Extend \p a and \p b to 33 bit and add them.
 1303 | /// \tparam [in] RetT The type of the return value
 1304 | /// \tparam [in] AT The type of the first value
 1305 | /// \tparam [in] BT The type of the second value
 1306 | /// \param [in] a The first value
 1307 | /// \param [in] b The second value
 1308 | /// \returns The extend addition of the two values
 1309 | template <typename RetT, typename AT, typename BT>
 1310 | inline constexpr RetT extend_add(AT a, BT b) {
 1311 |   return detail::extend_binary<RetT, false>(a, b, std::plus());
 1312 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 1314-1329

```text
 1314 | /// Extend Inputs to 33 bit, add \p a, \p b, then do \p second_op with \p c.
 1315 | /// \tparam [in] RetT The type of the return value
 1316 | /// \tparam [in] AT The type of the first value
 1317 | /// \tparam [in] BT The type of the second value
 1318 | /// \tparam [in] CT The type of the third value
 1319 | /// \tparam [in] BinaryOperation The type of the second operation
 1320 | /// \param [in] a The first value
 1321 | /// \param [in] b The second value
 1322 | /// \param [in] c The third value
 1323 | /// \param [in] second_op The operation to do with the third value
 1324 | /// \returns The extend addition of \p a, \p b and \p second_op with \p c
 1325 | template <typename RetT, typename AT, typename BT, typename CT,
 1326 |           typename BinaryOperation>
 1327 | inline constexpr RetT extend_add(AT a, BT b, CT c, BinaryOperation second_op) {
 1328 |   return detail::extend_binary<RetT, false>(a, b, c, std::plus(), second_op);
 1329 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 1331-1341

```text
 1331 | /// Extend \p a and \p b to 33 bit and add them with saturation.
 1332 | /// \tparam [in] RetT The type of the return value
 1333 | /// \tparam [in] AT The type of the first value
 1334 | /// \tparam [in] BT The type of the second value
 1335 | /// \param [in] a The first value
 1336 | /// \param [in] b The second value
 1337 | /// \returns The extend addition of the two values with saturation
 1338 | template <typename RetT, typename AT, typename BT>
 1339 | inline constexpr RetT extend_add_sat(AT a, BT b) {
 1340 |   return detail::extend_binary<RetT, true>(a, b, std::plus());
 1341 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 1343-1361

```text
 1343 | /// Extend Inputs to 33 bit, add \p a, \p b with saturation, then do \p
 1344 | /// second_op with \p c.
 1345 | /// \tparam [in] RetT The type of the return value
 1346 | /// \tparam [in] AT The type of the first value
 1347 | /// \tparam [in] BT The type of the second value
 1348 | /// \tparam [in] CT The type of the third value
 1349 | /// \tparam [in] BinaryOperation The type of the second operation
 1350 | /// \param [in] a The first value
 1351 | /// \param [in] b The second value
 1352 | /// \param [in] c The third value
 1353 | /// \param [in] second_op The operation to do with the third value
 1354 | /// \returns The extend addition of \p a, \p b with saturation and \p second_op
 1355 | /// with \p c
 1356 | template <typename RetT, typename AT, typename BT, typename CT,
 1357 |           typename BinaryOperation>
 1358 | inline constexpr RetT extend_add_sat(AT a, BT b, CT c,
 1359 |                                      BinaryOperation second_op) {
 1360 |   return detail::extend_binary<RetT, true>(a, b, c, std::plus(), second_op);
 1361 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 1363-1373

```text
 1363 | /// Extend \p a and \p b to 33 bit and minus them.
 1364 | /// \tparam [in] RetT The type of the return value
 1365 | /// \tparam [in] AT The type of the first value
 1366 | /// \tparam [in] BT The type of the second value
 1367 | /// \param [in] a The first value
 1368 | /// \param [in] b The second value
 1369 | /// \returns The extend subtraction of the two values
 1370 | template <typename RetT, typename AT, typename BT>
 1371 | inline constexpr RetT extend_sub(AT a, BT b) {
 1372 |   return detail::extend_binary<RetT, false>(a, b, std::minus());
 1373 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 1375-1390

```text
 1375 | /// Extend Inputs to 33 bit, minus \p a, \p b, then do \p second_op with \p c.
 1376 | /// \tparam [in] RetT The type of the return value
 1377 | /// \tparam [in] AT The type of the first value
 1378 | /// \tparam [in] BT The type of the second value
 1379 | /// \tparam [in] CT The type of the third value
 1380 | /// \tparam [in] BinaryOperation The type of the second operation
 1381 | /// \param [in] a The first value
 1382 | /// \param [in] b The second value
 1383 | /// \param [in] c The third value
 1384 | /// \param [in] second_op The operation to do with the third value
 1385 | /// \returns The extend subtraction of \p a, \p b and \p second_op with \p c
 1386 | template <typename RetT, typename AT, typename BT, typename CT,
 1387 |           typename BinaryOperation>
 1388 | inline constexpr RetT extend_sub(AT a, BT b, CT c, BinaryOperation second_op) {
 1389 |   return detail::extend_binary<RetT, false>(a, b, c, std::minus(), second_op);
 1390 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 1392-1402

```text
 1392 | /// Extend \p a and \p b to 33 bit and minus them with saturation.
 1393 | /// \tparam [in] RetT The type of the return value
 1394 | /// \tparam [in] AT The type of the first value
 1395 | /// \tparam [in] BT The type of the second value
 1396 | /// \param [in] a The first value
 1397 | /// \param [in] b The second value
 1398 | /// \returns The extend subtraction of the two values with saturation
 1399 | template <typename RetT, typename AT, typename BT>
 1400 | inline constexpr RetT extend_sub_sat(AT a, BT b) {
 1401 |   return detail::extend_binary<RetT, true>(a, b, std::minus());
 1402 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 1404-1422

```text
 1404 | /// Extend Inputs to 33 bit, minus \p a, \p b with saturation, then do \p
 1405 | /// second_op with \p c.
 1406 | /// \tparam [in] RetT The type of the return value
 1407 | /// \tparam [in] AT The type of the first value
 1408 | /// \tparam [in] BT The type of the second value
 1409 | /// \tparam [in] CT The type of the third value
 1410 | /// \tparam [in] BinaryOperation The type of the second operation
 1411 | /// \param [in] a The first value
 1412 | /// \param [in] b The second value
 1413 | /// \param [in] c The third value
 1414 | /// \param [in] second_op The operation to do with the third value
 1415 | /// \returns The extend subtraction of \p a, \p b with saturation and \p
 1416 | /// second_op with \p c
 1417 | template <typename RetT, typename AT, typename BT, typename CT,
 1418 |           typename BinaryOperation>
 1419 | inline constexpr RetT extend_sub_sat(AT a, BT b, CT c,
 1420 |                                      BinaryOperation second_op) {
 1421 |   return detail::extend_binary<RetT, true>(a, b, c, std::minus(), second_op);
 1422 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 1424-1434

```text
 1424 | /// Extend \p a and \p b to 33 bit and do abs_diff.
 1425 | /// \tparam [in] RetT The type of the return value
 1426 | /// \tparam [in] AT The type of the first value
 1427 | /// \tparam [in] BT The type of the second value
 1428 | /// \param [in] a The first value
 1429 | /// \param [in] b The second value
 1430 | /// \returns The extend abs_diff of the two values
 1431 | template <typename RetT, typename AT, typename BT>
 1432 | inline constexpr RetT extend_absdiff(AT a, BT b) {
 1433 |   return detail::extend_binary<RetT, false>(a, b, abs_diff());
 1434 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 1436-1453

```text
 1436 | /// Extend Inputs to 33 bit, abs_diff \p a, \p b, then do \p second_op with \p
 1437 | /// c.
 1438 | /// \tparam [in] RetT The type of the return value
 1439 | /// \tparam [in] AT The type of the first value
 1440 | /// \tparam [in] BT The type of the second value
 1441 | /// \tparam [in] CT The type of the third value
 1442 | /// \tparam [in] BinaryOperation The type of the second operation
 1443 | /// \param [in] a The first value
 1444 | /// \param [in] b The second value
 1445 | /// \param [in] c The third value
 1446 | /// \param [in] second_op The operation to do with the third value
 1447 | /// \returns The extend abs_diff of \p a, \p b and \p second_op with \p c
 1448 | template <typename RetT, typename AT, typename BT, typename CT,
 1449 |           typename BinaryOperation>
 1450 | inline constexpr RetT extend_absdiff(AT a, BT b, CT c,
 1451 |                                      BinaryOperation second_op) {
 1452 |   return detail::extend_binary<RetT, false>(a, b, c, abs_diff(), second_op);
 1453 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 1455-1465

```text
 1455 | /// Extend \p a and \p b to 33 bit and do abs_diff with saturation.
 1456 | /// \tparam [in] RetT The type of the return value
 1457 | /// \tparam [in] AT The type of the first value
 1458 | /// \tparam [in] BT The type of the second value
 1459 | /// \param [in] a The first value
 1460 | /// \param [in] b The second value
 1461 | /// \returns The extend abs_diff of the two values with saturation
 1462 | template <typename RetT, typename AT, typename BT>
 1463 | inline constexpr RetT extend_absdiff_sat(AT a, BT b) {
 1464 |   return detail::extend_binary<RetT, true>(a, b, abs_diff());
 1465 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 1467-1485

```text
 1467 | /// Extend Inputs to 33 bit, abs_diff \p a, \p b with saturation, then do \p
 1468 | /// second_op with \p c.
 1469 | /// \tparam [in] RetT The type of the return value
 1470 | /// \tparam [in] AT The type of the first value
 1471 | /// \tparam [in] BT The type of the second value
 1472 | /// \tparam [in] CT The type of the third value
 1473 | /// \tparam [in] BinaryOperation The type of the second operation
 1474 | /// \param [in] a The first value
 1475 | /// \param [in] b The second value
 1476 | /// \param [in] c The third value
 1477 | /// \param [in] second_op The operation to do with the third value
 1478 | /// \returns The extend abs_diff of \p a, \p b with saturation and \p
 1479 | /// second_op with \p c
 1480 | template <typename RetT, typename AT, typename BT, typename CT,
 1481 |           typename BinaryOperation>
 1482 | inline constexpr RetT extend_absdiff_sat(AT a, BT b, CT c,
 1483 |                                          BinaryOperation second_op) {
 1484 |   return detail::extend_binary<RetT, true>(a, b, c, abs_diff(), second_op);
 1485 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 1487-1497

```text
 1487 | /// Extend \p a and \p b to 33 bit and return smaller one.
 1488 | /// \tparam [in] RetT The type of the return value
 1489 | /// \tparam [in] AT The type of the first value
 1490 | /// \tparam [in] BT The type of the second value
 1491 | /// \param [in] a The first value
 1492 | /// \param [in] b The second value
 1493 | /// \returns The smaller one of the two extended values
 1494 | template <typename RetT, typename AT, typename BT>
 1495 | inline constexpr RetT extend_min(AT a, BT b) {
 1496 |   return detail::extend_binary<RetT, false>(a, b, minimum());
 1497 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 1499-1515

```text
 1499 | /// Extend Inputs to 33 bit, find the smaller one in \p a, \p b, then do \p
 1500 | /// second_op with \p c.
 1501 | /// \tparam [in] RetT The type of the return value
 1502 | /// \tparam [in] AT The type of the first value
 1503 | /// \tparam [in] BT The type of the second value
 1504 | /// \tparam [in] CT The type of the third value
 1505 | /// \tparam [in] BinaryOperation The type of the second operation
 1506 | /// \param [in] a The first value
 1507 | /// \param [in] b The second value
 1508 | /// \param [in] c The third value
 1509 | /// \param [in] second_op The operation to do with the third value
 1510 | /// \returns The smaller one of \p a, \p b and \p second_op with \p c
 1511 | template <typename RetT, typename AT, typename BT, typename CT,
 1512 |           typename BinaryOperation>
 1513 | inline constexpr RetT extend_min(AT a, BT b, CT c, BinaryOperation second_op) {
 1514 |   return detail::extend_binary<RetT, false>(a, b, c, minimum(), second_op);
 1515 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 1517-1527

```text
 1517 | /// Extend \p a and \p b to 33 bit and return smaller one with saturation.
 1518 | /// \tparam [in] RetT The type of the return value
 1519 | /// \tparam [in] AT The type of the first value
 1520 | /// \tparam [in] BT The type of the second value
 1521 | /// \param [in] a The first value
 1522 | /// \param [in] b The second value
 1523 | /// \returns The smaller one of the two extended values with saturation
 1524 | template <typename RetT, typename AT, typename BT>
 1525 | inline constexpr RetT extend_min_sat(AT a, BT b) {
 1526 |   return detail::extend_binary<RetT, true>(a, b, minimum());
 1527 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 1529-1547

```text
 1529 | /// Extend Inputs to 33 bit, find the smaller one in \p a, \p b with saturation,
 1530 | /// then do \p second_op with \p c.
 1531 | /// \tparam [in] RetT The type of the return value
 1532 | /// \tparam [in] AT The type of the first value
 1533 | /// \tparam [in] BT The type of the second value
 1534 | /// \tparam [in] CT The type of the third value
 1535 | /// \tparam [in] BinaryOperation The type of the second operation
 1536 | /// \param [in] a The first value
 1537 | /// \param [in] b The second value
 1538 | /// \param [in] c The third value
 1539 | /// \param [in] second_op The operation to do with the third value
 1540 | /// \returns The smaller one of \p a, \p b with saturation and \p
 1541 | /// second_op with \p c
 1542 | template <typename RetT, typename AT, typename BT, typename CT,
 1543 |           typename BinaryOperation>
 1544 | inline constexpr RetT extend_min_sat(AT a, BT b, CT c,
 1545 |                                      BinaryOperation second_op) {
 1546 |   return detail::extend_binary<RetT, true>(a, b, c, minimum(), second_op);
 1547 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 1549-1559

```text
 1549 | /// Extend \p a and \p b to 33 bit and return bigger one.
 1550 | /// \tparam [in] RetT The type of the return value
 1551 | /// \tparam [in] AT The type of the first value
 1552 | /// \tparam [in] BT The type of the second value
 1553 | /// \param [in] a The first value
 1554 | /// \param [in] b The second value
 1555 | /// \returns The bigger one of the two extended values
 1556 | template <typename RetT, typename AT, typename BT>
 1557 | inline constexpr RetT extend_max(AT a, BT b) {
 1558 |   return detail::extend_binary<RetT, false>(a, b, maximum());
 1559 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 1561-1577

```text
 1561 | /// Extend Inputs to 33 bit, find the bigger one in \p a, \p b, then do \p
 1562 | /// second_op with \p c.
 1563 | /// \tparam [in] RetT The type of the return value
 1564 | /// \tparam [in] AT The type of the first value
 1565 | /// \tparam [in] BT The type of the second value
 1566 | /// \tparam [in] CT The type of the third value
 1567 | /// \tparam [in] BinaryOperation The type of the second operation
 1568 | /// \param [in] a The first value
 1569 | /// \param [in] b The second value
 1570 | /// \param [in] c The third value
 1571 | /// \param [in] second_op The operation to do with the third value
 1572 | /// \returns The bigger one of \p a, \p b and \p second_op with \p c
 1573 | template <typename RetT, typename AT, typename BT, typename CT,
 1574 |           typename BinaryOperation>
 1575 | inline constexpr RetT extend_max(AT a, BT b, CT c, BinaryOperation second_op) {
 1576 |   return detail::extend_binary<RetT, false>(a, b, c, maximum(), second_op);
 1577 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 1579-1589

```text
 1579 | /// Extend \p a and \p b to 33 bit and return bigger one with saturation.
 1580 | /// \tparam [in] RetT The type of the return value
 1581 | /// \tparam [in] AT The type of the first value
 1582 | /// \tparam [in] BT The type of the second value
 1583 | /// \param [in] a The first value
 1584 | /// \param [in] b The second value
 1585 | /// \returns The bigger one of the two extended values with saturation
 1586 | template <typename RetT, typename AT, typename BT>
 1587 | inline constexpr RetT extend_max_sat(AT a, BT b) {
 1588 |   return detail::extend_binary<RetT, true>(a, b, maximum());
 1589 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 1591-1609

```text
 1591 | /// Extend Inputs to 33 bit, find the bigger one in \p a, \p b with saturation,
 1592 | /// then do \p second_op with \p c.
 1593 | /// \tparam [in] RetT The type of the return value
 1594 | /// \tparam [in] AT The type of the first value
 1595 | /// \tparam [in] BT The type of the second value
 1596 | /// \tparam [in] CT The type of the third value
 1597 | /// \tparam [in] BinaryOperation The type of the second operation
 1598 | /// \param [in] a The first value
 1599 | /// \param [in] b The second value
 1600 | /// \param [in] c The third value
 1601 | /// \param [in] second_op The operation to do with the third value
 1602 | /// \returns The bigger one of \p a, \p b with saturation and \p
 1603 | /// second_op with \p c
 1604 | template <typename RetT, typename AT, typename BT, typename CT,
 1605 |           typename BinaryOperation>
 1606 | inline constexpr RetT extend_max_sat(AT a, BT b, CT c,
 1607 |                                      BinaryOperation second_op) {
 1608 |   return detail::extend_binary<RetT, true>(a, b, c, maximum(), second_op);
 1609 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 1611-1619

```text
 1611 | /// Extend \p a and \p b to 33 bit and return a << clamp(b, 0, 32).
 1612 | /// \param [in] a The source value
 1613 | /// \param [in] b The offset to shift
 1614 | /// \returns a << clamp(b, 0, 32)
 1615 | template <typename RetT, typename T>
 1616 | inline constexpr RetT extend_shl_clamp(T a, uint32_t b) {
 1617 |   return detail::extend_binary<RetT, false>(a, sycl::clamp(b, 0u, 32u),
 1618 |                                             detail::shift_left());
 1619 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 1621-1632

```text
 1621 | /// Extend Inputs to 33 bit, and return second_op(a << clamp(b, 0, 32), c).
 1622 | /// \param [in] a The source value
 1623 | /// \param [in] b The offset to shift
 1624 | /// \param [in] c The value to merge
 1625 | /// \param [in] second_op The operation to do with the third value
 1626 | /// \returns second_op(a << clamp(b, 0, 32), c)
 1627 | template <typename RetT, typename T, typename BinaryOperation>
 1628 | inline constexpr RetT extend_shl_clamp(T a, uint32_t b, uint32_t c,
 1629 |                                        BinaryOperation second_op) {
 1630 |   return detail::extend_binary<RetT, false>(a, sycl::clamp(b, 0u, 32u), c,
 1631 |                                             detail::shift_left(), second_op);
 1632 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 1634-1642

```text
 1634 | /// Extend \p a and \p b to 33 bit and return sat(a << clamp(b, 0, 32)).
 1635 | /// \param [in] a The source value
 1636 | /// \param [in] b The offset to shift
 1637 | /// \returns sat(a << clamp(b, 0, 32))
 1638 | template <typename RetT, typename T>
 1639 | inline constexpr RetT extend_shl_sat_clamp(T a, uint32_t b) {
 1640 |   return detail::extend_binary<RetT, true>(a, sycl::clamp(b, 0u, 32u),
 1641 |                                            detail::shift_left());
 1642 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 1644-1655

```text
 1644 | /// Extend Inputs to 33 bit, and return second_op(sat(a << clamp(b, 0, 32)), c).
 1645 | /// \param [in] a The source value
 1646 | /// \param [in] b The offset to shift
 1647 | /// \param [in] c The value to merge
 1648 | /// \param [in] second_op The operation to do with the third value
 1649 | /// \returns second_op(sat(a << clamp(b, 0, 32)), c)
 1650 | template <typename RetT, typename T, typename BinaryOperation>
 1651 | inline constexpr RetT extend_shl_sat_clamp(T a, uint32_t b, uint32_t c,
 1652 |                                            BinaryOperation second_op) {
 1653 |   return detail::extend_binary<RetT, true>(a, sycl::clamp(b, 0u, 32u), c,
 1654 |                                            detail::shift_left(), second_op);
 1655 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 1657-1664

```text
 1657 | /// Extend \p a and \p b to 33 bit and return a << (b & 0x1F).
 1658 | /// \param [in] a The source value
 1659 | /// \param [in] b The offset to shift
 1660 | /// \returns a << (b & 0x1F)
 1661 | template <typename RetT, typename T>
 1662 | inline constexpr RetT extend_shl_wrap(T a, uint32_t b) {
 1663 |   return detail::extend_binary<RetT, false>(a, b & 0x1F, detail::shift_left());
 1664 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 1666-1677

```text
 1666 | /// Extend Inputs to 33 bit, and return second_op(a << (b & 0x1F), c).
 1667 | /// \param [in] a The source value
 1668 | /// \param [in] b The offset to shift
 1669 | /// \param [in] c The value to merge
 1670 | /// \param [in] second_op The operation to do with the third value
 1671 | /// \returns second_op(a << (b & 0x1F), c)
 1672 | template <typename RetT, typename T, typename BinaryOperation>
 1673 | inline constexpr RetT extend_shl_wrap(T a, uint32_t b, uint32_t c,
 1674 |                                       BinaryOperation second_op) {
 1675 |   return detail::extend_binary<RetT, false>(a, b & 0x1F, c,
 1676 |                                             detail::shift_left(), second_op);
 1677 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 1679-1686

```text
 1679 | /// Extend \p a and \p b to 33 bit and return sat(a << (b & 0x1F)).
 1680 | /// \param [in] a The source value
 1681 | /// \param [in] b The offset to shift
 1682 | /// \returns sat(a << (b & 0x1F))
 1683 | template <typename RetT, typename T>
 1684 | inline constexpr RetT extend_shl_sat_wrap(T a, uint32_t b) {
 1685 |   return detail::extend_binary<RetT, true>(a, b & 0x1F, detail::shift_left());
 1686 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 1688-1699

```text
 1688 | /// Extend Inputs to 33 bit, and return second_op(sat(a << (b & 0x1F)), c).
 1689 | /// \param [in] a The source value
 1690 | /// \param [in] b The offset to shift
 1691 | /// \param [in] c The value to merge
 1692 | /// \param [in] second_op The operation to do with the third value
 1693 | /// \returns second_op(sat(a << (b & 0x1F)), c)
 1694 | template <typename RetT, typename T, typename BinaryOperation>
 1695 | inline constexpr RetT extend_shl_sat_wrap(T a, uint32_t b, uint32_t c,
 1696 |                                           BinaryOperation second_op) {
 1697 |   return detail::extend_binary<RetT, true>(a, b & 0x1F, c, detail::shift_left(),
 1698 |                                            second_op);
 1699 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 1701-1709

```text
 1701 | /// Extend \p a and \p b to 33 bit and return a >> clamp(b, 0, 32).
 1702 | /// \param [in] a The source value
 1703 | /// \param [in] b The offset to shift
 1704 | /// \returns a >> clamp(b, 0, 32)
 1705 | template <typename RetT, typename T>
 1706 | inline constexpr RetT extend_shr_clamp(T a, uint32_t b) {
 1707 |   return detail::extend_binary<RetT, false>(a, sycl::clamp(b, 0u, 32u),
 1708 |                                             detail::shift_right());
 1709 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 1711-1722

```text
 1711 | /// Extend Inputs to 33 bit, and return second_op(a >> clamp(b, 0, 32), c).
 1712 | /// \param [in] a The source value
 1713 | /// \param [in] b The offset to shift
 1714 | /// \param [in] c The value to merge
 1715 | /// \param [in] second_op The operation to do with the third value
 1716 | /// \returns second_op(a >> clamp(b, 0, 32), c)
 1717 | template <typename RetT, typename T, typename BinaryOperation>
 1718 | inline constexpr RetT extend_shr_clamp(T a, uint32_t b, uint32_t c,
 1719 |                                        BinaryOperation second_op) {
 1720 |   return detail::extend_binary<RetT, false>(a, sycl::clamp(b, 0u, 32u), c,
 1721 |                                             detail::shift_right(), second_op);
 1722 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 1724-1732

```text
 1724 | /// Extend \p a and \p b to 33 bit and return sat(a >> clamp(b, 0, 32)).
 1725 | /// \param [in] a The source value
 1726 | /// \param [in] b The offset to shift
 1727 | /// \returns sat(a >> clamp(b, 0, 32))
 1728 | template <typename RetT, typename T>
 1729 | inline constexpr RetT extend_shr_sat_clamp(T a, uint32_t b) {
 1730 |   return detail::extend_binary<RetT, true>(a, sycl::clamp(b, 0u, 32u),
 1731 |                                            detail::shift_right());
 1732 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 1734-1745

```text
 1734 | /// Extend Inputs to 33 bit, and return second_op(sat(a >> clamp(b, 0, 32)), c).
 1735 | /// \param [in] a The source value
 1736 | /// \param [in] b The offset to shift
 1737 | /// \param [in] c The value to merge
 1738 | /// \param [in] second_op The operation to do with the third value
 1739 | /// \returns second_op(sat(a >> clamp(b, 0, 32)), c)
 1740 | template <typename RetT, typename T, typename BinaryOperation>
 1741 | inline constexpr RetT extend_shr_sat_clamp(T a, uint32_t b, uint32_t c,
 1742 |                                            BinaryOperation second_op) {
 1743 |   return detail::extend_binary<RetT, true>(a, sycl::clamp(b, 0u, 32u), c,
 1744 |                                            detail::shift_right(), second_op);
 1745 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 1747-1754

```text
 1747 | /// Extend \p a and \p b to 33 bit and return a >> (b & 0x1F).
 1748 | /// \param [in] a The source value
 1749 | /// \param [in] b The offset to shift
 1750 | /// \returns a >> (b & 0x1F)
 1751 | template <typename RetT, typename T>
 1752 | inline constexpr RetT extend_shr_wrap(T a, uint32_t b) {
 1753 |   return detail::extend_binary<RetT, false>(a, b & 0x1F, detail::shift_right());
 1754 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 1756-1767

```text
 1756 | /// Extend Inputs to 33 bit, and return second_op(a >> (b & 0x1F), c).
 1757 | /// \param [in] a The source value
 1758 | /// \param [in] b The offset to shift
 1759 | /// \param [in] c The value to merge
 1760 | /// \param [in] second_op The operation to do with the third value
 1761 | /// \returns second_op(a >> (b & 0x1F), c)
 1762 | template <typename RetT, typename T, typename BinaryOperation>
 1763 | inline constexpr RetT extend_shr_wrap(T a, uint32_t b, uint32_t c,
 1764 |                                       BinaryOperation second_op) {
 1765 |   return detail::extend_binary<RetT, false>(a, b & 0x1F, c,
 1766 |                                             detail::shift_right(), second_op);
 1767 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 1769-1776

```text
 1769 | /// Extend \p a and \p b to 33 bit and return sat(a >> (b & 0x1F)).
 1770 | /// \param [in] a The source value
 1771 | /// \param [in] b The offset to shift
 1772 | /// \returns sat(a >> (b & 0x1F))
 1773 | template <typename RetT, typename T>
 1774 | inline constexpr RetT extend_shr_sat_wrap(T a, uint32_t b) {
 1775 |   return detail::extend_binary<RetT, true>(a, b & 0x1F, detail::shift_right());
 1776 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 1778-1789

```text
 1778 | /// Extend Inputs to 33 bit, and return second_op(sat(a >> (b & 0x1F)), c).
 1779 | /// \param [in] a The source value
 1780 | /// \param [in] b The offset to shift
 1781 | /// \param [in] c The value to merge
 1782 | /// \param [in] second_op The operation to do with the third value
 1783 | /// \returns second_op(sat(a >> (b & 0x1F)), c)
 1784 | template <typename RetT, typename T, typename BinaryOperation>
 1785 | inline constexpr RetT extend_shr_sat_wrap(T a, uint32_t b, uint32_t c,
 1786 |                                           BinaryOperation second_op) {
 1787 |   return detail::extend_binary<RetT, true>(a, b & 0x1F, c,
 1788 |                                            detail::shift_right(), second_op);
 1789 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 1791-1803

```text
 1791 | /// Compute vectorized addition of \p a and \p b, with each value treated as a
 1792 | /// 2 elements vector type and extend each element to 17 bit.
 1793 | /// \tparam [in] RetT The type of the return value, can only be 32 bit integer
 1794 | /// \tparam [in] AT The type of the first value, can only be 32 bit integer
 1795 | /// \tparam [in] BT The type of the second value, can only be 32 bit integer
 1796 | /// \param [in] a The first value
 1797 | /// \param [in] b The second value
 1798 | /// \param [in] c The third value
 1799 | /// \returns The extend vectorized addition of the two values
 1800 | template <typename RetT, typename AT, typename BT>
 1801 | inline constexpr RetT extend_vadd2(AT a, BT b, RetT c) {
 1802 |   return detail::extend_vbinary2<RetT, false, false>(a, b, c, std::plus());
 1803 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 1805-1819

```text
 1805 | /// Compute vectorized addition of \p a and \p b, with each value treated as a 2
 1806 | /// elements vector type and extend each element to 17 bit. Then add each half
 1807 | /// of the result and add with \p c.
 1808 | /// \tparam [in] RetT The type of the return value, can only be 32 bit integer
 1809 | /// \tparam [in] AT The type of the first value, can only be 32 bit integer
 1810 | /// \tparam [in] BT The type of the second value, can only be 32 bit integer
 1811 | /// \param [in] a The first value
 1812 | /// \param [in] b The second value
 1813 | /// \param [in] c The third value
 1814 | /// \returns The addition of each half of extend vectorized addition of the two
 1815 | /// values and the third value
 1816 | template <typename RetT, typename AT, typename BT>
 1817 | inline constexpr RetT extend_vadd2_add(AT a, BT b, RetT c) {
 1818 |   return detail::extend_vbinary2<RetT, false, true>(a, b, c, std::plus());
 1819 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 1821-1833

```text
 1821 | /// Compute vectorized addition of \p a and \p b with saturation, with each
 1822 | /// value treated as a 2 elements vector type and extend each element to 17 bit.
 1823 | /// \tparam [in] RetT The type of the return value, can only be 32 bit integer
 1824 | /// \tparam [in] AT The type of the first value, can only be 32 bit integer
 1825 | /// \tparam [in] BT The type of the second value, can only be 32 bit integer
 1826 | /// \param [in] a The first value
 1827 | /// \param [in] b The second value
 1828 | /// \param [in] c The third value
 1829 | /// \returns The extend vectorized addition of the two values with saturation
 1830 | template <typename RetT, typename AT, typename BT>
 1831 | inline constexpr RetT extend_vadd2_sat(AT a, BT b, RetT c) {
 1832 |   return detail::extend_vbinary2<RetT, true, false>(a, b, c, std::plus());
 1833 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 1835-1847

```text
 1835 | /// Compute vectorized subtraction of \p a and \p b, with each value treated as
 1836 | /// a 2 elements vector type and extend each element to 17 bit.
 1837 | /// \tparam [in] RetT The type of the return value, can only be 32 bit integer
 1838 | /// \tparam [in] AT The type of the first value, can only be 32 bit integer
 1839 | /// \tparam [in] BT The type of the second value, can only be 32 bit integer
 1840 | /// \param [in] a The first value
 1841 | /// \param [in] b The second value
 1842 | /// \param [in] c The third value
 1843 | /// \returns The extend vectorized subtraction of the two values
 1844 | template <typename RetT, typename AT, typename BT>
 1845 | inline constexpr RetT extend_vsub2(AT a, BT b, RetT c) {
 1846 |   return detail::extend_vbinary2<RetT, false, false>(a, b, c, std::minus());
 1847 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 1849-1863

```text
 1849 | /// Compute vectorized subtraction of \p a and \p b, with each value treated as
 1850 | /// a 2 elements vector type and extend each element to 17 bit. Then add each
 1851 | /// half of the result and add with \p c.
 1852 | /// \tparam [in] RetT The type of the return value, can only be 32 bit integer
 1853 | /// \tparam [in] AT The type of the first value, can only be 32 bit integer
 1854 | /// \tparam [in] BT The type of the second value, can only be 32 bit integer
 1855 | /// \param [in] a The first value
 1856 | /// \param [in] b The second value
 1857 | /// \param [in] c The third value
 1858 | /// \returns The addition of each half of extend vectorized subtraction of the
 1859 | /// two values and the third value
 1860 | template <typename RetT, typename AT, typename BT>
 1861 | inline constexpr RetT extend_vsub2_add(AT a, BT b, RetT c) {
 1862 |   return detail::extend_vbinary2<RetT, false, true>(a, b, c, std::minus());
 1863 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 1865-1877

```text
 1865 | /// Compute vectorized subtraction of \p a and \p b with saturation, with each
 1866 | /// value treated as a 2 elements vector type and extend each element to 17 bit.
 1867 | /// \tparam [in] RetT The type of the return value, can only be 32 bit integer
 1868 | /// \tparam [in] AT The type of the first value, can only be 32 bit integer
 1869 | /// \tparam [in] BT The type of the second value, can only be 32 bit integer
 1870 | /// \param [in] a The first value
 1871 | /// \param [in] b The second value
 1872 | /// \param [in] c The third value
 1873 | /// \returns The extend vectorized subtraction of the two values with saturation
 1874 | template <typename RetT, typename AT, typename BT>
 1875 | inline constexpr RetT extend_vsub2_sat(AT a, BT b, RetT c) {
 1876 |   return detail::extend_vbinary2<RetT, true, false>(a, b, c, std::minus());
 1877 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 1879-1891

```text
 1879 | /// Compute vectorized abs_diff of \p a and \p b, with each value treated as a 2
 1880 | /// elements vector type and extend each element to 17 bit.
 1881 | /// \tparam [in] RetT The type of the return value, can only be 32 bit integer
 1882 | /// \tparam [in] AT The type of the first value, can only be 32 bit integer
 1883 | /// \tparam [in] BT The type of the second value, can only be 32 bit integer
 1884 | /// \param [in] a The first value
 1885 | /// \param [in] b The second value
 1886 | /// \param [in] c The third value
 1887 | /// \returns The extend vectorized abs_diff of the two values
 1888 | template <typename RetT, typename AT, typename BT>
 1889 | inline constexpr RetT extend_vabsdiff2(AT a, BT b, RetT c) {
 1890 |   return detail::extend_vbinary2<RetT, false, false>(a, b, c, abs_diff());
 1891 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 1893-1907

```text
 1893 | /// Compute vectorized abs_diff of \p a and \p b, with each value treated as a 2
 1894 | /// elements vector type and extend each element to 17 bit. Then add each half
 1895 | /// of the result and add with \p c.
 1896 | /// \tparam [in] RetT The type of the return value, can only be 32 bit integer
 1897 | /// \tparam [in] AT The type of the first value, can only be 32 bit integer
 1898 | /// \tparam [in] BT The type of the second value, can only be 32 bit integer
 1899 | /// \param [in] a The first value
 1900 | /// \param [in] b The second value
 1901 | /// \param [in] c The third value
 1902 | /// \returns The addition of each half of extend vectorized abs_diff of the
 1903 | /// two values and the third value
 1904 | template <typename RetT, typename AT, typename BT>
 1905 | inline constexpr RetT extend_vabsdiff2_add(AT a, BT b, RetT c) {
 1906 |   return detail::extend_vbinary2<RetT, false, true>(a, b, c, abs_diff());
 1907 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 1909-1921

```text
 1909 | /// Compute vectorized abs_diff of \p a and \p b with saturation, with each
 1910 | /// value treated as a 2 elements vector type and extend each element to 17 bit.
 1911 | /// \tparam [in] RetT The type of the return value, can only be 32 bit integer
 1912 | /// \tparam [in] AT The type of the first value, can only be 32 bit integer
 1913 | /// \tparam [in] BT The type of the second value, can only be 32 bit integer
 1914 | /// \param [in] a The first value
 1915 | /// \param [in] b The second value
 1916 | /// \param [in] c The third value
 1917 | /// \returns The extend vectorized abs_diff of the two values with saturation
 1918 | template <typename RetT, typename AT, typename BT>
 1919 | inline constexpr RetT extend_vabsdiff2_sat(AT a, BT b, RetT c) {
 1920 |   return detail::extend_vbinary2<RetT, true, false>(a, b, c, abs_diff());
 1921 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 1923-1935

```text
 1923 | /// Compute vectorized minimum of \p a and \p b, with each value treated as a 2
 1924 | /// elements vector type and extend each element to 17 bit.
 1925 | /// \tparam [in] RetT The type of the return value, can only be 32 bit integer
 1926 | /// \tparam [in] AT The type of the first value, can only be 32 bit integer
 1927 | /// \tparam [in] BT The type of the second value, can only be 32 bit integer
 1928 | /// \param [in] a The first value
 1929 | /// \param [in] b The second value
 1930 | /// \param [in] c The third value
 1931 | /// \returns The extend vectorized minimum of the two values
 1932 | template <typename RetT, typename AT, typename BT>
 1933 | inline constexpr RetT extend_vmin2(AT a, BT b, RetT c) {
 1934 |   return detail::extend_vbinary2<RetT, false, false>(a, b, c, minimum());
 1935 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 1937-1951

```text
 1937 | /// Compute vectorized minimum of \p a and \p b, with each value treated as a 2
 1938 | /// elements vector type and extend each element to 17 bit. Then add each half
 1939 | /// of the result and add with \p c.
 1940 | /// \tparam [in] RetT The type of the return value, can only be 32 bit integer
 1941 | /// \tparam [in] AT The type of the first value, can only be 32 bit integer
 1942 | /// \tparam [in] BT The type of the second value, can only be 32 bit integer
 1943 | /// \param [in] a The first value
 1944 | /// \param [in] b The second value
 1945 | /// \param [in] c The third value
 1946 | /// \returns The addition of each half of extend vectorized minimum of the
 1947 | /// two values and the third value
 1948 | template <typename RetT, typename AT, typename BT>
 1949 | inline constexpr RetT extend_vmin2_add(AT a, BT b, RetT c) {
 1950 |   return detail::extend_vbinary2<RetT, false, true>(a, b, c, minimum());
 1951 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 1953-1965

```text
 1953 | /// Compute vectorized minimum of \p a and \p b with saturation, with each value
 1954 | /// treated as a 2 elements vector type and extend each element to 17 bit.
 1955 | /// \tparam [in] RetT The type of the return value, can only be 32 bit integer
 1956 | /// \tparam [in] AT The type of the first value, can only be 32 bit integer
 1957 | /// \tparam [in] BT The type of the second value, can only be 32 bit integer
 1958 | /// \param [in] a The first value
 1959 | /// \param [in] b The second value
 1960 | /// \param [in] c The third value
 1961 | /// \returns The extend vectorized minimum of the two values with saturation
 1962 | template <typename RetT, typename AT, typename BT>
 1963 | inline constexpr RetT extend_vmin2_sat(AT a, BT b, RetT c) {
 1964 |   return detail::extend_vbinary2<RetT, true, false>(a, b, c, minimum());
 1965 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 1967-1979

```text
 1967 | /// Compute vectorized maximum of \p a and \p b, with each value treated as a 2
 1968 | /// elements vector type and extend each element to 17 bit.
 1969 | /// \tparam [in] RetT The type of the return value, can only be 32 bit integer
 1970 | /// \tparam [in] AT The type of the first value, can only be 32 bit integer
 1971 | /// \tparam [in] BT The type of the second value, can only be 32 bit integer
 1972 | /// \param [in] a The first value
 1973 | /// \param [in] b The second value
 1974 | /// \param [in] c The third value
 1975 | /// \returns The extend vectorized maximum of the two values
 1976 | template <typename RetT, typename AT, typename BT>
 1977 | inline constexpr RetT extend_vmax2(AT a, BT b, RetT c) {
 1978 |   return detail::extend_vbinary2<RetT, false, false>(a, b, c, maximum());
 1979 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 1981-1995

```text
 1981 | /// Compute vectorized maximum of \p a and \p b, with each value treated as a 2
 1982 | /// elements vector type and extend each element to 17 bit. Then add each half
 1983 | /// of the result and add with \p c.
 1984 | /// \tparam [in] RetT The type of the return value, can only be 32 bit integer
 1985 | /// \tparam [in] AT The type of the first value, can only be 32 bit integer
 1986 | /// \tparam [in] BT The type of the second value, can only be 32 bit integer
 1987 | /// \param [in] a The first value
 1988 | /// \param [in] b The second value
 1989 | /// \param [in] c The third value
 1990 | /// \returns The addition of each half of extend vectorized maximum of the
 1991 | /// two values and the third value
 1992 | template <typename RetT, typename AT, typename BT>
 1993 | inline constexpr RetT extend_vmax2_add(AT a, BT b, RetT c) {
 1994 |   return detail::extend_vbinary2<RetT, false, true>(a, b, c, maximum());
 1995 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 1997-2009

```text
 1997 | /// Compute vectorized maximum of \p a and \p b with saturation, with each value
 1998 | /// treated as a 2 elements vector type and extend each element to 17 bit.
 1999 | /// \tparam [in] RetT The type of the return value, can only be 32 bit integer
 2000 | /// \tparam [in] AT The type of the first value, can only be 32 bit integer
 2001 | /// \tparam [in] BT The type of the second value, can only be 32 bit integer
 2002 | /// \param [in] a The first value
 2003 | /// \param [in] b The second value
 2004 | /// \param [in] c The third value
 2005 | /// \returns The extend vectorized maximum of the two values with saturation
 2006 | template <typename RetT, typename AT, typename BT>
 2007 | inline constexpr RetT extend_vmax2_sat(AT a, BT b, RetT c) {
 2008 |   return detail::extend_vbinary2<RetT, true, false>(a, b, c, maximum());
 2009 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 2011-2024

```text
 2011 | /// Compute vectorized average of \p a and \p b, with each value treated as a 2
 2012 | /// elements vector type and extend each element to 17 bit.
 2013 | /// \tparam [in] RetT The type of the return value, can only be 32 bit integer
 2014 | /// \tparam [in] AT The type of the first value, can only be 32 bit integer
 2015 | /// \tparam [in] BT The type of the second value, can only be 32 bit integer
 2016 | /// \param [in] a The first value
 2017 | /// \param [in] b The second value
 2018 | /// \param [in] c The third value
 2019 | /// \returns The extend vectorized average of the two values
 2020 | template <typename RetT, typename AT, typename BT>
 2021 | inline constexpr RetT extend_vavrg2(AT a, BT b, RetT c) {
 2022 |   return detail::extend_vbinary2<RetT, false, false>(a, b, c,
 2023 |                                                      detail::average());
 2024 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 2026-2040

```text
 2026 | /// Compute vectorized average of \p a and \p b, with each value treated as a 2
 2027 | /// elements vector type and extend each element to 17 bit. Then add each half
 2028 | /// of the result and add with \p c.
 2029 | /// \tparam [in] RetT The type of the return value, can only be 32 bit integer
 2030 | /// \tparam [in] AT The type of the first value, can only be 32 bit integer
 2031 | /// \tparam [in] BT The type of the second value, can only be 32 bit integer
 2032 | /// \param [in] a The first value
 2033 | /// \param [in] b The second value
 2034 | /// \param [in] c The third value
 2035 | /// \returns The addition of each half of extend average maximum of the
 2036 | /// two values and the third value
 2037 | template <typename RetT, typename AT, typename BT>
 2038 | inline constexpr RetT extend_vavrg2_add(AT a, BT b, RetT c) {
 2039 |   return detail::extend_vbinary2<RetT, false, true>(a, b, c, detail::average());
 2040 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 2042-2054

```text
 2042 | /// Compute vectorized average of \p a and \p b with saturation, with each value
 2043 | /// treated as a 2 elements vector type and extend each element to 17 bit.
 2044 | /// \tparam [in] RetT The type of the return value, can only be 32 bit integer
 2045 | /// \tparam [in] AT The type of the first value, can only be 32 bit integer
 2046 | /// \tparam [in] BT The type of the second value, can only be 32 bit integer
 2047 | /// \param [in] a The first value
 2048 | /// \param [in] b The second value
 2049 | /// \param [in] c The third value
 2050 | /// \returns The extend vectorized average of the two values with saturation
 2051 | template <typename RetT, typename AT, typename BT>
 2052 | inline constexpr RetT extend_vavrg2_sat(AT a, BT b, RetT c) {
 2053 |   return detail::extend_vbinary2<RetT, true, false>(a, b, c, detail::average());
 2054 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 2056-2069

```text
 2056 | /// Extend \p a and \p b to 33 bit and vectorized compare input values using
 2057 | /// specified comparison \p cmp .
 2058 | ///
 2059 | /// \tparam [in] AT The type of the first value, can only be 32 bit integer
 2060 | /// \tparam [in] BT The type of the second value, can only be 32 bit integer
 2061 | /// \tparam [in] BinaryOperation The type of the compare operation
 2062 | /// \param [in] a The first value
 2063 | /// \param [in] b The second value
 2064 | /// \param [in] cmp The comparsion operator
 2065 | /// \returns The comparison result of the two extended values.
 2066 | template <typename AT, typename BT, typename BinaryOperation>
 2067 | inline constexpr unsigned extend_vcompare2(AT a, BT b, BinaryOperation cmp) {
 2068 |   return detail::extend_vbinary2<unsigned, false, false>(a, b, 0, cmp);
 2069 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 2071-2087

```text
 2071 | /// Extend Inputs to 33 bit, and vectorized compare input values using specified
 2072 | /// comparison \p cmp , then add the result with \p c .
 2073 | ///
 2074 | /// \tparam [in] AT The type of the first value, can only be 32 bit integer
 2075 | /// \tparam [in] BT The type of the second value, can only be 32 bit integer
 2076 | /// \tparam [in] BinaryOperation The type of the compare operation
 2077 | /// \param [in] a The first value
 2078 | /// \param [in] b The second value
 2079 | /// \param [in] c The third value
 2080 | /// \param [in] cmp The comparsion operator
 2081 | /// \returns The comparison result of the two extended values, and add the
 2082 | /// result with \p c .
 2083 | template <typename AT, typename BT, typename BinaryOperation>
 2084 | inline constexpr unsigned extend_vcompare2_add(AT a, BT b, unsigned c,
 2085 |                                                BinaryOperation cmp) {
 2086 |   return detail::extend_vbinary2<unsigned, false, true>(a, b, c, cmp);
 2087 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 2089-2101

```text
 2089 | /// Compute vectorized addition of \p a and \p b, with each value treated as a
 2090 | /// 4 elements vector type and extend each element to 9 bit.
 2091 | /// \tparam [in] RetT The type of the return value, can only be 32 bit integer
 2092 | /// \tparam [in] AT The type of the first value, can only be 32 bit integer
 2093 | /// \tparam [in] BT The type of the second value, can only be 32 bit integer
 2094 | /// \param [in] a The first value
 2095 | /// \param [in] b The second value
 2096 | /// \param [in] c The third value
 2097 | /// \returns The extend vectorized addition of the two values
 2098 | template <typename RetT, typename AT, typename BT>
 2099 | inline constexpr RetT extend_vadd4(AT a, BT b, RetT c) {
 2100 |   return detail::extend_vbinary4<RetT, false, false>(a, b, c, std::plus());
 2101 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 2103-2117

```text
 2103 | /// Compute vectorized addition of \p a and \p b, with each value treated as a 4
 2104 | /// elements vector type and extend each element to 9 bit. Then add each half
 2105 | /// of the result and add with \p c.
 2106 | /// \tparam [in] RetT The type of the return value, can only be 32 bit integer
 2107 | /// \tparam [in] AT The type of the first value, can only be 32 bit integer
 2108 | /// \tparam [in] BT The type of the second value, can only be 32 bit integer
 2109 | /// \param [in] a The first value
 2110 | /// \param [in] b The second value
 2111 | /// \param [in] c The third value
 2112 | /// \returns The addition of each half of extend vectorized addition of the two
 2113 | /// values and the third value
 2114 | template <typename RetT, typename AT, typename BT>
 2115 | inline constexpr RetT extend_vadd4_add(AT a, BT b, RetT c) {
 2116 |   return detail::extend_vbinary4<RetT, false, true>(a, b, c, std::plus());
 2117 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 2119-2131

```text
 2119 | /// Compute vectorized addition of \p a and \p b with saturation, with each
 2120 | /// value treated as a 4 elements vector type and extend each element to 9 bit.
 2121 | /// \tparam [in] RetT The type of the return value, can only be 32 bit integer
 2122 | /// \tparam [in] AT The type of the first value, can only be 32 bit integer
 2123 | /// \tparam [in] BT The type of the second value, can only be 32 bit integer
 2124 | /// \param [in] a The first value
 2125 | /// \param [in] b The second value
 2126 | /// \param [in] c The third value
 2127 | /// \returns The extend vectorized addition of the two values with saturation
 2128 | template <typename RetT, typename AT, typename BT>
 2129 | inline constexpr RetT extend_vadd4_sat(AT a, BT b, RetT c) {
 2130 |   return detail::extend_vbinary4<RetT, true, false>(a, b, c, std::plus());
 2131 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 2133-2145

```text
 2133 | /// Compute vectorized subtraction of \p a and \p b, with each value treated as
 2134 | /// a 4 elements vector type and extend each element to 9 bit.
 2135 | /// \tparam [in] RetT The type of the return value, can only be 32 bit integer
 2136 | /// \tparam [in] AT The type of the first value, can only be 32 bit integer
 2137 | /// \tparam [in] BT The type of the second value, can only be 32 bit integer
 2138 | /// \param [in] a The first value
 2139 | /// \param [in] b The second value
 2140 | /// \param [in] c The third value
 2141 | /// \returns The extend vectorized subtraction of the two values
 2142 | template <typename RetT, typename AT, typename BT>
 2143 | inline constexpr RetT extend_vsub4(AT a, BT b, RetT c) {
 2144 |   return detail::extend_vbinary4<RetT, false, false>(a, b, c, std::minus());
 2145 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 2147-2161

```text
 2147 | /// Compute vectorized subtraction of \p a and \p b, with each value treated as
 2148 | /// a 4 elements vector type and extend each element to 9 bit. Then add each
 2149 | /// half of the result and add with \p c.
 2150 | /// \tparam [in] RetT The type of the return value, can only be 32 bit integer
 2151 | /// \tparam [in] AT The type of the first value, can only be 32 bit integer
 2152 | /// \tparam [in] BT The type of the second value, can only be 32 bit integer
 2153 | /// \param [in] a The first value
 2154 | /// \param [in] b The second value
 2155 | /// \param [in] c The third value
 2156 | /// \returns The addition of each half of extend vectorized subtraction of the
 2157 | /// two values and the third value
 2158 | template <typename RetT, typename AT, typename BT>
 2159 | inline constexpr RetT extend_vsub4_add(AT a, BT b, RetT c) {
 2160 |   return detail::extend_vbinary4<RetT, false, true>(a, b, c, std::minus());
 2161 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 2163-2175

```text
 2163 | /// Compute vectorized subtraction of \p a and \p b with saturation, with each
 2164 | /// value treated as a 4 elements vector type and extend each element to 9 bit.
 2165 | /// \tparam [in] RetT The type of the return value, can only be 32 bit integer
 2166 | /// \tparam [in] AT The type of the first value, can only be 32 bit integer
 2167 | /// \tparam [in] BT The type of the second value, can only be 32 bit integer
 2168 | /// \param [in] a The first value
 2169 | /// \param [in] b The second value
 2170 | /// \param [in] c The third value
 2171 | /// \returns The extend vectorized subtraction of the two values with saturation
 2172 | template <typename RetT, typename AT, typename BT>
 2173 | inline constexpr RetT extend_vsub4_sat(AT a, BT b, RetT c) {
 2174 |   return detail::extend_vbinary4<RetT, true, false>(a, b, c, std::minus());
 2175 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 2177-2189

```text
 2177 | /// Compute vectorized abs_diff of \p a and \p b, with each value treated as a 4
 2178 | /// elements vector type and extend each element to 9 bit.
 2179 | /// \tparam [in] RetT The type of the return value, can only be 32 bit integer
 2180 | /// \tparam [in] AT The type of the first value, can only be 32 bit integer
 2181 | /// \tparam [in] BT The type of the second value, can only be 32 bit integer
 2182 | /// \param [in] a The first value
 2183 | /// \param [in] b The second value
 2184 | /// \param [in] c The third value
 2185 | /// \returns The extend vectorized abs_diff of the two values
 2186 | template <typename RetT, typename AT, typename BT>
 2187 | inline constexpr RetT extend_vabsdiff4(AT a, BT b, RetT c) {
 2188 |   return detail::extend_vbinary4<RetT, false, false>(a, b, c, abs_diff());
 2189 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 2191-2205

```text
 2191 | /// Compute vectorized abs_diff of \p a and \p b, with each value treated as a 4
 2192 | /// elements vector type and extend each element to 9 bit. Then add each half
 2193 | /// of the result and add with \p c.
 2194 | /// \tparam [in] RetT The type of the return value, can only be 32 bit integer
 2195 | /// \tparam [in] AT The type of the first value, can only be 32 bit integer
 2196 | /// \tparam [in] BT The type of the second value, can only be 32 bit integer
 2197 | /// \param [in] a The first value
 2198 | /// \param [in] b The second value
 2199 | /// \param [in] c The third value
 2200 | /// \returns The addition of each half of extend vectorized abs_diff of the
 2201 | /// two values and the third value
 2202 | template <typename RetT, typename AT, typename BT>
 2203 | inline constexpr RetT extend_vabsdiff4_add(AT a, BT b, RetT c) {
 2204 |   return detail::extend_vbinary4<RetT, false, true>(a, b, c, abs_diff());
 2205 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 2207-2219

```text
 2207 | /// Compute vectorized abs_diff of \p a and \p b with saturation, with each
 2208 | /// value treated as a 4 elements vector type and extend each element to 9 bit.
 2209 | /// \tparam [in] RetT The type of the return value, can only be 32 bit integer
 2210 | /// \tparam [in] AT The type of the first value, can only be 32 bit integer
 2211 | /// \tparam [in] BT The type of the second value, can only be 32 bit integer
 2212 | /// \param [in] a The first value
 2213 | /// \param [in] b The second value
 2214 | /// \param [in] c The third value
 2215 | /// \returns The extend vectorized abs_diff of the two values with saturation
 2216 | template <typename RetT, typename AT, typename BT>
 2217 | inline constexpr RetT extend_vabsdiff4_sat(AT a, BT b, RetT c) {
 2218 |   return detail::extend_vbinary4<RetT, true, false>(a, b, c, abs_diff());
 2219 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 2221-2233

```text
 2221 | /// Compute vectorized minimum of \p a and \p b, with each value treated as a 4
 2222 | /// elements vector type and extend each element to 9 bit.
 2223 | /// \tparam [in] RetT The type of the return value, can only be 32 bit integer
 2224 | /// \tparam [in] AT The type of the first value, can only be 32 bit integer
 2225 | /// \tparam [in] BT The type of the second value, can only be 32 bit integer
 2226 | /// \param [in] a The first value
 2227 | /// \param [in] b The second value
 2228 | /// \param [in] c The third value
 2229 | /// \returns The extend vectorized minimum of the two values
 2230 | template <typename RetT, typename AT, typename BT>
 2231 | inline constexpr RetT extend_vmin4(AT a, BT b, RetT c) {
 2232 |   return detail::extend_vbinary4<RetT, false, false>(a, b, c, minimum());
 2233 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 2235-2249

```text
 2235 | /// Compute vectorized minimum of \p a and \p b, with each value treated as a 4
 2236 | /// elements vector type and extend each element to 9 bit. Then add each half
 2237 | /// of the result and add with \p c.
 2238 | /// \tparam [in] RetT The type of the return value, can only be 32 bit integer
 2239 | /// \tparam [in] AT The type of the first value, can only be 32 bit integer
 2240 | /// \tparam [in] BT The type of the second value, can only be 32 bit integer
 2241 | /// \param [in] a The first value
 2242 | /// \param [in] b The second value
 2243 | /// \param [in] c The third value
 2244 | /// \returns The addition of each half of extend vectorized minimum of the
 2245 | /// two values and the third value
 2246 | template <typename RetT, typename AT, typename BT>
 2247 | inline constexpr RetT extend_vmin4_add(AT a, BT b, RetT c) {
 2248 |   return detail::extend_vbinary4<RetT, false, true>(a, b, c, minimum());
 2249 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 2251-2263

```text
 2251 | /// Compute vectorized minimum of \p a and \p b with saturation, with each value
 2252 | /// treated as a 4 elements vector type and extend each element to 9 bit.
 2253 | /// \tparam [in] RetT The type of the return value, can only be 32 bit integer
 2254 | /// \tparam [in] AT The type of the first value, can only be 32 bit integer
 2255 | /// \tparam [in] BT The type of the second value, can only be 32 bit integer
 2256 | /// \param [in] a The first value
 2257 | /// \param [in] b The second value
 2258 | /// \param [in] c The third value
 2259 | /// \returns The extend vectorized minimum of the two values with saturation
 2260 | template <typename RetT, typename AT, typename BT>
 2261 | inline constexpr RetT extend_vmin4_sat(AT a, BT b, RetT c) {
 2262 |   return detail::extend_vbinary4<RetT, true, false>(a, b, c, minimum());
 2263 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 2265-2277

```text
 2265 | /// Compute vectorized maximum of \p a and \p b, with each value treated as a 4
 2266 | /// elements vector type and extend each element to 9 bit.
 2267 | /// \tparam [in] RetT The type of the return value, can only be 32 bit integer
 2268 | /// \tparam [in] AT The type of the first value, can only be 32 bit integer
 2269 | /// \tparam [in] BT The type of the second value, can only be 32 bit integer
 2270 | /// \param [in] a The first value
 2271 | /// \param [in] b The second value
 2272 | /// \param [in] c The third value
 2273 | /// \returns The extend vectorized maximum of the two values
 2274 | template <typename RetT, typename AT, typename BT>
 2275 | inline constexpr RetT extend_vmax4(AT a, BT b, RetT c) {
 2276 |   return detail::extend_vbinary4<RetT, false, false>(a, b, c, maximum());
 2277 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 2279-2293

```text
 2279 | /// Compute vectorized maximum of \p a and \p b, with each value treated as a 4
 2280 | /// elements vector type and extend each element to 9 bit. Then add each half
 2281 | /// of the result and add with \p c.
 2282 | /// \tparam [in] RetT The type of the return value, can only be 32 bit integer
 2283 | /// \tparam [in] AT The type of the first value, can only be 32 bit integer
 2284 | /// \tparam [in] BT The type of the second value, can only be 32 bit integer
 2285 | /// \param [in] a The first value
 2286 | /// \param [in] b The second value
 2287 | /// \param [in] c The third value
 2288 | /// \returns The addition of each half of extend vectorized maximum of the
 2289 | /// two values and the third value
 2290 | template <typename RetT, typename AT, typename BT>
 2291 | inline constexpr RetT extend_vmax4_add(AT a, BT b, RetT c) {
 2292 |   return detail::extend_vbinary4<RetT, false, true>(a, b, c, maximum());
 2293 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 2295-2307

```text
 2295 | /// Compute vectorized maximum of \p a and \p b with saturation, with each value
 2296 | /// treated as a 4 elements vector type and extend each element to 9 bit.
 2297 | /// \tparam [in] RetT The type of the return value, can only be 32 bit integer
 2298 | /// \tparam [in] AT The type of the first value, can only be 32 bit integer
 2299 | /// \tparam [in] BT The type of the second value, can only be 32 bit integer
 2300 | /// \param [in] a The first value
 2301 | /// \param [in] b The second value
 2302 | /// \param [in] c The third value
 2303 | /// \returns The extend vectorized maximum of the two values with saturation
 2304 | template <typename RetT, typename AT, typename BT>
 2305 | inline constexpr RetT extend_vmax4_sat(AT a, BT b, RetT c) {
 2306 |   return detail::extend_vbinary4<RetT, true, false>(a, b, c, maximum());
 2307 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 2309-2322

```text
 2309 | /// Compute vectorized average of \p a and \p b, with each value treated as a 4
 2310 | /// elements vector type and extend each element to 9 bit.
 2311 | /// \tparam [in] RetT The type of the return value, can only be 32 bit integer
 2312 | /// \tparam [in] AT The type of the first value, can only be 32 bit integer
 2313 | /// \tparam [in] BT The type of the second value, can only be 32 bit integer
 2314 | /// \param [in] a The first value
 2315 | /// \param [in] b The second value
 2316 | /// \param [in] c The third value
 2317 | /// \returns The extend vectorized average of the two values
 2318 | template <typename RetT, typename AT, typename BT>
 2319 | inline constexpr RetT extend_vavrg4(AT a, BT b, RetT c) {
 2320 |   return detail::extend_vbinary4<RetT, false, false>(a, b, c,
 2321 |                                                      detail::average());
 2322 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 2324-2338

```text
 2324 | /// Compute vectorized average of \p a and \p b, with each value treated as a 4
 2325 | /// elements vector type and extend each element to 9 bit. Then add each half
 2326 | /// of the result and add with \p c.
 2327 | /// \tparam [in] RetT The type of the return value, can only be 32 bit integer
 2328 | /// \tparam [in] AT The type of the first value, can only be 32 bit integer
 2329 | /// \tparam [in] BT The type of the second value, can only be 32 bit integer
 2330 | /// \param [in] a The first value
 2331 | /// \param [in] b The second value
 2332 | /// \param [in] c The third value
 2333 | /// \returns The addition of each half of extend vectorized average of the
 2334 | /// two values and the third value
 2335 | template <typename RetT, typename AT, typename BT>
 2336 | inline constexpr RetT extend_vavrg4_add(AT a, BT b, RetT c) {
 2337 |   return detail::extend_vbinary4<RetT, false, true>(a, b, c, detail::average());
 2338 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 2340-2352

```text
 2340 | /// Compute vectorized average of \p a and \p b with saturation, with each value
 2341 | /// treated as a 4 elements vector type and extend each element to 9 bit.
 2342 | /// \tparam [in] RetT The type of the return value, can only be 32 bit integer
 2343 | /// \tparam [in] AT The type of the first value, can only be 32 bit integer
 2344 | /// \tparam [in] BT The type of the second value, can only be 32 bit integer
 2345 | /// \param [in] a The first value
 2346 | /// \param [in] b The second value
 2347 | /// \param [in] c The third value
 2348 | /// \returns The extend vectorized average of the two values with saturation
 2349 | template <typename RetT, typename AT, typename BT>
 2350 | inline constexpr RetT extend_vavrg4_sat(AT a, BT b, RetT c) {
 2351 |   return detail::extend_vbinary4<RetT, true, false>(a, b, c, detail::average());
 2352 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 2354-2367

```text
 2354 | /// Extend \p a and \p b to 33 bit and vectorized compare input values using
 2355 | /// specified comparison \p cmp .
 2356 | ///
 2357 | /// \tparam [in] AT The type of the first value, can only be 32 bit integer
 2358 | /// \tparam [in] BT The type of the second value, can only be 32 bit integer
 2359 | /// \tparam [in] BinaryOperation The type of the compare operation
 2360 | /// \param [in] a The first value
 2361 | /// \param [in] b The second value
 2362 | /// \param [in] cmp The comparsion operator
 2363 | /// \returns The comparison result of the two extended values.
 2364 | template <typename AT, typename BT, typename BinaryOperation>
 2365 | inline constexpr unsigned extend_vcompare4(AT a, BT b, BinaryOperation cmp) {
 2366 |   return detail::extend_vbinary4<unsigned, false, false>(a, b, 0, cmp);
 2367 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 2369-2387

```text
 2369 | /// Extend Inputs to 33 bit, and vectorized compare input values using specified
 2370 | /// comparison \p cmp , then add the result with \p c .
 2371 | ///
 2372 | /// \tparam [in] AT The type of the first value, can only be 32 bit integer
 2373 | /// \tparam [in] BT The type of the second value, can only be 32 bit integer
 2374 | /// \tparam [in] BinaryOperation The type of the compare operation
 2375 | /// \param [in] a The first value
 2376 | /// \param [in] b The second value
 2377 | /// \param [in] c The third value
 2378 | /// \param [in] cmp The comparsion operator
 2379 | /// \returns The comparison result of the two extended values, and add the
 2380 | /// result with \p c .
 2381 | template <typename AT, typename BT, typename BinaryOperation>
 2382 | inline constexpr unsigned extend_vcompare4_add(AT a, BT b, unsigned c,
 2383 |                                                BinaryOperation cmp) {
 2384 |   return detail::extend_vbinary4<unsigned, false, true>(a, b, c, cmp);
 2385 | }
 2387 | } // namespace compat
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

## Key Concepts / 关键概念

- Compile-time numeric metaprogramming / 编译期数值元编程
- Static containers and tuple-like storage / 静态容器与类元组存储
- Backend portability and SYCL-style compatibility / 后端可移植性与 SYCL 风格兼容层
- SYCL interoperability / SYCL 互操作
- Header-only template specialization patterns / 纯头文件模板特化模式

## Dependencies / 依赖关系

- Direct includes / 直接包含:
  - `limits`
  - `sycl/feature_test.hpp`
  - `type_traits`
  - `sycl/ext/oneapi/experimental/bfloat16_math.hpp`
  - `sycl/ext/oneapi/experimental/complex/complex.hpp`
  - `cute/util/compat/traits.hpp`
- Primary symbols / 主要符号: `abs`, `abs_diff`, `add_sat`, `rhadd`, `hadd`, `maximum`, `minimum`, `sub_sat`
- Dependency role / 依赖角色: Sits between CuTe algorithms and backend APIs, normalizing device, kernel, launch, memory, and math behavior across supported targets. / 位于 CuTe 算法与后端 API 之间，统一不同目标上的设备、内核、启动、内存和数学行为。
