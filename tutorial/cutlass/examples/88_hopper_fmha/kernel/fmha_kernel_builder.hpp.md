# fmha_kernel_builder.hpp — Code Analysis / 代码分析

**Source / 源文件**: `examples/88_hopper_fmha/kernel/fmha_kernel_builder.hpp`  
**Purpose / 用途**: Dispatches Hopper FMHA forward-kernel assembly from element/layout/dispatch-policy templates to the concrete mainloop, epilogue, and kernel type. / 把 Hopper FMHA 前向内核的模板参数（数据类型、布局、派发策略）组装成具体的 mainloop、epilogue 与 kernel 类型。

---

## Line-by-Line Analysis / 逐行分析

```cpp
/***************************************************************************************************
 * Copyright (c) 2024 - 2026 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
 * SPDX-License-Identifier: BSD-3-Clause
 *
 * Redistribution and use in source and binary forms, with or without
 * modification, are permitted provided that the following conditions are met:
 *
 * 1. Redistributions of source code must retain the above copyright notice, this
 * list of conditions and the following disclaimer.
 *
 * 2. Redistributions in binary form must reproduce the above copyright notice,
 * this list of conditions and the following disclaimer in the documentation
 * and/or other materials provided with the distribution.
 *
 * 3. Neither the name of the copyright holder nor the names of its
 * contributors may be used to endorse or promote products derived from
 * this software without specific prior written permission.
 *
 * THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
 * AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
 * IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
 * DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE
 * FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
 * DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
 * SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
 * CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
 * OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
 * OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
 *
 **************************************************************************************************/

#pragma once

#include "../collective/fmha_collective_tma.hpp"
#include "../collective/fmha_collective_tma_warpspecialized.hpp"
#include "../collective/fmha_epilogue.hpp"
#include "../kernel/fmha_kernel_tma.hpp"
#include "../kernel/fmha_kernel_tma_warpspecialized.hpp"
#include "../kernel/fmha_options.hpp"

namespace cutlass::fmha::kernel {
```

**EN**: This block wires CUTLASS collective types together. It is where architecture tags, operator classes, operand layouts, tile/cluster shapes, and stage policies are assembled so the library can deduce a concrete mainloop or epilogue implementation. Uses hopper-era gmma/tma building blocks. Relies on tma-style bulk movement or descriptor handling.  
**CN**: 这一块负责把 CUTLASS 的 collective 类型真正接起来：架构标签、算子类别、操作数布局、tile/cluster 形状以及 stage 策略都会在这里组合，从而让库推导出具体的 mainloop 或 epilogue 实现。使用 Hopper 时代的 GMMA/TMA 构件。 依赖 TMA 风格的批量搬运或描述符处理。

---

```cpp
template<
  class Element_,
  class ElementAccumulatorQK_,
  class ElementAccumulatorPV_,
  class TileShape_, // BlockQO, BlockKV, BlockHead
  class LayoutQ_,
  class LayoutK_,
  class LayoutV_,
  class Fusion,
  class DispatchPolicy,
  class... Options
>
struct FmhaBuilder;

template<
  class Element,
  class ElementAccumulator,
  class TileShape, // BlockQO, BlockKV, BlockHead
  class Fusion,
  class... Options
>
struct FmhaBuilder<
  Element,
  ElementAccumulator,
  ElementAccumulator,
  TileShape,
  cute::tuple<int, _1, cute::tuple<int, int>>,
  cute::tuple<int, _1, cute::tuple<int, int>>,
  cute::tuple<int, _1, cute::tuple<int, int>>,
  Fusion,
  cutlass::gemm::KernelTma,
  Options...
> {

  using CollectiveMainloop = cutlass::fmha::collective::FmhaMainloopTma<Element, ElementAccumulator, TileShape, Fusion, Options...>;

  using CollectiveEpilogue = cutlass::fmha::collective::FmhaFwdEpilogue<
      Element, ElementAccumulator, typename CollectiveMainloop::TileShapePV>;

  using Kernel = cutlass::fmha::kernel::FmhaKernelTma<CollectiveMainloop, CollectiveEpilogue, Options...>;
};
```

**EN**: This block wires CUTLASS collective types together. It is where architecture tags, operator classes, operand layouts, tile/cluster shapes, and stage policies are assembled so the library can deduce a concrete mainloop or epilogue implementation. Uses hopper-era gmma/tma building blocks. Relies on tma-style bulk movement or descriptor handling.  
**CN**: 这一块负责把 CUTLASS 的 collective 类型真正接起来：架构标签、算子类别、操作数布局、tile/cluster 形状以及 stage 策略都会在这里组合，从而让库推导出具体的 mainloop 或 epilogue 实现。使用 Hopper 时代的 GMMA/TMA 构件。 依赖 TMA 风格的批量搬运或描述符处理。

---

```cpp
template<
  class Element,
  class ElementAccumulatorQK,
  class ElementAccumulatorPV,
  class TileShape, // BlockQO, BlockKV, BlockHead
  class LayoutQ,
  class LayoutK,
  class LayoutV,
  class Fusion,
  class... Options
>
struct FmhaBuilder<
  Element,
  ElementAccumulatorQK,
  ElementAccumulatorPV,
  TileShape,
  LayoutQ,
  LayoutK,
  LayoutV,
  Fusion,
  cutlass::gemm::KernelTmaWarpSpecializedCooperative,
  Options...
> {

  using CollectiveMainloop = cutlass::fmha::collective::FmhaMainloopTmaWarpSpecialized<
      Element, ElementAccumulatorQK, ElementAccumulatorPV,
      TileShape, LayoutQ, LayoutK, LayoutV,
      Fusion, Options...>;

  using CollectiveEpilogue = cutlass::fmha::collective::FmhaFwdEpilogue<
      Element, ElementAccumulatorPV, typename CollectiveMainloop::TileShapePV>;

  static constexpr bool kIsPersistent = find_option_t<Tag::kIsPersistent, false_type, Options...>::value;
  using TileScheduler = std::conditional_t<kIsPersistent, cutlass::fmha::kernel::PersistentTileScheduler, cutlass::fmha::kernel::IndividualTileScheduler>;

  using Kernel = cutlass::fmha::kernel::FmhaKernelTmaWarpSpecialized<CollectiveMainloop, CollectiveEpilogue, TileScheduler, Options...>;
};
```

**EN**: This block wires CUTLASS collective types together. It is where architecture tags, operator classes, operand layouts, tile/cluster shapes, and stage policies are assembled so the library can deduce a concrete mainloop or epilogue implementation. Uses hopper-era gmma/tma building blocks. Relies on tma-style bulk movement or descriptor handling.  
**CN**: 这一块负责把 CUTLASS 的 collective 类型真正接起来：架构标签、算子类别、操作数布局、tile/cluster 形状以及 stage 策略都会在这里组合，从而让库推导出具体的 mainloop 或 epilogue 实现。使用 Hopper 时代的 GMMA/TMA 构件。 依赖 TMA 风格的批量搬运或描述符处理。

---

```cpp
template<
  class Element,
  class ElementAccumulatorQK,
  class ElementAccumulatorPV,
  class TileShape, // BlockQO, BlockKV, BlockHead
  class LayoutQ,
  class LayoutK,
  class LayoutV,
  class Fusion,
  class... Options
>
struct FmhaBuilder<
  Element,
  ElementAccumulatorQK,
  ElementAccumulatorPV,
  TileShape,
  LayoutQ,
  LayoutK,
  LayoutV,
  Fusion,
  cutlass::gemm::KernelTmaWarpSpecializedPingpong,
  Options...
> {
  using Kernel = typename FmhaBuilder<
      Element, ElementAccumulatorQK, ElementAccumulatorPV,
      TileShape,
      LayoutQ, LayoutK, LayoutV,
      Fusion,
      cutlass::gemm::KernelTmaWarpSpecializedCooperative,
      Options...,
      Option<Tag::kIsPersistent, true_type>,
      Option<Tag::kLoadsQSeparately, true_type>
  >::Kernel;
};
```

**EN**: This block wires CUTLASS collective types together. It is where architecture tags, operator classes, operand layouts, tile/cluster shapes, and stage policies are assembled so the library can deduce a concrete mainloop or epilogue implementation. Uses hopper-era gmma/tma building blocks. Relies on tma-style bulk movement or descriptor handling.  
**CN**: 这一块负责把 CUTLASS 的 collective 类型真正接起来：架构标签、算子类别、操作数布局、tile/cluster 形状以及 stage 策略都会在这里组合，从而让库推导出具体的 mainloop 或 epilogue 实现。使用 Hopper 时代的 GMMA/TMA 构件。 依赖 TMA 风格的批量搬运或描述符处理。

---

```cpp
}  // namespace cutlass::fmha::kernel
```

**EN**: This block wires CUTLASS collective types together. It is where architecture tags, operator classes, operand layouts, tile/cluster shapes, and stage policies are assembled so the library can deduce a concrete mainloop or epilogue implementation. Uses hopper-era gmma/tma building blocks. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一块负责把 CUTLASS 的 collective 类型真正接起来：架构标签、算子类别、操作数布局、tile/cluster 形状以及 stage 策略都会在这里组合，从而让库推导出具体的 mainloop 或 epilogue 实现。使用 Hopper 时代的 GMMA/TMA 构件。 属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

## Key Concepts / 关键概念

- TMA-driven data movement / 基于 TMA 的数据搬运
- Warp-specialized FMHA layering across collective, kernel, and device wrappers / 跨 collective、kernel 与 device wrapper 的 warp-specialized FMHA 分层
- Scheduler policy as a first-class optimization knob / 作为一级优化旋钮的调度策略

## Dependencies / 依赖项

- `../collective/fmha_collective_tma.hpp` — collective-stage helper for the attention/GEMM pipeline / 注意力或 GEMM 流水线的 collective 阶段辅助
- `../collective/fmha_collective_tma_warpspecialized.hpp` — collective-stage helper for the attention/GEMM pipeline / 注意力或 GEMM 流水线的 collective 阶段辅助
- `../collective/fmha_epilogue.hpp` — collective-stage helper for the attention/GEMM pipeline / 注意力或 GEMM 流水线的 collective 阶段辅助
- `../kernel/fmha_kernel_tma.hpp` — kernel-layer helper for launch-time execution / 启动期执行所需的内核层辅助
- `../kernel/fmha_kernel_tma_warpspecialized.hpp` — kernel-layer helper for launch-time execution / 启动期执行所需的内核层辅助
- `../kernel/fmha_options.hpp` — kernel-layer helper for launch-time execution / 启动期执行所需的内核层辅助
