# sm100_mla_tile_scheduler.hpp — Code Analysis / 代码分析

**Source / 源文件**: `examples/77_blackwell_fmha/kernel/sm100_mla_tile_scheduler.hpp`  
**Purpose / 用途**: Defines the tile schedulers used by MLA kernels, including both direct scheduling and persistent scheduling over split-KV work. / 定义 MLA kernel 使用的 tile scheduler，包括对 split-KV 工作的直接调度与 persistent 调度。

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

**EN**: This code block advances the file's control flow or type system in a focused step. In CUTLASS examples, even seemingly local code usually exists to reconcile runtime problem metadata with a heavily templated compile-time kernel configuration. Targets blackwell sm100 execution paths. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一代码块以一个较集中的步骤推进文件的控制流或类型系统。在 CUTLASS 示例中，即便看似局部的代码，通常也是为了把运行期问题元数据和高度模板化的编译期内核配置对齐起来。面向 Blackwell SM100 执行路径。 属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

```cpp
struct Sm100MlaIndividualTileScheduler {

  struct Params {
    dim3 grid;
  };

  bool valid_ = true;

  CUTLASS_DEVICE
  Sm100MlaIndividualTileScheduler(Params const&) {}

  template<class ProblemShape, class ClusterShape>
  static Params to_underlying_arguments(
      ProblemShape const& problem_shape, KernelHardwareInfo hw_info,
      ClusterShape const& cluster_shape, int const& split_kv) {
    using namespace cute;
    dim3 grid(get<0>(cluster_shape), get<3>(problem_shape) /* Batch */, split_kv /*Maximum Split KV*/);
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
    return make_coord(blockIdx.x, _0{}, blockIdx.y, blockIdx.z);
  }

  CUTLASS_DEVICE
  Sm100MlaIndividualTileScheduler& operator++() {
    valid_ = false;
    return *this;
  }
};
```

**EN**: This block defines the runtime argument/parameter packing used to cross the host/device boundary. CUTLASS separates high-level `Arguments` from lower-level `Params` so pointers, strides, schedules, and hardware metadata can be normalized before kernel launch. Targets blackwell sm100 execution paths. Makes cluster geometry an explicit compile-time tuning knob.  
**CN**: 这一块定义了跨越 host/device 边界时使用的运行期参数封装。CUTLASS 通常把高层 `Arguments` 与更底层的 `Params` 分开，这样指针、stride、调度信息和硬件元数据就能在 kernel 启动前统一整理。面向 Blackwell SM100 执行路径。 把 cluster 几何作为显式的编译期调优参数。

---

```cpp
struct Sm100MlaPersistentTileScheduler {

  struct Params {
    int num_blocks;
    FastDivmod divmod_m_block;
    FastDivmod divmod_b;
    FastDivmod divmod_split_kv;
    KernelHardwareInfo hw_info;
  };

  int block_idx = 0;
  Params params;

  CUTLASS_DEVICE
  Sm100MlaPersistentTileScheduler(Params const& params) : block_idx(blockIdx.x), params(params) {}

  template<class ProblemShape, class ClusterShape>
  static Params to_underlying_arguments(
      ProblemShape const& problem_shape, KernelHardwareInfo hw_info,
      ClusterShape const& cluster_shape, int const& split_kv) {
    using namespace cute;
    // Get SM count if needed, otherwise use user supplied SM count
    int sm_count = hw_info.sm_count;
    if (sm_count <= 1 || sm_count % size<0>(cluster_shape) != 0) {
      CUTLASS_TRACE_HOST("  WARNING: Arguments do not include a valid SM count.\n"
          "  For optimal performance, populate the arguments KernelHardwareInfo struct with the SM count.");
      sm_count = KernelHardwareInfo::query_device_multiprocessor_count(hw_info.device_id);
    }

    CUTLASS_TRACE_HOST("to_underlying_arguments(): Setting persistent grid SM count to " << sm_count);
    hw_info.sm_count = sm_count;

    int num_m_blocks = size<0>(cluster_shape);
    int num_blocks = num_m_blocks * get<3>(problem_shape)  /* Batch */;
    num_blocks *= split_kv; /* Maximum Split KV*/
```

**EN**: This block defines the runtime argument/parameter packing used to cross the host/device boundary. CUTLASS separates high-level `Arguments` from lower-level `Params` so pointers, strides, schedules, and hardware metadata can be normalized before kernel launch. Targets blackwell sm100 execution paths. Makes cluster geometry an explicit compile-time tuning knob.  
**CN**: 这一块定义了跨越 host/device 边界时使用的运行期参数封装。CUTLASS 通常把高层 `Arguments` 与更底层的 `Params` 分开，这样指针、stride、调度信息和硬件元数据就能在 kernel 启动前统一整理。面向 Blackwell SM100 执行路径。 把 cluster 几何作为显式的编译期调优参数。

---

```cpp
    return Params {
      num_blocks,
      { num_m_blocks}, { get<3>(problem_shape) }, {split_kv},
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
    int m_block, bidb, n_split_kv;
    params.divmod_m_block(block_decode, m_block, block_decode);
    params.divmod_split_kv(block_decode, n_split_kv, block_decode);
    params.divmod_b(block_decode, bidb, block_decode);
    return make_coord(m_block, _0{}, bidb, n_split_kv);
  }
```

**EN**: This block defines the runtime argument/parameter packing used to cross the host/device boundary. CUTLASS separates high-level `Arguments` from lower-level `Params` so pointers, strides, schedules, and hardware metadata can be normalized before kernel launch. Targets blackwell sm100 execution paths. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一块定义了跨越 host/device 边界时使用的运行期参数封装。CUTLASS 通常把高层 `Arguments` 与更底层的 `Params` 分开，这样指针、stride、调度信息和硬件元数据就能在 kernel 启动前统一整理。面向 Blackwell SM100 执行路径。 属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

```cpp
  CUTLASS_DEVICE
  Sm100MlaPersistentTileScheduler& operator++() {
    block_idx += gridDim.x;
    return *this;
  }
};
```

**EN**: This block defines the runtime argument/parameter packing used to cross the host/device boundary. CUTLASS separates high-level `Arguments` from lower-level `Params` so pointers, strides, schedules, and hardware metadata can be normalized before kernel launch. Targets blackwell sm100 execution paths. Exposes scheduler policy as a first-class performance choice.  
**CN**: 这一块定义了跨越 host/device 边界时使用的运行期参数封装。CUTLASS 通常把高层 `Arguments` 与更底层的 `Params` 分开，这样指针、stride、调度信息和硬件元数据就能在 kernel 启动前统一整理。面向 Blackwell SM100 执行路径。 把调度策略作为一级性能选择暴露出来。

---

```cpp
} // namespace cutlass::fmha::kernel
```

**EN**: This code block advances the file's control flow or type system in a focused step. In CUTLASS examples, even seemingly local code usually exists to reconcile runtime problem metadata with a heavily templated compile-time kernel configuration. Targets blackwell sm100 execution paths. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一代码块以一个较集中的步骤推进文件的控制流或类型系统。在 CUTLASS 示例中，即便看似局部的代码，通常也是为了把运行期问题元数据和高度模板化的编译期内核配置对齐起来。面向 Blackwell SM100 执行路径。 属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

## Key Concepts / 关键概念

- Architecture-tagged specialization (`Sm100`/`Sm103`/`Sm120`) / 基于架构标签的特化（`Sm100`/`Sm103`/`Sm120`）
- Cluster-shape tuning and possible multi-SM cooperation / Cluster 形状调优与可能的多 SM 协作
- Warp-specialized FMHA layering across collective, kernel, and device wrappers / 跨 collective、kernel 与 device wrapper 的 warp-specialized FMHA 分层
- Scheduler policy as a first-class optimization knob / 作为一级优化旋钮的调度策略

## Dependencies / 依赖项

- `cutlass/cutlass.h` — core CUTLASS types, architecture tags, and utilities / CUTLASS 核心类型、架构标签与工具
- `cutlass/fast_math.h` — CUTLASS core component used by this example / 此示例使用的 CUTLASS 核心组件
- `cutlass/kernel_hardware_info.h` — CUTLASS core component used by this example / 此示例使用的 CUTLASS 核心组件
