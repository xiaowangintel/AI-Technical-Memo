# fmha_configuration.hpp — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `benchmarks/flash_attention/fmha_configuration.hpp`
- **EN:** Current FMHA configuration layer that maps mode, head dimension, tile shape, and scheduler policy into a concrete Flash Attention kernel type.
- **CN:** 当前的 FMHA 配置层：把模式、头维、tile 形状和调度策略映射为具体的 Flash Attention 内核类型。

## Line-by-Line Analysis / 逐行分析

### Lines 1-42 — File prologue
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
  31 | #pragma once
  32 | 
  33 | #include <type_traits>
  34 | 
  35 | #include "cutlass/gemm/dispatch_policy.hpp"
  36 | #include "flash_attention_v2/collective/fmha_fusion.hpp"
  37 | #include "flash_attention_v2/kernel/xe_fmha_fwd_kernel.hpp"
  38 | #include "flash_attention_v2/kernel/xe_tile_scheduler.hpp"
  39 | 
  40 | namespace cutlass {
  41 | namespace flash_attention {
  42 | 
```
**EN:** License comments, include directives, and namespace/prologue code establish the compilation context for the definitions that follow.
**CN:** 许可证注释、include 指令以及命名空间/前导代码为后续定义建立编译上下文。

### Lines 43-136 — FMHAConfig core template
```cpp
  43 | template <typename ElementQ, typename ElementK, typename ElementV, typename ElementO,
  44 |           typename LayoutQ_, typename LayoutK_, typename LayoutV_, typename LayoutO_,
  45 |           typename TileShapeQK, typename TileShapePV, typename TileShapeOutput, 
  46 |           typename SubgroupLayoutQK, typename SubgroupLayoutPV_,
  47 |           bool Causal_, bool VarLen_, bool CachedKV_, bool PagedKV_, bool Persistent_,
  48 |           int PipelineStages,
  49 |           typename GmemTiledCopyQ = void, 
  50 |           typename GmemTiledCopyK = void, 
  51 |           typename GmemTiledCopyV = void, 
  52 |           typename GmemTiledCopyO = void,
  53 |           typename MMAOperation_ = void,
  54 |           typename StrideQ = Stride<int, _1, int, int>, 
  55 |           typename StrideK = Stride<int, _1, int, int>,
  56 |           typename StrideV = Stride<_1, int, int, int>, 
  57 |           typename StrideO = Stride<int, _1, int, int>>          
  58 | struct FMHAConfig {
  59 |   using LayoutQ = LayoutQ_;
  60 |   using LayoutK = LayoutK_;
  61 |   using LayoutV = LayoutV_;
  62 |   using LayoutO = LayoutO_;
  63 |   
  64 |   static constexpr bool Causal = Causal_;
  65 |   static constexpr bool VarLen = VarLen_;
  66 |   static constexpr bool CachedKV = CachedKV_;
  67 |   static constexpr bool PagedKV = PagedKV_;
  68 |   static constexpr bool Persistent = Persistent_;
  69 |   static_assert(!(Persistent & Causal), "persistent SDPA kernel not support Causal yet");
  70 |   
  71 |   static constexpr int SGTileQ = get<0>(shape_div(TileShapeQK{}, shape(SubgroupLayoutQK{})))();
  72 | 
  73 |   using DefaultMMA = XE_DPAS_TT<cute::gcd(SGTileQ, 8), float, ElementQ>;
  74 |   using MMAOperationPV = XE_DPAS_TT<cute::gcd(SGTileQ, 8), float, ElementV>;
  75 | 
  76 |   using MMAOperation = cute::conditional_t<is_void_v<MMAOperation_>,
  77 |                                            DefaultMMA,
  78 |                                            MMAOperation_>;
  79 | 
  80 |   using SubgroupLayoutPV = cute::conditional_t<is_void_v<SubgroupLayoutPV_>,
  81 |                                                decltype(cutlass::fmha::collective::get_sg_layout_pv(SubgroupLayoutQK{})),
  82 |                                                SubgroupLayoutPV_>;
  83 | 
  84 |   using TiledMMAQK = typename TiledMMAHelper<MMA_Atom<MMAOperation>, Layout<TileShapeQK>, SubgroupLayoutQK>::TiledMMA;
  85 |   using TiledMMAPV = typename TiledMMAHelper<MMA_Atom<MMAOperationPV>, Layout<TileShapePV>, SubgroupLayoutPV>::TiledMMA;
  86 |   static_assert(get<0>(TileShapeOutput{}) == get<0>(TileShapePV{}),
  87 |       "Output tile and P*V tile have different sizes in Q dimension");    
  88 |   static constexpr int VTiles = get<1>(TileShapeOutput{}) / get<1>(TileShapePV{});
  89 | 
  90 |   template <typename ElementType, typename Stride>
  91 |   static constexpr auto make_dummy_tensor(ElementType val, Stride stride) {
  92 |     return make_tensor(make_gmem_ptr(&val),
  93 |                         make_layout(repeat<rank_v<decltype(stride)>>(1), stride));
  94 |   };
  95 | 
  96 |   using TensorQ = decltype(make_dummy_tensor(ElementQ{}, StrideQ{}));
  97 |   using TensorK = decltype(make_dummy_tensor(ElementK{}, StrideK{}));
  98 |   using TensorV = decltype(make_dummy_tensor(ElementV{}, StrideV{}));
  99 |   using TensorO = decltype(make_dummy_tensor(ElementO{}, StrideO{}));
 100 |   using TensorK_cache = TensorK;
 101 |   using TensorV_cache = TensorV;
 102 |   using GmemTiledCopyK_cache = GmemTiledCopyK;
 103 |   using GmemTiledCopyV_cache = GmemTiledCopyV;
 104 | 
 105 |   using ProblemShapeType = cutlass::fmha::kernel::FMHAProblemShape<VarLen>;
 106 | 
 107 | 
 108 |   // Mainloop
 109 |   using MainloopDispatchPolicy = cutlass::fmha::XeDefault<PipelineStages>;
 110 |   using CollectiveMainloop = cutlass::fmha::collective::FMHAFwdMainloop<
 111 |       MainloopDispatchPolicy, Causal, CachedKV, PagedKV,
 112 |       TiledMMAQK, TiledMMAPV, VTiles,
 113 |       TensorQ, TensorK, TensorV,
 114 |       TensorK_cache, TensorV_cache,
 115 |       GmemTiledCopyQ, GmemTiledCopyK, GmemTiledCopyV,
 116 |       GmemTiledCopyK_cache, GmemTiledCopyV_cache
 117 |   >;
 118 | 
 119 |   using CollectiveEpilogue = cutlass::fmha::collective::FMHAFwdEpilogue<
 120 |     CollectiveMainloop,
 121 |     TileShapeOutput,
 122 |     TensorO,
 123 |     GmemTiledCopyO
 124 |   >;
 125 | 
 126 |   using Scheduler = cute::conditional_t<Persistent,
 127 |       cutlass::fmha::kernel::XeFHMAIndividualPersistentTileScheduler,
 128 |       cutlass::fmha::kernel::XeFHMAIndividualTileScheduler
 129 |   >;
 130 |   using FMHAKernel = cute::conditional_t<Persistent,
 131 |       cutlass::fmha::kernel::XeFMHAFwdDynamicSplitKernel<
 132 |         ProblemShapeType, CollectiveMainloop, CollectiveEpilogue, Scheduler>,
 133 |       cutlass::fmha::kernel::XeFMHAFwdKernel<
 134 |         ProblemShapeType, CollectiveMainloop, CollectiveEpilogue, Scheduler>
 135 |   >;
 136 | };
```
**EN:** Bundles layouts, mode flags, tensor views, tiled MMA objects, mainloop, epilogue, scheduler selection, and the final kernel alias into one reusable configuration type.
**CN:** 把布局、模式标志、张量视图、tiled MMA、mainloop、epilogue、调度器选择以及最终内核别名打包成一个可复用配置类型。

### Lines 140-184 — Prefill shape presets
```cpp
 140 | enum class FMHAMode { Decode, Prefill };
 141 | 
 142 | template <FMHAMode Mode, int HeadDim, bool Persistent>
 143 | struct ShapeConfig;
 144 | 
 145 | // Prefill configs
 146 | template <bool Persistent>
 147 | struct ShapeConfig<FMHAMode::Prefill, 16, Persistent> {
 148 |   using ShapeQK = Shape<_16, _16, _32>;
 149 |   using ShapePV = Shape<_16, _32, _16>;
 150 |   using ShapeOutput = Shape<_16, _16>;
 151 |   using SubgroupLayout = Layout<Shape<_1, _1, _1>>;
 152 | };
 153 | 
 154 | template <bool Persistent>
 155 | struct ShapeConfig<FMHAMode::Prefill, 64, Persistent> {
 156 |   using ShapeQK = Shape<_128, _64, _32>;
 157 |   using ShapePV = Shape<_128, _32, _64>;
 158 |   using ShapeOutput = Shape<_128, _64>;
 159 |   using SubgroupLayout = Layout<Shape<_8, _1, _1>>;
 160 | };
 161 | 
 162 | template <bool Persistent>
 163 | struct ShapeConfig<FMHAMode::Prefill, 96, Persistent> {
 164 |   using ShapeQK = Shape<_128, _64, _32>;
 165 |   using ShapePV = Shape<_128, _32, _64>;
 166 |   using ShapeOutput = Shape<_128, _96>;
 167 |   using SubgroupLayout = Layout<Shape<_8, _1, _1>>;
 168 | };
 169 | 
 170 | template <bool Persistent>
 171 | struct ShapeConfig<FMHAMode::Prefill, 128, Persistent> {
 172 |   using ShapeQK = Shape<_128, _64, _32>;
 173 |   using ShapePV = Shape<_128, _32, _64>;
 174 |   using ShapeOutput = Shape<_128, _128>;
 175 |   using SubgroupLayout = Layout<Shape<_16, _1, _1>>;
 176 | };
 177 | 
 178 | template <bool Persistent>
 179 | struct ShapeConfig<FMHAMode::Prefill, 192, Persistent> {
 180 |   using ShapeQK = Shape<_256, _64, _32>;
 181 |   using ShapePV = Shape<_256, _32, _64>;
 182 |   using ShapeOutput = Shape<_256, _192>;
 183 |   using SubgroupLayout = Layout<Shape<_16, _1, _1>>;
 184 | };
```
**EN:** Defines built-in tile shapes and subgroup layouts for prefill kernels across the supported head dimensions.
**CN:** 为受支持的各个头维定义 prefill 内核的内建 tile 形状与 subgroup 布局。

### Lines 186-233 — Decode shape presets
```cpp
 186 | // Decode configs
 187 | template <bool Persistent>
 188 | struct ShapeConfig<FMHAMode::Decode, 16, Persistent> {
 189 |   using ShapeQK = Shape<_1, _16, _16>;
 190 |   using ShapePV = Shape<_1, _16, _16>;
 191 |   using ShapeOutput = Shape<_1, _16>;
 192 |   using SubgroupLayout = Layout<Shape<_1, _2, _1>>;
 193 | };
 194 | 
 195 | template <bool Persistent>
 196 | struct ShapeConfig<FMHAMode::Decode, 64, Persistent> {
 197 |   using num_sg = cute::conditional_t<Persistent, _16, _8>;
 198 |   using kv_tile_size = cute::conditional_t<Persistent, _256, _512>;
 199 |   using ShapeQK = Shape<_1, kv_tile_size, _64>;
 200 |   using ShapePV = Shape<_1, _32, kv_tile_size>;
 201 |   using ShapeOutput = Shape<_1, _64>;
 202 |   using SubgroupLayout = Layout<Shape<_1, num_sg, _1>>;
 203 | };
 204 | 
 205 | template <bool Persistent>
 206 | struct ShapeConfig<FMHAMode::Decode, 96, Persistent> {
 207 |   using num_sg = cute::conditional_t<Persistent, _16, _8>;
 208 |   using kv_tile_size = cute::conditional_t<Persistent, _256, _512>;
 209 |   using ShapeQK = Shape<_1, kv_tile_size, _64>;
 210 |   using ShapePV = Shape<_1, _32, kv_tile_size>;
 211 |   using ShapeOutput = Shape<_1, _96>;
 212 |   using SubgroupLayout = Layout<Shape<_1, num_sg, _1>>;
 213 | };
 214 | 
 215 | template <bool Persistent>
 216 | struct ShapeConfig<FMHAMode::Decode, 128, Persistent> {
 217 |   using num_sg = cute::conditional_t<Persistent, _16, _8>;
 218 |   using kv_tile_size = cute::conditional_t<Persistent, _256, _512>;
 219 |   using ShapeQK = Shape<_1, kv_tile_size, _64>;
 220 |   using ShapePV = Shape<_1, _32, kv_tile_size>;
 221 |   using ShapeOutput = Shape<_1, _128>;
 222 |   using SubgroupLayout = Layout<Shape<_1, num_sg, _1>>;
 223 | };
 224 | 
 225 | template <bool Persistent>
 226 | struct ShapeConfig<FMHAMode::Decode, 192, Persistent> {
 227 |   using num_sg = cute::conditional_t<Persistent, _16, _8>;
 228 |   using kv_tile_size = cute::conditional_t<Persistent, _256, _512>;
 229 |   using ShapeQK = Shape<_1, kv_tile_size, _64>;
 230 |   using ShapePV = Shape<_1, _32, kv_tile_size>;
 231 |   using ShapeOutput = Shape<_1, _192>;
 232 |   using SubgroupLayout = Layout<Shape<_1, num_sg, _1>>;
 233 | };
```
**EN:** Defines decode-specific tile and subgroup presets, including the persistent-kernel-dependent subgroup and KV tile choices for larger head dimensions.
**CN:** 定义 decode 专用的 tile 与 subgroup 预设；对于较大的头维，还根据是否为持久化内核选择 subgroup 数量和 KV tile 大小。

### Lines 235-255 — Pipeline and lookup generator
```cpp
 235 | // Mode-dependent pipeline stages
 236 | template <FMHAMode Mode>
 237 | struct PipelineStagesConfig;
 238 | template <>
 239 | struct PipelineStagesConfig<FMHAMode::Decode> { static constexpr int value = 1; };
 240 | template <>
 241 | struct PipelineStagesConfig<FMHAMode::Prefill> { static constexpr int value = 2; };
 242 | 
 243 | // FMHAConfigGen using ShapeConfig lookup table
 244 | template<FMHAMode Mode,
 245 |          class ElementQ, class ElementK, class ElementV, class ElementO,
 246 |          class LayoutQ, class LayoutK, class LayoutV, class LayoutO,
 247 |          bool Causal, bool VarLen, bool CachedKV, bool PagedKV, bool Persistent, int HeadDim>
 248 | struct FMHAConfigGen{
 249 |   using TileShapeConfig = ShapeConfig<Mode, HeadDim, Persistent>;
 250 |   using type = cutlass::flash_attention::FMHAConfig<
 251 |     ElementQ, ElementK, ElementV, ElementO, LayoutQ, LayoutK, LayoutV, LayoutO,
 252 |     typename TileShapeConfig::ShapeQK, typename TileShapeConfig::ShapePV, typename TileShapeConfig::ShapeOutput,
 253 |     typename TileShapeConfig::SubgroupLayout, void,
 254 |     Causal, VarLen, CachedKV, PagedKV, Persistent, PipelineStagesConfig<Mode>::value>;
 255 | };
```
**EN:** Chooses pipeline stage counts per mode and turns a mode/head-dimension pair into a concrete FMHAConfig through the shape lookup table.
**CN:** 按模式选择 pipeline stage 数量，并通过形状查表把 mode/head-dimension 组合转换为具体的 FMHAConfig。

### Lines 257-295 — Explicit tile generator
```cpp
 257 | // FMHAConfigGen with explicit tile and subgroup specification
 258 | template<FMHAMode Mode,
 259 |          class ElementQ, class ElementK, class ElementV, class ElementO,
 260 |          class LayoutQ, class LayoutK, class LayoutV, class LayoutO,
 261 |          bool Causal, bool VarLen, bool CachedKV, bool PagedKV, bool Persistent,
 262 |          int WgTileQ, int WgTileK, int WgTileV,
 263 |          int SgTileQ, int SgTileK,
 264 |          int HeadDimQK, int HeadDimV>
 265 | struct FMHAConfigGenWithTileShape{
 266 |   using ShapeQK = Shape<Int<WgTileQ>, Int<WgTileK>, Int<HeadDimQK>>;
 267 |   using ShapePV = Shape<Int<WgTileQ>, Int<WgTileV>, Int<WgTileK>>;  // Third dimension = WgTileK (K sequence tile, shared with ShapeQK[1])
 268 |   using ShapeOutput = Shape<Int<WgTileQ>, Int<HeadDimV>>;
 269 | 
 270 |   // Derive subgroup counts from tile ratios for QK matmul
 271 |   static_assert(WgTileQ % SgTileQ == 0, "WgTileQ must be divisible by SgTileQ");
 272 |   static_assert(WgTileK % SgTileK == 0, "WgTileK must be divisible by SgTileK");
 273 | 
 274 |   // SubgroupLayoutQK: (num_sg_q, num_sg_k, 1)
 275 |   // Head dimension is never split across subgroups (always _1)
 276 |   using SubgroupLayoutQK = Layout<Shape<
 277 |     Int<WgTileQ / SgTileQ>,
 278 |     Int<WgTileK / SgTileK>,
 279 |     _1
 280 |   >>;
 281 | 
 282 |   // SubgroupLayoutPV: (num_sg_p = num_sg_q, 1, num_sg_k)
 283 |   // number of subgroups in PV GEMM equals that in QK GEMM
 284 |   using SubgroupLayoutPV = Layout<Shape<
 285 |     Int<WgTileQ / SgTileQ>,
 286 |     _1,
 287 |     Int<WgTileK / SgTileK>
 288 |   >>;
 289 | 
 290 |   using type = cutlass::flash_attention::FMHAConfig<
 291 |     ElementQ, ElementK, ElementV, ElementO, LayoutQ, LayoutK, LayoutV, LayoutO,
 292 |     ShapeQK, ShapePV, ShapeOutput,
 293 |     SubgroupLayoutQK, SubgroupLayoutPV,
 294 |     Causal, VarLen, CachedKV, PagedKV, Persistent, PipelineStagesConfig<Mode>::value>;
 295 | };
```
**EN:** Builds configuration types directly from user-specified workgroup and subgroup tile sizes, deriving both QK and PV subgroup layouts from the provided ratios.
**CN:** 直接根据用户指定的 workgroup/subgroup tile 大小构建配置类型，并从这些比例推导 QK 与 PV 的 subgroup 布局。

### Lines 296-298 — File epilogue
```cpp
 296 | 
 297 | } // namespace flash_attention
 298 | } // namespace cutlass
```
**EN:** This trailing block closes scopes or keeps small glue code that finalizes the file structure.
**CN:** 这一尾部代码块用于关闭作用域，或保留收尾用的小型胶水代码，从而完成文件结构。

## Key Concepts / 关键概念

- Type-level FMHA assembly / 类型级 FMHA 组装
- Mode-specific tile presets / 模式相关的 tile 预设
- Persistent versus regular scheduling / 持久化与常规调度
- Explicit tile-shape generation / 显式 tile 形状生成

## Dependencies / 依赖关系

- `flash_attention_v2/collective/fmha_fusion.hpp` — mainloop and epilogue pieces / mainloop 与 epilogue 组件
- `xe_fmha_fwd_kernel.hpp` — kernel shells / 内核外壳
- `xe_tile_scheduler.hpp` — tile schedulers / tile 调度器
- `cutlass/gemm/dispatch_policy.hpp` — pipeline policy / pipeline 策略
