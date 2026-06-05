# benchmarks_cuda.hpp — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `benchmarks/gemm/legacy/benchmarks_cuda.hpp`
- **EN:** Catalog of CUDA GEMM benchmarks for Ampere, covering BF16, FP16, and TF32 with several alignment variants.
- **CN:** 面向 Ampere 的 CUDA GEMM 基准目录，覆盖 BF16、FP16 与 TF32 以及若干对齐变体。

## Line-by-Line Analysis / 逐行分析

### Lines 1-35 — File prologue
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
  32 | #pragma once
  33 | 
  34 | #include "gemm_configuration_cuda.hpp"
  35 | 
```
**EN:** License comments, include directives, and namespace/prologue code establish the compilation context for the definitions that follow.
**CN:** 许可证注释、include 指令以及命名空间/前导代码为后续定义建立编译上下文。

### Lines 36-111 — Configuration aliases
```cpp
  36 | using AmpereGemmBF16BF16FP32_CCC = cutlass::gemm::device::GemmConfiguration<
  37 |         cutlass::arch::Sm80,
  38 |         cutlass::bfloat16_t, cutlass::layout::ColumnMajor, 8,
  39 |         cutlass::bfloat16_t, cutlass::layout::ColumnMajor, 8,
  40 |         float, cutlass::layout::ColumnMajor,
  41 |         float>;
  42 | 
  43 | using AmpereGemmBF16BF16FP32_CCC_kAlignmentA4 = cutlass::gemm::device::GemmConfiguration<
  44 |         cutlass::arch::Sm80,
  45 |         cutlass::bfloat16_t, cutlass::layout::ColumnMajor, 4,
  46 |         cutlass::bfloat16_t, cutlass::layout::ColumnMajor, 8,
  47 |         float, cutlass::layout::ColumnMajor,
  48 |         float>;
  49 | 
  50 | using AmpereGemmBF16BF16FP32_CCC_kAlignmentA1 = cutlass::gemm::device::GemmConfiguration<
  51 |         cutlass::arch::Sm80,
  52 |         cutlass::bfloat16_t, cutlass::layout::ColumnMajor, 1,
  53 |         cutlass::bfloat16_t, cutlass::layout::ColumnMajor, 8,
  54 |         float, cutlass::layout::ColumnMajor,
  55 |         float>;
  56 | 
  57 | using AmpereGemmBF16BF16FP32_CCC_kAlignment1 = cutlass::gemm::device::GemmConfiguration<
  58 |         cutlass::arch::Sm80,
  59 |         cutlass::bfloat16_t, cutlass::layout::ColumnMajor, 1,
  60 |         cutlass::bfloat16_t, cutlass::layout::ColumnMajor, 1,
  61 |         float, cutlass::layout::ColumnMajor,
  62 |         float>;
  63 | 
  64 | using AmpereGemmFP16FP16FP32_CCC = cutlass::gemm::device::GemmConfiguration<
  65 |         cutlass::arch::Sm80,
  66 |         cutlass::half_t, cutlass::layout::ColumnMajor, 8,
  67 |         cutlass::half_t, cutlass::layout::ColumnMajor, 8,
  68 |         float, cutlass::layout::ColumnMajor,
  69 |         float>;
  70 | 
  71 | using AmpereGemmFP16FP16FP32_CCC_kAlignmentA4 = cutlass::gemm::device::GemmConfiguration<
  72 |         cutlass::arch::Sm80,
  73 |         cutlass::half_t, cutlass::layout::ColumnMajor, 4,
  74 |         cutlass::half_t, cutlass::layout::ColumnMajor, 8,
  75 |         float, cutlass::layout::ColumnMajor,
  76 |         float>;
  77 | 
  78 | using AmpereGemmFP16FP16FP32_CCC_kAlignmentA1 = cutlass::gemm::device::GemmConfiguration<
  79 |         cutlass::arch::Sm80,
  80 |         cutlass::half_t, cutlass::layout::ColumnMajor, 1,
  81 |         cutlass::half_t, cutlass::layout::ColumnMajor, 8,
  82 |         float, cutlass::layout::ColumnMajor,
  83 |         float>;
  84 | 
  85 | using AmpereGemmFP16FP16FP32_CCC_kAlignment1 = cutlass::gemm::device::GemmConfiguration<
  86 |         cutlass::arch::Sm80,
  87 |         cutlass::half_t, cutlass::layout::ColumnMajor, 1,
  88 |         cutlass::half_t, cutlass::layout::ColumnMajor, 1,
  89 |         float, cutlass::layout::ColumnMajor,
  90 |         float>;
  91 | 
  92 | using AmpereGemmTF32TF32FP32_CCC = cutlass::gemm::device::GemmConfiguration<
  93 |         cutlass::arch::Sm80,
  94 |         float, cutlass::layout::ColumnMajor, 4,
  95 |         float, cutlass::layout::ColumnMajor, 4,
  96 |         float, cutlass::layout::ColumnMajor,
  97 |         float>;
  98 | 
  99 | using AmpereGemmTF32TF32FP32_CCC_kAlignmentA1 = cutlass::gemm::device::GemmConfiguration<
 100 |         cutlass::arch::Sm80,
 101 |         float, cutlass::layout::ColumnMajor, 1,
 102 |         float, cutlass::layout::ColumnMajor, 4,
 103 |         float, cutlass::layout::ColumnMajor,
 104 |         float>;
 105 | 
 106 | using AmpereGemmTF32TF32FP32_CCC_kAlignment1 = cutlass::gemm::device::GemmConfiguration<
 107 |         cutlass::arch::Sm80,
 108 |         float, cutlass::layout::ColumnMajor, 1,
 109 |         float, cutlass::layout::ColumnMajor, 1,
 110 |         float, cutlass::layout::ColumnMajor,
 111 |         float>;
```
**EN:** Declares Ampere BF16, FP16, and TF32 GEMM configurations with different input-alignment combinations.
**CN:** 声明 Ampere BF16、FP16 和 TF32 GEMM 配置，并覆盖不同的输入对齐组合。

### Lines 113-125 — Benchmark wrappers
```cpp
 113 | CUTLASS_CREATE_GEMM_BENCHMARK(AmpereGemmBF16BF16FP32_CCC);
 114 | CUTLASS_CREATE_GEMM_BENCHMARK(AmpereGemmBF16BF16FP32_CCC_kAlignment1);
 115 | CUTLASS_CREATE_GEMM_BENCHMARK(AmpereGemmBF16BF16FP32_CCC_kAlignmentA1);
 116 | CUTLASS_CREATE_GEMM_BENCHMARK(AmpereGemmBF16BF16FP32_CCC_kAlignmentA4);
 117 | 
 118 | CUTLASS_CREATE_GEMM_BENCHMARK(AmpereGemmFP16FP16FP32_CCC);
 119 | CUTLASS_CREATE_GEMM_BENCHMARK(AmpereGemmFP16FP16FP32_CCC_kAlignment1);
 120 | CUTLASS_CREATE_GEMM_BENCHMARK(AmpereGemmFP16FP16FP32_CCC_kAlignmentA1);
 121 | CUTLASS_CREATE_GEMM_BENCHMARK(AmpereGemmFP16FP16FP32_CCC_kAlignmentA4);
 122 | 
 123 | CUTLASS_CREATE_GEMM_BENCHMARK(AmpereGemmTF32TF32FP32_CCC);
 124 | CUTLASS_CREATE_GEMM_BENCHMARK(AmpereGemmTF32TF32FP32_CCC_kAlignment1);
 125 | CUTLASS_CREATE_GEMM_BENCHMARK(AmpereGemmTF32TF32FP32_CCC_kAlignmentA1);
```
**EN:** Expands the benchmark-wrapper macro once per configuration alias.
**CN:** 对每个配置别名展开一次基准包装宏。

### Lines 127-141 — Registration list
```cpp
 127 | static void register_gemm_benchmarks() {
 128 |   CUTLASS_BENCHMARK(AmpereGemmBF16BF16FP32_CCC);
 129 |   CUTLASS_BENCHMARK(AmpereGemmBF16BF16FP32_CCC_kAlignment1);
 130 |   CUTLASS_BENCHMARK(AmpereGemmBF16BF16FP32_CCC_kAlignmentA1);
 131 |   CUTLASS_BENCHMARK(AmpereGemmBF16BF16FP32_CCC_kAlignmentA4);
 132 | 
 133 |   CUTLASS_BENCHMARK(AmpereGemmFP16FP16FP32_CCC);
 134 |   CUTLASS_BENCHMARK(AmpereGemmFP16FP16FP32_CCC_kAlignment1);
 135 |   CUTLASS_BENCHMARK(AmpereGemmFP16FP16FP32_CCC_kAlignmentA1);
 136 |   CUTLASS_BENCHMARK(AmpereGemmFP16FP16FP32_CCC_kAlignmentA4);
 137 | 
 138 |   CUTLASS_BENCHMARK(AmpereGemmTF32TF32FP32_CCC);
 139 |   CUTLASS_BENCHMARK(AmpereGemmTF32TF32FP32_CCC_kAlignment1);
 140 |   CUTLASS_BENCHMARK(AmpereGemmTF32TF32FP32_CCC_kAlignmentA1);
 141 | }
```
**EN:** Registers the CUDA GEMM benchmarks with the suite entry point.
**CN:** 把 CUDA GEMM 基准注册到套件入口中。

## Key Concepts / 关键概念

- Ampere GEMM families / Ampere GEMM 家族
- Alignment variants / 对齐变体
- Macro-driven registration / 宏驱动注册

## Dependencies / 依赖关系

- `gemm_configuration_cuda.hpp` — CUDA config templates / CUDA 配置模板
- `CUTLASS_CREATE_GEMM_BENCHMARK` — wrapper macro / 包装宏
- `CUTLASS_BENCHMARK` — registry macro / 注册宏
