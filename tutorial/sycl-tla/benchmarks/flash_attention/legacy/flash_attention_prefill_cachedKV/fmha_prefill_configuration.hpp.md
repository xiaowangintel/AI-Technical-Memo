# fmha_prefill_configuration.hpp — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `benchmarks/flash_attention/legacy/flash_attention_prefill_cachedKV/fmha_prefill_configuration.hpp`
- **EN:** Legacy cached-KV prefill configuration template that selects MMA atoms and assembles the cache-aware mainloop, epilogue, and kernel types.
- **CN:** 旧版 cached-KV prefill 配置模板：选择 MMA 原子，并组装缓存感知的 mainloop、epilogue 和内核类型。

## Line-by-Line Analysis / 逐行分析

### Lines 1-37 — File prologue
```cpp
   1 | /***************************************************************************************************
   2 |  * Copyright (c) 2024 - 2025 Codeplay Software Ltd. All rights reserved.
   3 |  * Copyright (C) 2025 - 2026 Intel Corporation, All rights reserved.
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
  32 | #pragma once
  33 | #include "flash_attention_v2/kernel/legacy/xe_flash_attn_prefill_cachedKV.hpp"
  34 | #include "cutlass/gemm/dispatch_policy.hpp"
  35 | 
  36 | namespace cutlass {
  37 | namespace flash_attention{
```
**EN:** License comments, include directives, and namespace/prologue code establish the compilation context for the definitions that follow.
**CN:** 许可证注释、include 指令以及命名空间/前导代码为后续定义建立编译上下文。

### Lines 38-48 — MMAOP selector
```cpp
  38 | template<typename DispatchPolicy, typename input, typename output> struct MMAOP {
  39 |   static_assert(cutlass::detail::dependent_false<DispatchPolicy>, "Could not find a supported MMA ATOM Operation for flash attention");
  40 | };
  41 | 
  42 | template <typename DispatchPolicy> struct MMAOP <DispatchPolicy, bfloat16_t, float> {
  43 |   using Type=cute::XE_8x16x16_F32BF16BF16F32_TT;
  44 | };
  45 | 
  46 | template <typename DispatchPolicy> struct MMAOP <DispatchPolicy, half_t, float> {
  47 |   using Type = cute::XE_8x16x16_F32F16F16F32_TT;
  48 | };
```
**EN:** Maps BF16 and FP16 element types to the correct XMX MMA atom for the cached-KV kernel family.
**CN:** 把 BF16 与 FP16 元素类型映射到 cached-KV 内核家族所需的正确 XMX MMA 原子。

### Lines 50-100 — Cached-KV config type
```cpp
  50 | template<typename ElementInputType, typename ElementAccumulatorType, typename ElementOutputType,
  51 |           typename GmemTiledCopyQ, typename GmemTiledCopyK, typename GmemTiledCopyV, typename GmemTiledCopyO,
  52 |           typename TileShapeQK, typename TileShapePV, typename TileShapeOutput, typename SubgroupLayout,
  53 |           bool HasCausal, bool IsVarLen, bool IsPagedKV, int PipelineStages>
  54 | struct FMHAPrefillConfig {
  55 | 
  56 |   using ElementOutput = ElementOutputType;        // <- data type of output
  57 |   using ElementInputQ = ElementInputType;    // <- data type of elements in input matrix Q
  58 |   using ElementInputK = ElementInputType;    // <- data type of elements in input matrix K
  59 |   using ElementInputV = ElementInputType;    // <- data type of elements in input matrix V
  60 |   using ElementAccumulator = ElementAccumulatorType;   // <- data type of accumulator for mma operation
  61 |   using LayoutQ = cutlass::layout::RowMajor;
  62 |   using LayoutK = cutlass::layout::ColumnMajor;
  63 |   using LayoutV = cutlass::layout::RowMajor;
  64 |   using LayoutO = cutlass::layout::RowMajor;
  65 |   static constexpr bool Causal = HasCausal;
  66 |   static constexpr bool VarLen = IsVarLen;
  67 |   static constexpr bool PagedKV = IsPagedKV;
  68 |   using GEMMDispatchPolicy = cutlass::gemm::MainloopIntelXeXMX16<PipelineStages>;
  69 |   using EpilogueDispatchPolicy = cutlass::epilogue::IntelXeXMX16;
  70 |   using MMAOperation = typename MMAOP<GEMMDispatchPolicy, ElementInputType,ElementAccumulator>::Type;
  71 |   using CollectiveEpilogue = cutlass::flash_attention::collective::FlashPrefillCachedEpilogue<
  72 |                                   EpilogueDispatchPolicy, MMAOperation, TileShapeOutput,
  73 |                                   SubgroupLayout, ElementAccumulator, ElementOutput,
  74 |                                   cutlass::gemm::TagToStrideC_t<LayoutO>, ElementOutput,
  75 |                                   GmemTiledCopyO>;
  76 | 
  77 |   using CollectiveSoftmaxEpilogue = cutlass::flash_attention::collective::FlashPrefillSoftmaxEpilogue<Causal,
  78 |                                  EpilogueDispatchPolicy, ElementAccumulator>;
  79 | 
  80 |   using ProblemShapeRegular = cute::tuple<int, int, int, int, int, int, int, int>;
  81 |   using ProblemShapeVarlen = cute::tuple<int, int, int, fmha::collective::VariableLength,
  82 |                                          fmha::collective::VariableLength,
  83 |                                          fmha::collective::VariableLength, int, int>;
  84 |   using ProblemShapeType = std::conditional_t<VarLen, ProblemShapeVarlen, ProblemShapeRegular>;
  85 | 
  86 |   // Mainloop
  87 |   using CollectiveMainloop =
  88 |       cutlass::flash_attention::collective::FlashPrefillCachedMma<GEMMDispatchPolicy, ProblemShapeType,
  89 |                                                             ElementInputQ, cutlass::gemm::TagToStrideA_t<LayoutQ>,
  90 |                                                             ElementInputK,cutlass::gemm::TagToStrideB_t<LayoutK>,
  91 |                                                             ElementInputV, cutlass::gemm::TagToStrideB_t<LayoutV>,
  92 |                                                             MMAOperation, TileShapeQK, TileShapePV, SubgroupLayout,
  93 |                                                             GmemTiledCopyQ,
  94 |                                                             GmemTiledCopyK,
  95 |                                                             GmemTiledCopyV,
  96 |                                                             Causal, PagedKV>;
  97 | 
  98 |   using GemmKernel = cutlass::flash_attention::kernel::FMHAPrefillCached<ProblemShapeType, CollectiveMainloop,
  99 |                                                                     CollectiveSoftmaxEpilogue, CollectiveEpilogue>;
 100 | };
```
**EN:** Defines the cache-aware FMHA prefill configuration, including the paged-KV flag, problem-shape aliases, mainloop, epilogue, and final kernel type.
**CN:** 定义缓存感知的 FMHA prefill 配置，包括 paged-KV 标志、问题形状别名、mainloop、epilogue 与最终内核类型。

### Lines 101-103 — File epilogue
```cpp
 101 | 
 102 | } // namespace flash_attention
 103 | } // namespace cutlass
```
**EN:** This trailing block closes scopes or keeps small glue code that finalizes the file structure.
**CN:** 这一尾部代码块用于关闭作用域，或保留收尾用的小型胶水代码，从而完成文件结构。

## Key Concepts / 关键概念

- MMA selection / MMA 选择
- Cached-KV type assembly / cached-KV 类型组装
- Paged-KV flag threading / paged-KV 标志透传

## Dependencies / 依赖关系

- Legacy cached prefill kernels / 旧版缓存 prefill 内核
- `FlashPrefillCachedMma` and cached epilogue / cached mainloop 与 epilogue
