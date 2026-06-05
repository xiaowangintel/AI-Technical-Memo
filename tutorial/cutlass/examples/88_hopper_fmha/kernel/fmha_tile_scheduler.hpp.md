# fmha_tile_scheduler.hpp — Code Analysis / 代码分析

**Source / 源文件**: `examples/88_hopper_fmha/kernel/fmha_tile_scheduler.hpp`  
**Purpose / 用途**: Provides tile schedulers for Hopper FMHA, including one-shot CTA scheduling, persistent scheduling, and a backward adapter that swaps traversal axes. / 为 Hopper FMHA 提供 tile 调度器，包括一次性 CTA 调度、persistent 调度，以及一个用于交换遍历轴的反向适配器。

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
#include "cutlass/fast_math.h"
#include "cutlass/kernel_hardware_info.h"

namespace cutlass::fmha::kernel {
```

**EN**: This code block advances the file's control flow or type system in a focused step. In CUTLASS examples, even seemingly local code usually exists to reconcile runtime problem metadata with a heavily templated compile-time kernel configuration. Uses hopper-era gmma/tma building blocks. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一代码块以一个较集中的步骤推进文件的控制流或类型系统。在 CUTLASS 示例中，即便看似局部的代码，通常也是为了把运行期问题元数据和高度模板化的编译期内核配置对齐起来。使用 Hopper 时代的 GMMA/TMA 构件。 属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

```cpp
struct IndividualTileScheduler {

  struct Params {
    dim3 grid;
  };

  bool valid_ = true;

  CUTLASS_DEVICE
  IndividualTileScheduler(Params const&) {}

  template<class ProblemSize, class ClusterShape, class TileShape>
  static Params to_underlying_arguments(
      ProblemSize const& problem_size, KernelHardwareInfo hw_info,
      ClusterShape const& cluster_shape, TileShape const& tile_shape)
  {
    using namespace cute;
    dim3 grid(round_up(ceil_div(size<2>(problem_size), size<0>(tile_shape)), size<0>(cluster_shape)), size<0>(problem_size), size<1>(problem_size));
    return Params{ grid };
  }

  static dim3 get_grid_shape(Params const& params) {
    return params.grid;
  }

  CUTLASS_DEVICE
  bool is_valid() {
    return valid_;
  }

  CUTLASS_DEVICE
  auto get_block_coord() {
    using namespace cute;
    return make_coord(blockIdx.x, _0{}, make_coord(blockIdx.y, blockIdx.z));
  }

  CUTLASS_DEVICE
  IndividualTileScheduler& operator++() {
    valid_ = false;
    return *this;
  }
};
```

**EN**: This block defines the runtime argument/parameter packing used to cross the host/device boundary. CUTLASS separates high-level `Arguments` from lower-level `Params` so pointers, strides, schedules, and hardware metadata can be normalized before kernel launch. Uses hopper-era gmma/tma building blocks. Makes cluster geometry an explicit compile-time tuning knob.  
**CN**: 这一块定义了跨越 host/device 边界时使用的运行期参数封装。CUTLASS 通常把高层 `Arguments` 与更底层的 `Params` 分开，这样指针、stride、调度信息和硬件元数据就能在 kernel 启动前统一整理。使用 Hopper 时代的 GMMA/TMA 构件。 把 cluster 几何作为显式的编译期调优参数。

---

```cpp
struct PersistentTileScheduler {

  struct Params {
    int num_blocks;
    FastDivmod divmod_m_block;
    FastDivmod divmod_b;
    FastDivmod divmod_h;

    KernelHardwareInfo hw_info;
  };

  int block_idx = 0;
  Params params;

  CUTLASS_DEVICE
  PersistentTileScheduler(Params const& params) : block_idx(blockIdx.x), params(params) {}

  template<class ProblemSize, class ClusterShape, class TileShape>
  static Params to_underlying_arguments(
      ProblemSize const& problem_size, KernelHardwareInfo hw_info,
      ClusterShape const& cluster_shape, TileShape const& tile_shape)
  {
    using namespace cute;
    // Get SM count if needed, otherwise use user supplied SM count
    int sm_count = hw_info.sm_count;
    if (sm_count <= 0) {
      CUTLASS_TRACE_HOST("  WARNING: Arguments do not include a valid SM count.\n"
          "  For optimal performance, populate the arguments KernelHardwareInfo struct with the SM count.");
      sm_count = KernelHardwareInfo::query_device_multiprocessor_count(hw_info.device_id);
    }

    CUTLASS_TRACE_HOST("to_underlying_arguments(): Setting persistent grid SM count to " << sm_count);
    hw_info.sm_count = sm_count;

    int num_m_blocks = cutlass::round_up(ceil_div(size<2>(problem_size), size<0>(tile_shape)), size<0>(cluster_shape));
    int num_blocks = num_m_blocks * size<0>(problem_size) * size<1>(problem_size);
```

**EN**: This block defines the runtime argument/parameter packing used to cross the host/device boundary. CUTLASS separates high-level `Arguments` from lower-level `Params` so pointers, strides, schedules, and hardware metadata can be normalized before kernel launch. Uses hopper-era gmma/tma building blocks. Makes cluster geometry an explicit compile-time tuning knob.  
**CN**: 这一块定义了跨越 host/device 边界时使用的运行期参数封装。CUTLASS 通常把高层 `Arguments` 与更底层的 `Params` 分开，这样指针、stride、调度信息和硬件元数据就能在 kernel 启动前统一整理。使用 Hopper 时代的 GMMA/TMA 构件。 把 cluster 几何作为显式的编译期调优参数。

---

```cpp
    return Params {
      num_blocks,
      { num_m_blocks}, { size<0>(problem_size) }, { size<1>(problem_size) },
      hw_info
    };
  }

  static dim3 get_grid_shape(Params const& params) {
    dim3 grid(std::min(params.num_blocks, params.hw_info.sm_count), 1, 1);
    return grid;
  }

  CUTLASS_DEVICE
  bool is_valid() {
    return block_idx < params.num_blocks;
  }

  CUTLASS_DEVICE
  auto get_block_coord() {
    using namespace cute;
    int block_decode = block_idx;
    int m_block, bidb, bidh;
    params.divmod_m_block(block_decode, m_block, block_decode);
    params.divmod_b(block_decode, bidb, block_decode);
    params.divmod_h(block_decode, bidh, block_decode);
    return make_coord(m_block, _0{}, make_coord(bidb, bidh));
  }
```

**EN**: This block defines the runtime argument/parameter packing used to cross the host/device boundary. CUTLASS separates high-level `Arguments` from lower-level `Params` so pointers, strides, schedules, and hardware metadata can be normalized before kernel launch. Uses hopper-era gmma/tma building blocks. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一块定义了跨越 host/device 边界时使用的运行期参数封装。CUTLASS 通常把高层 `Arguments` 与更底层的 `Params` 分开，这样指针、stride、调度信息和硬件元数据就能在 kernel 启动前统一整理。使用 Hopper 时代的 GMMA/TMA 构件。 属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

```cpp
  CUTLASS_DEVICE
  PersistentTileScheduler& operator++() {
    block_idx += gridDim.x;
    return *this;
  }
};
```

**EN**: This block defines the runtime argument/parameter packing used to cross the host/device boundary. CUTLASS separates high-level `Arguments` from lower-level `Params` so pointers, strides, schedules, and hardware metadata can be normalized before kernel launch. Uses hopper-era gmma/tma building blocks. Exposes scheduler policy as a first-class performance choice.  
**CN**: 这一块定义了跨越 host/device 边界时使用的运行期参数封装。CUTLASS 通常把高层 `Arguments` 与更底层的 `Params` 分开，这样指针、stride、调度信息和硬件元数据就能在 kernel 启动前统一整理。使用 Hopper 时代的 GMMA/TMA 构件。 把调度策略作为一级性能选择暴露出来。

---

```cpp
template<typename Base>
struct TileSchedulerBwdAdapter {

  using Params = typename Base::Params;

  Base base_;

  CUTLASS_DEVICE
  TileSchedulerBwdAdapter(Params const& params) : base_(params) {}

  template<class ProblemSize, class ClusterShape, class TileShape>
  static Params to_underlying_arguments(
      ProblemSize const& problem_size, KernelHardwareInfo hw_info,
      ClusterShape const& cluster_shape, TileShape const& tile_shape)
  {
    using namespace cute;
    return Base::to_underlying_arguments(select<0,1,3,2,4>(problem_size), hw_info, select<1,0,2>(cluster_shape), select<1,0,2>(tile_shape));
  }

  static dim3 get_grid_shape(Params const& params) {
    return Base::get_grid_shape(params);
  }

  CUTLASS_DEVICE
  bool is_valid() {
    return base_.is_valid();
  }

  CUTLASS_DEVICE
  auto get_block_coord() {
    using namespace cute;
    return select<1,0,2>(base_.get_block_coord());
  }

  CUTLASS_DEVICE
  TileSchedulerBwdAdapter& operator++() {
    ++base_;
    return *this;
  }
};
```

**EN**: This block deals with work distribution across CTAs, clusters, or persistent loops. In these examples the scheduler choice often changes utilization more than arithmetic itself, because it controls how tiles are handed to available SMs and when the kernel can keep the machine busy. Uses hopper-era gmma/tma building blocks. Makes cluster geometry an explicit compile-time tuning knob.  
**CN**: 这一块处理的是 CTA、cluster 或 persistent 循环之间的工作分发。对于这些示例来说，调度器的选择对利用率的影响常常不亚于算术本身，因为它决定 tile 如何交给可用的 SM，以及内核怎样持续喂满机器。使用 Hopper 时代的 GMMA/TMA 构件。 把 cluster 几何作为显式的编译期调优参数。

---

```cpp
}  // namespace cutlass::fmha::kernel
```

**EN**: This code block advances the file's control flow or type system in a focused step. In CUTLASS examples, even seemingly local code usually exists to reconcile runtime problem metadata with a heavily templated compile-time kernel configuration. Uses hopper-era gmma/tma building blocks. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一代码块以一个较集中的步骤推进文件的控制流或类型系统。在 CUTLASS 示例中，即便看似局部的代码，通常也是为了把运行期问题元数据和高度模板化的编译期内核配置对齐起来。使用 Hopper 时代的 GMMA/TMA 构件。 属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

## Key Concepts / 关键概念

- Cluster-shape tuning and possible multi-SM cooperation / Cluster 形状调优与可能的多 SM 协作
- Warp-specialized FMHA layering across collective, kernel, and device wrappers / 跨 collective、kernel 与 device wrapper 的 warp-specialized FMHA 分层
- Scheduler policy as a first-class optimization knob / 作为一级优化旋钮的调度策略

## Dependencies / 依赖项

- `cutlass/cutlass.h` — core CUTLASS types, architecture tags, and utilities / CUTLASS 核心类型、架构标签与工具
- `cutlass/fast_math.h` — CUTLASS core component used by this example / 此示例使用的 CUTLASS 核心组件
- `cutlass/kernel_hardware_info.h` — CUTLASS core component used by this example / 此示例使用的 CUTLASS 核心组件
