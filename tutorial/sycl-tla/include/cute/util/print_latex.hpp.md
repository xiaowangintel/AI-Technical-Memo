# print_latex.hpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `include/cute/util/print_latex.hpp`
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
   44 | ///////////////////////////////////////
   45 | // Common LaTeX TikZ Color utilities //
   46 | ///////////////////////////////////////
   48 | struct TikzColor_White {
   49 |   CUTE_HOST_DEVICE char const*
   50 |   operator()(int idx) const {
   51 |     return "white";
   52 |   }
   53 | };
```
**EN:** Formats CuTe objects for human-readable inspection, often emitting structured text or visualization-oriented output.
**CN:** 把 CuTe 对象格式化为便于人工检查的输出，通常生成结构化文本或面向可视化的结果。

### Lines 55-62

```text
   55 | struct TikzColor_BWx8 {
   56 |   CUTE_HOST_DEVICE char const*
   57 |   operator()(int idx) const {
   58 |     static char const* color_map[8] = {"black!00", "black!40", "black!20", "black!60",
   59 |                                        "black!10", "black!50", "black!30", "black!70"};
   60 |     return color_map[idx % 8];
   61 |   }
   62 | };
```
**EN:** Formats CuTe objects for human-readable inspection, often emitting structured text or visualization-oriented output.
**CN:** 把 CuTe 对象格式化为便于人工检查的输出，通常生成结构化文本或面向可视化的结果。

### Lines 64-77

```text
   64 | struct TikzColor_TV {
   65 |   CUTE_HOST_DEVICE char const*
   66 |   operator()(int tid, int vid) const {
   67 |     static char const* color_map[8] = {"{rgb,255:red,175;green,175;blue,255}",
   68 |                                        "{rgb,255:red,175;green,255;blue,175}",
   69 |                                        "{rgb,255:red,255;green,255;blue,175}",
   70 |                                        "{rgb,255:red,255;green,175;blue,175}",
   71 |                                        "{rgb,255:red,210;green,210;blue,255}",
   72 |                                        "{rgb,255:red,210;green,255;blue,210}",
   73 |                                        "{rgb,255:red,255;green,255;blue,210}",
   74 |                                        "{rgb,255:red,255;green,210;blue,210}"};
   75 |     return color_map[tid % 8];
   76 |   }
   77 | };
```
**EN:** Formats CuTe objects for human-readable inspection, often emitting structured text or visualization-oriented output.
**CN:** 把 CuTe 对象格式化为便于人工检查的输出，通常生成结构化文本或面向可视化的结果。

### Lines 79-124

```text
   79 | /////////////////////////////
   80 | // Layout 2D to LaTeX TikZ //
   81 | /////////////////////////////
   83 | template <class LayoutA, class TikzColorFn = TikzColor_BWx8>
   84 | CUTE_HOST_DEVICE
   85 | void
   86 | print_latex(LayoutA const& layout_a,   // (m,n) -> idx
   87 |             TikzColorFn color = {})    // lambda(idx) -> tikz color string
   88 | {
   89 |   CUTE_STATIC_ASSERT_V(rank(layout_a) <= Int<2>{});
   90 |   auto layout = append<2>(layout_a, Layout<_1,_0>{});
   91 | 
   92 |   // Commented print(layout)
   93 |   printf("%% Layout: "); print(layout); printf("\n");
   94 |   // Header
   95 |   printf("\\documentclass[convert]{standalone}\n"
   96 |          "\\usepackage{tikz}\n\n"
   97 |          "\\begin{document}\n"
   98 |          "\\begin{tikzpicture}[x={(0cm,-1cm)},y={(1cm,0cm)},every node/.style={minimum size=1cm, outer sep=0pt}]\n\n");
   99 | 
  100 |   auto [M, N] = product_each(shape(layout));
  101 | 
  102 |   // Layout
  103 |   for (int m = 0; m < M; ++m) {
  104 |     for (int n = 0; n < N; ++n) {
  105 |       int idx = layout(m,n);
  106 |       printf("\\node[fill=%s] at (%d,%d) {%d};\n",
  107 |              color(idx), m, n, idx);
  108 |     }
  109 |   }
  110 |   // Grid
  111 |   printf("\\draw[color=black,thick,shift={(-0.5,-0.5)}] (0,0) grid (%d,%d);\n\n",
  112 |          int(M), int(N));
  113 |   // Labels
  114 |   for (int m =  0, n = -1; m < M; ++m) {
  115 |     printf("\\node at (%d,%d) {\\Large{\\texttt{%d}}};\n", m, n, m);
  116 |   }
  117 |   for (int m = -1, n =  0; n < N; ++n) {
  118 |     printf("\\node at (%d,%d) {\\Large{\\texttt{%d}}};\n", m, n, n);
  119 |   }
  120 | 
  121 |   // Footer
  122 |   printf("\\end{tikzpicture}\n"
  123 |          "\\end{document}\n");
  124 | }
```
**EN:** Formats CuTe objects for human-readable inspection, often emitting structured text or visualization-oriented output.
**CN:** 把 CuTe 对象格式化为便于人工检查的输出，通常生成结构化文本或面向可视化的结果。

### Lines 126-133

```text
  126 | template <class SwizzleFn, int B, class Layout, class TikzColorFn = TikzColor_BWx8>
  127 | CUTE_HOST_DEVICE
  128 | void
  129 | print_latex(ComposedLayout<SwizzleFn,smem_ptr_flag_bits<B>,Layout> const& layout,
  130 |             TikzColorFn color = {})    // lambda(idx) -> tikz color string)
  131 | {
  132 |   print_latex(as_position_independent_swizzle_layout(layout), color);
  133 | }
```
**EN:** Defines shared-memory pointer wrappers and constructors, including overloads that compose with swizzle-aware addressing.
**CN:** 定义共享内存指针包装器与构造函数，并包含可与 swizzle 感知寻址组合的重载。

### Lines 135-186

```text
  135 | ///////////////////////////////
  136 | // LayoutTV 2D to LaTeX TikZ //
  137 | ///////////////////////////////
  139 | template <class LayoutTV, class Tile_MN,
  140 |           class TikzColorFn = TikzColor_TV>
  141 | CUTE_HOST_DEVICE
  142 | void
  143 | print_latex_tv(LayoutTV const& layout_tv,   // (t,v) -> m,n coord
  144 |                Tile_MN  const& tile_mn,     // (M,N)
  145 |                TikzColorFn     color = {})  // (t,v) -> color
  146 | {
  147 |   CUTE_STATIC_ASSERT_V(rank(layout_tv) == Int<2>{});
  148 | 
  149 |   // Commented prints
  150 |   printf("%% Layout TV: "); print(layout_tv); printf("\n");
  151 |   // Header
  152 |   printf("\\documentclass[convert]{standalone}\n"
  153 |          "\\usepackage{tikz}\n\n"
  154 |          "\\begin{document}\n"
  155 |          "\\begin{tikzpicture}[x={(0cm,-1cm)},y={(1cm,0cm)},every node/.style={minimum size=1cm, outer sep=0pt}]\n\n");
  156 | 
  157 |   auto [M, N] = product_each(shape(tile_mn));
  158 |   Tensor filled = make_tensor<bool>(make_shape(M, N));
  159 |   clear(filled);
  160 | 
  161 |   // Layout
  162 |   for (int tid = 0; tid < size<0>(layout_tv); ++tid) {
  163 |     for (int vid = 0; vid < size<1>(layout_tv); ++vid) {
  164 |       auto [m, n] = layout_tv(tid, vid);
  165 |       if (not filled(m, n)) {
  166 |         filled(m, n) = true;
  167 |         printf("\\node[fill=%s] at (%d,%d) {\\shortstack{T%d \\\\ V%d}};\n",
  168 |                 color(tid, vid),
  169 |                 int(m), int(n),
  170 |                 tid, vid);
  171 |       }
  172 |     }
  173 |   }
  174 |   // Grid
  175 |   printf("\\draw[color=black,thick,shift={(-0.5,-0.5)}] (0,0) grid (%d,%d);\n\n", int(M), int(N));
  176 |   // Labels
  177 |   for (int m = 0, n = -1; m < M; ++m) {
  178 |     printf("\\node at (%d,%d) {\\Large{\\texttt{%d}}};\n", m, n, m);
  179 |   }
  180 |   for (int n = 0, m = -1; n < N; ++n) {
  181 |     printf("\\node at (%d,%d) {\\Large{\\texttt{%d}}};\n", m, n, n);
  182 |   }
  183 |   // Footer
  184 |   printf("\\end{tikzpicture}\n"
  185 |          "\\end{document}\n");
  186 | }
```
**EN:** Formats CuTe objects for human-readable inspection, often emitting structured text or visualization-oriented output.
**CN:** 把 CuTe 对象格式化为便于人工检查的输出，通常生成结构化文本或面向可视化的结果。

### Lines 188-192

```text
  188 | ////////////////////////////
  189 | // MMA Atom to LaTeX TikZ //
  190 | ////////////////////////////
  192 | namespace detail {
```
**EN:** Formats CuTe objects for human-readable inspection, often emitting structured text or visualization-oriented output.
**CN:** 把 CuTe 对象格式化为便于人工检查的输出，通常生成结构化文本或面向可视化的结果。

### Lines 194-296

```text
  194 | template <class LayoutC, class LayoutA, class LayoutB, class Tile_MNK,
  195 |           class TikzColorFn = TikzColor_TV>
  196 | CUTE_HOST_DEVICE
  197 | void
  198 | print_latex_mma(LayoutC const& C,         // (tid,vid) -> (m,n) coord
  199 |                 LayoutA const& A,         // (tid,vid) -> (m,k) coord
  200 |                 LayoutB const& B,         // (tid,vid) -> (n,k) coord
  201 |                 Tile_MNK const& tile_mnk, // (M,N,K)
  202 |                 TikzColorFn color = {})   // lambda(tid,vid) -> tikz color string
  203 | {
  204 |   CUTE_STATIC_ASSERT_V(rank(C) == Int<2>{});
  205 |   CUTE_STATIC_ASSERT_V(rank(A) == Int<2>{});
  206 |   CUTE_STATIC_ASSERT_V(rank(B) == Int<2>{});
  207 | 
  208 |   // Commented prints
  209 |   printf("%% LayoutC: "); print(C); printf("\n");
  210 |   printf("%% LayoutA: "); print(A); printf("\n");
  211 |   printf("%% LayoutB: "); print(B); printf("\n");
  212 |   // Header
  213 |   printf("\\documentclass[convert]{standalone}\n"
  214 |          "\\usepackage{tikz}\n\n"
  215 |          "\\begin{document}\n"
  216 |          "\\begin{tikzpicture}[x={(0cm,-1cm)},y={(1cm,0cm)},every node/.style={minimum size=1cm, outer sep=0pt}]\n\n");
  217 | 
  218 |   auto [M, N, K] = product_each(shape(tile_mnk));
  219 |   Tensor filled = make_tensor<bool>(make_shape(M, N, K));
  220 |   clear(filled);
  221 | 
  222 |   // C starting at 0,0
  223 |   for (int tid = 0; tid < size<0>(C); ++tid) {
  224 |     for (int vid = 0; vid < size<1>(C); ++vid) {
  225 |       auto [m, n] = C(tid, vid);
  226 |       if (not filled(m, n, 0)) {
  227 |         filled(m, n, 0) = true;
  228 |         printf("\\node[fill=%s] at (%d,%d) {\\shortstack{T%d \\\\ V%d}};\n",
  229 |                color(tid, vid),
  230 |                int(m), int(n),
  231 |                tid, vid);
  232 |       }
  233 |     }
  234 |   }
  235 |   // Grid
  236 |   printf("\\draw[color=black,thick,shift={(-0.5,-0.5)}] (%d,%d) grid (%d,%d);\n\n",
  237 |           0, 0, int(M), int(N));
  238 | 
  239 |   clear(filled);
  240 | 
  241 |   // A starting at 0,-K-1
  242 |   for (int tid = 0; tid < size<0>(A); ++tid) {
  243 |     for (int vid = 0; vid < size<1>(A); ++vid) {
  244 |       auto [m, k] = A(tid, vid);
  245 |       if (not filled(m, 0, k)) {
  246 |         filled(m, 0, k) = true;
  247 |         printf("\\node[fill=%s] at (%d,%d) {\\shortstack{T%d \\\\ V%d}};\n",
  248 |                color(tid, vid),
  249 |                int(m), int(k-K-1),
  250 |                tid, vid);
  251 |       }
  252 |     }
  253 |   }
  254 |   // Grid
  255 |   printf("\\draw[color=black,thick,shift={(-0.5,-0.5)}] (%d,%d) grid (%d,%d);\n\n",
  256 |          0, -int(K)-1, int(M), -1);
  257 |   // A labels
  258 |   for (int m =  0, k = -1; m < M; ++m) {
  259 |     printf("\\node at (%d,%d) {\\Large{\\texttt{%d}}};\n", m, int(k-K-1), m);
  260 |   }
  261 |   for (int m = -1, k =  0; k < K; ++k) {
  262 |     printf("\\node at (%d,%d) {\\Large{\\texttt{%d}}};\n", m, int(k-K-1), k);
  263 |   }
  264 | 
  265 |   clear(filled);
  266 | 
  267 |   // B starting at -K-1,0
  268 |   for (int tid = 0; tid < size<0>(B); ++tid) {
  269 |     for (int vid = 0; vid < size<1>(B); ++vid) {
  270 |       auto [n, k] = B(tid, vid);
  271 |       if (not filled(0, n, k)) {
  272 |         filled(0, n, k) = true;
  273 |         printf("\\node[fill=%s] at (%d,%d) {\\shortstack{T%d \\\\ V%d}};\n",
  274 |                color(tid, vid),
  275 |                int(k)-int(K)-1, int(n),
  276 |                tid, vid);
  277 |       }
  278 |     }
  279 |   }
  280 |   // Grid
  281 |   printf("\\draw[color=black,thick,shift={(-0.5,-0.5)}] (%d,%d) grid (%d,%d);\n\n",
  282 |          -int(K)-1, 0, -1, int(N));
  283 |   // B labels
  284 |   for (int n =  0, k = -1; n < N; ++n) {
  285 |     printf("\\node at (%d,%d) {\\Large{\\texttt{%d}}};\n", int(k-K-1), n, n);
  286 |   }
  287 |   for (int n = -1, k =  0; k < K; ++k) {
  288 |     printf("\\node at (%d,%d) {\\Large{\\texttt{%d}}};\n", int(k-K-1), n, k);
  289 |   }
  290 | 
  291 |   // Footer
  292 |   printf("\\end{tikzpicture}\n"
  293 |          "\\end{document}\n");
  294 | }
  296 | } // end namespace detail
```
**EN:** Formats CuTe objects for human-readable inspection, often emitting structured text or visualization-oriented output.
**CN:** 把 CuTe 对象格式化为便于人工检查的输出，通常生成结构化文本或面向可视化的结果。

### Lines 298-306

```text
  298 | // MMA Atom to LaTeX TikZ
  299 | template <class... Args, class TikzColorFn = TikzColor_TV>
  300 | CUTE_HOST_DEVICE
  301 | void
  302 | print_latex(MMA_Atom<Args...> const& mma_atom,
  303 |             TikzColorFn color = {})             // lambda(thr_idx,val_idx) -> tikz color string
  304 | {
  305 |   print_latex(make_tiled_mma(mma_atom));
  306 | }
```
**EN:** Formats CuTe objects for human-readable inspection, often emitting structured text or visualization-oriented output.
**CN:** 把 CuTe 对象格式化为便于人工检查的输出，通常生成结构化文本或面向可视化的结果。

### Lines 308-327

```text
  308 | // TiledMMA to LaTeX TikZ
  309 | template <class... Args, class TikzColorFn = TikzColor_TV>
  310 | CUTE_HOST_DEVICE
  311 | void
  312 | print_latex(TiledMMA<Args...> const& mma,
  313 |             TikzColorFn color = {})             // lambda(thr_idx,val_idx) -> tikz color string
  314 | {
  315 |   auto tile_mnk = tile_shape(mma);
  316 | 
  317 |   Tensor refC = make_identity_tensor(select<0,1>(tile_mnk));
  318 |   Tensor tensorC_TV = composition(refC, mma.get_layoutC_TV());
  319 | 
  320 |   Tensor refA = make_identity_tensor(select<0,2>(tile_mnk));
  321 |   Tensor tensorA_TV = composition(refA, mma.get_layoutA_TV());
  322 | 
  323 |   Tensor refB = make_identity_tensor(select<1,2>(tile_mnk));
  324 |   Tensor tensorB_TV = composition(refB, mma.get_layoutB_TV());
  325 | 
  326 |   detail::print_latex_mma(tensorC_TV, tensorA_TV, tensorB_TV, tile_mnk, color);
  327 | }
```
**EN:** Formats CuTe objects for human-readable inspection, often emitting structured text or visualization-oriented output.
**CN:** 把 CuTe 对象格式化为便于人工检查的输出，通常生成结构化文本或面向可视化的结果。

### Lines 329-333

```text
  329 | ////////////////////////////
  330 | // CopyAtom to LaTeX TikZ //
  331 | ////////////////////////////
  333 | namespace detail {
```
**EN:** Formats CuTe objects for human-readable inspection, often emitting structured text or visualization-oriented output.
**CN:** 把 CuTe 对象格式化为便于人工检查的输出，通常生成结构化文本或面向可视化的结果。

### Lines 335-417

```text
  335 | // Generic TV Layout to LaTeX TikZ
  336 | template <class LayoutS_TV, class LayoutD_TV, class Tile_MN,
  337 |           class TikzColorFn = TikzColor_TV>
  338 | CUTE_HOST_DEVICE
  339 | void
  340 | print_latex_copy(LayoutS_TV const& S,            // (t,v) -> m,n coord
  341 |                  LayoutD_TV const& D,            // (t,v) -> m,n coord
  342 |                  Tile_MN const& tile_mn,         // (M,N)
  343 |                  TikzColorFn       color = {})   // (t,v) -> color
  344 | {
  345 |   CUTE_STATIC_ASSERT_V(rank(S) == Int<2>{});
  346 |   CUTE_STATIC_ASSERT_V(rank(D) == Int<2>{});
  347 | 
  348 |   // Commented prints
  349 |   printf("%% Layout S TV: "); print(S); printf("\n");
  350 |   printf("%% Layout D TV: "); print(D); printf("\n");
  351 | 
  352 |   // Header
  353 |   printf("\\documentclass[convert]{standalone}\n"
  354 |          "\\usepackage{tikz}\n\n"
  355 |          "\\begin{document}\n"
  356 |          "\\begin{tikzpicture}[x={(0cm,-1cm)},y={(1cm,0cm)},every node/.style={minimum size=1cm, outer sep=0pt}]\n\n");
  357 | 
  358 |   auto [M, N] = product_each(shape(tile_mn));
  359 |   Tensor filled = make_tensor<bool>(make_shape(M, N));
  360 |   clear(filled);
  361 | 
  362 |   // S starting at 0,0
  363 |   for (int tid = 0; tid < size<0>(S); ++tid) {
  364 |     for (int vid = 0; vid < size<1>(S); ++vid) {
  365 |       auto [m, n] = S(tid, vid);
  366 |       if (not filled(m, n)) {
  367 |         filled(m, n) = true;
  368 |         printf("\\node[fill=%s] at (%d,%d) {\\shortstack{T%d \\\\ V%d}};\n",
  369 |               color(tid, vid),
  370 |               int(m), int(n),
  371 |               tid, vid);
  372 |       }
  373 |     }
  374 |   }
  375 |   // Grid
  376 |   printf("\\draw[color=black,thick,shift={(-0.5,-0.5)}] (%d,%d) grid (%d,%d);\n\n",
  377 |          0, 0, int(M), int(N));
  378 |   // S Labels
  379 |   for (int m =  0, n = -1; m < M; ++m) {
  380 |     printf("\\node at (%d,%d) {\\Large{\\texttt{%d}}};\n", m, n, m);
  381 |   }
  382 |   for (int m = -1, n =  0; n < N; ++n) {
  383 |     printf("\\node at (%d,%d) {\\Large{\\texttt{%d}}};\n", m, n, n);
  384 |   }
  385 | 
  386 |   clear(filled);
  387 | 
  388 |   // D starting at 0,N+3
  389 |   for (int tid = 0; tid < size<0>(D); ++tid) {
  390 |     for (int vid = 0; vid < size<1>(D); ++vid) {
  391 |       auto [m, n] = D(tid, vid);
  392 |       if (not filled(m, n)) {
  393 |         filled(m, n) = true;
  394 |         printf("\\node[fill=%s] at (%d,%d) {\\shortstack{T%d \\\\ V%d}};\n",
  395 |               color(tid, vid),
  396 |               int(m), int(n) + int(N) + 3,
  397 |               tid, vid);
  398 |       }
  399 |     }
  400 |   }
  401 |   // Grid
  402 |   printf("\\draw[color=black,thick,shift={(-0.5,-0.5)}] (%d,%d) grid (%d,%d);\n\n",
  403 |          0, int(N) + 3, int(M), int(N) + int(N) + 3);
  404 |   // D Labels
  405 |   for (int m =  0, n = N; m < M; ++m) {
  406 |     printf("\\node at (%d,%d) {\\Large{\\texttt{%d}}};\n", m, int(n+N+3), m);
  407 |   }
  408 |   for (int m = -1, n = 0; n < N; ++n) {
  409 |     printf("\\node at (%d,%d) {\\Large{\\texttt{%d}}};\n", m, int(n+N+3), n);
  410 |   }
  411 | 
  412 |   // Footer
  413 |   printf("\\end{tikzpicture}\n"
  414 |          "\\end{document}\n");
  415 | }
  417 | } // end namespace detail
```
**EN:** Formats CuTe objects for human-readable inspection, often emitting structured text or visualization-oriented output.
**CN:** 把 CuTe 对象格式化为便于人工检查的输出，通常生成结构化文本或面向可视化的结果。

### Lines 419-438

```text
  419 | // TiledCopy to LaTeX TikZ
  420 | template <class... Args, class TikzColorFn = TikzColor_TV>
  421 | CUTE_HOST_DEVICE
  422 | void
  423 | print_latex(TiledCopy<Args...> const& copy,
  424 |             TikzColorFn color = {})              // lambda(tid,vid) -> tikz color string
  425 | {
  426 |   auto tiler_mn = typename TiledCopy<Args...>::Tiler_MN{};
  427 |   auto tile_mn = product_each(shape(logical_divide(make_layout(Shape<_1,_1>{}), tiler_mn)));  // tile_shape
  428 | 
  429 |   Tensor refS = make_identity_tensor(tile_mn);
  430 |   Tensor layoutS_TV = copy.tidfrg_S(refS)(_,_,Int<0>{});
  431 | 
  432 |   Tensor refD = make_identity_tensor(tile_mn);
  433 |   Tensor layoutD_TV = copy.tidfrg_D(refD)(_,_,Int<0>{});
  434 | 
  435 |   detail::print_latex_copy(layoutS_TV, layoutD_TV, tile_mn, color);
  436 | }
  438 | } // end namespace cute
```
**EN:** Formats CuTe objects for human-readable inspection, often emitting structured text or visualization-oriented output.
**CN:** 把 CuTe 对象格式化为便于人工检查的输出，通常生成结构化文本或面向可视化的结果。

## Key Concepts / 关键概念

- Hardware MMA traits and instruction mapping / 硬件 MMA traits 与指令映射
- Layout algebra and coordinate linearization / 布局代数与坐标线性化
- Tensor views, tiling, and partitioning / 张量视图、平铺与分块
- Memory-space-aware pointer adaptation / 感知内存空间的指针适配
- Compile-time numeric metaprogramming / 编译期数值元编程
- Intel Xe-specific behavior / Intel Xe 特定行为

## Dependencies / 依赖关系

- Direct includes / 直接包含:
  - `cute/config.hpp`
  - `cute/atom/mma_atom.hpp`
  - `cute/atom/copy_atom.hpp`
  - `cute/layout.hpp`
  - `cute/tensor_impl.hpp`
- Primary symbols / 主要符号: `TikzColor_White`, `TikzColor_BWx8`, `TikzColor_TV`, `LayoutA`, `TikzColorFn`, `SwizzleFn`, `Layout`, `LayoutTV`
- Dependency role / 依赖角色: Depends on low-level architecture instruction wrappers and is consumed by higher-level `MMA_Atom`, tiling, and kernel-building code. / 依赖底层架构指令包装器，并被更高层的 `MMA_Atom`、平铺逻辑和内核构建代码消费。
