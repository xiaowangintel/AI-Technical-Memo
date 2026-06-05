# benchmarks_prefill_bf16.cpp — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `benchmarks/flash_attention/benchmarks_prefill_bf16.cpp`
- **EN:** Registers the current BF16 prefill benchmark configurations for Flash Attention, including cached/paged-KV and explicit-tile variants.
- **CN:** 注册当前版本 Flash Attention 的 BF16 prefill 基准配置，包括 cached/paged-KV 和显式 tile 变体。

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

### Lines 38-45 — Default cached/paged prefill config
```cpp
  38 | /* ---------------------------------------- HeadDim = 64 ------------------------------------------ */
  39 | using BmgFMHAPrefill_BF16_BF16_BF16_FP32_RCR_h64_NonCausal_FixedLen_CachedKV_PagedKV = FMHAConfigGen</*Mode*/FMHAMode::Prefill,
  40 |   /*ElementQ*/ cutlass::bfloat16_t, /*ElementK*/ cutlass::bfloat16_t, /*ElementV*/ cutlass::bfloat16_t, /*ElementO*/ float,
  41 |   /*LayoutQ*/ cutlass::layout::RowMajor, /*LayoutK*/ cutlass::layout::ColumnMajor, /*LayoutV*/ cutlass::layout::RowMajor, /*LayoutO*/ cutlass::layout::RowMajor,
  42 |   /*Causal*/ false, /*VarLen*/ false, /*CachedKV*/ true, /*PagedKV*/ true, /*Persistent*/ false, /*HeadDim*/ 64
  43 | >::type;
  44 | 
  45 | CUTLASS_CREATE_FMHA_BENCHMARK(BmgFMHAPrefill_BF16_BF16_BF16_FP32_RCR_h64_NonCausal_FixedLen_CachedKV_PagedKV);
```
**EN:** Defines a prefill benchmark that enables cached KV and paged KV on top of the default head-dimension-64 configuration.
**CN:** 定义一个基于默认 head-dim=64 配置的 prefill 基准，并启用 cached KV 与 paged KV。

### Lines 47-54 — Explicit-tile fixed-length config
```cpp
  47 | /* ---------------------------------------- Custom Tiles ------------------------------------------ */
  48 | 
  49 | using BmgFMHAPrefill_BF16_BF16_BF16_FP32_RCR_WgQ128K64V32_SgQ8K64_HDimQK32V64_NonCausal_FixedLen = FMHAConfigGenWithTileShape</*Mode*/FMHAMode::Prefill,
  50 |   /*ElementQ*/ cutlass::bfloat16_t, /*ElementK*/ cutlass::bfloat16_t, /*ElementV*/ cutlass::bfloat16_t, /*ElementO*/ float,
  51 |   /*LayoutQ*/ cutlass::layout::RowMajor, /*LayoutK*/ cutlass::layout::ColumnMajor, /*LayoutV*/ cutlass::layout::RowMajor, /*LayoutO*/ cutlass::layout::RowMajor,
  52 |   /*Causal*/ false, /*VarLen*/ false, /*CachedKV*/ false, /*PagedKV*/ false, /*Persistent*/ false, /*WgTileQ*/ 128, /*WgTileK*/ 64, /*WgTileV*/ 32,
  53 |   /*SgTileQ*/ 8, /*SgTileK*/ 64, /*HeadDimQK*/ 32, /*HeadDimV*/ 64
  54 | >::type;
```
**EN:** Creates a custom-tiled prefill benchmark for fixed-length, non-causal execution without cache features.
**CN:** 创建一个自定义 tile 的 prefill 基准，用于定长、非因果、无缓存特性的执行路径。

### Lines 56-64 — Explicit-tile causal varlen config
```cpp
  56 | using BmgFMHAPrefill_BF16_BF16_BF16_FP32_RCR_WgQ128K64V32_SgQ8K64_HDimQK32V64_Causal_VarLen = FMHAConfigGenWithTileShape</*Mode*/FMHAMode::Prefill,
  57 |   /*ElementQ*/ cutlass::bfloat16_t, /*ElementK*/ cutlass::bfloat16_t, /*ElementV*/ cutlass::bfloat16_t, /*ElementO*/ float,
  58 |   /*LayoutQ*/ cutlass::layout::RowMajor, /*LayoutK*/ cutlass::layout::ColumnMajor, /*LayoutV*/ cutlass::layout::RowMajor, /*LayoutO*/ cutlass::layout::RowMajor,
  59 |   /*Causal*/ true, /*VarLen*/ true, /*CachedKV*/ false, /*PagedKV*/ false, /*Persistent*/ false, /*WgTileQ*/ 128, /*WgTileK*/ 64, /*WgTileV*/ 32,
  60 |   /*SgTileQ*/ 8, /*SgTileK*/ 64, /*HeadDimQK*/ 32, /*HeadDimV*/ 64
  61 | >::type;
  62 | 
  63 | CUTLASS_CREATE_FMHA_BENCHMARK(BmgFMHAPrefill_BF16_BF16_BF16_FP32_RCR_WgQ128K64V32_SgQ8K64_HDimQK32V64_NonCausal_FixedLen);
  64 | CUTLASS_CREATE_FMHA_BENCHMARK(BmgFMHAPrefill_BF16_BF16_BF16_FP32_RCR_WgQ128K64V32_SgQ8K64_HDimQK32V64_Causal_VarLen);
```
**EN:** Adds a second custom-tiled prefill variant that turns on causal masking and variable-length sequence support.
**CN:** 添加第二个自定义 tile 的 prefill 变体，并开启因果 mask 与变长序列支持。

### Lines 67-72 — Suite registration
```cpp
  67 | static void register_flash_attention_prefill_benchmarks_bf16() {
  68 |   CUTLASS_FMHA_BENCHMARK(BmgFMHAPrefill_BF16_BF16_BF16_FP32_RCR_h64_NonCausal_FixedLen_CachedKV_PagedKV);
  69 | 
  70 |   CUTLASS_FMHA_BENCHMARK(BmgFMHAPrefill_BF16_BF16_BF16_FP32_RCR_WgQ128K64V32_SgQ8K64_HDimQK32V64_NonCausal_FixedLen);
  71 |   CUTLASS_FMHA_BENCHMARK(BmgFMHAPrefill_BF16_BF16_BF16_FP32_RCR_WgQ128K64V32_SgQ8K64_HDimQK32V64_Causal_VarLen);
  72 | }
```
**EN:** Registers all three prefill benchmarks into the current suite.
**CN:** 把这三个 prefill 基准全部注册到当前套件中。

## Key Concepts / 关键概念

- Prefill-mode FMHA aliases / prefill 模式 FMHA 类型别名
- Cached-KV and paged-KV support / cached-KV 与 paged-KV 支持
- Custom tile-shape experiments / 自定义 tile 形状实验

## Dependencies / 依赖关系

- `benchmark_runner.hpp` — FMHA benchmark harness / FMHA 基准运行器
- `fmha_configuration.hpp` — config generators / 配置生成器
- `CUTLASS_CREATE_FMHA_BENCHMARK` — wrapper macro / 包装宏
