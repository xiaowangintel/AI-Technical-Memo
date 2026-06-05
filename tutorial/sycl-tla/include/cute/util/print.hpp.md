# print.hpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `include/cute/util/print.hpp`
- Purpose (EN): Implements text, LaTeX, SVG, and tensor-oriented printers used to visualize CuTe layouts and tensor structures.
- 作用 (CN): 实现文本、LaTeX、SVG 以及张量专用打印器，用于可视化 CuTe 布局和张量结构。

## Line-by-Line Analysis / 逐行分析

### Lines 1-29

```text
    1 | /***************************************************************************************************
    2 |  * Copyright (c) 2023 - 2025 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
    3 |  * SPDX-License-Identifier: BSD-3-Clause
    4 |  *
    5 |  * Redistribution and use in source and binary forms, with or without
    6 |  * modification, are permitted provided that the following conditions are met:
    7 |  *
    8 |  * 1. Redistributions of source code must retain the above copyright notice, this
    9 |  * list of conditions and the following disclaimer.
   10 |  *
   11 |  * 2. Redistributions in binary form must reproduce the above copyright notice,
   12 |  * this list of conditions and the following disclaimer in the documentation
   13 |  * and/or other materials provided with the distribution.
   14 |  *
   15 |  * 3. Neither the name of the copyright holder nor the names of its
   16 |  * contributors may be used to endorse or promote products derived from
   17 |  * this software without specific prior written permission.
   18 |  *
   19 |  * THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
   20 |  * AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
   21 |  * IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
   22 |  * DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE
   23 |  * FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
   24 |  * DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
   25 |  * SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
   26 |  * CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
   27 |  * OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
   28 |  * OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
   29 |  *
```
**EN:** Provides the license notice, copyright ownership, and redistribution terms for this header.
**CN:** 给出该头文件的许可证声明、版权归属以及再分发条款。

### Lines 30-31

```text
   30 |  **************************************************************************************************/
   31 | #pragma once
```
**EN:** Formats CuTe objects for human-readable inspection, often emitting structured text or visualization-oriented output.
**CN:** 把 CuTe 对象格式化为便于人工检查的输出，通常生成结构化文本或面向可视化的结果。

### Lines 33-39

```text
   33 | #include <cute/config.hpp>           // CUTE_HOST_DEVICE
   34 | #include <cute/util/type_traits.hpp> // cute::is_valid
   35 | #include <cute/numeric/numeric_types.hpp> 
   37 | #if defined(CUTLASS_ENABLE_SYCL) && !defined(__CUDA__)
   38 | #define printf sycl::ext::oneapi::experimental::printf
   39 | #endif // defined(CUTLASS_ENABLE_SYCL) && !defined(__CUDA__)
```
**EN:** Sets up the header dependencies for this file by importing `cute/config.hpp`, `cute/util/type_traits.hpp`, `cute/numeric/numeric_types.hpp`.
**CN:** 通过引入 `cute/config.hpp`, `cute/util/type_traits.hpp`, `cute/numeric/numeric_types.hpp` 为该文件建立头文件依赖。

### Lines 41-46

```text
   41 | //
   42 | // CUDA compatible print and printf
   43 | //
   45 | namespace cute
   46 | {
```
**EN:** Formats CuTe objects for human-readable inspection, often emitting structured text or visualization-oriented output.
**CN:** 把 CuTe 对象格式化为便于人工检查的输出，通常生成结构化文本或面向可视化的结果。

### Lines 48-62

```text
   48 | CUTE_HOST_DEVICE
   49 | int
   50 | num_digits(int x)
   51 | {
   52 |   return (x < 10 ? 1 :
   53 |           (x < 100 ? 2 :
   54 |            (x < 1000 ? 3 :
   55 |             (x < 10000 ? 4 :
   56 |              (x < 100000 ? 5 :
   57 |               (x < 1000000 ? 6 :
   58 |                (x < 10000000 ? 7 :
   59 |                 (x < 100000000 ? 8 :
   60 |                  (x < 1000000000 ? 9 :
   61 |                   10)))))))));
   62 | }
```
**EN:** Formats CuTe objects for human-readable inspection, often emitting structured text or visualization-oriented output.
**CN:** 把 CuTe 对象格式化为便于人工检查的输出，通常生成结构化文本或面向可视化的结果。

### Lines 64-72

```text
   64 | //
   65 | // print dispatcher
   66 | //
   68 | CUTE_HOST_DEVICE
   69 | void
   70 | print(char c) {
   71 |   printf("%c", c);
   72 | }
```
**EN:** Formats CuTe objects for human-readable inspection, often emitting structured text or visualization-oriented output.
**CN:** 把 CuTe 对象格式化为便于人工检查的输出，通常生成结构化文本或面向可视化的结果。

### Lines 74-78

```text
   74 | CUTE_HOST_DEVICE
   75 | void
   76 | print(signed char a) {
   77 |   printf("%d", static_cast<int>(a));
   78 | }
```
**EN:** Formats CuTe objects for human-readable inspection, often emitting structured text or visualization-oriented output.
**CN:** 把 CuTe 对象格式化为便于人工检查的输出，通常生成结构化文本或面向可视化的结果。

### Lines 80-84

```text
   80 | CUTE_HOST_DEVICE
   81 | void
   82 | print(unsigned char a) {
   83 |   printf("%u", static_cast<unsigned int>(a));
   84 | }
```
**EN:** Formats CuTe objects for human-readable inspection, often emitting structured text or visualization-oriented output.
**CN:** 把 CuTe 对象格式化为便于人工检查的输出，通常生成结构化文本或面向可视化的结果。

### Lines 86-90

```text
   86 | CUTE_HOST_DEVICE
   87 | void
   88 | print(short a) {
   89 |   printf("%hd", a);
   90 | }
```
**EN:** Formats CuTe objects for human-readable inspection, often emitting structured text or visualization-oriented output.
**CN:** 把 CuTe 对象格式化为便于人工检查的输出，通常生成结构化文本或面向可视化的结果。

### Lines 92-96

```text
   92 | CUTE_HOST_DEVICE
   93 | void
   94 | print(unsigned short a) {
   95 |   printf("%hu", a);
   96 | }
```
**EN:** Formats CuTe objects for human-readable inspection, often emitting structured text or visualization-oriented output.
**CN:** 把 CuTe 对象格式化为便于人工检查的输出，通常生成结构化文本或面向可视化的结果。

### Lines 98-102

```text
   98 | CUTE_HOST_DEVICE
   99 | void
  100 | print(int a) {
  101 |   printf("%d", a);
  102 | }
```
**EN:** Formats CuTe objects for human-readable inspection, often emitting structured text or visualization-oriented output.
**CN:** 把 CuTe 对象格式化为便于人工检查的输出，通常生成结构化文本或面向可视化的结果。

### Lines 104-108

```text
  104 | CUTE_HOST_DEVICE
  105 | void
  106 | print(uint1b_t a) {
  107 |   printf("%d", int(a));
  108 | }
```
**EN:** Formats CuTe objects for human-readable inspection, often emitting structured text or visualization-oriented output.
**CN:** 把 CuTe 对象格式化为便于人工检查的输出，通常生成结构化文本或面向可视化的结果。

### Lines 110-114

```text
  110 | CUTE_HOST_DEVICE
  111 | void
  112 | print(int2b_t a) {
  113 |   printf("%d", int(a));
  114 | }
```
**EN:** Formats CuTe objects for human-readable inspection, often emitting structured text or visualization-oriented output.
**CN:** 把 CuTe 对象格式化为便于人工检查的输出，通常生成结构化文本或面向可视化的结果。

### Lines 116-120

```text
  116 | CUTE_HOST_DEVICE
  117 | void
  118 | print(uint2b_t a) {
  119 |   printf("%d", int(a));
  120 | }
```
**EN:** Formats CuTe objects for human-readable inspection, often emitting structured text or visualization-oriented output.
**CN:** 把 CuTe 对象格式化为便于人工检查的输出，通常生成结构化文本或面向可视化的结果。

### Lines 122-126

```text
  122 | CUTE_HOST_DEVICE
  123 | void
  124 | print(int4b_t a) {
  125 |   printf("%d", int(a));
  126 | }
```
**EN:** Formats CuTe objects for human-readable inspection, often emitting structured text or visualization-oriented output.
**CN:** 把 CuTe 对象格式化为便于人工检查的输出，通常生成结构化文本或面向可视化的结果。

### Lines 128-132

```text
  128 | CUTE_HOST_DEVICE
  129 | void
  130 | print(uint4b_t a) {
  131 |   printf("%d", int(a));
  132 | }
```
**EN:** Formats CuTe objects for human-readable inspection, often emitting structured text or visualization-oriented output.
**CN:** 把 CuTe 对象格式化为便于人工检查的输出，通常生成结构化文本或面向可视化的结果。

### Lines 134-138

```text
  134 | CUTE_HOST_DEVICE
  135 | void
  136 | print(bin1_t a) {
  137 |   printf("%d", int(a));
  138 | }
```
**EN:** Formats CuTe objects for human-readable inspection, often emitting structured text or visualization-oriented output.
**CN:** 把 CuTe 对象格式化为便于人工检查的输出，通常生成结构化文本或面向可视化的结果。

### Lines 140-144

```text
  140 | CUTE_HOST_DEVICE
  141 | void
  142 | print(unsigned int a) {
  143 |   printf("%u", a);
  144 | }
```
**EN:** Formats CuTe objects for human-readable inspection, often emitting structured text or visualization-oriented output.
**CN:** 把 CuTe 对象格式化为便于人工检查的输出，通常生成结构化文本或面向可视化的结果。

### Lines 146-150

```text
  146 | CUTE_HOST_DEVICE
  147 | void
  148 | print(long a) {
  149 |   printf("%ld", a);
  150 | }
```
**EN:** Formats CuTe objects for human-readable inspection, often emitting structured text or visualization-oriented output.
**CN:** 把 CuTe 对象格式化为便于人工检查的输出，通常生成结构化文本或面向可视化的结果。

### Lines 152-156

```text
  152 | CUTE_HOST_DEVICE
  153 | void
  154 | print(unsigned long a) {
  155 |   printf("%lu", a);
  156 | }
```
**EN:** Formats CuTe objects for human-readable inspection, often emitting structured text or visualization-oriented output.
**CN:** 把 CuTe 对象格式化为便于人工检查的输出，通常生成结构化文本或面向可视化的结果。

### Lines 158-162

```text
  158 | CUTE_HOST_DEVICE
  159 | void
  160 | print(long long a) {
  161 |   printf("%lld", a);
  162 | }
```
**EN:** Formats CuTe objects for human-readable inspection, often emitting structured text or visualization-oriented output.
**CN:** 把 CuTe 对象格式化为便于人工检查的输出，通常生成结构化文本或面向可视化的结果。

### Lines 164-168

```text
  164 | CUTE_HOST_DEVICE
  165 | void
  166 | print(unsigned long long a) {
  167 |   printf("%llu", a);
  168 | }
```
**EN:** Formats CuTe objects for human-readable inspection, often emitting structured text or visualization-oriented output.
**CN:** 把 CuTe 对象格式化为便于人工检查的输出，通常生成结构化文本或面向可视化的结果。

### Lines 170-174

```text
  170 | CUTE_HOST_DEVICE
  171 | void
  172 | print(float a) {
  173 |   printf("%f", a);
  174 | }
```
**EN:** Formats CuTe objects for human-readable inspection, often emitting structured text or visualization-oriented output.
**CN:** 把 CuTe 对象格式化为便于人工检查的输出，通常生成结构化文本或面向可视化的结果。

### Lines 176-180

```text
  176 | CUTE_HOST_DEVICE
  177 | void
  178 | print(double a) {
  179 |   printf("%f", a);
  180 | }
```
**EN:** Formats CuTe objects for human-readable inspection, often emitting structured text or visualization-oriented output.
**CN:** 把 CuTe 对象格式化为便于人工检查的输出，通常生成结构化文本或面向可视化的结果。

### Lines 182-187

```text
  182 | template <class... T>
  183 | CUTE_HOST_DEVICE
  184 | void
  185 | print(char const* format, T const&... t) {
  186 |   printf(format, t...);
  187 | }
```
**EN:** Formats CuTe objects for human-readable inspection, often emitting structured text or visualization-oriented output.
**CN:** 把 CuTe 对象格式化为便于人工检查的输出，通常生成结构化文本或面向可视化的结果。

### Lines 189-193

```text
  189 | CUTE_HOST_DEVICE
  190 | void
  191 | print(char const* format) {
  192 |   printf("%s", format);
  193 | }
```
**EN:** Formats CuTe objects for human-readable inspection, often emitting structured text or visualization-oriented output.
**CN:** 把 CuTe 对象格式化为便于人工检查的输出，通常生成结构化文本或面向可视化的结果。

### Lines 195-208

```text
  195 | //
  196 | // pretty printing
  197 | //
  199 | CUTE_HOST_DEVICE void
  200 | pretty_print(uint1b_t a) {
  201 |   //TODO(Codeplay) Remove this once DPC++ bugfix is in release.
  202 | #if defined(SYCL_INTEL_TARGET)
  203 |   printf("%d", int(a));
  204 |   printf(" ");
  205 | #else
  206 |   printf("%*d", 3, int(a));
  207 | #endif
  208 | }
```
**EN:** Formats CuTe objects for human-readable inspection, often emitting structured text or visualization-oriented output.
**CN:** 把 CuTe 对象格式化为便于人工检查的输出，通常生成结构化文本或面向可视化的结果。

### Lines 210-219

```text
  210 | CUTE_HOST_DEVICE void
  211 | pretty_print(int2b_t a) {
  212 |   //TODO(Codeplay) Remove this once DPC++ bugfix is in release.
  213 | #if defined(SYCL_INTEL_TARGET)
  214 |   printf("%d", int(a));
  215 |   printf(" ");
  216 | #else
  217 |   printf("%*d", 5, int(a));
  218 | #endif
  219 | }
```
**EN:** Formats CuTe objects for human-readable inspection, often emitting structured text or visualization-oriented output.
**CN:** 把 CuTe 对象格式化为便于人工检查的输出，通常生成结构化文本或面向可视化的结果。

### Lines 221-230

```text
  221 | CUTE_HOST_DEVICE void
  222 | pretty_print(uint2b_t a) {
  223 |   //TODO(Codeplay) Remove this once DPC++ bugfix is in release.
  224 | #if defined(SYCL_INTEL_TARGET)
  225 |   printf("%d", int(a));
  226 |   printf(" ");
  227 | #else
  228 |   printf("%*d", 5, int(a));
  229 | #endif
  230 | }
```
**EN:** Formats CuTe objects for human-readable inspection, often emitting structured text or visualization-oriented output.
**CN:** 把 CuTe 对象格式化为便于人工检查的输出，通常生成结构化文本或面向可视化的结果。

### Lines 232-241

```text
  232 | CUTE_HOST_DEVICE void
  233 | pretty_print(int4b_t a) {
  234 |   //TODO(Codeplay) Remove this once DPC++ bugfix is in release.
  235 | #if defined(SYCL_INTEL_TARGET)
  236 |   printf("%d", int(a));
  237 |   printf(" ");
  238 | #else
  239 |   printf("%*d", 5, int(a));
  240 | #endif
  241 | }
```
**EN:** Formats CuTe objects for human-readable inspection, often emitting structured text or visualization-oriented output.
**CN:** 把 CuTe 对象格式化为便于人工检查的输出，通常生成结构化文本或面向可视化的结果。

### Lines 243-252

```text
  243 | CUTE_HOST_DEVICE void
  244 | pretty_print(uint4b_t a) {
  245 |   //TODO(Codeplay) Remove this once DPC++ bugfix is in release.
  246 | #if defined(SYCL_INTEL_TARGET)
  247 |   printf("%d", int(a));
  248 |   printf(" ");
  249 | #else
  250 |   printf("%*d", 5, int(a));
  251 | #endif
  252 | }
```
**EN:** Formats CuTe objects for human-readable inspection, often emitting structured text or visualization-oriented output.
**CN:** 把 CuTe 对象格式化为便于人工检查的输出，通常生成结构化文本或面向可视化的结果。

### Lines 254-263

```text
  254 | CUTE_HOST_DEVICE void
  255 | pretty_print(bool v) {
  256 |   //TODO(Codeplay) Remove this once DPC++ bugfix is in release.
  257 | #if defined(SYCL_INTEL_TARGET)
  258 |   printf("%d", int(v));
  259 |   printf(" ");
  260 | #else
  261 |   printf("%*d", 3, int(v));
  262 | #endif
  263 | }
```
**EN:** Formats CuTe objects for human-readable inspection, often emitting structured text or visualization-oriented output.
**CN:** 把 CuTe 对象格式化为便于人工检查的输出，通常生成结构化文本或面向可视化的结果。

### Lines 265-274

```text
  265 | CUTE_HOST_DEVICE void
  266 | pretty_print(int32_t v) {
  267 |   //TODO(Codeplay) Remove this once DPC++ bugfix is in release.
  268 | #if defined(SYCL_INTEL_TARGET)
  269 |   printf("%d", int(v));
  270 |   printf(" ");
  271 | #else
  272 |   printf("%*d", 5, v);
  273 | #endif
  274 | }
```
**EN:** Formats CuTe objects for human-readable inspection, often emitting structured text or visualization-oriented output.
**CN:** 把 CuTe 对象格式化为便于人工检查的输出，通常生成结构化文本或面向可视化的结果。

### Lines 276-285

```text
  276 | CUTE_HOST_DEVICE void
  277 | pretty_print(uint32_t v) {
  278 |   //TODO(Codeplay) Remove this once DPC++ bugfix is in release.
  279 | #if defined(SYCL_INTEL_TARGET)
  280 |   printf("%d", int(v));
  281 |   printf(" ");
  282 | #else
  283 |   printf("%*d", 5, v);
  284 | #endif
  285 | }
```
**EN:** Formats CuTe objects for human-readable inspection, often emitting structured text or visualization-oriented output.
**CN:** 把 CuTe 对象格式化为便于人工检查的输出，通常生成结构化文本或面向可视化的结果。

### Lines 287-296

```text
  287 | CUTE_HOST_DEVICE void
  288 | pretty_print(int64_t v) {
  289 |   //TODO(Codeplay) Remove this once DPC++ bugfix is in release.
  290 | #if defined(SYCL_INTEL_TARGET)
  291 |   printf("%lld", static_cast<long long>(v));
  292 |   printf(" ");
  293 | #else
  294 |   printf("%*lld", 5, static_cast<long long>(v));
  295 | #endif
  296 | }
```
**EN:** Formats CuTe objects for human-readable inspection, often emitting structured text or visualization-oriented output.
**CN:** 把 CuTe 对象格式化为便于人工检查的输出，通常生成结构化文本或面向可视化的结果。

### Lines 298-307

```text
  298 | CUTE_HOST_DEVICE void
  299 | pretty_print(uint64_t v) {
  300 |   //TODO(Codeplay) Remove this once DPC++ bugfix is in release.
  301 | #if defined(SYCL_INTEL_TARGET)
  302 |   printf("%llu", static_cast<unsigned long long>(v));
  303 |   printf(" ");
  304 | #else
  305 |   printf("%*llu", 5, static_cast<unsigned long long>(v));
  306 | #endif
  307 | }
```
**EN:** Formats CuTe objects for human-readable inspection, often emitting structured text or visualization-oriented output.
**CN:** 把 CuTe 对象格式化为便于人工检查的输出，通常生成结构化文本或面向可视化的结果。

### Lines 309-317

```text
  309 | CUTE_HOST_DEVICE void
  310 | pretty_print(float v) {
  311 | //TODO(Codeplay) Remove this once DPC++ bugfix is in release.
  312 | #if defined(SYCL_INTEL_TARGET)
  313 |   printf("%.2f", v);
  314 | #else 
  315 |   printf("%*.2e", 10, v);
  316 | #endif  // SYCL_INTEL_TARGET
  317 | }
```
**EN:** Formats CuTe objects for human-readable inspection, often emitting structured text or visualization-oriented output.
**CN:** 把 CuTe 对象格式化为便于人工检查的输出，通常生成结构化文本或面向可视化的结果。

### Lines 319-327

```text
  319 | CUTE_HOST_DEVICE void
  320 | pretty_print(double v) {
  321 |   //TODO(Codeplay) Remove this once DPC++ bugfix is in release.
  322 | #if defined(SYCL_INTEL_TARGET)
  323 |   printf("%.3e", v);
  324 | #else
  325 |   printf("%*.3e", 11, v);
  326 | #endif  // SYCL_INTEL_TARGET
  327 | }
```
**EN:** Formats CuTe objects for human-readable inspection, often emitting structured text or visualization-oriented output.
**CN:** 把 CuTe 对象格式化为便于人工检查的输出，通常生成结构化文本或面向可视化的结果。

### Lines 329-340

```text
  329 | template <class T>
  330 | CUTE_HOST_DEVICE void
  331 | pretty_print(T t) {
  332 |   constexpr auto has_print_exmy_base = cute::is_valid([](auto t) -> decltype(pretty_print_float_exmy_base(t)) {}, t);  
  333 |   if constexpr (has_print_exmy_base) {   
  334 |   pretty_print_float_exmy_base(t);       
  335 |   } else {                               
  336 |   printf("  "); print(t);
  337 |   }                                      
  338 | }
  340 | } // end namespace cute
```
**EN:** Formats CuTe objects for human-readable inspection, often emitting structured text or visualization-oriented output.
**CN:** 把 CuTe 对象格式化为便于人工检查的输出，通常生成结构化文本或面向可视化的结果。

## Key Concepts / 关键概念

- Compile-time numeric metaprogramming / 编译期数值元编程
- Backend portability and SYCL-style compatibility / 后端可移植性与 SYCL 风格兼容层
- SYCL interoperability / SYCL 互操作
- Intel Xe-specific behavior / Intel Xe 特定行为
- Diagnostics and visualization output / 诊断与可视化输出
- Header-only template specialization patterns / 纯头文件模板特化模式

## Dependencies / 依赖关系

- Direct includes / 直接包含:
  - `cute/config.hpp`
  - `cute/util/type_traits.hpp`
  - `cute/numeric/numeric_types.hpp`
- Primary symbols / 主要符号: `T`, `num_digits`, `print`, `pretty_print`
- Dependency role / 依赖角色: Sits between CuTe algorithms and backend APIs, normalizing device, kernel, launch, memory, and math behavior across supported targets. / 位于 CuTe 算法与后端 API 之间，统一不同目标上的设备、内核、启动、内存和数学行为。
