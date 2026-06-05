# benchmarks_sycl.hpp — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `benchmarks/gemm/legacy/benchmarks_sycl.hpp`
- **EN:** Large legacy SYCL GEMM benchmark catalog covering PVC/Xe BF16, FP16, split-K, stream-K, fused epilogues, and mixed-precision experiments.
- **CN:** 大型旧版 SYCL GEMM 基准目录，覆盖 PVC/Xe 上的 BF16、FP16、split-K、stream-K、融合 epilogue 以及混合精度实验。

## Line-by-Line Analysis / 逐行分析

### Lines 1-36 — File prologue
```cpp
   1 | /***************************************************************************************************
   2 | * Copyright (c) 2024 - 2024 Codeplay Software Ltd. All rights reserved.
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
  30 |  **************************************************************************************************/
  31 | 
  32 |  #pragma once
  33 | 
  34 | #include "gemm_configuration_sycl.hpp"
  35 | 
  36 | using Scheduler = cutlass::gemm::device::Scheduler;
```
**EN:** License comments, include directives, and namespace/prologue code establish the compilation context for the definitions that follow.
**CN:** 许可证注释、include 指令以及命名空间/前导代码为后续定义建立编译上下文。

### Lines 37-84 — BF16 RRR families
```cpp
  37 | 
  38 | using MMAAtom = MMA_Atom<XE_8x16x16_F32BF16BF16F32_TT>;
  39 | 
  40 | template <
  41 |   typename TileShape,
  42 |   typename Tiler,
  43 |   typename GmemTiledCopyA,
  44 |   typename GmemTiledCopyB>
  45 | using Gemm_Bench_BF16FP32_RRR = cutlass::gemm::device::GemmConfiguration<
  46 |     cutlass::arch::IntelXe,
  47 |     cutlass::bfloat16_t, cutlass::layout::RowMajor,
  48 |     cutlass::bfloat16_t, cutlass::layout::RowMajor,
  49 |     float, cutlass::layout::RowMajor,
  50 |     float,
  51 |     TileShape, Scheduler::Gemm, Tiler,
  52 |     GmemTiledCopyA, GmemTiledCopyB>;
  53 | 
  54 | 
  55 | using Tile_1 = TiledMMA<MMAAtom, Layout<Shape<_8,_4,_1>, Stride<_4,_1,_0>>, Tile<Layout<Shape<_8, _8, _4>, Stride<_1, _32, _8>>, Layout<Shape<_16, _4, _4>, Stride<_1, _64, _16>>, _32>>;
  56 | using PvcGemmBF16BF16FP32_RRR_1 = Gemm_Bench_BF16FP32_RRR<Shape<_256, _256, _32>, Tile_1, XE_2D_U16x32x32_LD_N, XE_2D_U16x32x32_LD_V>;
  57 | 
  58 | using Tile_2 = TiledMMA<MMAAtom, Layout<Shape<_4,_8,_1>, Stride<_8,_1,_0>>, Tile<Layout<Shape<_8, _4, _4>, Stride<_1, _32, _8>>, Layout<Shape<_16, _8, _4>, Stride<_1, _64, _16>>, _32>>;
  59 | using PvcGemmBF16BF16FP32_RRR_2 = Gemm_Bench_BF16FP32_RRR<Shape<_128, _512, _32>, Tile_2, XE_2D_U16x32x32_LD_N, XE_2D_U16x32x32_LD_V>;
  60 | 
  61 | using Tile_3 = TiledMMA<MMAAtom, Layout<Shape<_8,_4,_1>, Stride<_4,_1,_0>>, Tile<Layout<Shape<_8, _8, _4>, Stride<_1, _32, _8>>, Layout<Shape<_16, _4, _2>, Stride<_1, _32, _16>>, _32>>;
  62 | using PvcGemmBF16BF16FP32_RRR_3 = Gemm_Bench_BF16FP32_RRR<Shape<_256, _128, _32>, Tile_3, XE_2D_U16x32x32_LD_N, XE_2D_U16x32x32_LD_V>;
  63 | 
  64 | using Tile_4 = TiledMMA<MMAAtom, Layout<Shape<_4,_8,_1>, Stride<_8,_1,_0>>, Tile<Layout<Shape<_8, _4, _4>, Stride<_1, _32, _8>>, Layout<Shape<_16, _8, _1>, Stride<_1, _16, _0>>, _16>>;
  65 | using PvcGemmBF16BF16FP32_RRR_4 = Gemm_Bench_BF16FP32_RRR<Shape<_128, _256, _16>, Tile_4, XE_2D_U16x32x16_LD_N, XE_2D_U16x16x16_LD_V>;
  66 | 
  67 | using Tile_5 = TiledMMA<MMAAtom, Layout<Shape<_1,_4,_1>, Stride<_0,_1,_0>>, Tile<Layout<Shape<_8, _1, _1>, Stride<_1, _0, _0>>, Layout<Shape<_16, _4, _2>, Stride<_1, _32, _16>>, _32>>;
  68 | using PvcGemmBF16BF16FP32_RRR_5 = Gemm_Bench_BF16FP32_RRR<Shape<_8, _128, _32>, Tile_5, XE_2D_U16x8x32_LD_N, XE_2D_U16x32x32_LD_V>;
  69 | 
  70 | CUTLASS_CREATE_GEMM_BENCHMARK(PvcGemmBF16BF16FP32_RRR_1);
  71 | CUTLASS_CREATE_GEMM_BENCHMARK(PvcGemmBF16BF16FP32_RRR_2);
  72 | CUTLASS_CREATE_GEMM_BENCHMARK(PvcGemmBF16BF16FP32_RRR_3);
  73 | CUTLASS_CREATE_GEMM_BENCHMARK(PvcGemmBF16BF16FP32_RRR_4);
  74 | CUTLASS_CREATE_GEMM_BENCHMARK(PvcGemmBF16BF16FP32_RRR_5);
  75 | 
  76 | using PvcGemmCollectiveBF16BF16FP32_RRR_256x256x32 = cutlass::gemm::device::GemmConfiguration<
  77 |     cutlass::arch::IntelXe,
  78 |     cutlass::bfloat16_t, cutlass::layout::RowMajor,
  79 |     cutlass::bfloat16_t, cutlass::layout::RowMajor,
  80 |     float, cutlass::layout::RowMajor,
  81 |     float,
  82 |     Shape<_256,_256,_32>, Scheduler::Gemm>;
  83 | 
  84 | CUTLASS_CREATE_GEMM_BENCHMARK(PvcGemmCollectiveBF16BF16FP32_RRR_256x256x32);
```
**EN:** Defines several row-major BF16 GEMM configurations for different tile shapes, tiled MMA layouts, and one collective-builder-based variant.
**CN:** 定义多个行主序 BF16 GEMM 配置，覆盖不同的 tile 形状、tiled MMA 布局，以及一个基于 collective builder 的变体。

### Lines 86-119 — BF16 RCR families
```cpp
  86 | template <
  87 |   typename TileShape,
  88 |   typename Tiler,
  89 |   typename GmemTiledCopyA,
  90 |   typename GmemTiledCopyB>
  91 | using Gemm_Bench_BF16FP32_RCR = cutlass::gemm::device::GemmConfiguration<
  92 |     cutlass::arch::IntelXe,
  93 |     cutlass::bfloat16_t, cutlass::layout::RowMajor,
  94 |     cutlass::bfloat16_t, cutlass::layout::ColumnMajor,
  95 |     float, cutlass::layout::RowMajor,
  96 |     float,
  97 |     TileShape, Scheduler::Gemm, Tiler,
  98 |     GmemTiledCopyA, GmemTiledCopyB>;
  99 | 
 100 | using Tile_6 = TiledMMAHelper<MMAAtom, Layout<Shape<_8, _128, _32>>, Layout<Shape<_1, _4, _1>, Stride<_0, _1, _0>>>::TiledMMA;
 101 | using PvcGemmBF16BF16FP32_RCR_5 = Gemm_Bench_BF16FP32_RCR<Shape<_8, _128, _32>, Tile_6, XE_2D_U16x8x32_LD_N, XE_2D_U16x16x16_LD_T>;
 102 | 
 103 | using PvcGemmBF16BF16FP32_RCR_6 = Gemm_Bench_BF16FP32_RCR<Shape<_256, _256, _32>, Tile_1, XE_2D_U16x8x32_LD_N, XE_2D_U16x16x16_LD_T>;
 104 | 
 105 | using Tile_7 = TiledMMAHelper<MMAAtom, Layout<Shape<_8, _128, _32>>, Layout<Shape<_1, _8, _1>, Stride<_8, _1, _0>>>::TiledMMA;
 106 | using PvcGemmBF16BF16FP32_RCR_7 = Gemm_Bench_BF16FP32_RCR<Shape<_8, _128, _32>, Tile_7, XE_2D_U16x8x32_LD_N, XE_2D_U16x16x16_LD_T>;
 107 | 
 108 | 
 109 | using Tile_8 = TiledMMAHelper<MMAAtom, Layout<Shape<_8, _64, _32>>, Layout<Shape<_1, _4, _1>, Stride<_4, _1, _0>>>::TiledMMA;
 110 | using PvcGemmBF16BF16FP32_RCR_9 = Gemm_Bench_BF16FP32_RCR<Shape<_8, _64, _32>, Tile_8, XE_2D_U16x8x32_LD_N, XE_2D_U16x16x16_LD_T>;
 111 | 
 112 | using Tile_9 = TiledMMAHelper<MMAAtom, Layout<Shape<_16, _64, _32>>, Layout<Shape<_2, _4, _1>, Stride<_4, _1, _0>>>::TiledMMA;
 113 | using PvcGemmBF16BF16FP32_RCR_16 = Gemm_Bench_BF16FP32_RCR<Shape<_16, _64, _32>, Tile_9, XE_2D_U16x8x32_LD_N, XE_2D_U16x16x16_LD_T>;
 114 | 
 115 | CUTLASS_CREATE_GEMM_BENCHMARK(PvcGemmBF16BF16FP32_RCR_5);
 116 | CUTLASS_CREATE_GEMM_BENCHMARK(PvcGemmBF16BF16FP32_RCR_6);
 117 | CUTLASS_CREATE_GEMM_BENCHMARK(PvcGemmBF16BF16FP32_RCR_7);
 118 | CUTLASS_CREATE_GEMM_BENCHMARK(PvcGemmBF16BF16FP32_RCR_9);
 119 | CUTLASS_CREATE_GEMM_BENCHMARK(PvcGemmBF16BF16FP32_RCR_16);
```
**EN:** Adds BF16 row-by-column benchmark families with several tile shapes and subgroup layouts.
**CN:** 增加 BF16 的 RCR（行乘列）基准家族，覆盖多个 tile 形状和 subgroup 布局。

### Lines 121-168 — Split-K and fused BF16 variants
```cpp
 121 | template <
 122 |   typename TileShape,
 123 |   typename Tiler,
 124 |   typename GmemTiledCopyA,
 125 |   typename GmemTiledCopyB,
 126 |   typename Epilogue>
 127 | using SplitK_Bench_BF16FP32_RCR_Epilogue = cutlass::gemm::device::GemmConfiguration<
 128 |     cutlass::arch::IntelXe,
 129 |     cutlass::bfloat16_t, cutlass::layout::RowMajor,
 130 |     cutlass::bfloat16_t, cutlass::layout::ColumnMajor,
 131 |     float, cutlass::layout::RowMajor,
 132 |     float,
 133 |     TileShape, Scheduler::GemmSplitK, Tiler,
 134 |     GmemTiledCopyA, GmemTiledCopyB, Epilogue>;
 135 | 
 136 | using Tile_10 = TiledMMAHelper<MMAAtom, Layout<Shape<_8, _64, _32>>, Layout<Shape<_1, _4, _1>, Stride<_4, _1, _0>>>::TiledMMA;
 137 | using SiLuF32 = cutlass::epilogue::fusion::LinCombEltAct<
 138 |   cutlass::epilogue::thread::SiLu,
 139 |   float, float, float, float,
 140 |   cutlass::FloatRoundStyle::round_to_nearest>;
 141 | using PvcGemmBF16BF16FP32_RCR_8_silu = SplitK_Bench_BF16FP32_RCR_Epilogue<
 142 |   Shape<_8, _64, _32>,
 143 |   Tile_10,
 144 |   XE_2D_U16x8x32_LD_N, XE_2D_U16x16x16_LD_T,
 145 |   SiLuF32>;
 146 | 
 147 | using Tile_11 = TiledMMAHelper<MMAAtom, Layout<Shape<_8, _64, _32>>, Layout<Shape<_1, _4, _1>, Stride<_4, _1, _0>>>::TiledMMA;
 148 | using PvcGemmBF16BF16FP32_RCR_7_mul = SplitK_Bench_BF16FP32_RCR_Epilogue<
 149 |   Shape<_8, _64, _32>,
 150 |   Tile_11,
 151 |   XE_2D_U16x8x32_LD_N, XE_2D_U16x16x16_LD_T,
 152 |   cutlass::epilogue::fusion::LinCombDeEltAct<
 153 |     cutlass::layout::RowMajor, std::multiplies, float, float>>;
 154 | 
 155 | CUTLASS_CREATE_GEMM_BENCHMARK(PvcGemmBF16BF16FP32_RCR_8_silu);
 156 | CUTLASS_CREATE_GEMM_BENCHMARK(PvcGemmBF16BF16FP32_RCR_7_mul);
 157 | 
 158 | using PvcGemmCollectiveBF16BF16FP32_RCR_silu_8x64x32 = cutlass::gemm::device::GemmConfiguration<
 159 |     cutlass::arch::IntelXe,
 160 |     cutlass::bfloat16_t, cutlass::layout::RowMajor,
 161 |     cutlass::bfloat16_t, cutlass::layout::ColumnMajor,
 162 |     float, cutlass::layout::RowMajor,
 163 |     float,
 164 |     Shape<_8,_64,_32>, Scheduler::GemmSplitK,
 165 |     void, void, void,
 166 |     SiLuF32>;
 167 | 
 168 | CUTLASS_CREATE_GEMM_BENCHMARK(PvcGemmCollectiveBF16BF16FP32_RCR_silu_8x64x32);
```
**EN:** Introduces split-K BF16 benchmarks and fused SiLU or multiply epilogues, including a collective-based fused configuration.
**CN:** 引入 split-K 的 BF16 基准以及融合 SiLU 或乘法 epilogue 的变体，其中还包括一个基于 collective 的融合配置。

### Lines 170-190 — CRR and CCR layouts
```cpp
 170 | using PvcGemmBF16BF16FP32_CRR_7 = cutlass::gemm::device::GemmConfiguration<
 171 |         cutlass::arch::IntelXe,
 172 |         cutlass::bfloat16_t, cutlass::layout::ColumnMajor,
 173 |         cutlass::bfloat16_t, cutlass::layout::RowMajor,
 174 |         float, cutlass::layout::RowMajor,
 175 |         float,
 176 |         Shape<_256, _256, _32>, Scheduler::Gemm, Tile_1,
 177 |         XE_2D_U16x16x16_LD_T, XE_2D_U16x32x32_LD_V
 178 |         >;
 179 | 
 180 | using PvcGemmBF16BF16FP32_CCR_8 = cutlass::gemm::device::GemmConfiguration<
 181 |         cutlass::arch::IntelXe,
 182 |         cutlass::bfloat16_t, cutlass::layout::ColumnMajor,
 183 |         cutlass::bfloat16_t, cutlass::layout::ColumnMajor,
 184 |         float, cutlass::layout::RowMajor,
 185 |         float,
 186 |         Shape<_256, _256, _32>, Scheduler::Gemm, Tile_1,
 187 |         XE_2D_U16x16x16_LD_T, XE_2D_U16x16x16_LD_T>;
 188 | 
 189 | CUTLASS_CREATE_GEMM_BENCHMARK(PvcGemmBF16BF16FP32_CRR_7);
 190 | CUTLASS_CREATE_GEMM_BENCHMARK(PvcGemmBF16BF16FP32_CCR_8);
```
**EN:** Defines additional BF16 layout combinations where A and/or B use column-major layouts.
**CN:** 定义额外的 BF16 布局组合，其中 A 和/或 B 使用列主序布局。

### Lines 193-233 — Commented future ideas
```cpp
 193 | // TODO(codeplay): Re-enable these once there is general support for epilogues
 194 | // using PvcGemmBF16BF16FP32_RCR_Linear = cutlass::gemm::device::GemmConfiguration<
 195 | //         cutlass::arch::IntelXe,
 196 | //         cutlass::bfloat16_t, cutlass::layout::RowMajor,
 197 | //         cutlass::bfloat16_t, cute::Stride<int64_t, _1, _0>, // Stride for batch is _0 (re-use the same B matrix)
 198 | //         float, cutlass::layout::RowMajor,
 199 | //         float, Shape<_256, _256, _32>,
 200 | //         TiledMMA<MMAAtom,
 201 | //                  Layout<Shape<_8,_4,_1>, Stride<_4,_1,_0>>,
 202 | //                  Tile<Layout<Shape<_8, _8, _4>, Stride<_1, _32, _8>>,
 203 | //                       Layout<Shape<_16, _4, _4>, Stride<_1, _64, _16>>,
 204 | //                       _32>>,
 205 | //         XE_2D_U16x8x32_LD_N, XE_2D_U16x16x16_LD_T,
 206 | //         Scheduler::Gemm,
 207 | //         cutlass::epilogue::fusion::LinCombPerColBias<
 208 | //             float, float, float, float,
 209 | //             float, 128 / sizeof_bits_v<float>,
 210 | //             cutlass::FloatRoundStyle::round_to_nearest>
 211 | // >;
 212 | 
 213 | // TODO(codeplay): Does this batch GEMM config correspond to 'MoE group gemm with 256 experts'?
 214 | // using PvcGemmBF16BF16FP32_RCR_Linear_MoE = cutlass::gemm::device::GemmConfiguration<
 215 | //         cutlass::arch::IntelXe,
 216 | //         cutlass::bfloat16_t, cute::Stride<int64_t, _1, _0>, // Stride for batch is _0 (re-use the same A matrix)
 217 | //         cutlass::bfloat16_t, cutlass::layout::ColumnMajor,
 218 | //         float, cutlass::layout::RowMajor,
 219 | //         float, Shape<_256, _256, _32>,
 220 | //         TiledMMA<MMAAtom,
 221 | //                  Layout<Shape<_8,_4,_1>, Stride<_4,_1,_0>>,
 222 | //                  Tile<Layout<Shape<_8, _8, _4>, Stride<_1, _32, _8>>,
 223 | //                       Layout<Shape<_16, _4, _4>, Stride<_1, _64, _16>>,
 224 | //                       _32>>,
 225 | //         XE_2D_U16x8x32_LD_N, XE_2D_U16x16x16_LD_T,
 226 | //         Scheduler::Gemm,
 227 | //         cutlass::epilogue::fusion::LinCombPerColBias<
 228 | //             float, float, float, float,
 229 | //             float, 128 / sizeof_bits_v<float>,
 230 | //             cutlass::FloatRoundStyle::round_to_nearest>
 231 | // >;
 232 | // CUTLASS_CREATE_GEMM_BENCHMARK(PvcGemmBF16BF16FP32_RCR_Linear);
 233 | // CUTLASS_CREATE_GEMM_BENCHMARK(PvcGemmBF16BF16FP32_RCR_Linear_MoE);
```
**EN:** Keeps currently disabled experiments for bias and MoE-style cases as commented reference material.
**CN:** 保留当前禁用的 bias 和类 MoE 场景实验，作为注释形式的参考材料。

### Lines 235-443 — FP16 and mixed-precision variants
```cpp
 235 | using PvcGemmFP16FP16FP32_RCR_5 = cutlass::gemm::device::GemmConfiguration<
 236 |         cutlass::arch::IntelXe,
 237 |         cutlass::half_t, cutlass::layout::RowMajor,
 238 |         cutlass::half_t, cutlass::layout::ColumnMajor,
 239 |         float, cutlass::layout::RowMajor,
 240 |         float, Shape<_8, _128, _32>, Scheduler::Gemm,
 241 |         typename TiledMMAHelper<MMA_Atom<XE_8x16x16_F32F16F16F32_TT>, Layout<Shape<_8, _128, _32>>,
 242 |         Layout<Shape<_1, _4, _1>, Stride<_0, _1, _0>>>::TiledMMA,
 243 |         XE_2D_U16x8x32_LD_N, XE_2D_U16x16x16_LD_T
 244 |         >;
 245 | 
 246 | using PvcGemmFP16FP16FP32_RCR_7 = cutlass::gemm::device::GemmConfiguration<
 247 |         cutlass::arch::IntelXe,
 248 |         cutlass::half_t, cutlass::layout::RowMajor,
 249 |         cutlass::half_t, cutlass::layout::ColumnMajor,
 250 |         float, cutlass::layout::RowMajor,
 251 |         float, Shape<_8, _128, _32>, Scheduler::Gemm,
 252 |         typename TiledMMAHelper<MMA_Atom<XE_8x16x16_F32F16F16F32_TT>, Layout<Shape<_8, _128, _32>>,
 253 |                                       Layout<Shape<_1, _8, _1>, Stride<_8, _1, _0>>>::TiledMMA,
 254 |         XE_2D_U16x8x32_LD_N, XE_2D_U16x16x16_LD_T
 255 |         >;
 256 | 
 257 | using PvcGemmFP16FP16FP32_RCR_9 = cutlass::gemm::device::GemmConfiguration<
 258 |         cutlass::arch::IntelXe,
 259 |         cutlass::half_t, cutlass::layout::RowMajor,
 260 |         cutlass::half_t, cutlass::layout::ColumnMajor,
 261 |         float, cutlass::layout::RowMajor,
 262 |         float, Shape<_8, _64, _32>, Scheduler::Gemm,
 263 |         typename TiledMMAHelper<MMA_Atom<XE_8x16x16_F32F16F16F32_TT>, Layout<Shape<_8, _64, _32>>,
 264 |                                       Layout<Shape<_1, _4, _1>, Stride<_4, _1, _0>>>::TiledMMA,
 265 |         XE_2D_U16x8x32_LD_N, XE_2D_U16x16x16_LD_T
 266 |         >;
 267 | 
 268 | using PvcGemmFP16FP16FP32_RCR_16 = cutlass::gemm::device::GemmConfiguration<
 269 |         cutlass::arch::IntelXe,
 270 |         cutlass::half_t, cutlass::layout::RowMajor,
 271 |         cutlass::half_t, cutlass::layout::ColumnMajor,
 272 |         float, cutlass::layout::RowMajor,
 273 |         float, Shape<_16, _64, _32>, Scheduler::Gemm,
 274 |         typename TiledMMAHelper<MMA_Atom<XE_8x16x16_F32F16F16F32_TT>, Layout<Shape<_16, _64, _32>>,
 275 |                                       Layout<Shape<_2, _4, _1>, Stride<_4, _1, _0>>>::TiledMMA,
 276 |         XE_2D_U16x8x32_LD_N, XE_2D_U16x16x16_LD_T
 277 |         >;
 278 | 
 279 | using PvcGemmFP16FP16FP32_RCR_7_mul = cutlass::gemm::device::GemmConfiguration<
 280 |         cutlass::arch::IntelXe,
 281 |         cutlass::half_t, cutlass::layout::RowMajor,
 282 |         cutlass::half_t, cutlass::layout::ColumnMajor,
 283 |         float, cutlass::layout::RowMajor,
 284 |         float, Shape<_8, _64, _32>, Scheduler::GemmSplitK,
 285 |         typename TiledMMAHelper<MMA_Atom<XE_8x16x16_F32F16F16F32_TT>, Layout<Shape<_8, _64, _32>>,
 286 |                                         Layout<Shape<_1, _4, _1>, Stride<_4, _1, _0>>>::TiledMMA,
 287 |         XE_2D_U16x8x32_LD_N, XE_2D_U16x16x16_LD_T,
 288 |         cutlass::epilogue::fusion::LinCombDeEltAct<
 289 |         cutlass::layout::RowMajor, std::multiplies, float, float>
 290 |         >;
 291 | 
 292 | using PvcGemmFP16FP16FP32_RCR_8_silu = cutlass::gemm::device::GemmConfiguration<
 293 |         cutlass::arch::IntelXe,
 294 |         cutlass::half_t, cutlass::layout::RowMajor,
 295 |         cutlass::half_t, cutlass::layout::ColumnMajor,
 296 |         float, cutlass::layout::RowMajor,
 297 |         float, Shape<_8, _64, _32>, Scheduler::GemmSplitK,
 298 |         typename TiledMMAHelper<MMA_Atom<XE_8x16x16_F32F16F16F32_TT>, Layout<Shape<_8, _64, _32>>,
 299 |                                       Layout<Shape<_1, _4, _1>, Stride<_4, _1, _0>>>::TiledMMA,
 300 |         XE_2D_U16x8x32_LD_N, XE_2D_U16x16x16_LD_T,
 301 |         cutlass::epilogue::fusion::LinCombEltAct<
 302 |             cutlass::epilogue::thread::SiLu,
 303 |             float, float, float, float,
 304 |             cutlass::FloatRoundStyle::round_to_nearest>>;
 305 | 
 306 | using PvcGemmFP16FP16FP32_SplitK_RCR_5 = cutlass::gemm::device::GemmConfiguration<
 307 |         cutlass::arch::IntelXe,
 308 |         cutlass::half_t, cutlass::layout::RowMajor,
 309 |         cutlass::half_t, cutlass::layout::ColumnMajor,
 310 |         float, cutlass::layout::RowMajor,
 311 |         float, Shape<_8, _64, _32>,  Scheduler::GemmSplitK,
 312 |         typename TiledMMAHelper<MMA_Atom<XE_8x16x16_F32F16F16F32_TT>, Layout<Shape<_8, _64, _32>>,
 313 |                                         Layout<Shape<_1, _4, _1>, Stride<_4, _1, _0>>>::TiledMMA,
 314 |         XE_2D_U16x8x32_LD_N, XE_2D_U16x16x16_LD_T
 315 |         >;
 316 | 
 317 | using PvcMixedPrecisionGemmFP16U4FP16F16FP16S4_RCR_1 = cutlass::gemm::device::MixedPrecisionGemmConfiguration<
 318 |         cutlass::arch::IntelXe,
 319 |         cutlass::half_t, cutlass::layout::RowMajor,
 320 |         cutlass::uint4_t, cutlass::layout::ColumnMajor,
 321 |         cutlass::half_t, cutlass::layout::RowMajor,
 322 |         cutlass::half_t, cute::Stride<_1, int64_t, int64_t>,
 323 |         cutlass::int4_t, cute::Stride<_8, cute::Stride<_1, int64_t>, int64_t>,
 324 |         Shape<_32, _128, _32>,  Scheduler::Gemm,
 325 |         typename TiledMMAHelper<MMA_Atom<XE_8x16x16_F32F16F16F32_TT>, Layout<Shape<_32, _128, _32>>,
 326 |                                         Layout<Shape<_1, _4, _1>, Stride<_4, _1, _0>>>::TiledMMA,
 327 |         XE_2D_U16x32x32_LD_N, XE_2D_U4x32x16_LD_T, XE_2D_U16x8x16_ST_N,
 328 |         cutlass::epilogue::fusion::LinearCombination<float, float,
 329 |           float, float, cutlass::FloatRoundStyle::round_to_nearest>,
 330 |         2
 331 |         >;
 332 | 
 333 | using PvcMixedPrecisionGemmBF16U4BF16BF16BF16S4_RCR_1 = cutlass::gemm::device::MixedPrecisionGemmConfiguration<
 334 |         cutlass::arch::IntelXe,
 335 |         cutlass::bfloat16_t, cutlass::layout::RowMajor,
 336 |         cutlass::uint4_t, cutlass::layout::ColumnMajor,
 337 |         cutlass::bfloat16_t, cutlass::layout::RowMajor,
 338 |         cutlass::bfloat16_t, cute::Stride<_1, int64_t, int64_t>,
 339 |         cutlass::int4_t, cute::Stride<_8, cute::Stride<_1, int64_t>, int64_t>,
 340 |         Shape<_32, _128, _32>,  Scheduler::Gemm,
 341 |         typename TiledMMAHelper<MMA_Atom<XE_8x16x16_F32BF16BF16F32_TT>, Layout<Shape<_32, _128, _32>>,
 342 |                                         Layout<Shape<_1, _4, _1>, Stride<_4, _1, _0>>>::TiledMMA,
 343 |         XE_2D_U16x32x32_LD_N, XE_2D_U4x32x16_LD_T, XE_2D_U16x8x16_ST_N,
 344 |         cutlass::epilogue::fusion::LinearCombination<float, float,
 345 |           float, float, cutlass::FloatRoundStyle::round_to_nearest>,
 346 |         2
 347 |         >;
 348 | 
 349 | using PvcMixedPrecisionGemmFP16U4FP16S8FP16S4_RCR_1 = cutlass::gemm::device::MixedPrecisionGemmConfiguration<
 350 |         cutlass::arch::IntelXe,
 351 |         cutlass::half_t, cutlass::layout::RowMajor,
 352 |         cutlass::uint4_t, cutlass::layout::ColumnMajor,
 353 |         cutlass::half_t, cutlass::layout::RowMajor,
 354 |         cutlass::half_t, cute::Stride<_1, int64_t, int64_t>,
 355 |         cutlass::int4_t, cute::Stride<_8, cute::Stride<_1, int64_t>, int64_t>,
 356 |         Shape<_32, _128, _32>,  Scheduler::Gemm,
 357 |         typename TiledMMAHelper<MMA_Atom<XE_8x16x32_S32S8S8S32_TT>, Layout<Shape<_32, _128, _32>>,
 358 |                                         Layout<Shape<_1, _4, _1>, Stride<_4, _1, _0>>>::TiledMMA,
 359 |         XE_2D_Packed_U16x32x32_LD_N, XE_2D_U4x32x16_LD_T, XE_2D_U16x8x16_ST_N,
 360 |         cutlass::epilogue::fusion::LinearCombination<int, int,
 361 |           int, int, cutlass::FloatRoundStyle::round_to_nearest>,
 362 |         2
 363 |         >;
 364 | 
 365 | using PvcMixedPrecisionGemmFP16U4S8S8FP16S4_RCR_1 = cutlass::gemm::device::MixedPrecisionGemmConfiguration<
 366 |         cutlass::arch::IntelXe,
 367 |         cutlass::half_t, cutlass::layout::RowMajor,
 368 |         cutlass::uint4_t, cutlass::layout::ColumnMajor,
 369 |         cutlass::int8_t, cutlass::layout::RowMajor,
 370 |         cutlass::half_t, cute::Stride<_1, int64_t, int64_t>,
 371 |         cutlass::int4_t, cute::Stride<_8, cute::Stride<_1, int64_t>, int64_t>,
 372 |         Shape<_32, _128, _32>,  Scheduler::Gemm,
 373 |         typename TiledMMAHelper<MMA_Atom<XE_8x16x32_S32S8S8S32_TT>, Layout<Shape<_32, _128, _32>>,
 374 |                                         Layout<Shape<_1, _4, _1>, Stride<_4, _1, _0>>>::TiledMMA,
 375 |         XE_2D_Packed_U16x32x32_LD_N, XE_2D_U4x32x16_LD_T, XE_2D_U8x8x16_ST_N,
 376 |         cutlass::epilogue::fusion::LinearCombination<int, int,
 377 |           int, int, cutlass::FloatRoundStyle::round_to_nearest>,
 378 |         2
 379 |         >;
 380 | 
 381 | using PvcMixedPrecisionGemmBF16U4BF16S8BF16S4_RCR_1 = cutlass::gemm::device::MixedPrecisionGemmConfiguration<
 382 |         cutlass::arch::IntelXe,
 383 |         cutlass::bfloat16_t, cutlass::layout::RowMajor,
 384 |         cutlass::uint4_t, cutlass::layout::ColumnMajor,
 385 |         cutlass::bfloat16_t, cutlass::layout::RowMajor,
 386 |         cutlass::bfloat16_t, cute::Stride<_1, int64_t, int64_t>,
 387 |         cutlass::int4_t, cute::Stride<_8, cute::Stride<_1, int64_t>, int64_t>,
 388 |         Shape<_32, _128, _32>,  Scheduler::Gemm,
 389 |         typename TiledMMAHelper<MMA_Atom<XE_8x16x32_S32S8S8S32_TT>, Layout<Shape<_32, _128, _32>>,
 390 |                                         Layout<Shape<_1, _4, _1>, Stride<_4, _1, _0>>>::TiledMMA,
 391 |         XE_2D_Packed_U16x32x32_LD_N, XE_2D_U4x32x16_LD_T, XE_2D_U16x8x16_ST_N,
 392 |         cutlass::epilogue::fusion::LinearCombination<int, int,
 393 |           int, int, cutlass::FloatRoundStyle::round_to_nearest>,
 394 |         2
 395 |         >;
 396 | 
 397 | using PvcMixedPrecisionGemmBF16U4S8S8BF16S4_RCR_1 = cutlass::gemm::device::MixedPrecisionGemmConfiguration<
 398 |         cutlass::arch::IntelXe,
 399 |         cutlass::bfloat16_t, cutlass::layout::RowMajor,
 400 |         cutlass::uint4_t, cutlass::layout::ColumnMajor,
 401 |         cutlass::int8_t, cutlass::layout::RowMajor,
 402 |         cutlass::bfloat16_t, cute::Stride<_1, int64_t, int64_t>,
 403 |         cutlass::int4_t, cute::Stride<_8, cute::Stride<_1, int64_t>, int64_t>,
 404 |         Shape<_32, _128, _32>,  Scheduler::Gemm,
 405 |         typename TiledMMAHelper<MMA_Atom<XE_8x16x32_S32S8S8S32_TT>, Layout<Shape<_32, _128, _32>>,
 406 |                                         Layout<Shape<_1, _4, _1>, Stride<_4, _1, _0>>>::TiledMMA,
 407 |         XE_2D_Packed_U16x32x32_LD_N, XE_2D_U4x32x16_LD_T, XE_2D_U8x8x16_ST_N,
 408 |         cutlass::epilogue::fusion::LinearCombination<int, int,
 409 |           int, int, cutlass::FloatRoundStyle::round_to_nearest>,
 410 |         2
 411 |         >;
 412 | 
 413 | using PvcMixedPrecisionGemmBF16S8BF16S8BF16S8_RCR_1 = cutlass::gemm::device::MixedPrecisionGemmConfiguration<
 414 |         cutlass::arch::IntelXe,
 415 |         cutlass::bfloat16_t, cutlass::layout::RowMajor,
 416 |         cutlass::int8_t, cutlass::layout::ColumnMajor,
 417 |         cutlass::bfloat16_t, cutlass::layout::RowMajor,
 418 |         cutlass::bfloat16_t, cute::Stride<_1, int64_t, int64_t>,
 419 |         cutlass::int8_t, cute::Stride<_1, int64_t, int64_t>,
 420 |         Shape<_32, _128, _32>,  Scheduler::Gemm,
 421 |         typename TiledMMAHelper<MMA_Atom<XE_8x16x32_S32S8S8S32_TT>, Layout<Shape<_32, _128, _32>>,
 422 |                                         Layout<Shape<_1, _4, _1>, Stride<_4, _1, _0>>>::TiledMMA,
 423 |         XE_2D_Packed_U16x32x32_LD_N, XE_2D_U8x16x32_LD_T, XE_2D_U16x8x16_ST_N,
 424 |         cutlass::epilogue::fusion::LinearCombination<int, int,
 425 |           int, int, cutlass::FloatRoundStyle::round_to_nearest>,
 426 |         2
 427 |         >;
 428 | 
 429 | using PvcMixedPrecisionGemmFP16S8FP16S8FP16S8_RCR_1 = cutlass::gemm::device::MixedPrecisionGemmConfiguration<
 430 |         cutlass::arch::IntelXe,
 431 |         cutlass::half_t, cutlass::layout::RowMajor,
 432 |         cutlass::int8_t, cutlass::layout::ColumnMajor,
 433 |         cutlass::half_t, cutlass::layout::RowMajor,
 434 |         cutlass::half_t, cute::Stride<_1, int64_t, int64_t>,
 435 |         cutlass::int8_t, cute::Stride<_1, int64_t, int64_t>,
 436 |         Shape<_32, _128, _32>,  Scheduler::Gemm,
 437 |         typename TiledMMAHelper<MMA_Atom<XE_8x16x32_S32S8S8S32_TT>, Layout<Shape<_32, _128, _32>>,
 438 |                                         Layout<Shape<_1, _4, _1>, Stride<_4, _1, _0>>>::TiledMMA,
 439 |         XE_2D_Packed_U16x32x32_LD_N, XE_2D_U8x16x32_LD_T, XE_2D_U16x8x16_ST_N,
 440 |         cutlass::epilogue::fusion::LinearCombination<int, int,
 441 |           int, int, cutlass::FloatRoundStyle::round_to_nearest>,
 442 |         2
 443 |         >;
```
**EN:** Defines FP16 benchmark families together with mixed-precision configurations that combine quantized inputs, scale/zero metadata, and different accumulator/output types.
**CN:** 定义 FP16 基准家族，以及结合量化输入、scale/zero 元数据和不同累加/输出类型的混合精度配置。

### Lines 445-488 — Wrapper generation
```cpp
 445 | CUTLASS_CREATE_GEMM_BENCHMARK(PvcGemmFP16FP16FP32_RCR_5);
 446 | CUTLASS_CREATE_GEMM_BENCHMARK(PvcGemmFP16FP16FP32_RCR_7);
 447 | CUTLASS_CREATE_GEMM_BENCHMARK(PvcGemmFP16FP16FP32_RCR_9);
 448 | CUTLASS_CREATE_GEMM_BENCHMARK(PvcGemmFP16FP16FP32_RCR_16);
 449 | CUTLASS_CREATE_GEMM_BENCHMARK(PvcGemmFP16FP16FP32_RCR_7_mul);
 450 | CUTLASS_CREATE_GEMM_BENCHMARK(PvcGemmFP16FP16FP32_RCR_8_silu);
 451 | CUTLASS_CREATE_GEMM_BENCHMARK(PvcGemmFP16FP16FP32_SplitK_RCR_5);
 452 | 
 453 | // Below are MixedPrecisionGemm, the data type are A, B, C, Mma, Scale, Zero
 454 | CUTLASS_CREATE_GEMM_BENCHMARK(PvcMixedPrecisionGemmFP16U4FP16F16FP16S4_RCR_1);
 455 | CUTLASS_CREATE_GEMM_BENCHMARK(PvcMixedPrecisionGemmBF16U4BF16BF16BF16S4_RCR_1);
 456 | CUTLASS_CREATE_GEMM_BENCHMARK(PvcMixedPrecisionGemmFP16U4FP16S8FP16S4_RCR_1);
 457 | CUTLASS_CREATE_GEMM_BENCHMARK(PvcMixedPrecisionGemmFP16U4S8S8FP16S4_RCR_1);
 458 | CUTLASS_CREATE_GEMM_BENCHMARK(PvcMixedPrecisionGemmBF16U4BF16S8BF16S4_RCR_1);
 459 | CUTLASS_CREATE_GEMM_BENCHMARK(PvcMixedPrecisionGemmBF16U4S8S8BF16S4_RCR_1);
 460 | CUTLASS_CREATE_GEMM_BENCHMARK(PvcMixedPrecisionGemmBF16S8BF16S8BF16S8_RCR_1);
 461 | CUTLASS_CREATE_GEMM_BENCHMARK(PvcMixedPrecisionGemmFP16S8FP16S8FP16S8_RCR_1);
 462 | 
 463 | using PvcGemmBF16BF16FP32_SplitK_RRR_1 = cutlass::gemm::device::GemmConfiguration<
 464 |         cutlass::arch::IntelXe,
 465 |         cutlass::bfloat16_t, cutlass::layout::RowMajor,
 466 |         cutlass::bfloat16_t, cutlass::layout::RowMajor,
 467 |         float, cutlass::layout::RowMajor,
 468 |         float,
 469 |         Shape<_256, _256, _32>, Scheduler::GemmSplitK, Tile_1,
 470 |         XE_2D_U16x32x32_LD_N, XE_2D_U16x32x32_LD_V>;
 471 | 
 472 | using Tile_12 = TiledMMAHelper<MMAAtom, Layout<Shape<_8, _64, _32>>, Layout<Shape<_1, _4, _1>, Stride<_4, _1, _0>>>::TiledMMA;
 473 | using PvcGemmBF16BF16FP32_SplitK_RCR_5 = cutlass::gemm::device::GemmConfiguration<
 474 |         cutlass::arch::IntelXe,
 475 |         cutlass::bfloat16_t, cutlass::layout::RowMajor,
 476 |         cutlass::bfloat16_t, cutlass::layout::ColumnMajor,
 477 |         float, cutlass::layout::RowMajor,
 478 |         float,
 479 |         Shape<_8, _64, _32>, Scheduler::GemmSplitK, Tile_12,
 480 |         XE_2D_U16x8x32_LD_N, XE_2D_U16x16x16_LD_T>;
 481 | 
 482 | CUTLASS_CREATE_GEMM_BENCHMARK(PvcGemmBF16BF16FP32_SplitK_RRR_1);
 483 | CUTLASS_CREATE_GEMM_BENCHMARK(PvcGemmBF16BF16FP32_SplitK_RCR_5);
 484 | 
 485 | // TODO: benchmark is named streamK but uses splitk scheduler
 486 | using PvcGemmBF16BF16FP32_StreamK_RRR_1 = PvcGemmBF16BF16FP32_SplitK_RRR_1;
 487 | 
 488 | CUTLASS_CREATE_GEMM_BENCHMARK(PvcGemmBF16BF16FP32_StreamK_RRR_1);
```
**EN:** Expands benchmark-wrapper macros for the FP16, mixed-precision, split-K, and stream-K legacy configurations.
**CN:** 为 FP16、混合精度、split-K 和 stream-K 旧版配置展开基准包装宏。

### Lines 490-533 — Registration list
```cpp
 490 | static void register_gemm_benchmarks() {
 491 |   CUTLASS_BENCHMARK(PvcGemmBF16BF16FP32_RRR_1);
 492 |   CUTLASS_BENCHMARK(PvcGemmBF16BF16FP32_RRR_2);
 493 |   CUTLASS_BENCHMARK(PvcGemmBF16BF16FP32_RRR_3);
 494 |   CUTLASS_BENCHMARK(PvcGemmBF16BF16FP32_RRR_4);
 495 |   CUTLASS_BENCHMARK(PvcGemmBF16BF16FP32_RRR_5);
 496 |   CUTLASS_BENCHMARK(PvcGemmCollectiveBF16BF16FP32_RRR_256x256x32);
 497 | 
 498 |   CUTLASS_BENCHMARK(PvcGemmBF16BF16FP32_RCR_5);
 499 |   CUTLASS_BENCHMARK(PvcGemmBF16BF16FP32_RCR_6);
 500 |   CUTLASS_BENCHMARK(PvcGemmBF16BF16FP32_RCR_7);
 501 |   CUTLASS_BENCHMARK(PvcGemmBF16BF16FP32_RCR_9);
 502 |   CUTLASS_BENCHMARK(PvcGemmBF16BF16FP32_RCR_16);
 503 | 
 504 |   CUTLASS_BENCHMARK(PvcGemmBF16BF16FP32_RCR_8_silu);
 505 |   CUTLASS_BENCHMARK(PvcGemmBF16BF16FP32_RCR_7_mul);
 506 |   CUTLASS_BENCHMARK(PvcGemmCollectiveBF16BF16FP32_RCR_silu_8x64x32);
 507 | 
 508 |   CUTLASS_BENCHMARK(PvcGemmBF16BF16FP32_CRR_7);
 509 |   CUTLASS_BENCHMARK(PvcGemmBF16BF16FP32_CCR_8);
 510 | 
 511 |   CUTLASS_BENCHMARK(PvcGemmBF16BF16FP32_SplitK_RRR_1);
 512 |   CUTLASS_BENCHMARK(PvcGemmBF16BF16FP32_SplitK_RCR_5);
 513 |   CUTLASS_BENCHMARK(PvcGemmBF16BF16FP32_StreamK_RRR_1);
 514 | 
 515 |   CUTLASS_BENCHMARK(PvcGemmFP16FP16FP32_RCR_5);
 516 |   CUTLASS_BENCHMARK(PvcGemmFP16FP16FP32_RCR_7);
 517 |   CUTLASS_BENCHMARK(PvcGemmFP16FP16FP32_RCR_9);
 518 |   CUTLASS_BENCHMARK(PvcGemmFP16FP16FP32_RCR_16);
 519 |   CUTLASS_BENCHMARK(PvcGemmFP16FP16FP32_RCR_7_mul);
 520 |   CUTLASS_BENCHMARK(PvcGemmFP16FP16FP32_RCR_8_silu);
 521 |   CUTLASS_BENCHMARK(PvcGemmFP16FP16FP32_SplitK_RCR_5);
 522 |   CUTLASS_BENCHMARK(PvcMixedPrecisionGemmFP16U4FP16F16FP16S4_RCR_1);
 523 |   CUTLASS_BENCHMARK(PvcMixedPrecisionGemmBF16U4BF16BF16BF16S4_RCR_1);
 524 |   CUTLASS_BENCHMARK(PvcMixedPrecisionGemmFP16U4FP16S8FP16S4_RCR_1);
 525 |   CUTLASS_BENCHMARK(PvcMixedPrecisionGemmFP16U4S8S8FP16S4_RCR_1);
 526 |   CUTLASS_BENCHMARK(PvcMixedPrecisionGemmBF16U4BF16S8BF16S4_RCR_1);
 527 |   CUTLASS_BENCHMARK(PvcMixedPrecisionGemmBF16U4S8S8BF16S4_RCR_1);
 528 |   CUTLASS_BENCHMARK(PvcMixedPrecisionGemmBF16S8BF16S8BF16S8_RCR_1);
 529 |   CUTLASS_BENCHMARK(PvcMixedPrecisionGemmFP16S8FP16S8FP16S8_RCR_1);
 530 | 
 531 |   // CUTLASS_BENCHMARK(PvcGemmBF16BF16FP32_RCR_Linear);
 532 |   // CUTLASS_BENCHMARK(PvcGemmBF16BF16FP32_RCR_Linear_MoE);
 533 | }
```
**EN:** Registers the wide legacy SYCL GEMM catalog with the benchmark suite entry point.
**CN:** 把广泛的旧版 SYCL GEMM 目录注册到基准套件入口中。

## Key Concepts / 关键概念

- Wide PVC/Xe benchmark coverage / 广泛的 PVC/Xe 基准覆盖
- Fused epilogue experiments / 融合 epilogue 实验
- Mixed-precision GEMM variants / 混合精度 GEMM 变体

## Dependencies / 依赖关系

- `gemm_configuration_sycl.hpp` — SYCL config templates / SYCL 配置模板
- `TiledMMAHelper` and Xe MMA atoms / TiledMMAHelper 与 Xe MMA 原子
- `CUTLASS_CREATE_GEMM_BENCHMARK` — wrapper generation / 包装生成
