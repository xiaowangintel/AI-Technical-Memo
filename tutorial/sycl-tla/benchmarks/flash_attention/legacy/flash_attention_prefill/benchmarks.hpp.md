# benchmarks.hpp — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `benchmarks/flash_attention/legacy/flash_attention_prefill/benchmarks.hpp`
- **EN:** Legacy prefill benchmark catalog that expands shape presets and dtype combinations into concrete benchmark registrations.
- **CN:** 旧版 prefill 基准目录：把形状预设与数据类型组合展开为具体的基准注册项。

## Line-by-Line Analysis / 逐行分析

### Lines 1-36 — File prologue
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
```
**EN:** License comments, include directives, and namespace/prologue code establish the compilation context for the definitions that follow.
**CN:** 许可证注释、include 指令以及命名空间/前导代码为后续定义建立编译上下文。

### Lines 37-67 — Shape presets
```cpp
  37 | 
  38 | struct Shape_h64 {
  39 |   static constexpr int PipelineStages = 2;
  40 |   using ShapeQK = Shape<_128, _64, _64>;
  41 |   using ShapePV = Shape<_128, _32, _64>;
  42 |   using ShapeOutPut = Shape<_128, _64, _64>;
  43 |   using SubgroupLayout = Layout<Shape<_8, _1, _1>, Stride<_1, _1, _1>>;
  44 | };
  45 | 
  46 | struct Shape_h96 {
  47 |   static constexpr int PipelineStages = 2;
  48 |   using ShapeQK = Shape<_128, _64, _32>;
  49 |   using ShapePV = Shape<_128, _32, _64>;
  50 |   using ShapeOutPut = Shape<_128, _96, _64>;
  51 |   using SubgroupLayout = Layout<Shape<_8, _1, _1>, Stride<_1, _1, _1>>; 
  52 | };
  53 | 
  54 | struct Shape_h128 {
  55 |   static constexpr int PipelineStages = 2;
  56 |   using ShapeQK = Shape<_128, _64, _64>;
  57 |   using ShapePV = Shape<_128, _32, _64>;
  58 |   using ShapeOutPut = Shape<_128, _128, _64>;
  59 |   using SubgroupLayout = Layout<Shape<_16, _1, _1>, Stride<_1, _1, _1>>; 
  60 | };
  61 | 
  62 | struct Shape_h192 {
  63 |   static constexpr int PipelineStages = 2;
  64 |   using ShapeQK = Shape<_256, _64, _64>;
  65 |   using ShapePV = Shape<_256, _32, _64>;
  66 |   using ShapeOutPut = Shape<_256, _192, _64>;
  67 |   using SubgroupLayout = Layout<Shape<_32, _1, _1>, Stride<_1, _1, _1>>; 
```
**EN:** Defines reusable tile shapes, subgroup layouts, and pipeline-stage constants for each supported head-size family.
**CN:** 为每个受支持的头维家族定义可复用的 tile 形状、subgroup 布局和 pipeline stage 常量。

### Lines 69-90 — Config generator
```cpp
  69 | 
  70 | template<class QKVType, bool Causal, bool VarLen, class TileShapeConfig>
  71 | struct FMHAPrefillConfigGen {
  72 |  // Todo(codeplay) this type should be passed as parameter as well since come shape may get better performace
  73 |  // with different copy
  74 |   using GmemTiledCopyQ = XE_2D_U16x8x32_LD_N;
  75 |   using GmemTiledCopyK = XE_2D_U16x16x16_LD_T; // _T designates a transposed block load operation
  76 |   using GmemTiledCopyV = XE_2D_U16x16x32_LD_V;
  77 |   using GmemTiledCopyO = XE_2D_U32x8x16_ST_N;
  78 |   using type = cutlass::flash_attention::FMHAPrefillConfig<
  79 |      // todo(codeplay) : accumulator type and output type should be pass as template parameter
  80 |       QKVType, float, float,  
  81 |       GmemTiledCopyQ ,
  82 |       GmemTiledCopyK,
  83 |       GmemTiledCopyV ,
  84 |       GmemTiledCopyO,
  85 |       typename TileShapeConfig::ShapeQK,
  86 |       typename TileShapeConfig::ShapePV,
  87 |       typename TileShapeConfig::ShapeOutPut,
  88 |       typename TileShapeConfig::SubgroupLayout,
  89 |       Causal, VarLen, TileShapeConfig::PipelineStages>;
  90 | };
```
**EN:** Combines element type, shape preset, and execution flags into a concrete legacy prefill configuration type.
**CN:** 把元素类型、形状预设和执行标志组合成具体的旧版 prefill 配置类型。

### Lines 93-127 — Type-alias matrix
```cpp
  93 | using PvcFMHAPrefillBF16BF16FP32_RCR_h96_Causal_FixedLen = FMHAPrefillConfigGen<cutlass::bfloat16_t, true, false, Shape_h96>::type;
  94 | using PvcFMHAPrefillBF16BF16FP32_RCR_h128_Causal_FixedLen = FMHAPrefillConfigGen<cutlass::bfloat16_t, true, false, Shape_h128>::type;
  95 | using PvcFMHAPrefillBF16BF16FP32_RCR_h192_Causal_FixedLen = FMHAPrefillConfigGen<cutlass::bfloat16_t, true, false, Shape_h192>::type;
  96 | using PvcFMHAPrefillBF16BF16FP32_RCR_h64_Causal_VarLen = FMHAPrefillConfigGen<cutlass::bfloat16_t,  true, true, Shape_h64>::type;
  97 | using PvcFMHAPrefillBF16BF16FP32_RCR_h96_Causal_VarLen = FMHAPrefillConfigGen<cutlass::bfloat16_t, true, true, Shape_h96>::type;
  98 | using PvcFMHAPrefillBF16BF16FP32_RCR_h128_Causal_VarLen = FMHAPrefillConfigGen<cutlass::bfloat16_t, true, true, Shape_h128>::type;
  99 | using PvcFMHAPrefillBF16BF16FP32_RCR_h192_Causal_VarLen = FMHAPrefillConfigGen<cutlass::bfloat16_t, true, true, Shape_h192>::type;
 100 | 
 101 | using PvcFMHAPrefillBF16BF16FP32_RCR_h64_NonCausal_FixedLen = FMHAPrefillConfigGen<cutlass::bfloat16_t,  false, false, Shape_h64>::type;
 102 | using PvcFMHAPrefillBF16BF16FP32_RCR_h96_NonCausal_FixedLen = FMHAPrefillConfigGen<cutlass::bfloat16_t, false, false, Shape_h96>::type;
 103 | using PvcFMHAPrefillBF16BF16FP32_RCR_h128_NonCausal_FixedLen = FMHAPrefillConfigGen<cutlass::bfloat16_t, false, false, Shape_h128>::type;
 104 | using PvcFMHAPrefillBF16BF16FP32_RCR_h192_NonCausal_FixedLen = FMHAPrefillConfigGen<cutlass::bfloat16_t, false, false, Shape_h192>::type;
 105 | using PvcFMHAPrefillBF16BF16FP32_RCR_h64_NonCausal_VarLen = FMHAPrefillConfigGen<cutlass::bfloat16_t,  false, true, Shape_h64>::type;
 106 | using PvcFMHAPrefillBF16BF16FP32_RCR_h96_NonCausal_VarLen = FMHAPrefillConfigGen<cutlass::bfloat16_t,  false, true, Shape_h96>::type;
 107 | using PvcFMHAPrefillBF16BF16FP32_RCR_h128_NonCausal_VarLen = FMHAPrefillConfigGen<cutlass::bfloat16_t, false, true, Shape_h128>::type;
 108 | using PvcFMHAPrefillBF16BF16FP32_RCR_h192_NonCausal_VarLen = FMHAPrefillConfigGen<cutlass::bfloat16_t, false, true, Shape_h192>::type;
 109 | 
 110 | using PvcFMHAPrefillFP16FP16FP32_RCR_h64_Causal_FixedLen = FMHAPrefillConfigGen<cutlass::half_t,  true, false, Shape_h64>::type;
 111 | using PvcFMHAPrefillFP16FP16FP32_RCR_h96_Causal_FixedLen = FMHAPrefillConfigGen<cutlass::half_t, true, false, Shape_h96>::type;
 112 | using PvcFMHAPrefillFP16FP16FP32_RCR_h128_Causal_FixedLen = FMHAPrefillConfigGen<cutlass::half_t, true, false, Shape_h128>::type;
 113 | using PvcFMHAPrefillFP16FP16FP32_RCR_h192_Causal_FixedLen = FMHAPrefillConfigGen<cutlass::half_t, true, false, Shape_h192>::type;
 114 | using PvcFMHAPrefillFP16FP16FP32_RCR_h64_Causal_VarLen = FMHAPrefillConfigGen<cutlass::half_t,  true, true, Shape_h64>::type;
 115 | using PvcFMHAPrefillFP16FP16FP32_RCR_h96_Causal_VarLen = FMHAPrefillConfigGen<cutlass::half_t,  true, true, Shape_h96>::type;
 116 | using PvcFMHAPrefillFP16FP16FP32_RCR_h128_Causal_VarLen = FMHAPrefillConfigGen<cutlass::half_t, true, true, Shape_h128>::type;
 117 | using PvcFMHAPrefillFP16FP16FP32_RCR_h192_Causal_VarLen = FMHAPrefillConfigGen<cutlass::half_t, true, true, Shape_h192>::type;
 118 | 
 119 | using PvcFMHAPrefillFP16FP16FP32_RCR_h64_NonCausal_FixedLen = FMHAPrefillConfigGen<cutlass::half_t,  false, false, Shape_h64>::type;
 120 | using PvcFMHAPrefillFP16FP16FP32_RCR_h96_NonCausal_FixedLen = FMHAPrefillConfigGen<cutlass::half_t,  false, false, Shape_h96>::type;
 121 | using PvcFMHAPrefillFP16FP16FP32_RCR_h128_NonCausal_FixedLen = FMHAPrefillConfigGen<cutlass::half_t, false, false, Shape_h128>::type;
 122 | using PvcFMHAPrefillFP16FP16FP32_RCR_h192_NonCausal_FixedLen = FMHAPrefillConfigGen<cutlass::half_t, false, false, Shape_h192>::type;
 123 | using PvcFMHAPrefillFP16FP16FP32_RCR_h64_NonCausal_VarLen = FMHAPrefillConfigGen<cutlass::half_t,  false, true, Shape_h64>::type;
 124 | using PvcFMHAPrefillFP16FP16FP32_RCR_h96_NonCausal_VarLen = FMHAPrefillConfigGen<cutlass::half_t,  false, true, Shape_h96>::type;
 125 | using PvcFMHAPrefillFP16FP16FP32_RCR_h128_NonCausal_VarLen = FMHAPrefillConfigGen<cutlass::half_t, false, true, Shape_h128>::type;
 126 | using PvcFMHAPrefillFP16FP16FP32_RCR_h192_NonCausal_VarLen = FMHAPrefillConfigGen<cutlass::half_t, false, true, Shape_h192>::type;
 127 | 
```
**EN:** Declares the BF16/FP16, causal/noncausal, and fixed-length/varlen benchmark families across the available head sizes.
**CN:** 声明跨所有可用头维的 BF16/FP16、因果/非因果、定长/变长基准家族。

### Lines 130-162 — Benchmark wrappers
```cpp
 130 | CUTLASS_CREATE_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillBF16BF16FP32_RCR_h64_NonCausal_FixedLen);
 131 | CUTLASS_CREATE_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillBF16BF16FP32_RCR_h96_Causal_FixedLen);
 132 | CUTLASS_CREATE_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillBF16BF16FP32_RCR_h96_NonCausal_FixedLen);
 133 | CUTLASS_CREATE_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillBF16BF16FP32_RCR_h128_Causal_FixedLen);
 134 | CUTLASS_CREATE_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillBF16BF16FP32_RCR_h128_NonCausal_FixedLen);
 135 | CUTLASS_CREATE_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillBF16BF16FP32_RCR_h192_Causal_FixedLen);
 136 | CUTLASS_CREATE_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillBF16BF16FP32_RCR_h192_NonCausal_FixedLen);
 137 | CUTLASS_CREATE_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillFP16FP16FP32_RCR_h64_Causal_FixedLen);
 138 | CUTLASS_CREATE_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillFP16FP16FP32_RCR_h64_NonCausal_FixedLen);
 139 | CUTLASS_CREATE_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillFP16FP16FP32_RCR_h96_Causal_FixedLen);
 140 | CUTLASS_CREATE_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillFP16FP16FP32_RCR_h96_NonCausal_FixedLen);
 141 | CUTLASS_CREATE_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillFP16FP16FP32_RCR_h128_Causal_FixedLen);
 142 | CUTLASS_CREATE_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillFP16FP16FP32_RCR_h128_NonCausal_FixedLen);
 143 | CUTLASS_CREATE_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillFP16FP16FP32_RCR_h192_Causal_FixedLen);
 144 | CUTLASS_CREATE_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillFP16FP16FP32_RCR_h192_NonCausal_FixedLen);
 145 | 
 146 | CUTLASS_CREATE_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillBF16BF16FP32_RCR_h64_Causal_VarLen);
 147 | CUTLASS_CREATE_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillBF16BF16FP32_RCR_h64_NonCausal_VarLen);
 148 | CUTLASS_CREATE_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillBF16BF16FP32_RCR_h96_Causal_VarLen);
 149 | CUTLASS_CREATE_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillBF16BF16FP32_RCR_h96_NonCausal_VarLen);
 150 | CUTLASS_CREATE_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillBF16BF16FP32_RCR_h128_Causal_VarLen);
 151 | CUTLASS_CREATE_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillBF16BF16FP32_RCR_h128_NonCausal_VarLen);
 152 | CUTLASS_CREATE_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillBF16BF16FP32_RCR_h192_Causal_VarLen);
 153 | CUTLASS_CREATE_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillBF16BF16FP32_RCR_h192_NonCausal_VarLen);
 154 | CUTLASS_CREATE_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillFP16FP16FP32_RCR_h64_Causal_VarLen);
 155 | CUTLASS_CREATE_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillFP16FP16FP32_RCR_h64_NonCausal_VarLen);
 156 | CUTLASS_CREATE_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillFP16FP16FP32_RCR_h96_Causal_VarLen);
 157 | CUTLASS_CREATE_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillFP16FP16FP32_RCR_h96_NonCausal_VarLen);
 158 | CUTLASS_CREATE_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillFP16FP16FP32_RCR_h128_Causal_VarLen);
 159 | CUTLASS_CREATE_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillFP16FP16FP32_RCR_h128_NonCausal_VarLen);
 160 | CUTLASS_CREATE_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillFP16FP16FP32_RCR_h192_Causal_VarLen);
 161 | CUTLASS_CREATE_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillFP16FP16FP32_RCR_h192_NonCausal_VarLen);
 162 | 
```
**EN:** Expands the wrapper macro for each configuration alias so each family becomes callable by the benchmark registry.
**CN:** 为每个配置别名展开包装宏，使每个家族都可以被基准注册表调用。

### Lines 164-197 — Registration list
```cpp
 164 |   CUTLASS_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillBF16BF16FP32_RCR_h64_Causal_FixedLen);
 165 |   CUTLASS_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillBF16BF16FP32_RCR_h64_NonCausal_FixedLen);
 166 |   CUTLASS_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillBF16BF16FP32_RCR_h96_Causal_FixedLen);
 167 |   CUTLASS_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillBF16BF16FP32_RCR_h96_NonCausal_FixedLen);
 168 |   CUTLASS_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillBF16BF16FP32_RCR_h128_Causal_FixedLen);
 169 |   CUTLASS_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillBF16BF16FP32_RCR_h128_NonCausal_FixedLen);
 170 |   CUTLASS_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillBF16BF16FP32_RCR_h192_Causal_FixedLen);
 171 |   CUTLASS_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillBF16BF16FP32_RCR_h192_NonCausal_FixedLen);
 172 |   CUTLASS_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillFP16FP16FP32_RCR_h64_Causal_FixedLen);
 173 |   CUTLASS_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillFP16FP16FP32_RCR_h64_NonCausal_FixedLen);
 174 |   CUTLASS_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillFP16FP16FP32_RCR_h96_NonCausal_FixedLen);
 175 |   CUTLASS_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillFP16FP16FP32_RCR_h96_Causal_FixedLen);
 176 |   CUTLASS_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillFP16FP16FP32_RCR_h128_Causal_FixedLen);
 177 |   CUTLASS_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillFP16FP16FP32_RCR_h128_NonCausal_FixedLen);
 178 |   CUTLASS_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillFP16FP16FP32_RCR_h192_Causal_FixedLen);
 179 |   CUTLASS_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillFP16FP16FP32_RCR_h192_NonCausal_FixedLen);
 180 | 
 181 |   CUTLASS_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillBF16BF16FP32_RCR_h64_Causal_VarLen);
 182 |   CUTLASS_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillBF16BF16FP32_RCR_h64_NonCausal_VarLen);
 183 |   CUTLASS_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillBF16BF16FP32_RCR_h96_Causal_VarLen);
 184 |   CUTLASS_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillBF16BF16FP32_RCR_h96_NonCausal_VarLen);
 185 |   CUTLASS_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillBF16BF16FP32_RCR_h128_Causal_VarLen);
 186 |   CUTLASS_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillBF16BF16FP32_RCR_h128_NonCausal_VarLen);
 187 |   CUTLASS_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillBF16BF16FP32_RCR_h192_Causal_VarLen);
 188 |   CUTLASS_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillBF16BF16FP32_RCR_h192_NonCausal_VarLen);
 189 |   CUTLASS_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillFP16FP16FP32_RCR_h64_Causal_VarLen);
 190 |   CUTLASS_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillFP16FP16FP32_RCR_h64_NonCausal_VarLen);
 191 |   CUTLASS_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillFP16FP16FP32_RCR_h96_Causal_VarLen);
 192 |   CUTLASS_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillFP16FP16FP32_RCR_h96_NonCausal_VarLen);
 193 |   CUTLASS_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillFP16FP16FP32_RCR_h128_Causal_VarLen);
 194 |   CUTLASS_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillFP16FP16FP32_RCR_h128_NonCausal_VarLen);
 195 |   CUTLASS_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillFP16FP16FP32_RCR_h192_Causal_VarLen);
 196 |   CUTLASS_FMHA_PREFILL_BENCHMARK(PvcFMHAPrefillFP16FP16FP32_RCR_h192_NonCausal_VarLen);
 197 | }
```
**EN:** Registers every generated prefill benchmark with the legacy prefill suite.
**CN:** 把所有生成的 prefill 基准注册到旧版 prefill 套件中。

## Key Concepts / 关键概念

- Head-size shape presets / 头维形状预设
- BF16 and FP16 families / BF16 与 FP16 家族
- Explicit registration matrix / 显式注册矩阵

## Dependencies / 依赖关系

- `benchmark_runner.hpp` — harness macros / 运行器宏
- `fmha_prefill_configuration.hpp` — config type builder / 配置类型构建器
