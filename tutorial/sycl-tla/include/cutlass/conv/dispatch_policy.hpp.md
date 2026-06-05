# dispatch_policy.hpp — Code Analysis / 代码分析

## Source / 来源
- **Path:** `include/cutlass/conv/dispatch_policy.hpp`
- **Purpose (EN):** Declares dispatch-policy tags that steer collective and kernel selection.
- **用途 (CN):** 声明用于引导 collective 与 kernel 选择的分发策略标签。

## Line-by-Line Analysis / 逐行分析

### Lines 1-30
```cpp
  1: /***************************************************************************************************
  2:  * Copyright (c) 2023 - 2025 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
  3:  * SPDX-License-Identifier: BSD-3-Clause
  4:  *
  5:  * Redistribution and use in source and binary forms, with or without
  6:  * modification, are permitted provided that the following conditions are met:
  7:  *
  8:  * 1. Redistributions of source code must retain the above copyright notice, this
  9:  * list of conditions and the following disclaimer.
 10:  *
 11:  * 2. Redistributions in binary form must reproduce the above copyright notice,
 12:  * this list of conditions and the following disclaimer in the documentation
 13:  * and/or other materials provided with the distribution.
 14:  *
 15:  * 3. Neither the name of the copyright holder nor the names of its
 16:  * contributors may be used to endorse or promote products derived from
 17:  * this software without specific prior written permission.
 18:  *
 19:  * THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
 20:  * AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
 21:  * IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
 22:  * DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE
 23:  * FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
 24:  * DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
 25:  * SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
 26:  * CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
 27:  * OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
 28:  * OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
 29:  *
 30:  **************************************************************************************************/
```
**EN:** Records the BSD-3-Clause license and redistribution conditions for the file.

**CN:** 记录该文件的 BSD-3-Clause 许可和再分发条件。

### Lines 31-31
```cpp
 31: #pragma once
```
**EN:** Marks this file as a single-inclusion header with `#pragma once`.

**CN:** 通过 `#pragma once` 把该文件标记为单次包含头文件。

### Lines 33-35
```cpp
 33: #include "cutlass/conv/convolution.h"
 34: #include "cutlass/epilogue/thread/activation.h"
 35: #include "cutlass/arch/arch.h"
```
**EN:** Imports direct dependencies used later in the file, including `convolution.h`, `activation.h`, `arch.h`.

**CN:** 引入本文件后续实现要用到的直接依赖，例如 `convolution.h`, `activation.h`, `arch.h`。

### Lines 37-38
```cpp
 37: #include "cute/layout.hpp"
 38: #include "cute/numeric/integral_constant.hpp"
```
**EN:** Imports direct dependencies used later in the file, including `layout.hpp`, `integral_constant.hpp`.

**CN:** 引入本文件后续实现要用到的直接依赖，例如 `layout.hpp`, `integral_constant.hpp`。

### Lines 40-40
```cpp
 40: #include "cutlass/gemm/dispatch_policy.hpp"
```
**EN:** Imports direct dependencies used later in the file, including `dispatch_policy.hpp`.

**CN:** 引入本文件后续实现要用到的直接依赖，例如 `dispatch_policy.hpp`。

### Lines 46-46
```cpp
 46: namespace cutlass::conv {
```
**EN:** Opens the namespace scope `cutlass::conv` that contains these definitions.

**CN:** 打开包含这些定义的命名空间作用域 `cutlass::conv`。

### Lines 50-55
```cpp
 50: //
 51: // Policies for categorical dispatch of mainloop against kernel grid schedules
 52: //
 53: struct KernelImplicitTmaWarpSpecializedSm90 : cutlass::gemm::KernelTmaWarpSpecialized { };
 54: struct KernelImplicitTmaWarpSpecializedSm90Cooperative { };
 55: struct KernelImplicitTmaWarpSpecializedSm90Pingpong { };
```
**EN:** Declares struct `KernelImplicitTmaWarpSpecializedSm90`. The nearby comment explains that it serves the surrounding kernel implicit TMA warp specialized SM90 logic.

**CN:** 声明结构体 `KernelImplicitTmaWarpSpecializedSm90`，相邻注释说明它服务于周围的 内核 隐式 TMA warp 级 特化 SM90 逻辑。

### Lines 57-78
```cpp
 57: //
 58: // Collective Mainloop Policies
 59: //
 61: // n-buffer in smem (Hopper TMA), pipelined with Hopper GMMA and TMA, static schedule between TMA and GMMA
 62: // for fprop
 63: template<
 64:   conv::Operator ConvOp_,
 65:   int Stages_,
 66:   int NumSpatialDimensions_,
 67:   class ClusterShape_ = cute::Shape<cute::C<1>,cute::C<1>,cute::C<1>>,
 68:   class KernelSchedule = KernelImplicitTmaWarpSpecializedSm90,
 69:   int PipelineAsyncMmaStages_ = 1
 70: >
 71: struct MainloopSm90TmaGmmaWarpSpecializedImplicitGemm {
 72:   static constexpr int Stages = Stages_;
 73:   static constexpr int NumSpatialDimensions = NumSpatialDimensions_;
 74:   static constexpr Operator ConvOp = ConvOp_;
 75:   static constexpr int PipelineAsyncMmaStages = PipelineAsyncMmaStages_;
 76:   using ClusterShape = ClusterShape_;
 77:   using ArchTag = arch::Sm90;
 78:   using Schedule = KernelSchedule;
```
**EN:** Declares struct `MainloopSm90TmaGmmaWarpSpecializedImplicitGemm`. The nearby comment explains that it serves the surrounding mainloop SM90 TMA GMMA warp specialized implicit GEMM logic.

**CN:** 声明结构体 `MainloopSm90TmaGmmaWarpSpecializedImplicitGemm`，相邻注释说明它服务于周围的 mainloop SM90 TMA GMMA warp 级 特化 隐式 GEMM 逻辑。

### Lines 80-84
```cpp
 80:   static_assert(NumSpatialDimensions >= 1);
 81:   static_assert(! (cute::is_same_v<KernelSchedule,KernelImplicitTmaWarpSpecializedSm90Cooperative> ||
 82:                    cute::is_same_v<KernelSchedule,KernelImplicitTmaWarpSpecializedSm90Pingpong>),
 83:     "Persistent schedules not support for conv yet.");
 84: };
```
**EN:** Defines function `static_assert` for this stage of the convolution workflow.

**CN:** 定义函数 `static_assert`，服务于卷积工作流的这一阶段。

### Lines 88-92
```cpp
 88: // SM100 tensor op kernel schedule
 89: struct KernelImplicitTmaWarpSpecializedSm100 {
 90:   static constexpr int SchedulerPipelineStageCount = 0;
 91:   static constexpr int AccumulatorPipelineStageCount = 0;
 92: };
```
**EN:** Declares struct `KernelImplicitTmaWarpSpecializedSm100`. The nearby comment explains that it serves the surrounding kernel implicit TMA warp specialized SM100 logic.

**CN:** 声明结构体 `KernelImplicitTmaWarpSpecializedSm100`，相邻注释说明它服务于周围的 内核 隐式 TMA warp 级 特化 SM100 逻辑。

### Lines 94-97
```cpp
 94: // Pseudo-policies for builder auto override that dispatches to the KernelImplicitTmaWarpSpecializedSm100
 95: // but for opting into 1 or 2 SM atoms
 96: struct KernelImplicitTmaWarpSpecialized1SmSm100 : KernelImplicitTmaWarpSpecializedSm100 { };
 97: struct KernelImplicitTmaWarpSpecialized2SmSm100 : KernelImplicitTmaWarpSpecializedSm100 { };
```
**EN:** Declares struct `KernelImplicitTmaWarpSpecialized1SmSm100`. The nearby comment explains that it serves the surrounding kernel implicit TMA warp specialized1 sm SM100 logic.

**CN:** 声明结构体 `KernelImplicitTmaWarpSpecialized1SmSm100`，相邻注释说明它服务于周围的 内核 隐式 TMA warp 级 specialized1 sm SM100 逻辑。

### Lines 99-100
```cpp
 99: struct KernelStridedDgradTmaWs1SmSm100 { };
100: struct KernelStridedDgradTmaWs2SmSm100 { };
```
**EN:** Declares struct `KernelStridedDgradTmaWs1SmSm100`, a kernel strided data-gradient TMA ws1 sm SM100 component in the convolution stack.

**CN:** 声明结构体 `KernelStridedDgradTmaWs1SmSm100`，它是卷积栈中的 内核 跨步 数据梯度 TMA ws1 sm SM100 组件。

### Lines 102-110
```cpp
102: // Policy for implicit gemm kernel
103: template<
104:   int SchedulerPipelineStageCount_,
105:   int AccumulatorPipelineStageCount_
106: >
107: struct KernelScheduleImplicitTmaWarpSpecializedSm100 : KernelImplicitTmaWarpSpecializedSm100 {
108:   static constexpr int SchedulerPipelineStageCount = SchedulerPipelineStageCount_;
109:   static constexpr int AccumulatorPipelineStageCount = AccumulatorPipelineStageCount_;
110: };
```
**EN:** Declares struct `KernelScheduleImplicitTmaWarpSpecializedSm100`. The nearby comment explains that it serves the surrounding kernel schedule implicit TMA warp specialized SM100 logic.

**CN:** 声明结构体 `KernelScheduleImplicitTmaWarpSpecializedSm100`，相邻注释说明它服务于周围的 内核 schedule 隐式 TMA warp 级 特化 SM100 逻辑。

### Lines 112-127
```cpp
112: // n-buffer in smem (Blackwell TMA), pipelined with Blackwell UMMA and TMA, fprop
113: template<
114:   conv::Operator ConvOp_,
115:   int Stages_,
116:   int NumSpatialDimensions_,
117:   int SchedulerPipelineStageCount_,
118:   int AccumulatorPipelineStageCount_,
119:   class ClusterShape_ = cute::Shape<cute::C<1>,cute::C<1>,cute::C<1>>
120: >
121: struct MainloopSm100TmaUmmaWarpSpecializedImplicitGemm {
122:   static constexpr int Stages = Stages_;
123:   static constexpr int NumSpatialDimensions = NumSpatialDimensions_;
124:   static constexpr Operator ConvOp = ConvOp_;
125:   using ClusterShape = ClusterShape_;
126:   using ArchTag = arch::Sm100;
127:   using Schedule = KernelScheduleImplicitTmaWarpSpecializedSm100<SchedulerPipelineStageCount_, AccumulatorPipelineStageCount_>;
```
**EN:** Declares struct `MainloopSm100TmaUmmaWarpSpecializedImplicitGemm`. The nearby comment explains that it serves the surrounding mainloop SM100 TMA UMMA warp specialized implicit GEMM logic.

**CN:** 声明结构体 `MainloopSm100TmaUmmaWarpSpecializedImplicitGemm`，相邻注释说明它服务于周围的 mainloop SM100 TMA UMMA warp 级 特化 隐式 GEMM 逻辑。

### Lines 129-130
```cpp
129:   static_assert(NumSpatialDimensions >= 1);
130: }; 
```
**EN:** Defines function `static_assert` for this stage of the convolution workflow.

**CN:** 定义函数 `static_assert`，服务于卷积工作流的这一阶段。

### Lines 134-134
```cpp
134: } // namespace cutlass::conv 
```
**EN:** Closes the current scope and finishes the surrounding definition block.

**CN:** 关闭当前作用域，并结束外围定义块。

## Key Concepts / 关键概念
- **EN:** Main role: Declares dispatch-policy tags that steer collective and kernel selection. **CN:** 核心作用：声明用于引导 collective 与 kernel 选择的分发策略标签。
- **EN:** Key exported symbols include `KernelImplicitTmaWarpSpecializedSm90`, `KernelImplicitTmaWarpSpecializedSm90Cooperative`, `KernelImplicitTmaWarpSpecializedSm90Pingpong`, `MainloopSm90TmaGmmaWarpSpecializedImplicitGemm`, `KernelImplicitTmaWarpSpecializedSm100`, `KernelImplicitTmaWarpSpecialized1SmSm100`. **CN:** 关键导出符号包括 `KernelImplicitTmaWarpSpecializedSm90`, `KernelImplicitTmaWarpSpecializedSm90Cooperative`, `KernelImplicitTmaWarpSpecializedSm90Pingpong`, `MainloopSm90TmaGmmaWarpSpecializedImplicitGemm`, `KernelImplicitTmaWarpSpecializedSm100`, `KernelImplicitTmaWarpSpecialized1SmSm100`。
- **EN:** MMA composition is central: the file either defines math operators directly or prepares data for them. **CN:** MMA 组合是核心：该文件要么直接定义数学算子，要么为其准备数据。
- **EN:** The implementation is architecture-aware and may specialize behavior for specific GPU generations. **CN:** 该实现具有架构感知能力，可能会针对特定 GPU 代际进行特化。

## Dependencies / 依赖关系
### Direct Includes / 直接包含
- `cutlass/conv/convolution.h`
- `cutlass/epilogue/thread/activation.h`
- `cutlass/arch/arch.h`
- `cute/layout.hpp`
- `cute/numeric/integral_constant.hpp`
- `cutlass/gemm/dispatch_policy.hpp`

### Internal Relationships / 内部关系
- **EN:** Uses CUTE metaprogramming and shape utilities. **CN:** 使用 CUTE 元编程与形状工具。
- **EN:** Builds on CUTLASS GEMM shapes, policies, or operators. **CN:** 构建在 CUTLASS GEMM 的形状、策略或算子之上。
