# print_svg.hpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `include/cute/util/print_svg.hpp`
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
   35 | #include <cute/atom/mma_atom.hpp>
   36 | #include <cute/atom/copy_atom.hpp>
   38 | #include <cute/layout.hpp>
   39 | #include <cute/tensor_impl.hpp>
```
**EN:** Sets up the header dependencies for this file by importing `cute/config.hpp`, `cute/atom/mma_atom.hpp`, `cute/atom/copy_atom.hpp`, `cute/layout.hpp`, `cute/tensor_impl.hpp`.
**CN:** 通过引入 `cute/config.hpp`, `cute/atom/mma_atom.hpp`, `cute/atom/copy_atom.hpp`, `cute/layout.hpp`, `cute/tensor_impl.hpp` 为该文件建立头文件依赖。

### Lines 41-42

```text
   41 | namespace cute
   42 | {
```
**EN:** Opens or closes namespace scope so the following declarations remain grouped under the intended CuTe or backend namespace hierarchy.
**CN:** 打开或关闭命名空间作用域，使后续声明保持在预期的 CuTe 或后端命名空间层级中。

### Lines 44-53

```text
   44 | ////////////////////////////////
   45 | // Common SVG Color utilities //
   46 | ////////////////////////////////
   48 | struct TSVGColor_White {
   49 |   CUTE_HOST_DEVICE char const*
   50 |   operator()(int idx) const {
   51 |     return "255,255,255";
   52 |   }
   53 | };
```
**EN:** Formats CuTe objects for human-readable inspection, often emitting structured text or visualization-oriented output.
**CN:** 把 CuTe 对象格式化为便于人工检查的输出，通常生成结构化文本或面向可视化的结果。

### Lines 55-62

```text
   55 | struct TSVGColor_BWx8 {
   56 |   CUTE_HOST_DEVICE char const*
   57 |   operator()(int idx) const {
   58 |     static char const* color_map[8] = {"255,255,255", "230,230,230", "205,205,205", "180,180,180",
   59 |                                        "155,155,155", "130,130,130", "105,105,105", "080,080,080"};
   60 |     return color_map[idx % 8];
   61 |   }
   62 | };
```
**EN:** Formats CuTe objects for human-readable inspection, often emitting structured text or visualization-oriented output.
**CN:** 把 CuTe 对象格式化为便于人工检查的输出，通常生成结构化文本或面向可视化的结果。

### Lines 64-71

```text
   64 | struct SVGColor_TV {
   65 |   CUTE_HOST_DEVICE char const*
   66 |   operator()(int tid, int vid) const {
   67 |     static char const* color_map[8] = {"175,175,255", "175,255,175", "255,255,175", "255,175,175",
   68 |                                        "210,210,255", "210,255,210", "255,255,210", "255,210,210"};
   69 |     return color_map[tid % 8];
   70 |   }
   71 | };
```
**EN:** Formats CuTe objects for human-readable inspection, often emitting structured text or visualization-oriented output.
**CN:** 把 CuTe 对象格式化为便于人工检查的输出，通常生成结构化文本或面向可视化的结果。

### Lines 73-77

```text
   73 | /////////////////////
   74 | // MMA Atom to SVG //
   75 | /////////////////////
   77 | namespace detail {
```
**EN:** Formats CuTe objects for human-readable inspection, often emitting structured text or visualization-oriented output.
**CN:** 把 CuTe 对象格式化为便于人工检查的输出，通常生成结构化文本或面向可视化的结果。

### Lines 79-224

```text
   79 | template <class LayoutC, class LayoutA, class LayoutB, class Tile_MNK,
   80 |           class SVGColorFn = SVGColor_TV>
   81 | CUTE_HOST_DEVICE
   82 | void
   83 | print_svg_mma(LayoutC const& C,
   84 |               LayoutA const& A,
   85 |               LayoutB const& B,
   86 |               Tile_MNK const& tile_mnk,
   87 |               SVGColorFn color = {})  // lambda(tid,vid) -> SVG color string
   88 | {
   89 |   CUTE_STATIC_ASSERT_V(rank(C) == Int<2>{});
   90 |   CUTE_STATIC_ASSERT_V(rank(A) == Int<2>{});
   91 |   CUTE_STATIC_ASSERT_V(rank(B) == Int<2>{});
   92 | 
   93 |   auto [M, N, K] = product_each(shape(tile_mnk));
   94 | 
   95 |   int cell_size = 20;
   96 | 
   97 |   int page_width  = (K + N + 2) * cell_size;
   98 |   int page_height = (K + M + 2) * cell_size;
   99 | 
  100 |   // Commented print
  101 |   printf("<!--  Tile: "); print(tile_mnk); printf("  -->\n");
  102 |   printf("<!--     A: "); print(A);        printf("  -->\n");
  103 |   printf("<!--     B: "); print(B);        printf("  -->\n");
  104 |   printf("<!--     C: "); print(C);        printf("  -->\n");
  105 | 
  106 |   // SVG Header
  107 |   printf("<svg width=\"100%%\" height=\"100%%\" viewBox=\"0 0 %d %d\" "
  108 |          "preserveAspectRatio=\"xMidYMid meet\" "
  109 |          "xmlns=\"http://www.w3.org/2000/svg\">\n",
  110 |          page_width, page_height);
  111 | 
  112 |   Tensor filled = make_tensor<bool>(make_shape(M, N, K));
  113 |   clear(filled);
  114 | 
  115 |   // --- Draw C ---
  116 |   for (int tid = 0; tid < size<0>(C); ++tid) {
  117 |     for (int vid = 0; vid < size<1>(C); ++vid) {
  118 |       auto [m, n] = C(tid, vid);
  119 |       if (!filled(m, n, 0)) {
  120 |         filled(m, n, 0) = true;
  121 | 
  122 |         int x = (n + K + 2) * cell_size;
  123 |         int y = (m + K + 2) * cell_size;
  124 | 
  125 |         printf("<rect x=\"%d\" y=\"%d\" width=\"%d\" height=\"%d\" "
  126 |               "fill=\"rgb(%s)\" stroke=\"black\"/>\n",
  127 |               x, y, cell_size, cell_size, color(tid,vid));
  128 |         printf("<text x=\"%d\" y=\"%d\" text-anchor=\"middle\" "
  129 |               "alignment-baseline=\"central\" font-size=\"8\">T%d</text>\n",
  130 |               x + cell_size/2, y + 1*cell_size/4, tid);
  131 |         printf("<text x=\"%d\" y=\"%d\" text-anchor=\"middle\" "
  132 |               "alignment-baseline=\"central\" font-size=\"8\">V%d</text>\n",
  133 |               x + cell_size/2, y + 3*cell_size/4, vid);
  134 |       }
  135 |     }
  136 |   }
  137 | 
  138 |   clear(filled);
  139 | 
  140 |   // --- Draw A ---
  141 |   for (int tid = 0; tid < size<0>(A); ++tid) {
  142 |     for (int vid = 0; vid < size<1>(A); ++vid) {
  143 |       auto [m, k] = A(tid, vid);
  144 |       if (!filled(m, 0, k)) {
  145 |         filled(m, 0, k) = true;
  146 | 
  147 |         int x = (k     + 1) * cell_size;
  148 |         int y = (m + K + 2) * cell_size;
  149 | 
  150 |         printf("<rect x=\"%d\" y=\"%d\" width=\"%d\" height=\"%d\" "
  151 |               "fill=\"rgb(%s)\" stroke=\"black\" />\n",
  152 |               x, y, cell_size, cell_size, color(tid,vid));
  153 |         printf("<text x=\"%d\" y=\"%d\" text-anchor=\"middle\" "
  154 |               "alignment-baseline=\"central\" font-size=\"8\">T%d</text>\n",
  155 |               x + cell_size/2, y + 1*cell_size/4, tid);
  156 |         printf("<text x=\"%d\" y=\"%d\" text-anchor=\"middle\" "
  157 |               "alignment-baseline=\"central\" font-size=\"8\">V%d</text>\n",
  158 |               x + cell_size/2, y + 3*cell_size/4, vid);
  159 |       }
  160 |     }
  161 |   }
  162 | 
  163 |   // A labels
  164 |   for (int m =  0, k = -1; m < M; ++m) {
  165 |     int x = (k     + 1) * cell_size;
  166 |     int y = (m + K + 2) * cell_size;
  167 |     printf("<text x=\"%d\" y=\"%d\" text-anchor=\"middle\" "
  168 |            "alignment-baseline=\"central\" font-size=\"12\">%d</text>\n",
  169 |            x + cell_size/2, y + cell_size/2, m);
  170 |   }
  171 |   for (int m = -1, k =  0; k < K; ++k) {
  172 |     int x = (k     + 1) * cell_size;
  173 |     int y = (m + K + 2) * cell_size;
  174 |     printf("<text x=\"%d\" y=\"%d\" text-anchor=\"middle\" "
  175 |            "alignment-baseline=\"central\" font-size=\"12\">%d</text>\n",
  176 |            x + cell_size/2, y + cell_size/2, k);
  177 |   }
  178 | 
  179 |   clear(filled);
  180 | 
  181 |   // --- Draw B ---
  182 |   for (int tid = 0; tid < size<0>(B); ++tid) {
  183 |     for (int vid = 0; vid < size<1>(B); ++vid) {
  184 |       auto [n, k] = B(tid, vid);
  185 |       if (!filled(0, n, k)) {
  186 |         filled(0, n, k) = true;
  187 | 
  188 |         int x = (n + K + 2) * cell_size;
  189 |         int y = (k     + 1) * cell_size;
  190 | 
  191 |         printf("<rect x=\"%d\" y=\"%d\" width=\"%d\" height=\"%d\" "
  192 |               "fill=\"rgb(%s)\" stroke=\"black\" />\n",
  193 |               x, y, cell_size, cell_size, color(tid,vid));
  194 |         printf("<text x=\"%d\" y=\"%d\" text-anchor=\"middle\" "
  195 |               "alignment-baseline=\"central\" font-size=\"8\">T%d</text>\n",
  196 |               x + cell_size/2, y + 1*cell_size/4, tid);
  197 |         printf("<text x=\"%d\" y=\"%d\" text-anchor=\"middle\" "
  198 |               "alignment-baseline=\"central\" font-size=\"8\">V%d</text>\n",
  199 |               x + cell_size/2, y + 3*cell_size/4, vid);
  200 |       }
  201 |     }
  202 |   }
  203 | 
  204 |   // B labels
  205 |   for (int n =  0, k = -1; n < N; ++n) {
  206 |     int x = (n + K + 2) * cell_size;
  207 |     int y = (k     + 1) * cell_size;
  208 |     printf("<text x=\"%d\" y=\"%d\" text-anchor=\"middle\" "
  209 |            "alignment-baseline=\"central\" font-size=\"12\">%d</text>\n",
  210 |            x + cell_size/2, y + cell_size/2, n);
  211 |   }
  212 |   for (int n = -1, k =  0; k < K; ++k) {
  213 |     int x = (n + K + 2) * cell_size;
  214 |     int y = (k     + 1) * cell_size;
  215 |     printf("<text x=\"%d\" y=\"%d\" text-anchor=\"middle\" "
  216 |            "alignment-baseline=\"central\" font-size=\"12\">%d</text>\n",
  217 |            x + cell_size/2, y + cell_size/2, k);
  218 |   }
  219 | 
  220 |   // SVG footer
  221 |   printf("</svg>\n");
  222 | }
  224 | } // end namespace detail
```
**EN:** Formats CuTe objects for human-readable inspection, often emitting structured text or visualization-oriented output.
**CN:** 把 CuTe 对象格式化为便于人工检查的输出，通常生成结构化文本或面向可视化的结果。

### Lines 226-234

```text
  226 | // MMA Atom to SVG
  227 | template <class... Args, class SVGColorFn = SVGColor_TV>
  228 | CUTE_HOST_DEVICE
  229 | void
  230 | print_svg(MMA_Atom<Args...> const& mma_atom,
  231 |           SVGColorFn color = {})             // lambda(thr_idx,val_idx) -> svg color string
  232 | {
  233 |   print_svg(make_tiled_mma(mma_atom));
  234 | }
```
**EN:** Formats CuTe objects for human-readable inspection, often emitting structured text or visualization-oriented output.
**CN:** 把 CuTe 对象格式化为便于人工检查的输出，通常生成结构化文本或面向可视化的结果。

### Lines 236-257

```text
  236 | // TiledMMA to SVG
  237 | template <class... Args, class SVGColorFn = SVGColor_TV>
  238 | CUTE_HOST_DEVICE
  239 | void
  240 | print_svg(TiledMMA<Args...> const& mma,
  241 |           SVGColorFn color = {})             // lambda(thr_idx,val_idx) -> svg color string
  242 | {
  243 |   auto tile_mnk = tile_shape(mma);
  244 | 
  245 |   Tensor refC = make_identity_tensor(select<0,1>(tile_mnk));
  246 |   Tensor tensorC_TV = composition(refC, mma.get_layoutC_TV());
  247 | 
  248 |   Tensor refA = make_identity_tensor(select<0,2>(tile_mnk));
  249 |   Tensor tensorA_TV = composition(refA, mma.get_layoutA_TV());
  250 | 
  251 |   Tensor refB = make_identity_tensor(select<1,2>(tile_mnk));
  252 |   Tensor tensorB_TV = composition(refB, mma.get_layoutB_TV());
  253 | 
  254 |   detail::print_svg_mma(tensorC_TV, tensorA_TV, tensorB_TV, tile_mnk, color);
  255 | }
  257 | } // end namespace cute
```
**EN:** Formats CuTe objects for human-readable inspection, often emitting structured text or visualization-oriented output.
**CN:** 把 CuTe 对象格式化为便于人工检查的输出，通常生成结构化文本或面向可视化的结果。

## Key Concepts / 关键概念

- Hardware MMA traits and instruction mapping / 硬件 MMA traits 与指令映射
- Layout algebra and coordinate linearization / 布局代数与坐标线性化
- Tensor views, tiling, and partitioning / 张量视图、平铺与分块
- Compile-time numeric metaprogramming / 编译期数值元编程
- Intel Xe-specific behavior / Intel Xe 特定行为
- Diagnostics and visualization output / 诊断与可视化输出

## Dependencies / 依赖关系

- Direct includes / 直接包含:
  - `cute/config.hpp`
  - `cute/atom/mma_atom.hpp`
  - `cute/atom/copy_atom.hpp`
  - `cute/layout.hpp`
  - `cute/tensor_impl.hpp`
- Primary symbols / 主要符号: `TSVGColor_White`, `TSVGColor_BWx8`, `SVGColor_TV`, `LayoutC`, `LayoutA`, `LayoutB`, `Tile_MNK`, `SVGColorFn`
- Dependency role / 依赖角色: Depends on low-level architecture instruction wrappers and is consumed by higher-level `MMA_Atom`, tiling, and kernel-building code. / 依赖底层架构指令包装器，并被更高层的 `MMA_Atom`、平铺逻辑和内核构建代码消费。
