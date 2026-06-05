# benchmarks.hpp — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `benchmarks/flash_attention/legacy/flash_attention_prefill_cachedKV/benchmarks.hpp`
- **EN:** Legacy cached-KV prefill benchmark catalog that mirrors the prefill family matrix while targeting the cache-aware kernel path.
- **CN:** 旧版 cached-KV prefill 基准目录：沿用 prefill 家族矩阵，但面向缓存感知内核路径。

## Line-by-Line Analysis / 逐行分析

### Lines 1-37 — File prologue
```cpp
   1 | /***************************************************************************************************
   2 | * Copyright (c) 2024 - 2025 Codeplay Software Ltd. All rights reserved.
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
  32 | #pragma once
  33 | 
  34 | #include "benchmark_runner.hpp"
  35 | #include "fmha_prefill_configuration.hpp"
  36 | 
  37 | struct Shape_h64 {
```
**EN:** License comments, include directives, and namespace/prologue code establish the compilation context for the definitions that follow.
**CN:** 许可证注释、include 指令以及命名空间/前导代码为后续定义建立编译上下文。

### Lines 38-68 — Shape presets
```cpp
  38 |   static constexpr int PipelineStages = 2;
  39 |   using ShapeQK = Shape<_128, _64, _64>;
  40 |   using ShapePV = Shape<_128, _32, _64>;
  41 |   using ShapeOutPut = Shape<_128, _64, _64>;
  42 |   using SubgroupLayout = Layout<Shape<_8, _1, _1>, Stride<_1, _1, _1>>;
  43 | };
  44 | 
  45 | struct Shape_h96 {
  46 |   static constexpr int PipelineStages = 2;
  47 |   using ShapeQK = Shape<_128, _64, _32>;
  48 |   using ShapePV = Shape<_128, _32, _64>;
  49 |   using ShapeOutPut = Shape<_128, _96, _64>;
  50 |   using SubgroupLayout = Layout<Shape<_8, _1, _1>, Stride<_1, _1, _1>>; 
  51 | };
  52 | 
  53 | struct Shape_h128 {
  54 |   static constexpr int PipelineStages = 2;
  55 |   using ShapeQK = Shape<_128, _64, _64>;
  56 |   using ShapePV = Shape<_128, _32, _64>;
  57 |   using ShapeOutPut = Shape<_128, _128, _64>;
  58 |   using SubgroupLayout = Layout<Shape<_16, _1, _1>, Stride<_1, _1, _1>>; 
  59 | };
  60 | 
  61 | struct Shape_h192 {
  62 |   static constexpr int PipelineStages = 2;
  63 |   using ShapeQK = Shape<_256, _64, _64>;
  64 |   using ShapePV = Shape<_256, _32, _64>;
  65 |   using ShapeOutPut = Shape<_256, _192, _64>;
  66 |   using SubgroupLayout = Layout<Shape<_32, _1, _1>, Stride<_1, _1, _1>>; 
  67 | };
  68 | 
```
**EN:** Defines the tile-shape families reused by cached-KV prefill benchmarks across the supported head sizes.
**CN:** 定义在各个受支持头维上由 cached-KV prefill 基准复用的 tile 形状家族。

### Lines 70-90 — Config generator
```cpp
  70 | struct FMHAPrefillConfigGen {
  71 |  // Todo(codeplay) this type should be passed as parameter as well since come shape may get better performace
  72 |  // with different copy
  73 |   using GmemTiledCopyQ = XE_2D_U16x8x32_LD_N;
  74 |   using GmemTiledCopyK = XE_2D_U16x16x16_LD_T; // _T designates a transposed block load operation
  75 |   using GmemTiledCopyV = XE_2D_U16x16x32_LD_V;
  76 |   using GmemTiledCopyO = XE_2D_U32x8x16_ST_N;
  77 |   using type = cutlass::flash_attention::FMHAPrefillConfig<
  78 |      // todo(codeplay) : accumulator type and output type should be pass as template parameter
  79 |       QKVType, float, float,  
  80 |       GmemTiledCopyQ ,
  81 |       GmemTiledCopyK,
  82 |       GmemTiledCopyV ,
  83 |       GmemTiledCopyO,
  84 |       typename TileShapeConfig::ShapeQK,
  85 |       typename TileShapeConfig::ShapePV,
  86 |       typename TileShapeConfig::ShapeOutPut,
  87 |       typename TileShapeConfig::SubgroupLayout,
  88 |       //TODO: the pagedKV has been set to false, the benchmark for the PagedKV needs to be added here and the parameter need to be set similar 
  89 |       // to causal
  90 |       Causal, VarLen, false, TileShapeConfig::PipelineStages>;
```
**EN:** Builds cached-KV prefill configuration aliases from the element type, shape family, and execution flags.
**CN:** 根据元素类型、形状家族和执行标志构建 cached-KV prefill 配置别名。

### Lines 92-126 — Type-alias matrix
```cpp
  92 | 
  93 | using PvcFMHAPrefillCachedKVBF16BF16FP32_RCR_h64_Causal_FixedLen = FMHAPrefillConfigGen<cutlass::bfloat16_t,  true, false, Shape_h64>::type;
  94 | using PvcFMHAPrefillCachedKVBF16BF16FP32_RCR_h96_Causal_FixedLen = FMHAPrefillConfigGen<cutlass::bfloat16_t, true, false, Shape_h96>::type;
  95 | using PvcFMHAPrefillCachedKVBF16BF16FP32_RCR_h128_Causal_FixedLen = FMHAPrefillConfigGen<cutlass::bfloat16_t, true, false, Shape_h128>::type;
  96 | using PvcFMHAPrefillCachedKVBF16BF16FP32_RCR_h192_Causal_FixedLen = FMHAPrefillConfigGen<cutlass::bfloat16_t, true, false, Shape_h192>::type;
  97 | using PvcFMHAPrefillCachedKVBF16BF16FP32_RCR_h64_Causal_VarLen = FMHAPrefillConfigGen<cutlass::bfloat16_t,  true, true, Shape_h64>::type;
  98 | using PvcFMHAPrefillCachedKVBF16BF16FP32_RCR_h96_Causal_VarLen = FMHAPrefillConfigGen<cutlass::bfloat16_t, true, true, Shape_h96>::type;
  99 | using PvcFMHAPrefillCachedKVBF16BF16FP32_RCR_h128_Causal_VarLen = FMHAPrefillConfigGen<cutlass::bfloat16_t, true, true, Shape_h128>::type;
 100 | using PvcFMHAPrefillCachedKVBF16BF16FP32_RCR_h192_Causal_VarLen = FMHAPrefillConfigGen<cutlass::bfloat16_t, true, true, Shape_h192>::type;
 101 | 
 102 | using PvcFMHAPrefillCachedKVBF16BF16FP32_RCR_h64_NonCausal_FixedLen = FMHAPrefillConfigGen<cutlass::bfloat16_t,  false, false, Shape_h64>::type;
 103 | using PvcFMHAPrefillCachedKVBF16BF16FP32_RCR_h96_NonCausal_FixedLen = FMHAPrefillConfigGen<cutlass::bfloat16_t, false, false, Shape_h96>::type;
 104 | using PvcFMHAPrefillCachedKVBF16BF16FP32_RCR_h128_NonCausal_FixedLen = FMHAPrefillConfigGen<cutlass::bfloat16_t, false, false, Shape_h128>::type;
 105 | using PvcFMHAPrefillCachedKVBF16BF16FP32_RCR_h192_NonCausal_FixedLen = FMHAPrefillConfigGen<cutlass::bfloat16_t, false, false, Shape_h192>::type;
 106 | using PvcFMHAPrefillCachedKVBF16BF16FP32_RCR_h64_NonCausal_VarLen = FMHAPrefillConfigGen<cutlass::bfloat16_t,  false, true, Shape_h64>::type;
 107 | using PvcFMHAPrefillCachedKVBF16BF16FP32_RCR_h96_NonCausal_VarLen = FMHAPrefillConfigGen<cutlass::bfloat16_t,  false, true, Shape_h96>::type;
 108 | using PvcFMHAPrefillCachedKVBF16BF16FP32_RCR_h128_NonCausal_VarLen = FMHAPrefillConfigGen<cutlass::bfloat16_t, false, true, Shape_h128>::type;
 109 | using PvcFMHAPrefillCachedKVBF16BF16FP32_RCR_h192_NonCausal_VarLen = FMHAPrefillConfigGen<cutlass::bfloat16_t, false, true, Shape_h192>::type;
 110 | 
 111 | using PvcFMHAPrefillCachedKVFP16FP16FP32_RCR_h64_Causal_FixedLen = FMHAPrefillConfigGen<cutlass::half_t,  true, false, Shape_h64>::type;
 112 | using PvcFMHAPrefillCachedKVFP16FP16FP32_RCR_h96_Causal_FixedLen = FMHAPrefillConfigGen<cutlass::half_t, true, false, Shape_h96>::type;
 113 | using PvcFMHAPrefillCachedKVFP16FP16FP32_RCR_h128_Causal_FixedLen = FMHAPrefillConfigGen<cutlass::half_t, true, false, Shape_h128>::type;
 114 | using PvcFMHAPrefillCachedKVFP16FP16FP32_RCR_h192_Causal_FixedLen = FMHAPrefillConfigGen<cutlass::half_t, true, false, Shape_h192>::type;
 115 | using PvcFMHAPrefillCachedKVFP16FP16FP32_RCR_h64_Causal_VarLen = FMHAPrefillConfigGen<cutlass::half_t,  true, true, Shape_h64>::type;
 116 | using PvcFMHAPrefillCachedKVFP16FP16FP32_RCR_h96_Causal_VarLen = FMHAPrefillConfigGen<cutlass::half_t,  true, true, Shape_h96>::type;
 117 | using PvcFMHAPrefillCachedKVFP16FP16FP32_RCR_h128_Causal_VarLen = FMHAPrefillConfigGen<cutlass::half_t, true, true, Shape_h128>::type;
 118 | using PvcFMHAPrefillCachedKVFP16FP16FP32_RCR_h192_Causal_VarLen = FMHAPrefillConfigGen<cutlass::half_t, true, true, Shape_h192>::type;
 119 | 
 120 | using PvcFMHAPrefillCachedKVFP16FP16FP32_RCR_h64_NonCausal_FixedLen = FMHAPrefillConfigGen<cutlass::half_t,  false, false, Shape_h64>::type;
 121 | using PvcFMHAPrefillCachedKVFP16FP16FP32_RCR_h96_NonCausal_FixedLen = FMHAPrefillConfigGen<cutlass::half_t,  false, false, Shape_h96>::type;
 122 | using PvcFMHAPrefillCachedKVFP16FP16FP32_RCR_h128_NonCausal_FixedLen = FMHAPrefillConfigGen<cutlass::half_t, false, false, Shape_h128>::type;
 123 | using PvcFMHAPrefillCachedKVFP16FP16FP32_RCR_h192_NonCausal_FixedLen = FMHAPrefillConfigGen<cutlass::half_t, false, false, Shape_h192>::type;
 124 | using PvcFMHAPrefillCachedKVFP16FP16FP32_RCR_h64_NonCausal_VarLen = FMHAPrefillConfigGen<cutlass::half_t,  false, true, Shape_h64>::type;
 125 | using PvcFMHAPrefillCachedKVFP16FP16FP32_RCR_h96_NonCausal_VarLen = FMHAPrefillConfigGen<cutlass::half_t,  false, true, Shape_h96>::type;
 126 | using PvcFMHAPrefillCachedKVFP16FP16FP32_RCR_h128_NonCausal_VarLen = FMHAPrefillConfigGen<cutlass::half_t, false, true, Shape_h128>::type;
```
**EN:** Declares the BF16/FP16, causal/noncausal, and fixed-length/varlen cached-KV benchmark families.
**CN:** 声明 BF16/FP16、因果/非因果、定长/变长的 cached-KV 基准家族。

### Lines 129-161 — Benchmark wrappers
```cpp
 129 | 
 130 | CUTLASS_CREATE_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillCachedKVBF16BF16FP32_RCR_h64_Causal_FixedLen);
 131 | CUTLASS_CREATE_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillCachedKVBF16BF16FP32_RCR_h64_NonCausal_FixedLen);
 132 | CUTLASS_CREATE_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillCachedKVBF16BF16FP32_RCR_h96_Causal_FixedLen);
 133 | CUTLASS_CREATE_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillCachedKVBF16BF16FP32_RCR_h96_NonCausal_FixedLen);
 134 | CUTLASS_CREATE_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillCachedKVBF16BF16FP32_RCR_h128_Causal_FixedLen);
 135 | CUTLASS_CREATE_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillCachedKVBF16BF16FP32_RCR_h128_NonCausal_FixedLen);
 136 | CUTLASS_CREATE_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillCachedKVBF16BF16FP32_RCR_h192_Causal_FixedLen);
 137 | CUTLASS_CREATE_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillCachedKVBF16BF16FP32_RCR_h192_NonCausal_FixedLen);
 138 | CUTLASS_CREATE_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillCachedKVFP16FP16FP32_RCR_h64_Causal_FixedLen);
 139 | CUTLASS_CREATE_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillCachedKVFP16FP16FP32_RCR_h64_NonCausal_FixedLen);
 140 | CUTLASS_CREATE_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillCachedKVFP16FP16FP32_RCR_h96_Causal_FixedLen);
 141 | CUTLASS_CREATE_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillCachedKVFP16FP16FP32_RCR_h96_NonCausal_FixedLen);
 142 | CUTLASS_CREATE_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillCachedKVFP16FP16FP32_RCR_h128_Causal_FixedLen);
 143 | CUTLASS_CREATE_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillCachedKVFP16FP16FP32_RCR_h128_NonCausal_FixedLen);
 144 | CUTLASS_CREATE_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillCachedKVFP16FP16FP32_RCR_h192_Causal_FixedLen);
 145 | CUTLASS_CREATE_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillCachedKVFP16FP16FP32_RCR_h192_NonCausal_FixedLen);
 146 | 
 147 | CUTLASS_CREATE_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillCachedKVBF16BF16FP32_RCR_h64_Causal_VarLen);
 148 | CUTLASS_CREATE_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillCachedKVBF16BF16FP32_RCR_h64_NonCausal_VarLen);
 149 | CUTLASS_CREATE_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillCachedKVBF16BF16FP32_RCR_h96_Causal_VarLen);
 150 | CUTLASS_CREATE_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillCachedKVBF16BF16FP32_RCR_h96_NonCausal_VarLen);
 151 | CUTLASS_CREATE_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillCachedKVBF16BF16FP32_RCR_h128_Causal_VarLen);
 152 | CUTLASS_CREATE_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillCachedKVBF16BF16FP32_RCR_h128_NonCausal_VarLen);
 153 | CUTLASS_CREATE_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillCachedKVBF16BF16FP32_RCR_h192_Causal_VarLen);
 154 | CUTLASS_CREATE_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillCachedKVBF16BF16FP32_RCR_h192_NonCausal_VarLen);
 155 | CUTLASS_CREATE_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillCachedKVFP16FP16FP32_RCR_h64_Causal_VarLen);
 156 | CUTLASS_CREATE_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillCachedKVFP16FP16FP32_RCR_h64_NonCausal_VarLen);
 157 | CUTLASS_CREATE_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillCachedKVFP16FP16FP32_RCR_h96_Causal_VarLen);
 158 | CUTLASS_CREATE_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillCachedKVFP16FP16FP32_RCR_h96_NonCausal_VarLen);
 159 | CUTLASS_CREATE_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillCachedKVFP16FP16FP32_RCR_h128_Causal_VarLen);
 160 | CUTLASS_CREATE_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillCachedKVFP16FP16FP32_RCR_h128_NonCausal_VarLen);
 161 | CUTLASS_CREATE_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillCachedKVFP16FP16FP32_RCR_h192_Causal_VarLen);
```
**EN:** Expands wrapper macros for every cached-KV family member so the registry can call them.
**CN:** 为每个 cached-KV 家族成员展开包装宏，使注册表能够调用它们。

### Lines 163-197 — Registration list
```cpp
 163 | 
 164 | static void register_flash_attention_prefill_benchmarks() {
 165 |   CUTLASS_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillCachedKVBF16BF16FP32_RCR_h64_Causal_FixedLen);
 166 |   CUTLASS_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillCachedKVBF16BF16FP32_RCR_h64_NonCausal_FixedLen);
 167 |   CUTLASS_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillCachedKVBF16BF16FP32_RCR_h96_Causal_FixedLen);
 168 |   CUTLASS_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillCachedKVBF16BF16FP32_RCR_h96_NonCausal_FixedLen);
 169 |   CUTLASS_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillCachedKVBF16BF16FP32_RCR_h128_Causal_FixedLen);
 170 |   CUTLASS_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillCachedKVBF16BF16FP32_RCR_h128_NonCausal_FixedLen);
 171 |   CUTLASS_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillCachedKVBF16BF16FP32_RCR_h192_Causal_FixedLen);
 172 |   CUTLASS_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillCachedKVBF16BF16FP32_RCR_h192_NonCausal_FixedLen);
 173 |   CUTLASS_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillCachedKVFP16FP16FP32_RCR_h64_Causal_FixedLen);
 174 |   CUTLASS_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillCachedKVFP16FP16FP32_RCR_h64_NonCausal_FixedLen);
 175 |   CUTLASS_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillCachedKVFP16FP16FP32_RCR_h96_NonCausal_FixedLen);
 176 |   CUTLASS_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillCachedKVFP16FP16FP32_RCR_h96_Causal_FixedLen);
 177 |   CUTLASS_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillCachedKVFP16FP16FP32_RCR_h128_Causal_FixedLen);
 178 |   CUTLASS_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillCachedKVFP16FP16FP32_RCR_h128_NonCausal_FixedLen);
 179 |   CUTLASS_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillCachedKVFP16FP16FP32_RCR_h192_Causal_FixedLen);
 180 |   CUTLASS_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillCachedKVFP16FP16FP32_RCR_h192_NonCausal_FixedLen);
 181 | 
 182 |   CUTLASS_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillCachedKVBF16BF16FP32_RCR_h64_Causal_VarLen);
 183 |   CUTLASS_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillCachedKVBF16BF16FP32_RCR_h64_NonCausal_VarLen);
 184 |   CUTLASS_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillCachedKVBF16BF16FP32_RCR_h96_Causal_VarLen);
 185 |   CUTLASS_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillCachedKVBF16BF16FP32_RCR_h96_NonCausal_VarLen);
 186 |   CUTLASS_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillCachedKVBF16BF16FP32_RCR_h128_Causal_VarLen);
 187 |   CUTLASS_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillCachedKVBF16BF16FP32_RCR_h128_NonCausal_VarLen);
 188 |   CUTLASS_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillCachedKVBF16BF16FP32_RCR_h192_Causal_VarLen);
 189 |   CUTLASS_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillCachedKVBF16BF16FP32_RCR_h192_NonCausal_VarLen);
 190 |   CUTLASS_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillCachedKVFP16FP16FP32_RCR_h64_Causal_VarLen);
 191 |   CUTLASS_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillCachedKVFP16FP16FP32_RCR_h64_NonCausal_VarLen);
 192 |   CUTLASS_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillCachedKVFP16FP16FP32_RCR_h96_Causal_VarLen);
 193 |   CUTLASS_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillCachedKVFP16FP16FP32_RCR_h96_NonCausal_VarLen);
 194 |   CUTLASS_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillCachedKVFP16FP16FP32_RCR_h128_Causal_VarLen);
 195 |   CUTLASS_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillCachedKVFP16FP16FP32_RCR_h128_NonCausal_VarLen);
 196 |   CUTLASS_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillCachedKVFP16FP16FP32_RCR_h192_Causal_VarLen);
 197 |   CUTLASS_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillCachedKVFP16FP16FP32_RCR_h192_NonCausal_VarLen);
```
**EN:** Registers all cached-KV prefill benchmarks with the legacy suite.
**CN:** 把所有 cached-KV prefill 基准注册到旧版套件中。

### Lines 198-198 — File epilogue
```cpp
 198 | }
```
**EN:** This trailing block closes scopes or keeps small glue code that finalizes the file structure.
**CN:** 这一尾部代码块用于关闭作用域，或保留收尾用的小型胶水代码，从而完成文件结构。

## Key Concepts / 关键概念

- Cached-KV family matrix / cached-KV 家族矩阵
- Shared shape presets / 共享的形状预设
- Explicit registration list / 显式注册列表

## Dependencies / 依赖关系

- `benchmark_runner.hpp` — harness macros / 运行器宏
- `fmha_prefill_configuration.hpp` — cached-KV config builder / cached-KV 配置构建器
