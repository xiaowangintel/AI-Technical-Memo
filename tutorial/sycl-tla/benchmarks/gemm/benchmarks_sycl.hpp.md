# benchmarks_sycl.hpp — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `benchmarks/gemm/benchmarks_sycl.hpp`
- **EN:** Current SYCL GEMM benchmark catalog focused on a single Intel Xe BF16→FP32 row-major benchmark.
- **CN:** 当前的 SYCL GEMM 基准目录，聚焦于一个 Intel Xe 的 BF16→FP32 行主序基准。

## Line-by-Line Analysis / 逐行分析

### Lines 1-33 — File prologue
```cpp
   1 | /***************************************************************************************************
   2 |  * Copyright (c) 2024 - 2024 Codeplay Software Ltd. All rights reserved.
   3 |  * Copyright (C) 2025 Intel Corporation, All rights reserved.
   4 |  * SPDX-License-Identifier: BSD-3-Clause
   5 |  *
   6 |  * Redistribution and use in source and binary forms, with or without
   7 |  * modification, are permitted provided that the following conditions are met:
   8 |  *
   9 |  * 1. Redistributions of source code must retain the above copyright notice, this
  10 |  * list of conditions and the following disclaimer.
  11 |  *
  12 |  * 2. Redistributions in binary form must reproduce the above copyright notice,
  13 |  * this list of conditions and the following disclaimer in the documentation
  14 |  * and/or other materials provided with the distribution.
  15 |  *
  16 |  * 3. Neither the name of the copyright holder nor the names of its
  17 |  * contributors may be used to endorse or promote products derived from
  18 |  * this software without specific prior written permission.
  19 |  *
  20 |  * THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
  21 |  * AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
  22 |  * IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
  23 |  * DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE
  24 |  * FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
  25 |  * DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
  26 |  * SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
  27 |  * CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
  28 |  * OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
  29 |  * OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
  30 |  *
  31 |  **************************************************************************************************/
  32 | 
  33 |  #pragma once
```
**EN:** License comments, include directives, and namespace/prologue code establish the compilation context for the definitions that follow.
**CN:** 许可证注释、include 指令以及命名空间/前导代码为后续定义建立编译上下文。

### Lines 34-56 — Configuration alias and wrapper
```cpp
  34 | 
  35 | #include "gemm_configuration_sycl.hpp"
  36 | 
  37 | using Scheduler = cutlass::gemm::device::Scheduler;
  38 | 
  39 | template <
  40 |   typename TileShape,
  41 |   typename Tiler,
  42 |   typename GmemTiledCopyA,
  43 |   typename GmemTiledCopyB>
  44 | using Gemm_Bench_BF16FP32_RRR = cutlass::gemm::device::GemmConfiguration<
  45 |     cutlass::arch::IntelXe,
  46 |     cutlass::bfloat16_t, cutlass::layout::RowMajor,
  47 |     cutlass::bfloat16_t, cutlass::layout::RowMajor,
  48 |     float, cutlass::layout::RowMajor,
  49 |     float,
  50 |     TileShape, Scheduler::Gemm, Tiler,
  51 |     GmemTiledCopyA, GmemTiledCopyB>;
  52 | 
  53 | using BmgGemm_BF16FP32_TileShape_512_256_32 = Shape<_512, _256, _32>;
  54 | using BmgGemm_BF16FP32_Tile_512_256_32 = typename TiledMMAHelper<MMA_Atom<XE_DPAS_TT<8, float, cute::bfloat16_t>>, Layout<BmgGemm_BF16FP32_TileShape_512_256_32>, Layout<Shape<_8, _4, _1>, Stride<_4, _1, _0>>>::TiledMMA;
  55 | using BmgGemmBF16BF16FP32_RRR_TileShape_512_256_32 = Gemm_Bench_BF16FP32_RRR<BmgGemm_BF16FP32_TileShape_512_256_32, BmgGemm_BF16FP32_Tile_512_256_32, void, void>;
  56 | CUTLASS_CREATE_GEMM_BENCHMARK(BmgGemmBF16BF16FP32_RRR_TileShape_512_256_32);
```
**EN:** Defines the tile shape, tiled MMA object, configuration alias, and wrapper macro expansion for the current Intel Xe benchmark.
**CN:** 定义当前 Intel Xe 基准所需的 tile 形状、tiled MMA 对象、配置别名以及包装宏展开。

### Lines 58-61 — Registration entry point
```cpp
  58 | static void register_gemm_benchmarks() {
  59 | // TODO: support sglang cases
  60 |   CUTLASS_BENCHMARK(BmgGemmBF16BF16FP32_RRR_TileShape_512_256_32);
  61 | }
```
**EN:** Registers the single current SYCL GEMM benchmark with the suite.
**CN:** 把当前唯一的 SYCL GEMM 基准注册到套件中。

## Key Concepts / 关键概念

- Intel Xe GEMM specialization / Intel Xe GEMM 特化
- Single representative benchmark / 单个代表性基准

## Dependencies / 依赖关系

- `gemm_configuration_sycl.hpp` — SYCL config template / SYCL 配置模板
- `TiledMMAHelper` and `XE_DPAS_TT` — Xe MMA description / Xe MMA 描述
