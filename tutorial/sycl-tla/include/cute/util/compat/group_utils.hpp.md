# group_utils.hpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `include/cute/util/compat/group_utils.hpp`
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
   18 |  *  group_utils.hpp
   19 |  *
   20 |  *  Description:
   21 |  *    Group util functionality for the SYCL compatibility extension
```
**EN:** Provides the license notice, copyright ownership, and redistribution terms for this header.
**CN:** 给出该头文件的许可证声明、版权归属以及再分发条款。

### Lines 22-34

```text
   22 |  **************************************************************************/
   24 | // The original source was under the license below:
   25 | //==---- group_utils.hpp ------------------*- C++ -*--------------------==//
   26 | //
   27 | // Copyright (C) Intel Corporation
   28 | // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   29 | // See https://llvm.org/LICENSE.txt for license information.
   30 | //
   31 | //===------------------------------------------------------------------===//
   33 | #pragma once
   34 | #pragma GCC system_header
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 36-41

```text
   36 | #include <iterator>
   37 | #include <stdexcept>
   38 | #include <sycl/sycl.hpp>
   40 | #include <cute/util/compat/defs.hpp>
   41 | #include <cute/util/compat/math.hpp>
```
**EN:** Sets up the header dependencies for this file by importing `iterator`, `stdexcept`, `sycl/sycl.hpp`, `cute/util/compat/defs.hpp`, `cute/util/compat/math.hpp`.
**CN:** 通过引入 `iterator`, `stdexcept`, `sycl/sycl.hpp`, `cute/util/compat/defs.hpp`, `cute/util/compat/math.hpp` 为该文件建立头文件依赖。

### Lines 43-45

```text
   43 | namespace compat {
   44 | namespace group {
   45 | namespace detail {
```
**EN:** Opens or closes namespace scope so the following declarations remain grouped under the intended CuTe or backend namespace hierarchy.
**CN:** 打开或关闭命名空间作用域，使后续声明保持在预期的 CuTe 或后端命名空间层级中。

### Lines 47-50

```text
   47 | template <typename... _Args>
   48 | constexpr auto __reduce_over_group(_Args... __args) {
   49 |   return sycl::reduce_over_group(__args...);
   50 | }
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 52-54

```text
   52 | template <typename... _Args> constexpr auto __group_broadcast(_Args... __args) {
   53 |   return sycl::group_broadcast(__args...);
   54 | }
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 56-59

```text
   56 | template <typename... _Args>
   57 | constexpr auto __exclusive_scan_over_group(_Args... __args) {
   58 |   return sycl::exclusive_scan_over_group(__args...);
   59 | }
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 61-64

```text
   61 | template <typename... _Args>
   62 | constexpr auto __inclusive_scan_over_group(_Args... __args) {
   63 |   return sycl::inclusive_scan_over_group(__args...);
   64 | }
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 66-71

```text
   66 | template <typename Item, typename T, class BinaryOperation,
   67 |           class GroupPrefixCallbackOperation>
   68 | __compat_inline__ T
   69 | exclusive_scan(const Item &item, T input, BinaryOperation binary_op,
   70 |                GroupPrefixCallbackOperation &prefix_callback_op) {
   71 |   T group_aggregate;
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 73-77

```text
   73 |   T output =
   74 |       detail::__exclusive_scan_over_group(item.get_group(), input, binary_op);
   75 |   if (item.get_local_linear_id() == item.get_local_range().size() - 1) {
   76 |     group_aggregate = binary_op(output, input);
   77 |   }
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 79-80

```text
   79 |   group_aggregate = detail::__group_broadcast(
   80 |       item.get_group(), group_aggregate, item.get_local_range().size() - 1);
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 82-87

```text
   82 |   T group_prefix = prefix_callback_op(group_aggregate);
   83 |   if (item.get_local_linear_id() == 0) {
   84 |     output = group_prefix;
   85 |   } else {
   86 |     output = binary_op(group_prefix, output);
   87 |   }
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 89-90

```text
   89 |   return output;
   90 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数。

### Lines 92-93

```text
   92 | typedef uint16_t digit_counter_type;
   93 | typedef uint32_t packed_counter_type;
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。

### Lines 95-97

```text
   95 | template <int N, int CURRENT_VAL = N, int COUNT = 0> struct log2 {
   96 |   enum { VALUE = log2<N, (CURRENT_VAL >> 1), COUNT + 1>::VALUE };
   97 | };
```
**EN:** Defines `log2` as a reusable type-level building block in this header. It also leans heavily on compile-time specialization.
**CN:** 将 `log2` 定义为本头文件中的可复用类型级构件。 它还大量依赖编译期特化。

### Lines 99-101

```text
   99 | template <int N, int COUNT> struct log2<N, 0, COUNT> {
  100 |   enum { VALUE = (1 << (COUNT - 1) < N) ? COUNT : COUNT - 1 };
  101 | };
```
**EN:** Defines `log2` as a reusable type-level building block in this header. It also leans heavily on compile-time specialization.
**CN:** 将 `log2` 定义为本头文件中的可复用类型级构件。 它还大量依赖编译期特化。

### Lines 103-107

```text
  103 | template <int RADIX_BITS, bool DESCENDING = false> class radix_rank {
  104 | public:
  105 |   static size_t get_local_memory_size(size_t group_threads) {
  106 |     return group_threads * PADDED_COUNTER_LANES * sizeof(packed_counter_type);
  107 |   }
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 109-109

```text
  109 |   radix_rank(uint8_t *local_memory) : _local_memory(local_memory) {}
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 111-153

```text
  111 |   template <typename Item, int VALUES_PER_THREAD>
  112 |   __compat_inline__ void
  113 |   rank_keys(const Item &item, uint32_t (&keys)[VALUES_PER_THREAD],
  114 |             int (&ranks)[VALUES_PER_THREAD], int current_bit, int num_bits) {
  115 | 
  116 |     digit_counter_type thread_prefixes[VALUES_PER_THREAD];
  117 |     digit_counter_type *digit_counters[VALUES_PER_THREAD];
  118 |     digit_counter_type *buffer =
  119 |         reinterpret_cast<digit_counter_type *>(_local_memory);
  120 |     auto g = item.get_group();
  121 |     reset_local_memory(item);
  122 | 
  123 |     sycl::group_barrier(g, sycl::memory_scope::work_group);
  124 | 
  125 | #pragma unroll
  126 |     for (int i = 0; i < VALUES_PER_THREAD; ++i) {
  127 |       uint32_t digit =
  128 |           ::compat::detail::bfe(keys[i], current_bit, num_bits);
  129 |       uint32_t sub_counter = digit >> LOG_COUNTER_LANES;
  130 |       uint32_t counter_lane = digit & (COUNTER_LANES - 1);
  131 | 
  132 |       if (DESCENDING) {
  133 |         sub_counter = PACKING_RATIO - 1 - sub_counter;
  134 |         counter_lane = COUNTER_LANES - 1 - counter_lane;
  135 |       }
  136 | 
  137 |       digit_counters[i] =
  138 |           &buffer[counter_lane * item.get_local_range().size() * PACKING_RATIO +
  139 |                   item.get_local_linear_id() * PACKING_RATIO + sub_counter];
  140 |       thread_prefixes[i] = *digit_counters[i];
  141 |       *digit_counters[i] = thread_prefixes[i] + 1;
  142 |     }
  143 | 
  144 |     sycl::group_barrier(g, sycl::memory_scope::work_group);
  145 | 
  146 |     scan_counters(item);
  147 | 
  148 |     sycl::group_barrier(g, sycl::memory_scope::work_group);
  149 | 
  150 |     for (int i = 0; i < VALUES_PER_THREAD; ++i) {
  151 |       ranks[i] = thread_prefixes[i] + *digit_counters[i];
  152 |     }
  153 |   }
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 155-165

```text
  155 | private:
  156 |   template <typename Item>
  157 |   __compat_inline__ void reset_local_memory(const Item &item) {
  158 |     packed_counter_type *ptr =
  159 |         reinterpret_cast<packed_counter_type *>(_local_memory);
  160 | 
  161 | #pragma unroll
  162 |     for (int i = 0; i < PADDED_COUNTER_LANES; ++i) {
  163 |       ptr[i * item.get_local_range().size() + item.get_local_linear_id()] = 0;
  164 |     }
  165 |   }
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 167-185

```text
  167 |   template <typename Item>
  168 |   __compat_inline__ packed_counter_type upsweep(const Item &item) {
  169 |     packed_counter_type sum = 0;
  170 |     packed_counter_type *ptr =
  171 |         reinterpret_cast<packed_counter_type *>(_local_memory);
  172 | 
  173 | #pragma unroll
  174 |     for (int i = 0; i < PADDED_COUNTER_LANES; i++) {
  175 |       cached_segment[i] =
  176 |           ptr[item.get_local_linear_id() * PADDED_COUNTER_LANES + i];
  177 |     }
  178 | 
  179 | #pragma unroll
  180 |     for (int i = 0; i < PADDED_COUNTER_LANES; ++i) {
  181 |       sum += cached_segment[i];
  182 |     }
  183 | 
  184 |     return sum;
  185 |   }
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 187-206

```text
  187 |   template <typename Item>
  188 |   __compat_inline__ void
  189 |   exclusive_downsweep(const Item &item, packed_counter_type raking_partial) {
  190 |     packed_counter_type *ptr =
  191 |         reinterpret_cast<packed_counter_type *>(_local_memory);
  192 |     packed_counter_type sum = raking_partial;
  193 | 
  194 | #pragma unroll
  195 |     for (int i = 0; i < PADDED_COUNTER_LANES; ++i) {
  196 |       packed_counter_type value = cached_segment[i];
  197 |       cached_segment[i] = sum;
  198 |       sum += value;
  199 |     }
  200 | 
  201 | #pragma unroll
  202 |     for (int i = 0; i < PADDED_COUNTER_LANES; ++i) {
  203 |       ptr[item.get_local_linear_id() * PADDED_COUNTER_LANES + i] =
  204 |           cached_segment[i];
  205 |     }
  206 |   }
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 208-221

```text
  208 |   struct prefix_callback {
  209 |     __compat_inline__ packed_counter_type
  210 |     operator()(packed_counter_type block_aggregate) {
  211 |       packed_counter_type block_prefix = 0;
  212 | 
  213 | #pragma unroll
  214 |       for (int packed = 1; packed < PACKING_RATIO; packed++) {
  215 |         block_prefix += block_aggregate
  216 |                         << (sizeof(digit_counter_type) * 8 * packed);
  217 |       }
  218 | 
  219 |       return block_prefix;
  220 |     }
  221 |   };
```
**EN:** Defines `prefix_callback` as a reusable type-level building block in this header. It also implements coordinate mapping or slicing behavior and acts as a factory/helper for constructing derived objects.
**CN:** 将 `prefix_callback` 定义为本头文件中的可复用类型级构件。 它还实现坐标映射或切片行为并充当构造派生对象的工厂/辅助函数。

### Lines 223-233

```text
  223 |   template <typename Item>
  224 |   __compat_inline__ void scan_counters(const Item &item) {
  225 |     packed_counter_type raking_partial = upsweep(item);
  226 | 
  227 |     prefix_callback callback;
  228 |     packed_counter_type exclusive_partial = exclusive_scan(
  229 |         item, raking_partial, sycl::ext::oneapi::plus<packed_counter_type>(),
  230 |         callback);
  231 | 
  232 |     exclusive_downsweep(item, exclusive_partial);
  233 |   }
```
**EN:** Implements `scan_counters`, a helper routine used by the surrounding CuTe abstractions. It also leans heavily on compile-time specialization and bridges to SYCL execution or group abstractions.
**CN:** 实现 `scan_counters`，这是周围 CuTe 抽象所使用的辅助例程。 它还大量依赖编译期特化并桥接到 SYCL 执行模型或 group 抽象。

### Lines 235-241

```text
  235 | private:
  236 |   static constexpr int PACKING_RATIO =
  237 |       sizeof(packed_counter_type) / sizeof(digit_counter_type);
  238 |   static constexpr int LOG_PACKING_RATIO = log2<PACKING_RATIO>::VALUE;
  239 |   static constexpr int LOG_COUNTER_LANES = RADIX_BITS - LOG_PACKING_RATIO;
  240 |   static constexpr int COUNTER_LANES = 1 << LOG_COUNTER_LANES;
  241 |   static constexpr int PADDED_COUNTER_LANES = COUNTER_LANES + 1;
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还大量依赖编译期特化。

### Lines 243-245

```text
  243 |   packed_counter_type cached_segment[PADDED_COUNTER_LANES];
  244 |   uint8_t *_local_memory;
  245 | };
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 247-247

```text
  247 | template <typename T, typename U> struct base_traits {
```
**EN:** Defines `base_traits` as a reusable type-level building block in this header. It also leans heavily on compile-time specialization.
**CN:** 将 `base_traits` 定义为本头文件中的可复用类型级构件。 它还大量依赖编译期特化。

### Lines 249-255

```text
  249 |   static __compat_inline__ U twiddle_in(U key) {
  250 |     throw std::runtime_error("Not implemented");
  251 |   }
  252 |   static __compat_inline__ U twiddle_out(U key) {
  253 |     throw std::runtime_error("Not implemented");
  254 |   }
  255 | };
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。

### Lines 257-260

```text
  257 | template <typename U> struct base_traits<uint32_t, U> {
  258 |   static __compat_inline__ U twiddle_in(U key) { return key; }
  259 |   static __compat_inline__ U twiddle_out(U key) { return key; }
  260 | };
```
**EN:** Defines `base_traits` as a reusable type-level building block in this header. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 将 `base_traits` 定义为本头文件中的可复用类型级构件。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 262-266

```text
  262 | template <typename U> struct base_traits<int, U> {
  263 |   static constexpr U HIGH_BIT = U(1) << ((sizeof(U) * 8) - 1);
  264 |   static __compat_inline__ U twiddle_in(U key) { return key ^ HIGH_BIT; }
  265 |   static __compat_inline__ U twiddle_out(U key) { return key ^ HIGH_BIT; }
  266 | };
```
**EN:** Defines `base_traits` as a reusable type-level building block in this header. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 将 `base_traits` 定义为本头文件中的可复用类型级构件。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 268-278

```text
  268 | template <typename U> struct base_traits<float, U> {
  269 |   static constexpr U HIGH_BIT = U(1) << ((sizeof(U) * 8) - 1);
  270 |   static __compat_inline__ U twiddle_in(U key) {
  271 |     U mask = (key & HIGH_BIT) ? U(-1) : HIGH_BIT;
  272 |     return key ^ mask;
  273 |   }
  274 |   static __compat_inline__ U twiddle_out(U key) {
  275 |     U mask = (key & HIGH_BIT) ? HIGH_BIT : U(-1);
  276 |     return key ^ mask;
  277 |   }
  278 | };
```
**EN:** Defines `base_traits` as a reusable type-level building block in this header. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 将 `base_traits` 定义为本头文件中的可复用类型级构件。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 280-283

```text
  280 | template <typename T> struct traits : base_traits<T, T> {};
  281 | template <> struct traits<uint32_t> : base_traits<uint32_t, uint32_t> {};
  282 | template <> struct traits<int> : base_traits<int, uint32_t> {};
  283 | template <> struct traits<float> : base_traits<float, uint32_t> {};
```
**EN:** Defines `traits` as a reusable type-level building block in this header. It also leans heavily on compile-time specialization.
**CN:** 将 `traits` 定义为本头文件中的可复用类型级构件。 它还大量依赖编译期特化。

### Lines 285-287

```text
  285 | template <int N> struct power_of_two {
  286 |   enum { VALUE = ((N & (N - 1)) == 0) };
  287 | };
```
**EN:** Defines `power_of_two` as a reusable type-level building block in this header. It also leans heavily on compile-time specialization.
**CN:** 将 `power_of_two` 定义为本头文件中的可复用类型级构件。 它还大量依赖编译期特化。

### Lines 289-294

```text
  289 | __compat_inline__ uint32_t shr_add(uint32_t x, uint32_t shift,
  290 |                                        uint32_t addend) {
  291 |   return (x >> shift) + addend;
  292 | }
  294 | } // namespace detail
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数。

### Lines 296-309

```text
  296 | /// Rearranging data partitioned across a work-group.
  297 | ///
  298 | /// \tparam T The type of the data elements.
  299 | /// \tparam ElementsPerWorkItem The number of data elements assigned to a
  300 | /// work-item.
  301 | template <typename T, size_t ElementsPerWorkItem> class exchange {
  302 | public:
  303 |   static size_t get_local_memory_size(size_t group_threads) {
  304 |     size_t padding_values =
  305 |         (INSERT_PADDING)
  306 |             ? ((group_threads * ElementsPerWorkItem) >> LOG_LOCAL_MEMORY_BANKS)
  307 |             : 0;
  308 |     return (group_threads * ElementsPerWorkItem + padding_values) * sizeof(T);
  309 |   }
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 311-311

```text
  311 |   exchange(uint8_t *local_memory) : _local_memory(local_memory) {}
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 313-321

```text
  313 |   // TODO: Investigate if padding is required for performance,
  314 |   // and if specializations are required for specific target hardware.
  315 |   static size_t adjust_by_padding(size_t offset) {
  316 | 
  317 |     if constexpr (INSERT_PADDING) {
  318 |       offset = detail::shr_add(offset, LOG_LOCAL_MEMORY_BANKS, offset);
  319 |     }
  320 |     return offset;
  321 |   }
```
**EN:** Implements `adjust_by_padding`, a helper routine used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 实现 `adjust_by_padding`，这是周围 CuTe 抽象所使用的辅助例程。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 323-328

```text
  323 |   struct blocked_offset {
  324 |     template <typename Item> size_t operator()(Item item, size_t i) {
  325 |       size_t offset = item.get_local_linear_id() * ElementsPerWorkItem + i;
  326 |       return adjust_by_padding(offset);
  327 |     }
  328 |   };
```
**EN:** Defines `blocked_offset` as a reusable type-level building block in this header. It also implements coordinate mapping or slicing behavior and acts as a factory/helper for constructing derived objects.
**CN:** 将 `blocked_offset` 定义为本头文件中的可复用类型级构件。 它还实现坐标映射或切片行为并充当构造派生对象的工厂/辅助函数。

### Lines 330-337

```text
  330 |   struct striped_offset {
  331 |     template <typename Item> size_t operator()(Item item, size_t i) {
  332 |       size_t offset = i * item.get_local_range(2) * item.get_local_range(1) *
  333 |                           item.get_local_range(0) +
  334 |                       item.get_local_linear_id();
  335 |       return adjust_by_padding(offset);
  336 |     }
  337 |   };
```
**EN:** Defines `striped_offset` as a reusable type-level building block in this header. It also implements coordinate mapping or slicing behavior and acts as a factory/helper for constructing derived objects.
**CN:** 将 `striped_offset` 定义为本头文件中的可复用类型级构件。 它还实现坐标映射或切片行为并充当构造派生对象的工厂/辅助函数。

### Lines 339-348

```text
  339 |   template <typename Iterator> struct scatter_offset {
  340 |     Iterator begin;
  341 |     scatter_offset(const int (&ranks)[ElementsPerWorkItem]) {
  342 |       begin = std::begin(ranks);
  343 |     }
  344 |     template <typename Item> size_t operator()(Item item, size_t i) const {
  345 |       // iterator i is expected to be within bounds [0,VALUES_PER_THREAD)
  346 |       return adjust_by_padding(begin[i]);
  347 |     }
  348 |   };
```
**EN:** Defines `scatter_offset` as a reusable type-level building block in this header. It also implements coordinate mapping or slicing behavior and acts as a factory/helper for constructing derived objects.
**CN:** 将 `scatter_offset` 定义为本头文件中的可复用类型级构件。 它还实现坐标映射或切片行为并充当构造派生对象的工厂/辅助函数。

### Lines 350-371

```text
  350 |   /// Inplace rearrange elements from blocked order to striped order.
  351 |   ///
  352 |   /// Suppose 512 integer data elements partitioned across 128 work-items, where
  353 |   /// each work-item owns 4 ( \p ElementsPerWorkItem ) data elements and the
  354 |   /// blocked \p input across the work-group is:
  355 |   ///
  356 |   ///   {[0, 1, 2, 3], [4, 5, 6, 7], ..., [508, 509, 510, 511]}.
  357 |   ///
  358 |   /// The striped order output is:
  359 |   ///
  360 |   ///   {[0, 128, 256, 384], [1, 129, 257, 385], ..., [127, 255, 383, 511]}.
  361 |   ///
  362 |   /// \tparam Item The work-item identifier type.
  363 |   /// \param item The work-item identifier.
  364 |   /// \param input The input data of each work-item.
  365 |   template <typename Item>
  366 |   __compat_inline__ void
  367 |   blocked_to_striped(Item item, T (&input)[ElementsPerWorkItem]) {
  368 |     striped_offset get_striped_offset;
  369 |     blocked_offset get_blocked_offset;
  370 |     helper_exchange(item, input, input, get_blocked_offset, get_striped_offset);
  371 |   }
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 373-394

```text
  373 |   /// Inplace rearrange elements from striped order to blocked order.
  374 |   ///
  375 |   /// Suppose 512 integer data elements partitioned across 128 work-items, where
  376 |   /// each work-item owns 4 ( \p ElementsPerWorkItem ) data elements and the
  377 |   /// striped \p input across the work-group is:
  378 |   ///
  379 |   ///   { [0, 128, 256, 384], [1, 129, 257, 385], ..., [127, 255, 383, 511] }.
  380 |   ///
  381 |   /// The blocked order output is:
  382 |   ///
  383 |   ///   { [0, 1, 2, 3], [4, 5, 6, 7], ..., [508, 509, 510, 511] }.
  384 |   ///
  385 |   /// \tparam Item The work-item identifier type.
  386 |   /// \param item The work-item identifier.
  387 |   /// \param input The input data of each work-item.
  388 |   template <typename Item>
  389 |   __compat_inline__ void
  390 |   striped_to_blocked(Item item, T (&input)[ElementsPerWorkItem]) {
  391 |     blocked_offset get_blocked_offset;
  392 |     striped_offset get_striped_offset;
  393 |     helper_exchange(item, input, input, get_striped_offset, get_blocked_offset);
  394 |   }
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 396-420

```text
  396 |   /// Rearrange elements from blocked order to striped order.
  397 |   ///
  398 |   /// Suppose 512 integer data elements partitioned across 128 work-items, where
  399 |   /// each work-item owns 4 ( \p ElementsPerWorkItem ) data elements and the
  400 |   /// blocked \p input across the work-group is:
  401 |   ///
  402 |   ///   { [0, 1, 2, 3], [4, 5, 6, 7], ..., [508, 509, 510, 511] }.
  403 |   ///
  404 |   /// The striped order output is:
  405 |   ///
  406 |   ///   { [0, 128, 256, 384], [1, 129, 257, 385], ..., [127, 255, 383, 511] }.
  407 |   ///
  408 |   /// \tparam Item The work-item identifier type.
  409 |   /// \param item The work-item identifier.
  410 |   /// \param input The input data of each work-item.
  411 |   /// \param output The corresponding output data of each work-item.
  412 |   template <typename Item>
  413 |   __compat_inline__ void
  414 |   blocked_to_striped(Item item, T (&input)[ElementsPerWorkItem],
  415 |                      T (&output)[ElementsPerWorkItem]) {
  416 |     striped_offset get_striped_offset;
  417 |     blocked_offset get_blocked_offset;
  418 |     helper_exchange(item, input, output, get_blocked_offset,
  419 |                     get_striped_offset);
  420 |   }
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 422-446

```text
  422 |   /// Rearrange elements from striped order to blocked order.
  423 |   ///
  424 |   /// Suppose 512 integer data elements partitioned across 128 work-items, where
  425 |   /// each work-item owns 4 ( \p ElementsPerWorkItem ) data elements and the
  426 |   /// striped \p input across the work-group is:
  427 |   ///
  428 |   ///   { [0, 128, 256, 384], [1, 129, 257, 385], ..., [127, 255, 383, 511] }.
  429 |   ///
  430 |   /// The blocked order output is:
  431 |   ///
  432 |   ///   { [0, 1, 2, 3], [4, 5, 6, 7], ..., [508, 509, 510, 511] }.
  433 |   ///
  434 |   /// \tparam Item The work-item identifier type.
  435 |   /// \param item The work-item identifier.
  436 |   /// \param input The input data of each work-item.
  437 |   /// \param output The corresponding output data of each work-item.
  438 |   template <typename Item>
  439 |   __compat_inline__ void
  440 |   striped_to_blocked(Item item, T (&input)[ElementsPerWorkItem],
  441 |                      T (&output)[ElementsPerWorkItem]) {
  442 |     blocked_offset get_blocked_offset;
  443 |     striped_offset get_striped_offset;
  444 |     helper_exchange(item, input, output, get_striped_offset,
  445 |                     get_blocked_offset);
  446 |   }
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 448-475

```text
  448 |   /// Inplace exchanges data items annotated by rank into blocked arrangement.
  449 |   ///
  450 |   /// Suppose 512 integer data elements partitioned across 128 work-items, where
  451 |   /// each work-item owns 4 ( \p ElementsPerWorkItem ) data elements and the
  452 |   /// striped \p input across the work-group is:
  453 |   ///
  454 |   ///   { [0, 128, 256, 384], [1, 129, 257, 385], ..., [127, 255, 383, 511] }.
  455 |   ///
  456 |   /// The rank across the work-group is:
  457 |   ///
  458 |   ///   { [0, 1, 2, 3], [4, 5, 6, 7], ..., [508, 509, 510, 511] }.
  459 |   ///
  460 |   /// The blocked order output is:
  461 |   ///
  462 |   ///   { [0, 1, 2, 3], [4, 5, 6, 7], ..., [508, 509, 510, 511] }.
  463 |   ///
  464 |   /// \tparam Item The work-item identifier type.
  465 |   /// \param item The work-item identifier.
  466 |   /// \param input The input data of each work-item.
  467 |   /// \param ranks The corresponding rank annotation of each work-item.
  468 |   template <typename Item>
  469 |   __compat_inline__ void
  470 |   scatter_to_blocked(Item item, T (&input)[ElementsPerWorkItem],
  471 |                      int (&ranks)[ElementsPerWorkItem]) {
  472 |     scatter_offset<const int *> get_scatter_offset(ranks);
  473 |     blocked_offset get_blocked_offset;
  474 |     helper_exchange(item, input, input, get_scatter_offset, get_blocked_offset);
  475 |   }
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 477-504

```text
  477 |   /// Inplace exchanges data items annotated by rank into striped arrangement.
  478 |   ///
  479 |   /// Suppose 512 integer data elements partitioned across 128 work-items, where
  480 |   /// each work-item owns 4 ( \p ElementsPerWorkItem ) data elements and the
  481 |   /// blocked \p input across the work-group is:
  482 |   ///
  483 |   ///   { [0, 1, 2, 3], [4, 5, 6, 7], ..., [508, 509, 510, 511] }.
  484 |   ///
  485 |   /// The rank across the work-group is:
  486 |   ///
  487 |   ///   { [16, 20, 24, 28], [32, 36, 40, 44], ..., [499, 503, 507, 511] }.
  488 |   ///
  489 |   /// The striped order output of each work-item will be:
  490 |   ///
  491 |   ///   { [0, 128, 256, 384], [1, 129, 257, 385], ..., [127, 255, 383, 511] }.
  492 |   ///
  493 |   /// \tparam Item The work-item identifier type.
  494 |   /// \param item The work-item identifier.
  495 |   /// \param input The input data of each work-item.
  496 |   /// \param ranks The corresponding rank annotation of each work-item.
  497 |   template <typename Item>
  498 |   __compat_inline__ void
  499 |   scatter_to_striped(Item item, T (&input)[ElementsPerWorkItem],
  500 |                      int (&ranks)[ElementsPerWorkItem]) {
  501 |     scatter_offset<const int *> get_scatter_offset(ranks);
  502 |     striped_offset get_striped_offset;
  503 |     helper_exchange(item, input, input, get_scatter_offset, get_striped_offset);
  504 |   }
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 506-526

```text
  506 | private:
  507 |   template <typename Item, typename offsetFunctorTypeFW,
  508 |             typename offsetFunctorTypeRV>
  509 |   __compat_inline__ void
  510 |   helper_exchange(Item item, T (&input)[ElementsPerWorkItem],
  511 |                   T (&output)[ElementsPerWorkItem],
  512 |                   offsetFunctorTypeFW &offset_functor_fw,
  513 |                   offsetFunctorTypeRV &offset_functor_rv) {
  514 |     T *buffer = reinterpret_cast<T *>(_local_memory);
  515 | #pragma unroll
  516 |     for (size_t i = 0; i < ElementsPerWorkItem; i++) {
  517 |       size_t offset = offset_functor_fw(item, i);
  518 |       buffer[offset] = input[i];
  519 |     }
  520 |     sycl::group_barrier(item.get_group());
  521 | #pragma unroll
  522 |     for (size_t i = 0; i < ElementsPerWorkItem; i++) {
  523 |       size_t offset = offset_functor_rv(item, i);
  524 |       output[i] = buffer[offset];
  525 |     }
  526 |   }
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 528-531

```text
  528 |   static constexpr int LOG_LOCAL_MEMORY_BANKS = 4;
  529 |   static constexpr bool INSERT_PADDING =
  530 |       (ElementsPerWorkItem > 4) &&
  531 |       (detail::power_of_two<ElementsPerWorkItem>::VALUE);
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还大量依赖编译期特化。

### Lines 533-534

```text
  533 |   uint8_t *_local_memory;
  534 | };
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 536-545

```text
  536 | /// The work-group wide radix sort to sort integer data elements
  537 | /// assigned to all work-items in the work-group.
  538 | ///
  539 | /// \tparam T The type of the data elements.
  540 | /// \tparam ElementsPerWorkItem The number of data elements assigned to
  541 | /// a work-item.
  542 | /// \tparam RADIX_BITS The number of radix bits per digit place.
  543 | template <typename T, int ElementsPerWorkItem, int RADIX_BITS = 4>
  544 | class group_radix_sort {
  545 |   uint8_t *_local_memory;
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 547-548

```text
  547 | public:
  548 |   group_radix_sort(uint8_t *local_memory) : _local_memory(local_memory) {}
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 550-556

```text
  550 |   static size_t get_local_memory_size(size_t group_threads) {
  551 |     size_t ranks_size =
  552 |         detail::radix_rank<RADIX_BITS>::get_local_memory_size(group_threads);
  553 |     size_t exchange_size =
  554 |         exchange<T, ElementsPerWorkItem>::get_local_memory_size(group_threads);
  555 |     return sycl::max(ranks_size, exchange_size);
  556 |   }
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 558-600

```text
  558 | private:
  559 |   template <typename Item, bool DESCENDING>
  560 |   __compat_inline__ void
  561 |   helper_sort(const Item &item, T (&keys)[ElementsPerWorkItem],
  562 |               int begin_bit = 0, int end_bit = 8 * sizeof(T),
  563 |               bool is_striped = false) {
  564 | 
  565 |     uint32_t(&unsigned_keys)[ElementsPerWorkItem] =
  566 |         reinterpret_cast<uint32_t(&)[ElementsPerWorkItem]>(keys);
  567 | 
  568 | #pragma unroll
  569 |     for (int i = 0; i < ElementsPerWorkItem; ++i) {
  570 |       unsigned_keys[i] = detail::traits<T>::twiddle_in(unsigned_keys[i]);
  571 |     }
  572 | 
  573 |     for (int i = begin_bit; i < end_bit; i += RADIX_BITS) {
  574 |       int pass_bits = sycl::min(RADIX_BITS, end_bit - begin_bit);
  575 | 
  576 |       int ranks[ElementsPerWorkItem];
  577 |       detail::radix_rank<RADIX_BITS, DESCENDING>(_local_memory)
  578 |           .template rank_keys<Item, ElementsPerWorkItem>(item, unsigned_keys,
  579 |                                                          ranks, i, pass_bits);
  580 | 
  581 |       sycl::group_barrier(item.get_group());
  582 | 
  583 |       bool last_iter = i + RADIX_BITS >= end_bit;
  584 |       if (last_iter && is_striped) {
  585 |         exchange<T, ElementsPerWorkItem>(_local_memory)
  586 |             .scatter_to_striped(item, keys, ranks);
  587 | 
  588 |       } else {
  589 |         exchange<T, ElementsPerWorkItem>(_local_memory)
  590 |             .scatter_to_blocked(item, keys, ranks);
  591 |       }
  592 | 
  593 |       sycl::group_barrier(item.get_group());
  594 |     }
  595 | 
  596 | #pragma unroll
  597 |     for (int i = 0; i < ElementsPerWorkItem; ++i) {
  598 |       unsigned_keys[i] = detail::traits<T>::twiddle_out(unsigned_keys[i]);
  599 |     }
  600 |   }
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 602-628

```text
  602 | public:
  603 |   /// Performs an ascending work-group wide radix sort over a blocked
  604 |   /// arrangement of input elements.
  605 |   ///
  606 |   /// Suppose 512 integer data elements partitioned across 128 work-items, where
  607 |   /// each work-item owns 4 ( \p ElementsPerWorkItem ) data elements and the
  608 |   /// \p input across the work-group is:
  609 |   ///
  610 |   ///   { [0,511,1,510], [2,509,3,508], [4,507,5,506], ..., [254,257,255,256] }.
  611 |   ///
  612 |   /// The ascending order output is:
  613 |   ///
  614 |   ///   { [0,1,2,3], [4,5,6,7], [8,9,10,11], ..., [508,509,510,511] }.
  615 |   ///
  616 |   /// \tparam Item The work-item identifier type.
  617 |   /// \param item The work-item identifier.
  618 |   /// \param input The input data of each work-item.
  619 |   /// \param begin_bit The beginning (least-significant) bit index needed for
  620 |   /// key comparison.
  621 |   /// \param end_bit The past-the-end (most-significant) bit
  622 |   /// index needed for key comparison.
  623 |   template <typename Item>
  624 |   __compat_inline__ void
  625 |   sort(const Item &item, T (&input)[ElementsPerWorkItem], int begin_bit = 0,
  626 |        int end_bit = 8 * sizeof(T)) {
  627 |     helper_sort<Item, /*DESCENDING=*/false>(item, input, begin_bit, end_bit);
  628 |   }
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 630-655

```text
  630 |   /// Performs an descending work-group wide radix sort over a blocked
  631 |   /// arrangement of input elements.
  632 |   ///
  633 |   /// Suppose 512 integer data elements partitioned across 128 work-items, where
  634 |   /// each work-item owns 4 ( \p ElementsPerWorkItem ) data elements and the
  635 |   /// \p input across the work-group is:
  636 |   ///
  637 |   ///   { [0,511,1,510], [2,509,3,508], [4,507,5,506], ..., [254,257,255,256] }.
  638 |   ///
  639 |   /// The descending order output is:
  640 |   ///
  641 |   ///   { [511,510,509,508], [11,10,9,8], [7,6,5,4], ..., [3,2,1,0] }.
  642 |   ///
  643 |   /// \tparam Item The work-item identifier type.
  644 |   /// \param item The work-item identifier.
  645 |   /// \param input The input data of each work-item.
  646 |   /// \param begin_bit The beginning (least-significant) bit index needed for
  647 |   /// key comparison.
  648 |   /// \param end_bit The past-the-end (most-significant) bit
  649 |   /// index needed for key comparison.
  650 |   template <typename Item>
  651 |   __compat_inline__ void
  652 |   sort_descending(const Item &item, T (&input)[ElementsPerWorkItem],
  653 |                   int begin_bit = 0, int end_bit = 8 * sizeof(T)) {
  654 |     helper_sort<Item, /*DESCENDING=*/true>(item, input, begin_bit, end_bit);
  655 |   }
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 657-684

```text
  657 |   /// Performs an ascending radix sort across a blocked arrangement of input
  658 |   /// elements, leaving them in a striped arrangement.
  659 |   ///
  660 |   /// Suppose 512 integer data elements partitioned across 128 work-items, where
  661 |   /// each work-item owns 4 ( \p ElementsPerWorkItem ) data elements and the
  662 |   /// \p input across the work-group is:
  663 |   ///
  664 |   ///   { [0,511,1,510], [2,509,3,508], [4,507,5,506], ..., [254,257,255,256] }.
  665 |   ///
  666 |   /// The corresponding output of each work-item will be:
  667 |   ///
  668 |   ///   { [0,128,256,384], [1,129,257,385], [2,130,258,386], ...,
  669 |   ///   [127,255,383,511] }.
  670 |   ///
  671 |   /// \tparam Item The work-item identifier type.
  672 |   /// \param item The work-item identifier.
  673 |   /// \param input The input data of each work-item.
  674 |   /// \param begin_bit The beginning (least-significant) bit index needed for
  675 |   /// key comparison.
  676 |   /// \param end_bit The past-the-end (most-significant) bit
  677 |   /// index needed for key comparison.
  678 |   template <typename Item>
  679 |   __compat_inline__ void
  680 |   sort_blocked_to_striped(const Item &item, T (&input)[ElementsPerWorkItem],
  681 |                           int begin_bit = 0, int end_bit = 8 * sizeof(T)) {
  682 |     helper_sort<Item, /*DESCENDING=*/false>(item, input, begin_bit, end_bit,
  683 |                                             /*is_striped=*/true);
  684 |   }
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 686-714

```text
  686 |   /// Performs an descending radix sort across a blocked arrangement of input
  687 |   /// elements, leaving them in a striped arrangement.
  688 |   ///
  689 |   /// Suppose 512 integer data elements partitioned across 128 work-items, where
  690 |   /// each work-item owns 4 ( \p ElementsPerWorkItem ) data elements and the
  691 |   /// \p input across the work-group is:
  692 |   ///
  693 |   ///   { [0,511,1,510], [2,509,3,508], [4,507,5,506], ..., [254,257,255,256] }.
  694 |   ///
  695 |   /// The descending striped order output is:
  696 |   ///
  697 |   ///   { [0,128,256,384], [1,129,257,385], [2,130,258,386], ...,
  698 |   ///   [127,255,383,511] }.
  699 |   ///
  700 |   /// \tparam Item The work-item identifier type.
  701 |   /// \param item The work-item identifier.
  702 |   /// \param input The input data of each work-item.
  703 |   /// \param begin_bit The beginning (least-significant) bit index needed for
  704 |   /// key comparison.
  705 |   /// \param end_bit The past-the-end (most-significant) bit
  706 |   /// index needed for key comparison.
  707 |   template <typename Item>
  708 |   __compat_inline__ void sort_descending_blocked_to_striped(
  709 |       const Item &item, T (&input)[ElementsPerWorkItem], int begin_bit = 0,
  710 |       int end_bit = 8 * sizeof(T)) {
  711 |     helper_sort<Item, /*DESCENDING=*/true>(item, input, begin_bit, end_bit,
  712 |                                            /*is_striped=*/true);
  713 |   }
  714 | };
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 716-721

```text
  716 | /// Load linear segment items into block format across threads
  717 | /// Helper for Block Load
  718 | enum load_algorithm {
  719 |   BLOCK_LOAD_DIRECT,
  720 |   BLOCK_LOAD_STRIPED,
  721 | };
```
**EN:** Defines `load_algorithm` as a reusable type-level building block in this header.
**CN:** 将 `load_algorithm` 定义为本头文件中的可复用类型级构件。

### Lines 723-743

```text
  723 | /// Load a linear segment of elements into a blocked arrangement across the
  724 | /// work-group.
  725 | ///
  726 | /// \tparam T The data type to load.
  727 | /// \tparam ElementsPerWorkItem The number of consecutive elements partitioned
  728 | /// onto each work-item.
  729 | /// \tparam InputIteratorT  The random-access iterator type for input \iterator.
  730 | /// \tparam ItemT The sycl::nd_item index space class.
  731 | /// \param item The calling work-item.
  732 | /// \param input_iter The work-group's base input iterator for loading from.
  733 | /// \param data Data to load.
  734 | template <typename T, size_t ElementsPerWorkItem, typename InputIteratorT,
  735 |           typename ItemT>
  736 | __compat_inline__ void load_direct_blocked(const ItemT &item,
  737 |                                                InputIteratorT input_iter,
  738 |                                                T (&data)[ElementsPerWorkItem]) {
  739 |   size_t work_item_id = item.get_local_linear_id();
  740 | #pragma unroll
  741 |   for (size_t i = 0; i < ElementsPerWorkItem; i++)
  742 |     data[i] = input_iter[(work_item_id * ElementsPerWorkItem) + i];
  743 | }
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 745-766

```text
  745 | /// Load a linear segment of elements into a striped arrangement across the
  746 | /// work-group.
  747 | ///
  748 | /// \tparam T The data type to load.
  749 | /// \tparam ElementsPerWorkItem The number of consecutive elements partitioned
  750 | /// onto each work-item.
  751 | /// \tparam InputIteratorT  The random-access iterator type for input \iterator.
  752 | /// \tparam ItemT The sycl::nd_item index space class.
  753 | /// \param item The calling work-item.
  754 | /// \param input_iter The work-group's base input iterator for loading from.
  755 | /// \param data Data to load.
  756 | template <typename T, int ElementsPerWorkItem, typename InputIteratorT,
  757 |           typename ItemT>
  758 | __compat_inline__ void load_direct_striped(const ItemT &item,
  759 |                                                InputIteratorT input_iter,
  760 |                                                T (&data)[ElementsPerWorkItem]) {
  761 |   size_t work_group_size = item.get_group().get_local_linear_range();
  762 |   size_t work_item_id = item.get_local_linear_id();
  763 | #pragma unroll
  764 |   for (size_t i = 0; i < ElementsPerWorkItem; i++)
  765 |     data[i] = input_iter[work_item_id + i * work_group_size];
  766 | }
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 768-790

```text
  768 | /// Load a linear segment of elements into a blocked arrangement across the
  769 | /// work-group, guarded by range.
  770 | ///
  771 | /// \tparam T The data type to load.
  772 | /// \tparam ElementsPerWorkItem The number of consecutive elements partitioned
  773 | /// onto each work-item.
  774 | /// \tparam InputIteratorT  The random-access iterator type for input \iterator.
  775 | /// \tparam ItemT The sycl::nd_item index space class.
  776 | /// \param item The calling work-item.
  777 | /// \param input_iter The work-group's base input iterator for loading from.
  778 | /// \param data Data to load.
  779 | /// \param valid_items Number of valid items to load
  780 | template <typename T, size_t ElementsPerWorkItem, typename InputIteratorT,
  781 |           typename ItemT>
  782 | __compat_inline__ void
  783 | load_direct_blocked(const ItemT &item, InputIteratorT input_iter,
  784 |                     T (&data)[ElementsPerWorkItem], int valid_items) {
  785 |   size_t work_item_id = item.get_local_linear_id();
  786 | #pragma unroll
  787 |   for (size_t i = 0; i < ElementsPerWorkItem; i++)
  788 |     if ((work_item_id * ElementsPerWorkItem) + i < valid_items)
  789 |       data[i] = input_iter[(work_item_id * ElementsPerWorkItem) + i];
  790 | }
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 792-815

```text
  792 | /// Load a linear segment of elements into a striped arrangement across the
  793 | /// work-group, guarded by range.
  794 | ///
  795 | /// \tparam T The data type to load.
  796 | /// \tparam ElementsPerWorkItem The number of consecutive elements partitioned
  797 | /// onto each work-item.
  798 | /// \tparam InputIteratorT  The random-access iterator type for input \iterator.
  799 | /// \tparam ItemT The sycl::nd_item index space class.
  800 | /// \param item The calling work-item.
  801 | /// \param input_iter The work-group's base input iterator for loading from.
  802 | /// \param data Data to load.
  803 | /// \param valid_items Number of valid items to load
  804 | template <typename T, int ElementsPerWorkItem, typename InputIteratorT,
  805 |           typename ItemT>
  806 | __compat_inline__ void
  807 | load_direct_striped(const ItemT &item, InputIteratorT input_iter,
  808 |                     T (&data)[ElementsPerWorkItem], int valid_items) {
  809 |   size_t work_group_size = item.get_group().get_local_linear_range();
  810 |   size_t work_item_id = item.get_local_linear_id();
  811 | #pragma unroll
  812 |   for (size_t i = 0; i < ElementsPerWorkItem; i++)
  813 |     if (work_item_id + (i * work_group_size) < valid_items)
  814 |       data[i] = input_iter[work_item_id + i * work_group_size];
  815 | }
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 817-840

```text
  817 | /// Store a blocked arrangement of items across a work-group into a linear
  818 | /// segment of items.
  819 | ///
  820 | /// \tparam T The data type to store.
  821 | /// \tparam ElementsPerWorkItem The number of consecutive elements partitioned
  822 | /// onto each work-item.
  823 | /// \tparam OutputIteratorT  The random-access iterator type for output.
  824 | /// \iterator.
  825 | /// \tparam ItemT The sycl::nd_item index space class.
  826 | /// \param item The calling work-item.
  827 | /// \param output_iter The work-group's base output iterator for writing.
  828 | /// \param data Data to store.
  829 | template <typename T, size_t ElementsPerWorkItem, typename OutputIteratorT,
  830 |           typename ItemT>
  831 | __compat_inline__ void
  832 | store_direct_blocked(const ItemT &item, OutputIteratorT output_iter,
  833 |                      T (&data)[ElementsPerWorkItem]) {
  834 |   size_t work_item_id = item.get_local_linear_id();
  835 |   OutputIteratorT work_item_iter =
  836 |       output_iter + (work_item_id * ElementsPerWorkItem);
  837 | #pragma unroll
  838 |   for (size_t i = 0; i < ElementsPerWorkItem; i++)
  839 |     work_item_iter[i] = data[i];
  840 | }
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 842-865

```text
  842 | /// Store a striped arrangement of items across a work-group into a linear
  843 | /// segment of items.
  844 | ///
  845 | /// \tparam T The data type to store.
  846 | /// \tparam ElementsPerWorkItem The number of consecutive elements partitioned
  847 | /// onto each work-item.
  848 | /// \tparam OutputIteratorT  The random-access iterator type for output.
  849 | /// \iterator.
  850 | /// \tparam ItemT The sycl::nd_item index space class.
  851 | /// \param item The calling work-item.
  852 | /// \param output_iter The work-group's base output iterator for writing.
  853 | /// \param items Data to store.
  854 | template <typename T, size_t ElementsPerWorkItem, typename OutputIteratorT,
  855 |           typename ItemT>
  856 | __compat_inline__ void
  857 | store_direct_striped(const ItemT &item, OutputIteratorT output_iter,
  858 |                      T (&data)[ElementsPerWorkItem]) {
  859 |   size_t work_group_size = item.get_group().get_local_linear_range();
  860 |   size_t work_item_id = item.get_local_linear_id();
  861 |   OutputIteratorT work_item_iter = output_iter + work_item_id;
  862 | #pragma unroll
  863 |   for (size_t i = 0; i < ElementsPerWorkItem; i++)
  864 |     work_item_iter[i * work_group_size] = data[i];
  865 | }
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 867-892

```text
  867 | /// Store a blocked arrangement of items across a work-group into a linear
  868 | /// segment of items, guarded by range.
  869 | ///
  870 | /// \tparam T The data type to store.
  871 | /// \tparam ElementsPerWorkItem The number of consecutive elements partitioned
  872 | /// onto each work-item.
  873 | /// \tparam OutputIteratorT  The random-access iterator type for output.
  874 | /// \iterator.
  875 | /// \tparam ItemT The sycl::nd_item index space class.
  876 | /// \param item The calling work-item.
  877 | /// \param output_iter The work-group's base output iterator for writing.
  878 | /// \param data Data to store.
  879 | /// \param valid_items Number of valid items to load
  880 | template <typename T, size_t ElementsPerWorkItem, typename OutputIteratorT,
  881 |           typename ItemT>
  882 | __compat_inline__ void
  883 | store_direct_blocked(const ItemT &item, OutputIteratorT output_iter,
  884 |                      T (&data)[ElementsPerWorkItem], size_t valid_items) {
  885 |   size_t work_item_id = item.get_local_linear_id();
  886 |   OutputIteratorT work_item_iter =
  887 |       output_iter + (work_item_id * ElementsPerWorkItem);
  888 | #pragma unroll
  889 |   for (size_t i = 0; i < ElementsPerWorkItem; i++)
  890 |     if (i + (work_item_id * ElementsPerWorkItem) < valid_items)
  891 |       work_item_iter[i] = data[i];
  892 | }
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 894-919

```text
  894 | /// Store a striped arrangement of items across a work-group into a linear
  895 | /// segment of items, guarded by range.
  896 | ///
  897 | /// \tparam T The data type to store.
  898 | /// \tparam ElementsPerWorkItem The number of consecutive elements partitioned
  899 | /// onto each work-item.
  900 | /// \tparam OutputIteratorT  The random-access iterator type for output.
  901 | /// \iterator.
  902 | /// \tparam ItemT The sycl::nd_item index space class.
  903 | /// \param item The calling work-item.
  904 | /// \param output_iter The work-group's base output iterator for writing.
  905 | /// \param items Data to store.
  906 | /// \param valid_items Number of valid items to load
  907 | template <typename T, size_t ElementsPerWorkItem, typename OutputIteratorT,
  908 |           typename ItemT>
  909 | __compat_inline__ void
  910 | store_direct_striped(const ItemT &item, OutputIteratorT output_iter,
  911 |                      T (&data)[ElementsPerWorkItem], size_t valid_items) {
  912 |   size_t work_group_size = item.get_group().get_local_linear_range();
  913 |   size_t work_item_id = item.get_local_linear_id();
  914 |   OutputIteratorT work_item_iter = output_iter + work_item_id;
  915 | #pragma unroll
  916 |   for (size_t i = 0; i < ElementsPerWorkItem; i++)
  917 |     if ((i * work_group_size) + work_item_id < valid_items)
  918 |       work_item_iter[i * work_group_size] = data[i];
  919 | }
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 921-926

```text
  921 | /// Enumerates alternative algorithms for compat::group::group_load to read
  922 | /// a linear segment of data from memory into a blocked arrangement across a
  923 | /// work-group.
  924 | enum class group_load_algorithm {
  925 |   /// A blocked arrangement of data is read directly from memory.
  926 |   blocked,
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 928-930

```text
  928 |   /// A striped arrangement of data is read directly from memory.
  929 |   striped
  930 | };
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 932-946

```text
  932 | /// Provide methods for loading a linear segment of items from memory into a
  933 | /// blocked arrangement across a work-group.
  934 | ///
  935 | /// \tparam T The input data type.
  936 | /// \tparam ElementsPerWorkItem The number of data elements assigned to a
  937 | /// work-item.
  938 | /// \tparam LoadAlgorithm The data movement strategy, default is blocked.
  939 | template <typename T, size_t ElementsPerWorkItem,
  940 |           group_load_algorithm LoadAlgorithm = group_load_algorithm::blocked>
  941 | class group_load {
  942 | public:
  943 |   static size_t get_local_memory_size([[maybe_unused]] size_t work_group_size) {
  944 |     return 0;
  945 |   }
  946 |   group_load(uint8_t *) {}
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 948-980

```text
  948 |   /// Load a linear segment of items from memory.
  949 |   ///
  950 |   /// Suppose 512 integer data elements partitioned across 128 work-items, where
  951 |   /// each work-item owns 4 ( \p ElementsPerWorkItem ) data elements and the
  952 |   /// \p input across the work-group is:
  953 |   ///
  954 |   ///   1, 2, 3, 4, 5, 6, 7, ..., 508, 509, 510, 511.
  955 |   ///
  956 |   /// The blocked order \p data of each work-item will be:
  957 |   ///
  958 |   ///   {[0,1,2,3], [4,5,6,7], ..., [508,509,510,511]}.
  959 |   ///
  960 |   /// The striped order \p output of each work-item will be:
  961 |   ///
  962 |   ///   {[0,128,256,384], [1,129,257,385], ..., [127,255,383,511]}.
  963 |   ///
  964 |   /// \tparam ItemT The sycl::nd_item index space class.
  965 |   /// \tparam InputIteratorT The random-access iterator type for input
  966 |   /// \iterator.
  967 |   /// \param item The work-item identifier.
  968 |   /// \param input_iter The work-group's base input iterator for loading from.
  969 |   /// \param data The data to load.
  970 |   template <typename ItemT, typename InputIteratorT>
  971 |   __compat_inline__ void load(const ItemT &item, InputIteratorT input_iter,
  972 |                                   T (&data)[ElementsPerWorkItem]) {
  973 |     if constexpr (LoadAlgorithm == group_load_algorithm::blocked) {
  974 |       load_direct_blocked<T, ElementsPerWorkItem, InputIteratorT, ItemT>(
  975 |           item, input_iter, data);
  976 |     } else if constexpr (LoadAlgorithm == group_load_algorithm::striped) {
  977 |       load_direct_striped<T, ElementsPerWorkItem, InputIteratorT, ItemT>(
  978 |           item, input_iter, data);
  979 |     }
  980 |   }
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 982-1017

```text
  982 |   /// Load a linear segment of items from memory, guarded by range.
  983 |   ///
  984 |   /// Suppose 512 integer data elements partitioned across 128 work-items, where
  985 |   /// each work-item owns 4 ( \p ElementsPerWorkItem ) data elements and
  986 |   /// valid_items is 5, the \p input across the work-group is:
  987 |   ///
  988 |   ///   0, 1, 2, 3, 4, 5, 6, 7, ..., 508, 509, 510, 511.
  989 |   ///
  990 |   /// The blocked order \p data of each work-item will be:
  991 |   ///
  992 |   ///   {[0,1,2,3], [4,?,?,?], ..., [?,?,?,?]}.
  993 |   ///
  994 |   /// The striped order \p output of each work-item will be:
  995 |   ///
  996 |   ///   {[0,?,?,?], [1,?,?,?], [2,?,?,?], [3,?,?,?] ..., [?,?,?,?]}.
  997 |   ///
  998 |   /// \tparam ItemT The sycl::nd_item index space class.
  999 |   /// \tparam InputIteratorT The random-access iterator type for input
 1000 |   /// \iterator.
 1001 |   /// \param item The work-item identifier.
 1002 |   /// \param input_iter The work-group's base input iterator for loading from.
 1003 |   /// \param data The data to load.
 1004 |   /// \param valid_items Number of valid items to load
 1005 |   template <typename ItemT, typename InputIteratorT>
 1006 |   __compat_inline__ void load(const ItemT &item, InputIteratorT input_iter,
 1007 |                                   T (&data)[ElementsPerWorkItem],
 1008 |                                   int valid_items) {
 1009 |     if constexpr (LoadAlgorithm == group_load_algorithm::blocked) {
 1010 |       load_direct_blocked<T, ElementsPerWorkItem, InputIteratorT, ItemT>(
 1011 |           item, input_iter, data, valid_items);
 1012 |     } else if constexpr (LoadAlgorithm == group_load_algorithm::striped) {
 1013 |       load_direct_striped<T, ElementsPerWorkItem, InputIteratorT, ItemT>(
 1014 |           item, input_iter, data, valid_items);
 1015 |     }
 1016 |   }
 1017 | };
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 1019-1024

```text
 1019 | /// Enumerates alternative algorithms for compat::group::group_load to write
 1020 | /// a blocked arrangement of items across a work-group to a linear segment of
 1021 | /// memory.
 1022 | enum class group_store_algorithm {
 1023 |   /// A blocked arrangement of data is written directly to memory.
 1024 |   blocked,
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 1026-1028

```text
 1026 |   /// A striped arrangement of data is written directly to memory.
 1027 |   striped,
 1028 | };
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 1030-1044

```text
 1030 | /// Provide methods for writing a blocked arrangement of elements partitioned
 1031 | /// across a work-group to a linear segment of memory.
 1032 | ///
 1033 | /// \tparam T The output data type.
 1034 | /// \tparam ElementsPerWorkItem The number of data elements assigned to a
 1035 | /// work-item.
 1036 | /// \tparam StoreAlgorithm The data movement strategy, default is blocked.
 1037 | template <typename T, size_t ElementsPerWorkItem,
 1038 |           group_store_algorithm StoreAlgorithm = group_store_algorithm::blocked>
 1039 | class group_store {
 1040 | public:
 1041 |   static size_t get_local_memory_size([[maybe_unused]] size_t work_group_size) {
 1042 |     return 0;
 1043 |   }
 1044 |   group_store(uint8_t *) {}
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 1046-1079

```text
 1046 |   /// Store items into a linear segment of memory.
 1047 |   ///
 1048 |   /// Suppose 512 integer data elements partitioned across 128 work-items, where
 1049 |   /// each work-item owns 4 ( \p ElementsPerWorkItem ) data elements and the
 1050 |   /// \p input across the work-group is:
 1051 |   ///
 1052 |   ///   {[0,1,2,3], [4,5,6,7], ..., [508,509,510,511]}.
 1053 |   ///
 1054 |   /// The blocked order \p output will be:
 1055 |   ///
 1056 |   ///   1, 2, 3, 4, 5, 6, 7, ..., 508, 509, 510, 511.
 1057 |   ///
 1058 |   /// The striped order \p output will be:
 1059 |   ///
 1060 |   ///   0, 128, 256, 384, 1, 129, 257, 385, ..., 127, 255, 383, 511.
 1061 |   ///
 1062 |   /// \tparam ItemT The sycl::nd_item index space class.
 1063 |   /// \tparam OutputIteratorT The random-access iterator type for \p output
 1064 |   /// iterator.
 1065 |   /// \param item The work-item identifier.
 1066 |   /// \param input The input data of each work-item.
 1067 |   /// \param data The data to store.
 1068 |   template <typename ItemT, typename OutputIteratorT>
 1069 |   __compat_inline__ void store(const ItemT &item,
 1070 |                                    OutputIteratorT output_iter,
 1071 |                                    T (&data)[ElementsPerWorkItem]) {
 1072 |     if constexpr (StoreAlgorithm == group_store_algorithm::blocked) {
 1073 |       store_direct_blocked<T, ElementsPerWorkItem, OutputIteratorT, ItemT>(
 1074 |           item, output_iter, data);
 1075 |     } else if constexpr (StoreAlgorithm == group_store_algorithm::striped) {
 1076 |       store_direct_striped<T, ElementsPerWorkItem, OutputIteratorT, ItemT>(
 1077 |           item, output_iter, data);
 1078 |     }
 1079 |   }
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 1081-1116

```text
 1081 |   /// Store items into a linear segment of memory, guarded by range.
 1082 |   ///
 1083 |   /// Suppose 512 integer data elements partitioned across 128 work-items, where
 1084 |   /// each work-item owns 4 ( \p ElementsPerWorkItem ) data elements and
 1085 |   /// \p valid_items is 5, the \p output across the work-group is:
 1086 |   ///
 1087 |   ///   {[0,0,0,0], [0,0,0,0], ..., [0,0,0,0]}.
 1088 |   ///
 1089 |   /// The blocked order \p output will be:
 1090 |   ///
 1091 |   ///   0, 1, 2, 3, 4, 5, 0, 0, ..., 0, 0, 0, 0.
 1092 |   ///
 1093 |   /// The striped order \p output will be:
 1094 |   ///
 1095 |   ///   0, 4, 8, 12, 16, 0, 0, 0, ..., 0, 0, 0, 0.
 1096 |   ///
 1097 |   /// \tparam ItemT The sycl::nd_item index space class.
 1098 |   /// \tparam OutputIteratorT The random-access iterator type for \p output
 1099 |   /// iterator.
 1100 |   /// \param item The work-item identifier.
 1101 |   /// \param input The input data of each work-item.
 1102 |   /// \param data The data to store.
 1103 |   /// \param valid_items Number of valid items to load
 1104 |   template <typename ItemT, typename OutputIteratorT>
 1105 |   __compat_inline__ void
 1106 |   store(const ItemT &item, OutputIteratorT output_iter,
 1107 |         T (&data)[ElementsPerWorkItem], size_t valid_items) {
 1108 |     if constexpr (StoreAlgorithm == group_store_algorithm::blocked) {
 1109 |       store_direct_blocked<T, ElementsPerWorkItem, OutputIteratorT, ItemT>(
 1110 |           item, output_iter, data, valid_items);
 1111 |     } else if constexpr (StoreAlgorithm == group_store_algorithm::striped) {
 1112 |       store_direct_striped<T, ElementsPerWorkItem, OutputIteratorT, ItemT>(
 1113 |           item, output_iter, data, valid_items);
 1114 |     }
 1115 |   }
 1116 | };
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 1118-1129

```text
 1118 | /// The work-group wide shuffle operations that allow work-items to exchange
 1119 | /// data elements with other work-items within the same work-group.
 1120 | ///
 1121 | /// \tparam T The type of the data elements.
 1122 | /// \tparam group_dim_0 The first dimension size of the work-group.
 1123 | /// \tparam group_dim_1 The second dimension size of the work-group.
 1124 | /// \tparam group_dim_2 The third dimension size of the work-group.
 1125 | template <typename T, int group_dim_0, int group_dim_1 = 1, int group_dim_2 = 1>
 1126 | class group_shuffle {
 1127 |   T *_local_memory = nullptr;
 1128 |   static constexpr size_t group_work_items =
 1129 |       group_dim_0 * group_dim_1 * group_dim_2;
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 1131-1135

```text
 1131 | public:
 1132 |   static constexpr size_t get_local_memory_size(size_t work_group_size) {
 1133 |     return sizeof(T) * work_group_size;
 1134 |   }
 1135 |   group_shuffle(uint8_t *local_memory) : _local_memory((T *)local_memory) {}
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 1137-1271

```text
 1137 |   /// Selects a value from a work-item at a given distance in the work-group
 1138 |   /// and stores the value in the output.
 1139 |   ///
 1140 |   /// \tparam ItemT The work-item identifier type.
 1141 |   /// \param item The work-item identifier.
 1142 |   /// \param input The input from the calling work-item.
 1143 |   /// \param output The output where the selected data will be stored.
 1144 |   /// \param distance The distance of work-items to look ahead or behind in the
 1145 |   /// work-group.
 1146 |   template <typename ItemT>
 1147 |   __compat_inline__ void select(const ItemT &item, T input, T &output,
 1148 |                                     int distance = 1) {
 1149 |     auto g = item.get_group();
 1150 |     size_t id = g.get_local_linear_id();
 1151 |     _local_memory[id] = input;
 1152 | 
 1153 |     sycl::group_barrier(g, sycl::memory_scope::work_group);
 1154 | 
 1155 |     const int target_id = static_cast<int>(id) + distance;
 1156 |     if ((target_id >= 0) && (target_id < group_work_items)) {
 1157 |       output = _local_memory[static_cast<size_t>(target_id)];
 1158 |     }
 1159 |   }
 1160 |   /// Selects a value from a work-item at a given distance in the work-group
 1161 |   /// and stores the value in the output, using a wrapped index to handle
 1162 |   /// overflow.
 1163 |   ///
 1164 |   /// \tparam ItemT The work-item identifier type.
 1165 |   /// \param item The work-item identifier.
 1166 |   /// \param input The input data to be selected.
 1167 |   /// \param output The output where the selected data will be stored.
 1168 |   /// \param distance The number of work-items to look ahead in the
 1169 |   /// work-group.
 1170 |   template <typename ItemT>
 1171 |   __compat_inline__ void select2(const ItemT &item, T input, T &output,
 1172 |                                      unsigned int distance = 1) {
 1173 |     auto g = item.get_group();
 1174 |     size_t id = g.get_local_linear_id();
 1175 |     _local_memory[id] = input;
 1176 | 
 1177 |     sycl::group_barrier(g, sycl::memory_scope::work_group);
 1178 | 
 1179 |     unsigned int offset = id + distance;
 1180 |     if (offset >= group_work_items)
 1181 |       offset -= group_work_items;
 1182 | 
 1183 |     output = _local_memory[offset];
 1184 |   }
 1185 |   /// Performs a shuffle operation to move data to the right across the
 1186 |   /// work-items, shifting elements in a work-item array by one position to the
 1187 |   /// right.
 1188 |   ///
 1189 |   /// \tparam ElementsPerWorkItem The number of data elements per work-item.
 1190 |   /// \tparam ItemT The work-item identifier type.
 1191 |   /// \param item The work-item identifier.
 1192 |   /// \param input The input data to be shuffled.
 1193 |   /// \param output The array that will store the shuffle result.
 1194 |   template <int ElementsPerWorkItem, typename ItemT>
 1195 |   __compat_inline__ void shuffle_right(const ItemT &item,
 1196 |                                            T (&input)[ElementsPerWorkItem],
 1197 |                                            T (&output)[ElementsPerWorkItem]) {
 1198 |     auto g = item.get_group();
 1199 |     size_t id = g.get_local_linear_id();
 1200 |     _local_memory[id] = input[ElementsPerWorkItem - 1];
 1201 | 
 1202 |     sycl::group_barrier(g, sycl::memory_scope::work_group);
 1203 | 
 1204 | #pragma unroll
 1205 |     for (int index = ElementsPerWorkItem - 1; index > 0; --index)
 1206 |       output[index] = input[index - 1];
 1207 | 
 1208 |     if (id > 0)
 1209 |       output[0] = _local_memory[id - 1];
 1210 |   }
 1211 |   /// Performs a shuffle operation to move data to the right across the
 1212 |   /// work-items, storing the suffix of the group after the shuffle operation.
 1213 |   ///
 1214 |   /// \tparam ElementsPerWorkItem The number of data elements per work-item.
 1215 |   /// \tparam ItemT The work-item identifier type.
 1216 |   /// \param item The work-item identifier.
 1217 |   /// \param input The input data to be shuffled.
 1218 |   /// \param output The array that will store the shuffle result.
 1219 |   /// \param group_suffix The suffix of the group after the shuffle.
 1220 |   template <int ElementsPerWorkItem, typename ItemT>
 1221 |   __compat_inline__ void
 1222 |   shuffle_right(const ItemT &item, T (&input)[ElementsPerWorkItem],
 1223 |                 T (&output)[ElementsPerWorkItem], T &group_suffix) {
 1224 |     shuffle_right(item, input, output);
 1225 |     group_suffix = _local_memory[group_work_items - 1];
 1226 |   }
 1227 |   /// Performs a shuffle operation to move data to the left across the
 1228 |   /// work-items, shifting elements in a work-item array by one position to the
 1229 |   /// left.
 1230 |   ///
 1231 |   /// \tparam ElementsPerWorkItem The number of data elements per work-item.
 1232 |   /// \tparam ItemT The work-item identifier type.
 1233 |   /// \param item The work-item identifier.
 1234 |   /// \param input The input data to be shuffled.
 1235 |   /// \param output The array that will store the shuffle result.
 1236 |   template <int ElementsPerWorkItem, typename ItemT>
 1237 |   __compat_inline__ void shuffle_left(const ItemT &item,
 1238 |                                           T (&input)[ElementsPerWorkItem],
 1239 |                                           T (&output)[ElementsPerWorkItem]) {
 1240 |     auto g = item.get_group();
 1241 |     size_t id = g.get_local_linear_id();
 1242 |     _local_memory[id] = input[0];
 1243 | 
 1244 |     sycl::group_barrier(g, sycl::memory_scope::work_group);
 1245 | 
 1246 | #pragma unroll
 1247 |     for (int index = 0; index < ElementsPerWorkItem - 1; index++)
 1248 |       output[index] = input[index + 1];
 1249 | 
 1250 |     if (id < group_work_items - 1)
 1251 |       output[ElementsPerWorkItem - 1] = _local_memory[id + 1];
 1252 |   }
 1253 |   /// Performs a shuffle operation to move data to the left across the
 1254 |   /// work-items, storing the prefix of the group before the shuffle operation.
 1255 |   ///
 1256 |   /// \tparam ElementsPerWorkItem The number of data elements per work-item.
 1257 |   /// \tparam ItemT The work-item identifier type.
 1258 |   /// \param item The work-item identifier.
 1259 |   /// \param input The input data to be shuffled.
 1260 |   /// \param output The array that will store the shuffle result.
 1261 |   /// \param group_prefix The prefix of the group before the shuffle.
 1262 |   template <int ElementsPerWorkItem, typename ItemT>
 1263 |   __compat_inline__ void
 1264 |   shuffle_left(const ItemT &item, T (&input)[ElementsPerWorkItem],
 1265 |                T (&output)[ElementsPerWorkItem], T &group_prefix) {
 1266 |     shuffle_left(item, input, output);
 1267 |     group_prefix = _local_memory[0];
 1268 |   }
 1269 | };
 1270 | } // namespace group
 1271 | } // namespace compat
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

## Key Concepts / 关键概念

- Memory-space-aware pointer adaptation / 感知内存空间的指针适配
- Compile-time numeric metaprogramming / 编译期数值元编程
- Static containers and tuple-like storage / 静态容器与类元组存储
- Backend portability and SYCL-style compatibility / 后端可移植性与 SYCL 风格兼容层
- SYCL interoperability / SYCL 互操作
- Intel Xe-specific behavior / Intel Xe 特定行为

## Dependencies / 依赖关系

- Direct includes / 直接包含:
  - `iterator`
  - `stdexcept`
  - `sycl/sycl.hpp`
  - `cute/util/compat/defs.hpp`
  - `cute/util/compat/math.hpp`
- Primary symbols / 主要符号: `log2`, `prefix_callback`, `base_traits`, `traits`, `power_of_two`, `blocked_offset`, `striped_offset`, `scatter_offset`
- Dependency role / 依赖角色: Sits between CuTe algorithms and backend APIs, normalizing device, kernel, launch, memory, and math behavior across supported targets. / 位于 CuTe 算法与后端 API 之间，统一不同目标上的设备、内核、启动、内存和数学行为。
