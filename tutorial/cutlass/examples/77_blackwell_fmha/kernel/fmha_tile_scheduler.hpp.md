# fmha_tile_scheduler.hpp — Code Analysis / 代码分析

**Source / 源文件**: `examples/77_blackwell_fmha/kernel/fmha_tile_scheduler.hpp`  
**Purpose / 用途**: Defines the default non-causal tile schedulers for FMHA, covering both ordinary grid launches and persistent launches. / 定义 FMHA 默认的非因果 tile scheduler，包括普通 grid 启动与 persistent 启动。

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
```

**EN**: This comment block frames the file: it usually states the target architecture, problem family, usage pattern, or design motivation before the code starts. That context matters because later template choices are easier to understand once the intended hardware path and workload are known. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一段注释用于给文件定调：通常先说明目标架构、问题类型、使用方式或设计动机，再进入具体代码。先理解这些背景，再看后面的模板选择与硬件路径会更清晰。属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

```cpp
#pragma once
```

**EN**: This preprocessor block gates architecture-specific code or closes a specialized compilation path. It keeps the file buildable across toolchains and GPU targets even when the most optimized path only exists on a subset of architectures. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一预处理块负责为特定架构的代码设置编译门槛，或者结束某条专门化路径。这样即便最优路径只存在于部分 GPU 架构上，文件依旧可以在不同工具链与目标上保持可编译。属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

```cpp
#include "cutlass/cutlass.h"
#include "cutlass/fast_math.h"
#include "cutlass/kernel_hardware_info.h"

namespace cutlass::fmha::kernel {
```

**EN**: This code block advances the file's control flow or type system in a focused step. In CUTLASS examples, even seemingly local code usually exists to reconcile runtime problem metadata with a heavily templated compile-time kernel configuration. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一代码块以一个较集中的步骤推进文件的控制流或类型系统。在 CUTLASS 示例中，即便看似局部的代码，通常也是为了把运行期问题元数据和高度模板化的编译期内核配置对齐起来。属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

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
      ClusterShape const& cluster_shape, TileShape const& tile_shape) {
    using namespace cute;
    dim3 grid(round_up(ceil_div(size<0>(problem_size), size<0>(tile_shape)), size<0>(cluster_shape)), size<3,0>(problem_size), size<3,1>(problem_size));
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

**EN**: This block defines the runtime argument/parameter packing used to cross the host/device boundary. CUTLASS separates high-level `Arguments` from lower-level `Params` so pointers, strides, schedules, and hardware metadata can be normalized before kernel launch. Makes cluster geometry an explicit compile-time tuning knob. Exposes scheduler policy as a first-class performance choice.  
**CN**: 这一块定义了跨越 host/device 边界时使用的运行期参数封装。CUTLASS 通常把高层 `Arguments` 与更底层的 `Params` 分开，这样指针、stride、调度信息和硬件元数据就能在 kernel 启动前统一整理。把 cluster 几何作为显式的编译期调优参数。 把调度策略作为一级性能选择暴露出来。

---

```cpp
struct PersistentTileScheduler {

  struct Params {
    int num_blocks;
    FastDivmod divmod_m_block;
    FastDivmod divmod_h;
    FastDivmod divmod_b;

    KernelHardwareInfo hw_info;
  };

  int block_idx = 0;
  Params params;

  CUTLASS_DEVICE
  PersistentTileScheduler(Params const& params) : block_idx(blockIdx.x), params(params) {}

  template<class ProblemSize, class ClusterShape, class TileShape>
  static Params to_underlying_arguments(
      ProblemSize const& problem_size, KernelHardwareInfo hw_info,
      ClusterShape const& cluster_shape, TileShape const& tile_shape) {
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

    int num_m_blocks = cutlass::round_up(ceil_div(size<0>(problem_size), size<0>(tile_shape)), size<0>(cluster_shape));
    int num_blocks = num_m_blocks * size<3,0>(problem_size) * size<3,1>(problem_size);
```

**EN**: This block defines the runtime argument/parameter packing used to cross the host/device boundary. CUTLASS separates high-level `Arguments` from lower-level `Params` so pointers, strides, schedules, and hardware metadata can be normalized before kernel launch. Makes cluster geometry an explicit compile-time tuning knob. Exposes scheduler policy as a first-class performance choice.  
**CN**: 这一块定义了跨越 host/device 边界时使用的运行期参数封装。CUTLASS 通常把高层 `Arguments` 与更底层的 `Params` 分开，这样指针、stride、调度信息和硬件元数据就能在 kernel 启动前统一整理。把 cluster 几何作为显式的编译期调优参数。 把调度策略作为一级性能选择暴露出来。

---

```cpp
    return Params {
      num_blocks,
      { num_m_blocks}, { size<3,0>(problem_size) }, { size<3,1>(problem_size) },
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
    return make_coord(m_block, _0{}, make_coord(bidh, bidb));
  }
```

**EN**: This block defines the runtime argument/parameter packing used to cross the host/device boundary. CUTLASS separates high-level `Arguments` from lower-level `Params` so pointers, strides, schedules, and hardware metadata can be normalized before kernel launch. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一块定义了跨越 host/device 边界时使用的运行期参数封装。CUTLASS 通常把高层 `Arguments` 与更底层的 `Params` 分开，这样指针、stride、调度信息和硬件元数据就能在 kernel 启动前统一整理。属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

```cpp
  CUTLASS_DEVICE
  PersistentTileScheduler& operator++() {
    block_idx += gridDim.x;
    return *this;
  }
};
```

**EN**: This block defines the runtime argument/parameter packing used to cross the host/device boundary. CUTLASS separates high-level `Arguments` from lower-level `Params` so pointers, strides, schedules, and hardware metadata can be normalized before kernel launch. Exposes scheduler policy as a first-class performance choice. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一块定义了跨越 host/device 边界时使用的运行期参数封装。CUTLASS 通常把高层 `Arguments` 与更底层的 `Params` 分开，这样指针、stride、调度信息和硬件元数据就能在 kernel 启动前统一整理。把调度策略作为一级性能选择暴露出来。 属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

```cpp
}  // namespace cutlass::fmha::kernel
```

**EN**: This code block advances the file's control flow or type system in a focused step. In CUTLASS examples, even seemingly local code usually exists to reconcile runtime problem metadata with a heavily templated compile-time kernel configuration. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一代码块以一个较集中的步骤推进文件的控制流或类型系统。在 CUTLASS 示例中，即便看似局部的代码，通常也是为了把运行期问题元数据和高度模板化的编译期内核配置对齐起来。属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

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
