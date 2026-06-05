# fmha_epilogue_bwd.hpp — Code Analysis / 代码分析

**Source / 源文件**: `examples/88_hopper_fmha/collective/fmha_epilogue_bwd.hpp`  
**Purpose / 用途**: Backward epilogue that stores final dK and dV tensors from the backward collective accumulators / 反向 epilogue，把反向 collective 产生的累加器写回为最终 dK 和 dV 张量

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

#include "cutlass/cutlass.h"
#include "cutlass/epilogue/collective/default_epilogue.hpp"

#include "../collective/fmha_epilogue.hpp"

namespace cutlass::fmha::collective {
```

**EN**: This block defines the runtime argument/parameter packing used to cross the host/device boundary. CUTLASS separates high-level `Arguments` from lower-level `Params` so pointers, strides, schedules, and hardware metadata can be normalized before kernel launch. Uses hopper-era gmma/tma building blocks. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一块定义了跨越 host/device 边界时使用的运行期参数封装。CUTLASS 通常把高层 `Arguments` 与更底层的 `Params` 分开，这样指针、stride、调度信息和硬件元数据就能在 kernel 启动前统一整理。使用 Hopper 时代的 GMMA/TMA 构件。 属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

```cpp
template<class Element, class ElementAccumulator, class TileShape_WG>
struct FmhaBwdEpilogueKV {

  static constexpr int Alignment = 16 / sizeof(Element);

  struct Arguments {
    Element* ptr_K;
    cute::tuple<int, int, int, cute::_1> dK;

    Element* ptr_V;
    cute::tuple<int, int, int, _1> dV;
  };

  //using DefaultOperation = cutlass::epilogue::fusion::LinearCombination<Element, ElementAccumulator, void>;
  static constexpr auto RoundStyle = cutlass::FloatRoundStyle::round_to_nearest;
  using DefaultOperation = cutlass::epilogue::fusion::Sm90EVT<
    cutlass::epilogue::fusion::Sm90Compute<cutlass::first, Element, ElementAccumulator, RoundStyle>,
    cutlass::epilogue::fusion::Sm90AccFetch
  >;
  using CollectiveEpilogueTMA = typename cutlass::epilogue::collective::CollectiveBuilder<
        cutlass::arch::Sm90, cutlass::arch::OpClassTensorOp,
        TileShape_WG, Shape<_1,_1,_1>, cutlass::epilogue::collective::EpilogueTileAuto,
        ElementAccumulator, ElementAccumulator,
        void, cute::tuple<int, _1, cute::tuple<int, int>>, Alignment,
        Element, cute::tuple<int, _1, cute::tuple<int, int>>, Alignment,
        cutlass::epilogue::TmaWarpSpecialized,
        DefaultOperation
    >::CollectiveOp;
```

**EN**: This block defines the runtime argument/parameter packing used to cross the host/device boundary. CUTLASS separates high-level `Arguments` from lower-level `Params` so pointers, strides, schedules, and hardware metadata can be normalized before kernel launch. Uses hopper-era gmma/tma building blocks. Selects tensor-core mma execution instead of a simt fallback.  
**CN**: 这一块定义了跨越 host/device 边界时使用的运行期参数封装。CUTLASS 通常把高层 `Arguments` 与更底层的 `Params` 分开，这样指针、stride、调度信息和硬件元数据就能在 kernel 启动前统一整理。使用 Hopper 时代的 GMMA/TMA 构件。 选择 Tensor Core MMA 执行，而不是 SIMT 回退路径。

---

```cpp
  struct Params {
    typename CollectiveEpilogueTMA::Params epilogue_K;
    typename CollectiveEpilogueTMA::Params epilogue_V;
  };
```

**EN**: This block defines the runtime argument/parameter packing used to cross the host/device boundary. CUTLASS separates high-level `Arguments` from lower-level `Params` so pointers, strides, schedules, and hardware metadata can be normalized before kernel launch. Uses hopper-era gmma/tma building blocks. Relies on tma-style bulk movement or descriptor handling.  
**CN**: 这一块定义了跨越 host/device 边界时使用的运行期参数封装。CUTLASS 通常把高层 `Arguments` 与更底层的 `Params` 分开，这样指针、stride、调度信息和硬件元数据就能在 kernel 启动前统一整理。使用 Hopper 时代的 GMMA/TMA 构件。 依赖 TMA 风格的批量搬运或描述符处理。

---

```cpp
  using TensorStorage = typename CollectiveEpilogueTMA::TensorStorage[2];
  using PipelineStorage = typename CollectiveEpilogueTMA::PipelineStorage;
  using LoadPipeline = typename CollectiveEpilogueTMA::LoadPipeline;
  static constexpr int TmaTransactionBytes = CollectiveEpilogueTMA::TmaTransactionBytes;

  template<class ProblemShape>
  static Params to_underlying_arguments(ProblemShape const& problem_size, Arguments const& args, void* workspace = nullptr) {
    auto dK = make_stride(get<2>(args.dK), get<3>(args.dK),
        make_stride(get<0>(args.dK), get<1>(args.dK)));
    auto dV = make_stride(get<2>(args.dV), get<3>(args.dV),
        make_stride(get<0>(args.dV), get<1>(args.dV)));
      
    auto problem_size_kv = make_shape(get<3>(problem_size), get<4>(problem_size), 1,
              make_shape(get<0>(problem_size), get<1>(problem_size)));
    typename CollectiveEpilogueTMA::Arguments args_k{{}, args.ptr_K, dK, args.ptr_K, dK};
    typename CollectiveEpilogueTMA::Arguments args_v{{}, args.ptr_V, dV, args.ptr_V, dV};
    return Params{
      CollectiveEpilogueTMA::to_underlying_arguments(problem_size_kv, args_k, nullptr),
      CollectiveEpilogueTMA::to_underlying_arguments(problem_size_kv, args_v, nullptr)
    };
  }

  template<class TileShape, class BlkCoord, class ResultTuple, class TiledMma, class ProblemShape>
  CUTLASS_DEVICE void operator()(
      TileShape const& tile_shape, BlkCoord const& blk_coord,
      ResultTuple const& result, TiledMma const& tiled_mma,
      ProblemShape const& problem_size, Params const& params,
      LoadPipeline epi_load_pipeline, TensorStorage& epi_tensor_storage)
  {
    auto acc_k = get<0>(result);
    auto acc_v = get<1>(result);
  
    auto problem_size_kv = make_shape(get<3>(problem_size), get<4>(problem_size), _,
              make_shape(get<0>(problem_size), get<1>(problem_size)));
  
    using EpiStorePipeline = typename CollectiveEpilogueTMA::StorePipeline;
    typename EpiStorePipeline::Params epi_store_pipeline_params;
    epi_store_pipeline_params.always_wait = true;
    EpiStorePipeline epi_store_pipeline(epi_store_pipeline_params);
```

**EN**: This pipeline block sets up or advances explicit producer/consumer state. Rather than using one monolithic loop, the code lets load, compute, softmax/reduction, or epilogue phases overlap through staged synchronization and cursor movement. Uses hopper-era gmma/tma building blocks. Uses explicit producer/consumer pipeline state instead of a single monolithic loop.  
**CN**: 这一流水线块负责建立或推进显式的 producer/consumer 状态。代码不是采用单体式循环，而是让加载、计算、softmax/规约或 epilogue 阶段通过分阶段同步与游标推进实现重叠。使用 Hopper 时代的 GMMA/TMA 构件。 使用显式 producer/consumer 流水线状态，而非单体式循环。

---

```cpp
    typename CollectiveEpilogueTMA::LoadPipelineState epi_load_pipe_consumer_state;
    PipelineState epi_store_pipe_producer_state = cutlass::make_producer_start_state<EpiStorePipeline>();

    CollectiveEpilogueTMA epilogue_k{params.epilogue_K, epi_tensor_storage[0]};
    CollectiveEpilogueTMA epilogue_v{params.epilogue_V, epi_tensor_storage[1]};

    {
      auto [epi_load_pipe_consumer_state_next, epi_store_pipe_producer_state_next] =
      epilogue_k.store(
        epi_load_pipeline, epi_load_pipe_consumer_state,
        epi_store_pipeline, epi_store_pipe_producer_state,
        problem_size_kv, tile_shape, make_coord(get<1>(blk_coord), _0{}, _, get<2>(blk_coord)),
        acc_k, tiled_mma, threadIdx.x % cutlass::NumThreadsPerWarpGroup,
        epi_tensor_storage[0]
      );

    }

    {
      auto [epi_load_pipe_consumer_state_next, epi_store_pipe_producer_state_next] =
      epilogue_v.store(
        epi_load_pipeline, epi_load_pipe_consumer_state,
        epi_store_pipeline, epi_store_pipe_producer_state,
        problem_size_kv, tile_shape, make_coord(get<1>(blk_coord), _0{}, _, get<2>(blk_coord)),
        acc_v, tiled_mma, threadIdx.x % cutlass::NumThreadsPerWarpGroup,
        epi_tensor_storage[1]
      );

      epilogue_k.store_tail(
        epi_load_pipeline, epi_load_pipe_consumer_state_next,
        epi_store_pipeline, epi_store_pipe_producer_state_next
      );

      epilogue_v.store_tail(
        epi_load_pipeline, epi_load_pipe_consumer_state_next,
        epi_store_pipeline, epi_store_pipe_producer_state_next
      );
    }
  }
};
```

**EN**: This pipeline block sets up or advances explicit producer/consumer state. Rather than using one monolithic loop, the code lets load, compute, softmax/reduction, or epilogue phases overlap through staged synchronization and cursor movement. Uses hopper-era gmma/tma building blocks. Uses explicit producer/consumer pipeline state instead of a single monolithic loop.  
**CN**: 这一流水线块负责建立或推进显式的 producer/consumer 状态。代码不是采用单体式循环，而是让加载、计算、softmax/规约或 epilogue 阶段通过分阶段同步与游标推进实现重叠。使用 Hopper 时代的 GMMA/TMA 构件。 使用显式 producer/consumer 流水线状态，而非单体式循环。

---

```cpp
}  // namespace cutlass::fmha::collective
```

**EN**: This pipeline block sets up or advances explicit producer/consumer state. Rather than using one monolithic loop, the code lets load, compute, softmax/reduction, or epilogue phases overlap through staged synchronization and cursor movement. Uses hopper-era gmma/tma building blocks. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一流水线块负责建立或推进显式的 producer/consumer 状态。代码不是采用单体式循环，而是让加载、计算、softmax/规约或 epilogue 阶段通过分阶段同步与游标推进实现重叠。使用 Hopper 时代的 GMMA/TMA 构件。 属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

## Key Concepts / 关键概念

- CUTLASS collective builders that deduce mainloop and epilogue implementations / 推导 mainloop 与 epilogue 实现的 CUTLASS collective 构建器
- TMA-driven data movement / 基于 TMA 的数据搬运
- Explicit producer/consumer pipelines / 显式 producer/consumer 流水线
- Warp-specialized FMHA layering across collective, kernel, and device wrappers / 跨 collective、kernel 与 device wrapper 的 warp-specialized FMHA 分层

## Dependencies / 依赖项

- `cutlass/cutlass.h` — core CUTLASS types, architecture tags, and utilities / CUTLASS 核心类型、架构标签与工具
- `cutlass/epilogue/collective/default_epilogue.hpp` — epilogue collective implementation detail / 尾声 collective 实现细节
- `../collective/fmha_epilogue.hpp` — collective-stage helper for the attention/GEMM pipeline / 注意力或 GEMM 流水线的 collective 阶段辅助
