# fmha_prefill_configuration.hpp — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `benchmarks/flash_attention/legacy/flash_attention_prefill/fmha_prefill_configuration.hpp`
- **EN:** Legacy prefill configuration template that maps element types and execution flags to concrete prefill kernels.
- **CN:** 旧版 prefill 配置模板：把元素类型和执行标志映射为具体的 prefill 内核。

## Line-by-Line Analysis / 逐行分析

### Lines 1-33 — File prologue
```cpp
   1 | /***************************************************************************************************
   2 |  * Copyright (c) 2024 - 2025 Codeplay Software Ltd. All rights reserved.
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
  31 | #pragma once
  32 | #include "cutlass/gemm/dispatch_policy.hpp"
  33 | 
```
**EN:** License comments, include directives, and namespace/prologue code establish the compilation context for the definitions that follow.
**CN:** 许可证注释、include 指令以及命名空间/前导代码为后续定义建立编译上下文。

### Lines 34-46 — MMAOP selector
```cpp
  34 | namespace cutlass {
  35 | namespace flash_attention{
  36 |   template<typename DispatchPolicy, typename input, typename output> struct MMAOP {
  37 |     static_assert(cutlass::detail::dependent_false<DispatchPolicy>, "Could not find a supported MMA ATOM Operation for flash attention");
  38 |   };
  39 |   
  40 |   template <typename DispatchPolicy> struct MMAOP <DispatchPolicy, bfloat16_t, float> {
  41 |     using Type=cute::XE_8x16x16_F32BF16BF16F32_TT;
  42 |   };
  43 |   
  44 |   template <typename DispatchPolicy> struct MMAOP <DispatchPolicy, half_t, float> {
  45 |     using Type = cute::XE_8x16x16_F32F16F16F32_TT;
  46 |   };
```
**EN:** Selects the BF16 or FP16 XMX MMA atom used by the legacy prefill kernel family.
**CN:** 选择旧版 prefill 内核家族使用的 BF16 或 FP16 XMX MMA 原子。

### Lines 48-79 — FMHAPrefillConfig core type
```cpp
  48 | template<typename ElementInputType, typename ElementAccumulatorType, typename ElementOutputType,  
  49 |           typename GmemTiledCopyQ, typename GmemTiledCopyK, typename GmemTiledCopyV, typename GmemTiledCopyO,
  50 |           typename TileShapeQK, typename TileShapePV, typename TileShapeOutput, typename SubgroupLayout, 
  51 |           bool HasCausal, bool IsVarLen, int PipelineStages>
  52 | struct FMHAPrefillConfig {
  53 | 
  54 |   using ElementOutput = ElementOutputType;        // <- data type of output
  55 |   using ElementInputQ = ElementInputType;    // <- data type of elements in input matrix Q
  56 |   using ElementInputK = ElementInputType;    // <- data type of elements in input matrix K
  57 |   using ElementInputV = ElementInputType;    // <- data type of elements in input matrix V
  58 |   using ElementAccumulator = ElementAccumulatorType;   // <- data type of accumulator for mma operation
  59 |   using LayoutQ = cutlass::layout::RowMajor;
  60 |   using LayoutK = cutlass::layout::ColumnMajor;
  61 |   using LayoutV = cutlass::layout::RowMajor;
  62 |   using LayoutO = cutlass::layout::RowMajor;
  63 |   static constexpr bool Causal = HasCausal;
  64 |   static constexpr bool VarLen = IsVarLen;
  65 |   using GEMMDispatchPolicy = cutlass::gemm::MainloopIntelXeXMX16<PipelineStages>;
  66 |   using EpilogueDispatchPolicy = cutlass::epilogue::IntelXeXMX16;
  67 |   using MMAOperation = typename MMAOP<GEMMDispatchPolicy, ElementInputType,ElementAccumulator>::Type;
  68 |   using CollectiveEpilogue = cutlass::flash_attention::collective::FlashPrefillEpilogue<
  69 |                                     EpilogueDispatchPolicy, MMAOperation, TileShapeOutput, 
  70 |                                     SubgroupLayout, ElementAccumulator, ElementOutputType,
  71 |                                     cutlass::gemm::TagToStrideC_t<LayoutO>, ElementOutput,
  72 |                                     GmemTiledCopyO>;
  73 | 
  74 |  using CollectiveSoftmaxEpilogue = cutlass::flash_attention::collective::FlashPrefillSoftmaxEpilogue<Causal, 
  75 |                                 EpilogueDispatchPolicy, ElementAccumulator>;
  76 | 
  77 |   using ProblemShapeRegular = cute::tuple<int, int, int, int, int, int, int>;
  78 |   using ProblemShapeVarlen = cute::tuple<int, int, int, fmha::collective::VariableLength, fmha::collective::VariableLength, int, int>;
  79 |   using ProblemShapeType = std::conditional_t<VarLen, ProblemShapeVarlen, ProblemShapeRegular>;
```
**EN:** Defines layouts, causal and variable-length flags, dispatch-policy aliases, and the problem-shape type used by the kernel.
**CN:** 定义布局、因果与变长标志、dispatch 策略别名，以及内核使用的问题形状类型。

### Lines 81-95 — Kernel assembly aliases
```cpp
  81 |   // Mainloop
  82 |   using CollectiveMainloop = 
  83 |         cutlass::flash_attention::collective::FlashPrefillMma<GEMMDispatchPolicy, ProblemShapeType, 
  84 |                                                               ElementInputQ, cutlass::gemm::TagToStrideA_t<LayoutQ>, 
  85 |                                                               ElementInputK,cutlass::gemm::TagToStrideB_t<LayoutK>, 
  86 |                                                               ElementInputV, cutlass::gemm::TagToStrideB_t<LayoutV>, 
  87 |                                                               MMAOperation, TileShapeQK, TileShapePV, SubgroupLayout,
  88 |                                                               GmemTiledCopyQ, 
  89 |                                                               GmemTiledCopyK, 
  90 |                                                               GmemTiledCopyV, 
  91 |                                                               Causal>;
  92 | 
  93 |   using GemmKernel = cutlass::flash_attention::kernel::FMHAPrefill<ProblemShapeType, CollectiveMainloop,
  94 |                                                                     CollectiveSoftmaxEpilogue, CollectiveEpilogue>;
  95 | };
```
**EN:** Composes the mainloop, softmax epilogue, output epilogue, and final kernel alias for the legacy prefill path.
**CN:** 为旧版 prefill 路径组合 mainloop、softmax epilogue、输出 epilogue 和最终内核别名。

### Lines 96-98 — File epilogue
```cpp
  96 | 
  97 | } // namespace flash_attention
  98 | } // namespace cutlass
```
**EN:** This trailing block closes scopes or keeps small glue code that finalizes the file structure.
**CN:** 这一尾部代码块用于关闭作用域，或保留收尾用的小型胶水代码，从而完成文件结构。

## Key Concepts / 关键概念

- MMA atom selection / MMA 原子选择
- Prefill config type assembly / prefill 配置类型组装
- Mainloop plus epilogue composition / mainloop 与 epilogue 组合

## Dependencies / 依赖关系

- Legacy prefill kernels and epilogues / 旧版 prefill 内核与 epilogue
- `cutlass/gemm/dispatch_policy.hpp` — dispatch selection / dispatch 选择
