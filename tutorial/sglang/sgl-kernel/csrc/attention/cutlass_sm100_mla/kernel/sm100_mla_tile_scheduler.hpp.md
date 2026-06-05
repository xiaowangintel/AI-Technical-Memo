# sm100_mla_tile_scheduler.hpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/csrc/attention/cutlass_sm100_mla/kernel/sm100_mla_tile_scheduler.hpp`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Implements attention-related kernels, layouts, or dispatch helpers for high-throughput inference. This header primarily defines interfaces, templates, constants, and inline helpers shared by compilation units. / 实现面向高吞吐推理的 attention 内核、数据布局或分发辅助逻辑。 该头文件主要定义编译单元共享的接口、模板、常量和内联辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-22: Local implementation details
```cpp
/***************************************************************************************************
 * Copyright (c) 2024 - 2025 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
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
```
**EN:** This section fills in the local implementation details around the surrounding logic, completing the behavior required by the file.
**CN:** 本段补充了相关逻辑周边的局部实现细节，以完成该文件所需的具体行为。

### Lines 23-44: Types and data layout
```cpp
 * FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
 * DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
 * SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
 * CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
 * OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
 * OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
 *
 **************************************************************************************************/

// clang-format off
#pragma once

#include "cutlass/cutlass.h"
#include "cutlass/fast_math.h"
#include "cutlass/kernel_hardware_info.h"

namespace cutlass::fmha::kernel {

////////////////////////////////////////////////////////////////////////////////

struct Sm100MlaIndividualTileScheduler {
```
**EN:** This section defines `Sm100MlaIndividualTileScheduler`, packaging state, metadata, or memory layout information that later kernels and wrappers consume.
**CN:** 本段定义了`Sm100MlaIndividualTileScheduler`等内容，用于封装后续内核与封装层会使用的状态、元数据或内存布局信息。

### Lines 45-66: Types and data layout
```cpp
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
```
**EN:** This section defines `Params`, `ProblemShape`, `ClusterShape`, packaging state, metadata, or memory layout information that later kernels and wrappers consume.
**CN:** 本段定义了`Params`、`ProblemShape`、`ClusterShape`等内容，用于封装后续内核与封装层会使用的状态、元数据或内存布局信息。

### Lines 67-88: Types and data layout
```cpp
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

////////////////////////////////////////////////////////////////////////////////

struct Sm100MlaPersistentTileScheduler {
```
**EN:** This section defines `Sm100MlaPersistentTileScheduler`, `is_valid`, `get_block_coord`, packaging state, metadata, or memory layout information that later kernels and wrappers consume.
**CN:** 本段定义了`Sm100MlaPersistentTileScheduler`、`is_valid`、`get_block_coord`等内容，用于封装后续内核与封装层会使用的状态、元数据或内存布局信息。

### Lines 89-114: Types and data layout
```cpp
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
```
**EN:** This section defines `Params`, `ProblemShape`, `ClusterShape`, packaging state, metadata, or memory layout information that later kernels and wrappers consume.
**CN:** 本段定义了`Params`、`ProblemShape`、`ClusterShape`等内容，用于封装后续内核与封装层会使用的状态、元数据或内存布局信息。

### Lines 115-138: Local implementation details
```cpp

    CUTLASS_TRACE_HOST("to_underlying_arguments(): Setting persistent grid SM count to " << sm_count);
    hw_info.sm_count = sm_count;

    int num_m_blocks = size<0>(cluster_shape);
    int num_blocks = num_m_blocks * get<3>(problem_shape)  /* Batch */;
    num_blocks *= split_kv; /* Maximum Split KV*/

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
```
**EN:** This section fills in the local implementation details around `get_grid_shape`, `is_valid`, `CUTLASS_TRACE_HOST`, completing the behavior required by the file.
**CN:** 本段补充了`get_grid_shape`、`is_valid`、`CUTLASS_TRACE_HOST`周边的局部实现细节，以完成该文件所需的具体行为。

### Lines 139-159: Templates, aliases, and constants
```cpp

  CUTLASS_DEVICE
  auto get_block_coord() {
    using namespace cute;
    int block_decode = block_idx;
    int m_block, bidb, n_split_kv;
    params.divmod_m_block(block_decode, m_block, block_decode);
    params.divmod_b(block_decode, bidb, block_decode);
    params.divmod_split_kv(block_decode, n_split_kv, block_decode);
    return make_coord(m_block, _0{}, bidb, n_split_kv);
  }

  CUTLASS_DEVICE
  Sm100MlaPersistentTileScheduler& operator++() {
    block_idx += gridDim.x;
    return *this;
  }
};

////////////////////////////////////////////////////////////////////////////////
```
**EN:** This section defines `get_block_coord`, `divmod_m_block`, `divmod_b`, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了`get_block_coord`、`divmod_m_block`、`divmod_b`等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 160-160: Local implementation details
```cpp
} // namespace cutlass::fmha::kernel
```
**EN:** This section fills in the local implementation details around the surrounding logic, completing the behavior required by the file.
**CN:** 本段补充了相关逻辑周边的局部实现细节，以完成该文件所需的具体行为。

## Key Concepts / 关键概念
- **CUTLASS integration / CUTLASS 集成**: Builds on CUTLASS/CUTE abstractions for optimized kernel generation. / 基于 CUTLASS/CUTE 抽象构建优化内核。

## Dependencies / 依赖关系
- **Internal headers / 内部头文件**: `cutlass/cutlass.h`, `cutlass/fast_math.h`, `cutlass/kernel_hardware_info.h`
- **External headers / 外部头文件**: Standard library or none / 标准库或无
- **Path context / 路径上下文**: attention / cutlass_sm100_mla / kernel / sm100_mla_tile_scheduler.hpp
