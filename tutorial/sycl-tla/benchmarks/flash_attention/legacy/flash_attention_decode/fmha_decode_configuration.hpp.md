# fmha_decode_configuration.hpp — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `benchmarks/flash_attention/legacy/flash_attention_decode/fmha_decode_configuration.hpp`
- **EN:** Legacy decode configuration templates that map head-size presets and data types into concrete FMHA decode kernel types.
- **CN:** 旧版 decode 配置模板：把头维预设和数据类型映射成具体的 FMHA decode 内核类型。

## Line-by-Line Analysis / 逐行分析

### Lines 1-36 — File prologue
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
  34 | namespace cutlass {
  35 | namespace flash_attention{
  36 | 
```
**EN:** License comments, include directives, and namespace/prologue code establish the compilation context for the definitions that follow.
**CN:** 许可证注释、include 指令以及命名空间/前导代码为后续定义建立编译上下文。

### Lines 37-47 — MMAOP selector
```cpp
  37 | template<typename DispatchPolicy, typename input, typename output> struct MMAOP {
  38 |     static_assert(cutlass::detail::dependent_false<DispatchPolicy>, "Could not find a supported MMA ATOM Operation for flash attention");
  39 | };
  40 | 
  41 | template <typename DispatchPolicy> struct MMAOP <DispatchPolicy, bfloat16_t, float> {
  42 |     using TYPE = cute::XE_1x16x16_F32BF16BF16F32_TT;
  43 | };
  44 | 
  45 | template <typename DispatchPolicy> struct MMAOP <DispatchPolicy, half_t, float> {
  46 |     using TYPE = cute::XE_1x16x16_F32F16F16F32_TT;
  47 | };
```
**EN:** Chooses the correct XMX MMA operation for BF16 or FP16 decode kernels and rejects unsupported combinations at compile time.
**CN:** 为 BF16 或 FP16 decode 内核选择正确的 XMX MMA 操作，并在编译期拒绝不支持的组合。

### Lines 49-107 — FMHADecodeConfig
```cpp
  49 | template <typename ElementInputType_, typename ElementAccumulatorType_, typename ElementOutputType_,
  50 |           typename GmemTiledCopyQ_, typename GmemTiledCopyK_, typename GmemTiledCopyV_, typename GmemTiledCopyO_, 
  51 |           typename TileShapeQK_, typename TileShapePV_, typename TileShapeOutput_, typename SubgroupLayout_,
  52 |           bool Causal_, bool VarLen_, bool PagedKV_>
  53 | struct FMHADecodeConfig {
  54 | 
  55 |   using ElementO = ElementOutputType_;     // <- data type of output
  56 |   using ElementInputQ = ElementInputType_;     // <- data type of elements in input matrix Q
  57 |   using ElementInputK = ElementInputType_;    // <- data type of elements in input matrix K
  58 |   using ElementInputV = ElementInputType_;    // <- data type of elements in input matrix V
  59 |   using ElementAccumulator = ElementAccumulatorType_; // <- data type of accumulator
  60 | 
  61 |   using LayoutQ = cutlass::layout::RowMajor;
  62 |   using LayoutK = cutlass::layout::ColumnMajor;
  63 |   using LayoutV = cutlass::layout::RowMajor;
  64 |   using LayoutO = cutlass::layout::RowMajor;
  65 | 
  66 |   using TileShapeQK = TileShapeQK_;
  67 |   using TileShapePV = TileShapePV_;
  68 |   using TileShapeOutput = TileShapeOutput_;
  69 |   using SubgroupLayout = SubgroupLayout_;
  70 | 
  71 |   static constexpr bool Causal = Causal_;
  72 |   static constexpr bool VarLen = VarLen_;
  73 |   static constexpr bool PagedKV = PagedKV_;
  74 |   
  75 |   static constexpr int PipelineStages = 2;
  76 |   using GEMMDispatchPolicy = cutlass::gemm::MainloopIntelXeXMX16<PipelineStages>;
  77 |   using EpilogueDispatchPolicy = cutlass::epilogue::IntelXeXMX16;
  78 | 
  79 |   using MMAOperation = typename MMAOP<GEMMDispatchPolicy, ElementInputQ, ElementAccumulator>::TYPE;
  80 | 
  81 |   using GmemTiledCopyQ = GmemTiledCopyQ_;
  82 |   using GmemTiledCopyK = GmemTiledCopyK_;
  83 |   using GmemTiledCopyV = GmemTiledCopyV_;
  84 |   using GmemTiledCopyO = GmemTiledCopyO_;
  85 |   using CollectiveEpilogue = cutlass::flash_attention::collective::FlashDecodeEpilogue<
  86 |       EpilogueDispatchPolicy, MMAOperation, TileShapeOutput, SubgroupLayout, ElementAccumulator, ElementO, cutlass::gemm::TagToStrideC_t<LayoutO>, ElementO,
  87 |       GmemTiledCopyO>;
  88 | 
  89 |   using CollectiveSoftmaxEpilogue = cutlass::flash_attention::collective::FlashDecodeSoftmaxEpilogue<Causal, EpilogueDispatchPolicy, ElementAccumulator>;
  90 | 
  91 |   using ProblemShapeRegular = cute::tuple<int, int, int, int, int, int, int, int>;
  92 |   using ProblemShapeVarlen = cute::tuple<int, int, int, fmha::collective::VariableLength, fmha::collective::VariableLength, fmha::collective::VariableLength, int, int>;
  93 |   using ProblemShapeType = std::conditional_t<VarLen, ProblemShapeVarlen, ProblemShapeRegular>;
  94 | 
  95 |   // Mainloop
  96 |   using CollectiveMainloop = cutlass::flash_attention::collective::FlashDecodeMma<
  97 |       GEMMDispatchPolicy, ProblemShapeType, ElementInputQ, cutlass::gemm::TagToStrideA_t<LayoutQ>, ElementInputK,
  98 |       cutlass::gemm::TagToStrideB_t<LayoutK>, ElementInputV, cutlass::gemm::TagToStrideB_t<LayoutV>, MMAOperation,
  99 |       TileShapeQK, TileShapePV, SubgroupLayout,
 100 |       GmemTiledCopyQ, // Q
 101 |       GmemTiledCopyK, // K
 102 |       GmemTiledCopyV, // V,
 103 |       Causal, PagedKV>;
 104 | 
 105 |   using FMHADecodeKernel = cutlass::flash_attention::kernel::FMHADecode<ProblemShapeType, CollectiveMainloop,
 106 |                                                                     CollectiveSoftmaxEpilogue, CollectiveEpilogue>;
 107 | };
```
**EN:** Bundles element types, layouts, tile shapes, execution flags, collective types, and the final kernel alias into one decode configuration type.
**CN:** 把元素类型、布局、tile 形状、执行标志、collective 类型以及最终内核别名打包成一个 decode 配置类型。

### Lines 110-140 — Shape helpers
```cpp
 110 | template <int KVTile, int NumSGs>
 111 | struct Shape_h64 {
 112 |   using ShapeQK = Shape<_1, Int<KVTile>, _64>;
 113 |   using ShapePV = Shape<_1, _32, Int<KVTile>>;
 114 |   using ShapeOutput = Shape<_1, _64, Int<KVTile>>;
 115 |   using SubgroupLayout = Layout<Shape<Int<NumSGs>, _1, _1>>;
 116 | };
 117 | 
 118 | template <int KVTile, int NumSGs>
 119 | struct Shape_h96 {
 120 |   using ShapeQK = Shape<_1, Int<KVTile>, _64>;
 121 |   using ShapePV = Shape<_1, _32, Int<KVTile>>;
 122 |   using ShapeOutput = Shape<_1, _96, Int<KVTile>>;
 123 |   using SubgroupLayout = Layout<Shape<Int<NumSGs>, _1, _1>>;
 124 | };
 125 | 
 126 | template <int KVTile, int NumSGs>
 127 | struct Shape_h128 {
 128 |   using ShapeQK = Shape<_1, Int<KVTile>, _64>;
 129 |   using ShapePV = Shape<_1, _32, Int<KVTile>>;
 130 |   using ShapeOutput = Shape<_1, _128, Int<KVTile>>;
 131 |   using SubgroupLayout = Layout<Shape<Int<NumSGs>, _1, _1>>;
 132 | };
 133 | 
 134 | template <int KVTile, int NumSGs>
 135 | struct Shape_h192 {
 136 |   using ShapeQK = Shape<_1, Int<KVTile>, _64>;
 137 |   using ShapePV = Shape<_1, _32, Int<KVTile>>;
 138 |   using ShapeOutput = Shape<_1, _192, Int<KVTile>>;
 139 |   using SubgroupLayout = Layout<Shape<Int<NumSGs>, _1, _1>>;
 140 | };
```
**EN:** Defines the built-in tile-shape families used by the h64/h96/h128/h192 decode benchmarks.
**CN:** 定义 h64/h96/h128/h192 decode 基准所使用的内建 tile 形状家族。

### Lines 142-188 — Configuration generators
```cpp
 142 | template<class QKVType, class AccumulatorType, class OutputType, bool Causal, bool VarLen, class TileShapeConfig, bool PagedKV>
 143 | struct FMHADecodeConfigGen;
 144 | 
 145 | template<class QKVType, bool Causal, bool VarLen, class TileShapeConfig, bool PagedKV>
 146 | struct FMHADecodeConfigGen<QKVType, float, float, Causal, VarLen, TileShapeConfig, PagedKV> {
 147 | 
 148 | using GmemTiledCopyQ = cute::XE_2D_U16x1x16_LD_N;
 149 | using GmemTiledCopyK = cute::XE_2D_U16x16x16_LD_T;
 150 | using GmemTiledCopyV = cute::XE_2D_U16x32x32_LD_V;
 151 | using GmemTiledCopyO = cute::XE_2D_U32x1x16_ST_N;
 152 | 
 153 | using type = cutlass::flash_attention::FMHADecodeConfig<
 154 |       QKVType, float, float, GmemTiledCopyQ, GmemTiledCopyK, GmemTiledCopyV,
 155 |       GmemTiledCopyO, typename TileShapeConfig::ShapeQK, typename TileShapeConfig::ShapePV,
 156 |       typename TileShapeConfig::ShapeOutput, typename TileShapeConfig::SubgroupLayout,
 157 |       Causal, VarLen, PagedKV>;
 158 | };
 159 | 
 160 | template<class QKVType, bool Causal, bool VarLen, class TileShapeConfig, bool PagedKV>
 161 | struct FMHADecodeConfigGen<QKVType, float, cutlass::bfloat16_t, Causal, VarLen, TileShapeConfig, PagedKV> {
 162 | 
 163 | using GmemTiledCopyQ = cute::XE_2D_U16x1x16_LD_N;
 164 | using GmemTiledCopyK = cute::XE_2D_U16x16x16_LD_T;
 165 | using GmemTiledCopyV = cute::XE_2D_U16x32x32_LD_V;
 166 | using GmemTiledCopyO = cute::XE_2D_U16x1x16_ST_N;
 167 | 
 168 | using type = cutlass::flash_attention::FMHADecodeConfig<
 169 |       QKVType, float, cutlass::bfloat16_t, GmemTiledCopyQ, GmemTiledCopyK, GmemTiledCopyV,
 170 |       GmemTiledCopyO, typename TileShapeConfig::ShapeQK, typename TileShapeConfig::ShapePV,
 171 |       typename TileShapeConfig::ShapeOutput, typename TileShapeConfig::SubgroupLayout,
 172 |       Causal, VarLen, PagedKV>;
 173 | };
 174 | 
 175 | template<class QKVType, bool Causal, bool VarLen, class TileShapeConfig, bool PagedKV>
 176 | struct FMHADecodeConfigGen<QKVType, float, cutlass::half_t, Causal, VarLen, TileShapeConfig, PagedKV> {
 177 | 
 178 | using GmemTiledCopyQ = cute::XE_2D_U16x1x16_LD_N;
 179 | using GmemTiledCopyK = cute::XE_2D_U16x16x16_LD_T;
 180 | using GmemTiledCopyV = cute::XE_2D_U16x32x32_LD_V;
 181 | using GmemTiledCopyO = cute::XE_2D_U16x1x16_ST_N;
 182 | 
 183 | using type = cutlass::flash_attention::FMHADecodeConfig<
 184 |       QKVType, float, cutlass::half_t, GmemTiledCopyQ, GmemTiledCopyK, GmemTiledCopyV,
 185 |       GmemTiledCopyO, typename TileShapeConfig::ShapeQK, typename TileShapeConfig::ShapePV,
 186 |       typename TileShapeConfig::ShapeOutput, typename TileShapeConfig::SubgroupLayout,
 187 |       Causal, VarLen, PagedKV>;
 188 | };
```
**EN:** Combines output type specializations with the shape helpers to produce ready-to-use decode configuration aliases.
**CN:** 把输出类型特化与形状辅助类型结合，生成可直接使用的 decode 配置别名。

### Lines 189-191 — File epilogue
```cpp
 189 | 
 190 | } // namespace flash_attention
 191 | } // namespace cutlass
```
**EN:** This trailing block closes scopes or keeps small glue code that finalizes the file structure.
**CN:** 这一尾部代码块用于关闭作用域，或保留收尾用的小型胶水代码，从而完成文件结构。

## Key Concepts / 关键概念

- MMA atom selection / MMA 原子选择
- Head-size shape presets / 头维形状预设
- Type-level kernel assembly / 类型级内核组装

## Dependencies / 依赖关系

- `cutlass/gemm/dispatch_policy.hpp` — dispatch policy / 分发策略
- `cute::Shape` and `cute::Layout` — tile descriptions / tile 描述
- Legacy decode collectives and kernels / 旧版 decode collective 与内核
