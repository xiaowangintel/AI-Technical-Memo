# atomic.hpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `include/cute/util/compat/atomic.hpp`
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
   18 |  *  atomic.hpp
   19 |  *
   20 |  *  Description:
   21 |  *    Atomic functionality for the SYCL compatibility extension
```
**EN:** Provides the license notice, copyright ownership, and redistribution terms for this header.
**CN:** 给出该头文件的许可证声明、版权归属以及再分发条款。

### Lines 22-34

```text
   22 |  **************************************************************************/
   24 | // The original source was under the license below:
   25 | //==---- atomic.hpp -------------------------------*- C++ -*----------------==//
   26 | //
   27 | // Copyright (C) Intel Corporation
   28 | // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   29 | // See https://llvm.org/LICENSE.txt for license information.
   30 | //
   31 | //===----------------------------------------------------------------------===//
   33 | #pragma once
   34 | #pragma GCC system_header
```
**EN:** Wraps backend-specific atomic behavior behind a compatibility-friendly CuTe interface.
**CN:** 在兼容层友好的 CuTe 接口后面封装后端特定的原子行为。

### Lines 36-43

```text
   36 | #include <cassert>
   38 | #include <sycl/access/access.hpp>
   39 | #include <sycl/atomic_ref.hpp>
   40 | #include <sycl/memory_enums.hpp>
   41 | #include <sycl/multi_ptr.hpp>
   43 | #include <cute/util/compat/traits.hpp>
```
**EN:** Sets up the header dependencies for this file by importing `cassert`, `sycl/access/access.hpp`, `sycl/atomic_ref.hpp`, `sycl/memory_enums.hpp`, `sycl/multi_ptr.hpp`, and 1 more include(s).
**CN:** 通过引入 `cassert`, `sycl/access/access.hpp`, `sycl/atomic_ref.hpp`, `sycl/memory_enums.hpp`, `sycl/multi_ptr.hpp`，以及另外 1 个头文件 为该文件建立头文件依赖。

### Lines 45-45

```text
   45 | namespace compat {
```
**EN:** Opens or closes namespace scope so the following declarations remain grouped under the intended CuTe or backend namespace hierarchy.
**CN:** 打开或关闭命名空间作用域，使后续声明保持在预期的 CuTe 或后端命名空间层级中。

### Lines 47-62

```text
   47 | /// Atomically add the value operand to the value at the addr and assign the
   48 | /// result to the value at addr.
   49 | /// \param [in, out] addr The pointer to the data.
   50 | /// \param operand The value to add to the value at \p addr.
   51 | /// \param memoryOrder The memory ordering used.
   52 | /// \returns The value at the \p addr before the call.
   53 | template <sycl::access::address_space addressSpace =
   54 |               sycl::access::address_space::generic_space,
   55 |           sycl::memory_order memoryOrder = sycl::memory_order::relaxed,
   56 |           sycl::memory_scope memoryScope = sycl::memory_scope::device,
   57 |           typename T>
   58 | inline T atomic_fetch_add(T *addr, arith_t<T> operand) {
   59 |   auto atm =
   60 |       sycl::atomic_ref<T, memoryOrder, memoryScope, addressSpace>(addr[0]);
   61 |   return atm.fetch_add(operand);
   62 | }
```
**EN:** Wraps backend-specific atomic behavior behind a compatibility-friendly CuTe interface.
**CN:** 在兼容层友好的 CuTe 接口后面封装后端特定的原子行为。

### Lines 64-79

```text
   64 | /// Atomically subtract the value operand from the value at the addr and
   65 | /// assign the result to the value at addr.
   66 | /// \param [in, out] addr The pointer to the data.
   67 | /// \param operand The value to subtract from the value at \p addr.
   68 | /// \param memoryOrder The memory ordering used.
   69 | /// \returns The value at the \p addr before the call.
   70 | template <sycl::access::address_space addressSpace =
   71 |               sycl::access::address_space::generic_space,
   72 |           sycl::memory_order memoryOrder = sycl::memory_order::relaxed,
   73 |           sycl::memory_scope memoryScope = sycl::memory_scope::device,
   74 |           typename T>
   75 | inline T atomic_fetch_sub(T *addr, arith_t<T> operand) {
   76 |   auto atm =
   77 |       sycl::atomic_ref<T, memoryOrder, memoryScope, addressSpace>(addr[0]);
   78 |   return atm.fetch_sub(operand);
   79 | }
```
**EN:** Wraps backend-specific atomic behavior behind a compatibility-friendly CuTe interface.
**CN:** 在兼容层友好的 CuTe 接口后面封装后端特定的原子行为。

### Lines 81-97

```text
   81 | /// Atomically perform a bitwise AND between the value operand and the value
   82 | /// at the addr and assign the result to the value at addr.
   83 | /// \param [in, out] addr The pointer to the data.
   84 | /// \param operand The value to use in bitwise AND operation with the value at
   85 | /// the \p addr.
   86 | /// \param memoryOrder The memory ordering used.
   87 | /// \returns The value at the \p addr before the call.
   88 | template <sycl::access::address_space addressSpace =
   89 |               sycl::access::address_space::generic_space,
   90 |           sycl::memory_order memoryOrder = sycl::memory_order::relaxed,
   91 |           sycl::memory_scope memoryScope = sycl::memory_scope::device,
   92 |           typename T>
   93 | inline T atomic_fetch_and(T *addr, type_identity_t<T> operand) {
   94 |   auto atm =
   95 |       sycl::atomic_ref<T, memoryOrder, memoryScope, addressSpace>(addr[0]);
   96 |   return atm.fetch_and(operand);
   97 | }
```
**EN:** Wraps backend-specific atomic behavior behind a compatibility-friendly CuTe interface.
**CN:** 在兼容层友好的 CuTe 接口后面封装后端特定的原子行为。

### Lines 99-115

```text
   99 | /// Atomically or the value at the addr with the value operand, and assign
  100 | /// the result to the value at addr.
  101 | /// \param [in, out] addr The pointer to the data.
  102 | /// \param operand The value to use in bitwise OR operation with the value at
  103 | /// the \p addr.
  104 | /// \param memoryOrder The memory ordering used.
  105 | /// \returns The value at the \p addr before the call.
  106 | template <sycl::access::address_space addressSpace =
  107 |               sycl::access::address_space::generic_space,
  108 |           sycl::memory_order memoryOrder = sycl::memory_order::relaxed,
  109 |           sycl::memory_scope memoryScope = sycl::memory_scope::device,
  110 |           typename T>
  111 | inline T atomic_fetch_or(T *addr, type_identity_t<T> operand) {
  112 |   auto atm =
  113 |       sycl::atomic_ref<T, memoryOrder, memoryScope, addressSpace>(addr[0]);
  114 |   return atm.fetch_or(operand);
  115 | }
```
**EN:** Wraps backend-specific atomic behavior behind a compatibility-friendly CuTe interface.
**CN:** 在兼容层友好的 CuTe 接口后面封装后端特定的原子行为。

### Lines 117-133

```text
  117 | /// Atomically xor the value at the addr with the value operand, and assign
  118 | /// the result to the value at addr.
  119 | /// \param [in, out] addr The pointer to the data.
  120 | /// \param operand The value to use in bitwise XOR operation with the value at
  121 | /// the \p addr.
  122 | /// \param memoryOrder The memory ordering used.
  123 | /// \returns The value at the \p addr before the call.
  124 | template <sycl::access::address_space addressSpace =
  125 |               sycl::access::address_space::generic_space,
  126 |           sycl::memory_order memoryOrder = sycl::memory_order::relaxed,
  127 |           sycl::memory_scope memoryScope = sycl::memory_scope::device,
  128 |           typename T>
  129 | inline T atomic_fetch_xor(T *addr, type_identity_t<T> operand) {
  130 |   auto atm =
  131 |       sycl::atomic_ref<T, memoryOrder, memoryScope, addressSpace>(addr[0]);
  132 |   return atm.fetch_xor(operand);
  133 | }
```
**EN:** Wraps backend-specific atomic behavior behind a compatibility-friendly CuTe interface.
**CN:** 在兼容层友好的 CuTe 接口后面封装后端特定的原子行为。

### Lines 135-149

```text
  135 | /// Atomically calculate the minimum of the value at addr and the value
  136 | /// operand and assign the result to the value at addr.
  137 | /// \param [in, out] addr The pointer to the data.
  138 | /// \param operand. \param memoryOrder The memory ordering used.
  139 | /// \returns The value at the \p addr before the call.
  140 | template <sycl::access::address_space addressSpace =
  141 |               sycl::access::address_space::generic_space,
  142 |           sycl::memory_order memoryOrder = sycl::memory_order::relaxed,
  143 |           sycl::memory_scope memoryScope = sycl::memory_scope::device,
  144 |           typename T>
  145 | inline T atomic_fetch_min(T *addr, type_identity_t<T> operand) {
  146 |   auto atm =
  147 |       sycl::atomic_ref<T, memoryOrder, memoryScope, addressSpace>(addr[0]);
  148 |   return atm.fetch_min(operand);
  149 | }
```
**EN:** Wraps backend-specific atomic behavior behind a compatibility-friendly CuTe interface.
**CN:** 在兼容层友好的 CuTe 接口后面封装后端特定的原子行为。

### Lines 151-166

```text
  151 | /// Atomically calculate the maximum of the value at addr and the value
  152 | /// operand and assign the result to the value at addr.
  153 | /// \param [in, out] addr The pointer to the data.
  154 | /// \param operand.
  155 | /// \param memoryOrder The memory ordering used.
  156 | /// \returns The value at the \p addr before the call.
  157 | template <sycl::access::address_space addressSpace =
  158 |               sycl::access::address_space::generic_space,
  159 |           sycl::memory_order memoryOrder = sycl::memory_order::relaxed,
  160 |           sycl::memory_scope memoryScope = sycl::memory_scope::device,
  161 |           typename T>
  162 | inline T atomic_fetch_max(T *addr, type_identity_t<T> operand) {
  163 |   auto atm =
  164 |       sycl::atomic_ref<T, memoryOrder, memoryScope, addressSpace>(addr[0]);
  165 |   return atm.fetch_max(operand);
  166 | }
```
**EN:** Wraps backend-specific atomic behavior behind a compatibility-friendly CuTe interface.
**CN:** 在兼容层友好的 CuTe 接口后面封装后端特定的原子行为。

### Lines 168-183

```text
  168 | /// Atomically set \p operand to the value stored in \p addr, if old value
  169 | /// stored in \p addr is equal to zero or greater than \p operand, else decrease
  170 | /// the value stored in \p addr. \param [in, out] addr The pointer to the data.
  171 | /// \param operand The threshold value.
  172 | /// \param memoryOrder The memory ordering used.
  173 | /// \returns The old value stored in \p addr.
  174 | template <sycl::access::address_space addressSpace =
  175 |               sycl::access::address_space::generic_space,
  176 |           sycl::memory_order memoryOrder = sycl::memory_order::relaxed,
  177 |           sycl::memory_scope memoryScope = sycl::memory_scope::device>
  178 | unsigned int atomic_fetch_compare_dec(unsigned int *addr,
  179 |                                       unsigned int operand) {
  180 |   auto atm =
  181 |       sycl::atomic_ref<unsigned int, memoryOrder, memoryScope, addressSpace>(
  182 |           addr[0]);
  183 |   unsigned int old;
```
**EN:** Wraps backend-specific atomic behavior behind a compatibility-friendly CuTe interface.
**CN:** 在兼容层友好的 CuTe 接口后面封装后端特定的原子行为。

### Lines 185-192

```text
  185 |   while (true) {
  186 |     old = atm.load();
  187 |     if (old == 0 || old > operand) {
  188 |       if (atm.compare_exchange_strong(old, operand))
  189 |         break;
  190 |     } else if (atm.compare_exchange_strong(old, old - 1))
  191 |       break;
  192 |   }
```
**EN:** Wraps backend-specific atomic behavior behind a compatibility-friendly CuTe interface.
**CN:** 在兼容层友好的 CuTe 接口后面封装后端特定的原子行为。

### Lines 194-195

```text
  194 |   return old;
  195 | }
```
**EN:** Wraps backend-specific atomic behavior behind a compatibility-friendly CuTe interface.
**CN:** 在兼容层友好的 CuTe 接口后面封装后端特定的原子行为。

### Lines 197-222

```text
  197 | /// Atomically increment the value stored in \p addr if old value stored in \p
  198 | /// addr is less than \p operand, else set 0 to the value stored in \p addr.
  199 | /// \param [in, out] addr The pointer to the data.
  200 | /// \param operand The threshold value.
  201 | /// \param memoryOrder The memory ordering used.
  202 | /// \returns The old value stored in \p addr.
  203 | template <sycl::access::address_space addressSpace =
  204 |               sycl::access::address_space::generic_space,
  205 |           sycl::memory_order memoryOrder = sycl::memory_order::relaxed,
  206 |           sycl::memory_scope memoryScope = sycl::memory_scope::device>
  207 | inline unsigned int atomic_fetch_compare_inc(unsigned int *addr,
  208 |                                              unsigned int operand) {
  209 |   auto atm =
  210 |       sycl::atomic_ref<unsigned int, memoryOrder, memoryScope, addressSpace>(
  211 |           addr[0]);
  212 |   unsigned int old;
  213 |   while (true) {
  214 |     old = atm.load();
  215 |     if (old >= operand) {
  216 |       if (atm.compare_exchange_strong(old, 0))
  217 |         break;
  218 |     } else if (atm.compare_exchange_strong(old, old + 1))
  219 |       break;
  220 |   }
  221 |   return old;
  222 | }
```
**EN:** Wraps backend-specific atomic behavior behind a compatibility-friendly CuTe interface.
**CN:** 在兼容层友好的 CuTe 接口后面封装后端特定的原子行为。

### Lines 224-238

```text
  224 | /// Atomically exchange the value at the address addr with the value operand.
  225 | /// \param [in, out] addr The pointer to the data.
  226 | /// \param operand The value to be exchanged with the value pointed by \p addr.
  227 | /// \param memoryOrder The memory ordering used.
  228 | /// \returns The value at the \p addr before the call.
  229 | template <sycl::access::address_space addressSpace =
  230 |               sycl::access::address_space::generic_space,
  231 |           sycl::memory_order memoryOrder = sycl::memory_order::relaxed,
  232 |           sycl::memory_scope memoryScope = sycl::memory_scope::device,
  233 |           typename T>
  234 | inline T atomic_exchange(T *addr, type_identity_t<T> operand) {
  235 |   auto atm =
  236 |       sycl::atomic_ref<T, memoryOrder, memoryScope, addressSpace>(addr[0]);
  237 |   return atm.exchange(operand);
  238 | }
```
**EN:** Wraps backend-specific atomic behavior behind a compatibility-friendly CuTe interface.
**CN:** 在兼容层友好的 CuTe 接口后面封装后端特定的原子行为。

### Lines 240-260

```text
  240 | /// Atomically compare the value at \p addr to the value expected and exchange
  241 | /// with the value desired if the value at \p addr is equal to the value
  242 | /// expected. Returns the value at the \p addr before the call.
  243 | /// \param [in, out] addr Multi_ptr.
  244 | /// \param expected The value to compare against the value at \p addr.
  245 | /// \param desired The value to assign to \p addr if the value at \p addr
  246 | /// is expected.
  247 | /// \param success The memory ordering used when comparison succeeds.
  248 | /// \param fail The memory ordering used when comparison fails.
  249 | /// \returns The value at the \p addr before the call.
  250 | template <sycl::access::address_space addressSpace =
  251 |               sycl::access::address_space::generic_space,
  252 |           sycl::memory_order memoryOrder = sycl::memory_order::relaxed,
  253 |           sycl::memory_scope memoryScope = sycl::memory_scope::device,
  254 |           typename T>
  255 | T atomic_compare_exchange_strong(
  256 |     sycl::multi_ptr<T, addressSpace> addr, type_identity_t<T> expected,
  257 |     type_identity_t<T> desired,
  258 |     sycl::memory_order success = sycl::memory_order::relaxed,
  259 |     sycl::memory_order fail = sycl::memory_order::relaxed) {
  260 |   auto atm = sycl::atomic_ref<T, memoryOrder, memoryScope, addressSpace>(*addr);
```
**EN:** Wraps backend-specific atomic behavior behind a compatibility-friendly CuTe interface.
**CN:** 在兼容层友好的 CuTe 接口后面封装后端特定的原子行为。

### Lines 262-264

```text
  262 |   atm.compare_exchange_strong(expected, desired, success, fail);
  263 |   return expected;
  264 | }
```
**EN:** Wraps backend-specific atomic behavior behind a compatibility-friendly CuTe interface.
**CN:** 在兼容层友好的 CuTe 接口后面封装后端特定的原子行为。

### Lines 266-289

```text
  266 | /// Atomically compare the value at \p addr to the value expected and exchange
  267 | /// with the value desired if the value at \p addr is equal to the value
  268 | /// expected. Returns the value at the \p addr before the call.
  269 | /// \param [in] addr The pointer to the data.
  270 | /// \param expected The value to compare against the value at \p addr.
  271 | /// \param desired The value to assign to \p addr if the value at \p addr is
  272 | /// expected.
  273 | /// \param success The memory ordering used when comparison succeeds.
  274 | /// \param fail The memory ordering used when comparison fails.
  275 | /// \returns The value at the \p addr before the call.
  276 | template <sycl::access::address_space addressSpace =
  277 |               sycl::access::address_space::generic_space,
  278 |           sycl::memory_order memoryOrder = sycl::memory_order::relaxed,
  279 |           sycl::memory_scope memoryScope = sycl::memory_scope::device,
  280 |           typename T>
  281 | T atomic_compare_exchange_strong(
  282 |     T *addr, type_identity_t<T> expected, type_identity_t<T> desired,
  283 |     sycl::memory_order success = sycl::memory_order::relaxed,
  284 |     sycl::memory_order fail = sycl::memory_order::relaxed) {
  285 |   auto atm =
  286 |       sycl::atomic_ref<T, memoryOrder, memoryScope, addressSpace>(addr[0]);
  287 |   atm.compare_exchange_strong(expected, desired, success, fail);
  288 |   return expected;
  289 | }
```
**EN:** Wraps backend-specific atomic behavior behind a compatibility-friendly CuTe interface.
**CN:** 在兼容层友好的 CuTe 接口后面封装后端特定的原子行为。

### Lines 291-302

```text
  291 | /// Atomic extension to implement standard APIs in std::atomic
  292 | namespace detail {
  293 | template <typename T> struct IsValidAtomicType {
  294 |   static constexpr bool value =
  295 |       (std::is_same<T, int>::value || std::is_same<T, unsigned int>::value ||
  296 |        std::is_same<T, long>::value || std::is_same<T, unsigned long>::value ||
  297 |        std::is_same<T, long long>::value ||
  298 |        std::is_same<T, unsigned long long>::value ||
  299 |        std::is_same<T, float>::value || std::is_same<T, double>::value ||
  300 |        std::is_pointer<T>::value);
  301 | };
  302 | } // namespace detail
```
**EN:** Wraps backend-specific atomic behavior behind a compatibility-friendly CuTe interface.
**CN:** 在兼容层友好的 CuTe 接口后面封装后端特定的原子行为。

### Lines 304-315

```text
  304 | template <typename T,
  305 |           sycl::memory_scope DefaultScope = sycl::memory_scope::system,
  306 |           sycl::memory_order DefaultOrder = sycl::memory_order::seq_cst,
  307 |           sycl::access::address_space Space =
  308 |               sycl::access::address_space::generic_space>
  309 | class atomic {
  310 |   static_assert(
  311 |       detail::IsValidAtomicType<T>::value,
  312 |       "Invalid atomic type.  Valid types are int, unsigned int, long, "
  313 |       "unsigned long, long long, unsigned long long, float, double "
  314 |       "and pointer types");
  315 |   T __d;
```
**EN:** Wraps backend-specific atomic behavior behind a compatibility-friendly CuTe interface.
**CN:** 在兼容层友好的 CuTe 接口后面封装后端特定的原子行为。

### Lines 317-327

```text
  317 | public:
  318 |   /// default memory synchronization order
  319 |   static constexpr sycl::memory_order default_read_order =
  320 |       sycl::atomic_ref<T, DefaultOrder, DefaultScope,
  321 |                        Space>::default_read_order;
  322 |   static constexpr sycl::memory_order default_write_order =
  323 |       sycl::atomic_ref<T, DefaultOrder, DefaultScope,
  324 |                        Space>::default_write_order;
  325 |   static constexpr sycl::memory_scope default_scope = DefaultScope;
  326 |   static constexpr sycl::memory_order default_read_modify_write_order =
  327 |       DefaultOrder;
```
**EN:** Wraps backend-specific atomic behavior behind a compatibility-friendly CuTe interface.
**CN:** 在兼容层友好的 CuTe 接口后面封装后端特定的原子行为。

### Lines 329-332

```text
  329 |   /// Default constructor.
  330 |   constexpr atomic() noexcept = default;
  331 |   /// Constructor with initialize value.
  332 |   constexpr atomic(T d) noexcept : __d(d){};
```
**EN:** Wraps backend-specific atomic behavior behind a compatibility-friendly CuTe interface.
**CN:** 在兼容层友好的 CuTe 接口后面封装后端特定的原子行为。

### Lines 334-343

```text
  334 |   /// atomically replaces the value of the referenced object with a non-atomic
  335 |   /// argument
  336 |   /// \param operand The value to replace the pointed value.
  337 |   /// \param memoryOrder The memory ordering used.
  338 |   /// \param memoryScope The memory scope used.
  339 |   void store(T operand, sycl::memory_order memoryOrder = default_write_order,
  340 |              sycl::memory_scope memoryScope = default_scope) noexcept {
  341 |     sycl::atomic_ref<T, DefaultOrder, DefaultScope, Space> atm(__d);
  342 |     atm.store(operand, memoryOrder, memoryScope);
  343 |   }
```
**EN:** Wraps backend-specific atomic behavior behind a compatibility-friendly CuTe interface.
**CN:** 在兼容层友好的 CuTe 接口后面封装后端特定的原子行为。

### Lines 345-354

```text
  345 |   /// atomically obtains the value of the referenced object
  346 |   /// \param memoryOrder The memory ordering used.
  347 |   /// \param memoryScope The memory scope used.
  348 |   /// \returns The value of the referenced object
  349 |   T load(sycl::memory_order memoryOrder = default_read_order,
  350 |          sycl::memory_scope memoryScope = default_scope) const noexcept {
  351 |     sycl::atomic_ref<T, DefaultOrder, DefaultScope, Space> atm(
  352 |         const_cast<T &>(__d));
  353 |     return atm.load(memoryOrder, memoryScope);
  354 |   }
```
**EN:** Wraps backend-specific atomic behavior behind a compatibility-friendly CuTe interface.
**CN:** 在兼容层友好的 CuTe 接口后面封装后端特定的原子行为。

### Lines 356-368

```text
  356 |   /// atomically replaces the value of the referenced object and obtains the
  357 |   /// value held previously
  358 |   /// \param operand The value to replace the pointed value.
  359 |   /// \param memoryOrder The memory ordering used.
  360 |   /// \param memoryScope The memory scope used.
  361 |   /// \returns The value of the referenced object before the call.
  362 |   T exchange(T operand,
  363 |              sycl::memory_order memoryOrder = default_read_modify_write_order,
  364 |              sycl::memory_scope memoryScope = default_scope) noexcept {
  365 | 
  366 |     sycl::atomic_ref<T, DefaultOrder, DefaultScope, Space> atm(__d);
  367 |     return atm.exchange(operand, memoryOrder, memoryScope);
  368 |   }
```
**EN:** Wraps backend-specific atomic behavior behind a compatibility-friendly CuTe interface.
**CN:** 在兼容层友好的 CuTe 接口后面封装后端特定的原子行为。

### Lines 370-405

```text
  370 |   /// atomically compares the value of the referenced object with non-atomic
  371 |   /// argument and performs atomic exchange if equal or atomic load if not
  372 |   /// \param expected The value expected to be found in the object referenced by
  373 |   /// the atomic_ref object
  374 |   /// \param desired  The value to store in the referenced object if it is as
  375 |   /// expected
  376 |   /// \param success The memory models for the read-modify-write
  377 |   /// \param failure The memory models for load operations
  378 |   /// \param memoryScope The memory scope used.
  379 |   /// \returns true if the referenced object was successfully changed, false
  380 |   /// otherwise.
  381 |   bool compare_exchange_weak(
  382 |       T &expected, T desired, sycl::memory_order success,
  383 |       sycl::memory_order failure,
  384 |       sycl::memory_scope memoryScope = default_scope) noexcept {
  385 |     sycl::atomic_ref<T, DefaultOrder, DefaultScope, Space> atm(__d);
  386 |     return atm.compare_exchange_weak(expected, desired, success, failure,
  387 |                                      memoryScope);
  388 |   }
  389 |   /// \param expected The value expected to be found in the object referenced by
  390 |   /// the atomic_ref object
  391 |   /// \param desired  The value to store in the referenced
  392 |   /// object if it is as expected
  393 |   /// \param memoryOrder 	The memory synchronization ordering for
  394 |   /// operations
  395 |   /// \param memoryScope The memory scope used.
  396 |   /// \returns true if the referenced object was successfully
  397 |   /// changed, false otherwise.
  398 |   bool compare_exchange_weak(
  399 |       T &expected, T desired,
  400 |       sycl::memory_order memoryOrder = default_read_modify_write_order,
  401 |       sycl::memory_scope memoryScope = default_scope) noexcept {
  402 |     sycl::atomic_ref<T, DefaultOrder, DefaultScope, Space> atm(__d);
  403 |     return atm.compare_exchange_weak(expected, desired, memoryOrder,
  404 |                                      memoryScope);
  405 |   }
```
**EN:** Wraps backend-specific atomic behavior behind a compatibility-friendly CuTe interface.
**CN:** 在兼容层友好的 CuTe 接口后面封装后端特定的原子行为。

### Lines 407-444

```text
  407 |   /// atomically compares the value of the referenced object with non-atomic
  408 |   /// argument and performs atomic exchange if equal or atomic load if not
  409 |   /// \param expected The value expected to be found in the object referenced by
  410 |   /// the atomic_ref object
  411 |   /// \param desired  The value to store in the referenced
  412 |   /// object if it is as expected
  413 |   /// \param success The memory models for the
  414 |   /// read-modify-write
  415 |   /// \param failure The memory models for load operations
  416 |   /// \param memoryScope The memory scope used.
  417 |   /// \returns true if the referenced object was successfully changed, false
  418 |   /// otherwise.
  419 |   bool compare_exchange_strong(
  420 |       T &expected, T desired, sycl::memory_order success,
  421 |       sycl::memory_order failure,
  422 |       sycl::memory_scope memoryScope = default_scope) noexcept {
  423 | 
  424 |     sycl::atomic_ref<T, DefaultOrder, DefaultScope, Space> atm(__d);
  425 |     return atm.compare_exchange_strong(expected, desired, success, failure,
  426 |                                        memoryScope);
  427 |   }
  428 |   /// \param expected The value expected to be found in the object referenced by
  429 |   /// the atomic_ref object
  430 |   /// \param desired The value to store in the referenced
  431 |   /// object if it is as expected
  432 |   /// \param memoryOrder 	The memory synchronization ordering for
  433 |   /// operations
  434 |   /// \param memoryScope The memory scope used.
  435 |   /// \returns true if the referenced object was successfully changed, false
  436 |   /// otherwise.
  437 |   bool compare_exchange_strong(
  438 |       T &expected, T desired,
  439 |       sycl::memory_order memoryOrder = default_read_modify_write_order,
  440 |       sycl::memory_scope memoryScope = default_scope) noexcept {
  441 |     sycl::atomic_ref<T, DefaultOrder, DefaultScope, Space> atm(__d);
  442 |     return atm.compare_exchange_strong(expected, desired, memoryOrder,
  443 |                                        memoryScope);
  444 |   }
```
**EN:** Wraps backend-specific atomic behavior behind a compatibility-friendly CuTe interface.
**CN:** 在兼容层友好的 CuTe 接口后面封装后端特定的原子行为。

### Lines 446-458

```text
  446 |   /// atomically adds the argument to the value stored in the atomic object and
  447 |   /// obtains the value held previously
  448 |   /// \param operand 	The other argument of arithmetic addition
  449 |   /// \param memoryOrder The memory ordering used.
  450 |   /// \param memoryScope The memory scope used.
  451 |   /// \returns The value of the referenced object before the call.
  452 |   T fetch_add(arith_t<T> operand,
  453 |               sycl::memory_order memoryOrder = default_read_modify_write_order,
  454 |               sycl::memory_scope memoryScope = default_scope) noexcept {
  455 | 
  456 |     auto atm = sycl::atomic_ref<T, DefaultOrder, DefaultScope, Space>(__d);
  457 |     return atm.fetch_add(operand, memoryOrder, memoryScope);
  458 |   }
```
**EN:** Wraps backend-specific atomic behavior behind a compatibility-friendly CuTe interface.
**CN:** 在兼容层友好的 CuTe 接口后面封装后端特定的原子行为。

### Lines 460-475

```text
  460 |   /// atomically subtracts the argument from the value stored in the atomic
  461 |   /// object and obtains the value held previously
  462 |   /// \param operand 	The other argument of arithmetic subtraction
  463 |   /// \param memoryOrder The memory ordering used.
  464 |   /// \param memoryScope The memory scope used.
  465 |   /// \returns The value of the referenced object before the call.
  466 |   T fetch_sub(arith_t<T> operand,
  467 |               sycl::memory_order memoryOrder = default_read_modify_write_order,
  468 |               sycl::memory_scope memoryScope = default_scope) noexcept {
  469 | 
  470 |     auto atm = sycl::atomic_ref<T, DefaultOrder, DefaultScope, Space>(__d);
  471 |     return atm.fetch_sub(operand, memoryOrder, memoryScope);
  472 |   }
  473 | };
  475 | } // namespace compat
```
**EN:** Wraps backend-specific atomic behavior behind a compatibility-friendly CuTe interface.
**CN:** 在兼容层友好的 CuTe 接口后面封装后端特定的原子行为。

## Key Concepts / 关键概念

- Memory-space-aware pointer adaptation / 感知内存空间的指针适配
- Compile-time numeric metaprogramming / 编译期数值元编程
- Backend portability and SYCL-style compatibility / 后端可移植性与 SYCL 风格兼容层
- SYCL interoperability / SYCL 互操作
- Intel Xe-specific behavior / Intel Xe 特定行为
- Atomic synchronization primitives / 原子同步原语

## Dependencies / 依赖关系

- Direct includes / 直接包含:
  - `cassert`
  - `sycl/access/access.hpp`
  - `sycl/atomic_ref.hpp`
  - `sycl/memory_enums.hpp`
  - `sycl/multi_ptr.hpp`
  - `cute/util/compat/traits.hpp`
- Primary symbols / 主要符号: `IsValidAtomicType`, `atomic`, `atomic_fetch_compare_dec`, `atomic_fetch_compare_inc`, `store`, `compare_exchange_weak`, `compare_exchange_strong`
- Dependency role / 依赖角色: Sits between CuTe algorithms and backend APIs, normalizing device, kernel, launch, memory, and math behavior across supported targets. / 位于 CuTe 算法与后端 API 之间，统一不同目标上的设备、内核、启动、内存和数学行为。
