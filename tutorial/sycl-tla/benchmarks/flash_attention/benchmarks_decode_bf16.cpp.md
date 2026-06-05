# benchmarks_decode_bf16.cpp — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `benchmarks/flash_attention/benchmarks_decode_bf16.cpp`
- **EN:** Registers the current BF16 decode benchmark configurations for Flash Attention.
- **CN:** 注册当前版本 Flash Attention 的 BF16 decode 基准配置。

## Line-by-Line Analysis / 逐行分析

### Lines 1-37 — File prologue
```cpp
   1 | /***************************************************************************************************
   2 |  * Copyright (c) 2026 Intel Corporation. All rights reserved.
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
  32 | 
  33 | #include "benchmark_runner.hpp"
  34 | #include "fmha_configuration.hpp"
  35 | 
  36 | using namespace cutlass::flash_attention;
  37 | 
```
**EN:** License comments, include directives, and namespace/prologue code establish the compilation context for the definitions that follow.
**CN:** 许可证注释、include 指令以及命名空间/前导代码为后续定义建立编译上下文。

### Lines 38-45 — Default h64 decode config
```cpp
  38 | /* ---------------------------------------- HeadDim = 64 ------------------------------------------ */
  39 | using BmgFMHADecode_BF16_BF16_BF16_FP32_RCR_h64_Causal_VarLen = FMHAConfigGen</*Mode*/FMHAMode::Decode,
  40 |   /*ElementQ*/ cutlass::bfloat16_t, /*ElementK*/ cutlass::bfloat16_t, /*ElementV*/ cutlass::bfloat16_t, /*ElementO*/ float,
  41 |   /*LayoutQ*/ cutlass::layout::RowMajor, /*LayoutK*/ cutlass::layout::ColumnMajor, /*LayoutV*/ cutlass::layout::RowMajor, /*LayoutO*/ cutlass::layout::RowMajor,
  42 |   /*Causal*/ true, /*VarLen*/ true, /*CachedKV*/ false, /*PagedKV*/ false, /*Persistent*/ false, /*HeadDim*/ 64
  43 | >::type;
  44 | 
  45 | CUTLASS_CREATE_FMHA_BENCHMARK(BmgFMHADecode_BF16_BF16_BF16_FP32_RCR_h64_Causal_VarLen);
```
**EN:** Defines a BF16 decode benchmark using the standard head-dimension-64 shape with causal and variable-length execution.
**CN:** 定义一个使用标准 head-dim=64 形状的 BF16 decode 基准，启用因果与变长执行。

### Lines 47-56 — Explicit-tile decode config
```cpp
  47 | /* ---------------------------------------- Custom Tiles ------------------------------------------ */
  48 | 
  49 | using BmgFMHADecode_BF16_BF16_BF16_FP32_RCR_WgQ128K64V32_SgQ8K64_HDimQK32V64_NonCausal_FixedLen = FMHAConfigGenWithTileShape</*Mode*/FMHAMode::Decode,
  50 |   /*ElementQ*/ cutlass::bfloat16_t, /*ElementK*/ cutlass::bfloat16_t, /*ElementV*/ cutlass::bfloat16_t, /*ElementO*/ float,
  51 |   /*LayoutQ*/ cutlass::layout::RowMajor, /*LayoutK*/ cutlass::layout::ColumnMajor, /*LayoutV*/ cutlass::layout::RowMajor, /*LayoutO*/ cutlass::layout::RowMajor,
  52 |   /*Causal*/ false, /*VarLen*/ false, /*CachedKV*/ false, /*PagedKV*/ false, /*Persistent*/ false, /*WgTileQ*/ 128, /*WgTileK*/ 64, /*WgTileV*/ 32,
  53 |   /*SgTileQ*/ 8, /*SgTileK*/ 64, /*HeadDimQK*/ 32, /*HeadDimV*/ 64
  54 | >::type;
  55 | 
  56 | CUTLASS_CREATE_FMHA_BENCHMARK(BmgFMHADecode_BF16_BF16_BF16_FP32_RCR_WgQ128K64V32_SgQ8K64_HDimQK32V64_NonCausal_FixedLen);
```
**EN:** Builds a second decode benchmark with explicitly specified workgroup and subgroup tile sizes instead of the default shape lookup.
**CN:** 构建第二个 decode 基准：不使用默认形状查表，而是显式指定 workgroup 和 subgroup tile 大小。

### Lines 58-62 — Suite registration
```cpp
  58 | static void register_flash_attention_decode_benchmarks_bf16() {
  59 |   CUTLASS_FMHA_BENCHMARK(BmgFMHADecode_BF16_BF16_BF16_FP32_RCR_h64_Causal_VarLen);
  60 | 
  61 |   CUTLASS_FMHA_BENCHMARK(BmgFMHADecode_BF16_BF16_BF16_FP32_RCR_WgQ128K64V32_SgQ8K64_HDimQK32V64_NonCausal_FixedLen);
  62 | }
```
**EN:** Registers both decode benchmark wrappers into the current Flash Attention decode suite.
**CN:** 把两个 decode 基准包装函数注册到当前 Flash Attention decode 套件中。

## Key Concepts / 关键概念

- Decode-mode FMHA aliases / decode 模式 FMHA 类型别名
- Default and explicit tile-shape generation / 默认与显式 tile 生成
- Macro-based benchmark registration / 基于宏的基准注册

## Dependencies / 依赖关系

- `benchmark_runner.hpp` — FMHA benchmark harness / FMHA 基准运行器
- `fmha_configuration.hpp` — config generators / 配置生成器
- `CUTLASS_CREATE_FMHA_BENCHMARK` — wrapper macro / 包装宏
