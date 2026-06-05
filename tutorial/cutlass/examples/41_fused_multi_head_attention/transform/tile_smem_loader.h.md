# tile_smem_loader.h — Code Analysis / 代码分析
**Source / 源文件**: `examples/41_fused_multi_head_attention/transform/tile_smem_loader.h`
**Purpose / 用途**: A tiny reusable loader that copies one tile from global memory to shared memory using CUTLASS threadblock iterators. / 这是一个小型可复用 loader，利用 CUTLASS 的线程块 iterator 把一个 tile 从全局内存复制到共享内存。
---
## Line-by-Line Analysis / 逐行分析

### Lines 1-43 / 第 1-43 行
```cpp
/***************************************************************************************************
 * Copyright (c) 2017 - 2026 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
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

#include <cutlass/cutlass.h>
#include "cutlass/aligned_buffer.h"
#include "cutlass/array.h"
#include "cutlass/layout/matrix.h"
#include "cutlass/layout/pitch_linear.h"
#include "cutlass/numeric_types.h"
#include "cutlass/transform/pitch_linear_thread_map.h"
#include "cutlass/transform/threadblock/predicated_tile_iterator.h"
#include "cutlass/transform/threadblock/regular_tile_iterator.h"
```
**EN**: License, includes, and namespace setup.
**CN**: 许可证、依赖头文件与命名空间准备。

### Lines 44-90 / 第 44-90 行
```cpp
template <
    typename scalar_t, // scalar type
    typename ThreadblockTileShape, // size of tile to load
    int Threads, // number of participating threads
    int ElementsPerAccess> // thread access width in elements
class TileSmemLoader {
 public:
  using SmemTile =
      cutlass::AlignedBuffer<scalar_t, ThreadblockTileShape::kCount>;

  using ThreadMap = cutlass::transform::PitchLinearStripminedThreadMap<
      cutlass::layout::PitchLinearShape<
          ThreadblockTileShape::kColumn, // contiguous
          ThreadblockTileShape::kRow>, // strided
      Threads, // Threads
      ElementsPerAccess>; // ElementsPerAccess

  using GmemTileIterator =
      cutlass::transform::threadblock::PredicatedTileIterator<
          ThreadblockTileShape, // Shape
          scalar_t, // Element
          cutlass::layout::RowMajor, // Layout
          0, // AdvanceRank
          ThreadMap>; // ThreadMap

  using SmemTileIterator = cutlass::transform::threadblock::RegularTileIterator<
      ThreadblockTileShape, // Shape
      scalar_t, // Element
      cutlass::layout::RowMajor, // Layout
      0, // AdvanceRank
      ThreadMap>; // ThreadMap

  using Fragment = typename GmemTileIterator::Fragment;

  /// load a tile from global memory into shared memory
  CUTLASS_DEVICE
  static void load(
      GmemTileIterator tile_load_iter,
      SmemTileIterator tile_store_iter) {
    Fragment tb_frag;
    tb_frag.clear();
    tile_load_iter.load(tb_frag);
    tile_store_iter.store(tb_frag);

    __syncthreads();
  }
};
```
**EN**: `TileSmemLoader` declaration, iterator aliases, fragment type, and the `load()` routine.
**CN**: `TileSmemLoader` 声明、iterator 别名、fragment 类型以及 `load()` 实现。

---
## Key Concepts / 关键概念
- **EN:** FMHA depends on on-chip staging because query, key, and value tiles must be reused immediately by tensor-core MMA instructions.
  **CN:** FMHA 强依赖片上 staging，因为 query、key、value tile 需要被 Tensor Core MMA 指令立即复用。
- **EN:** The loader uses a predicated global iterator so tail tiles remain safe even when sequence lengths are not multiples of the threadblock shape.
  **CN:** 该 loader 使用带 predication 的全局 iterator，因此即使序列长度不是 threadblock 形状的整数倍，尾部 tile 也能安全处理。
- **EN:** This is a clean example of CUTLASS iterator/kernel composition: layout, tile shape, thread mapping, and access width are encoded in types rather than handwritten pointer arithmetic.
  **CN:** 这是 CUTLASS iterator / kernel 组合方式的典型例子：布局、tile 形状、线程映射和访问宽度都编码在类型中，而不是手写指针运算。
- **EN:** The FMHA pipeline uses this kind of staging utility whenever a global tensor tile must be transformed into a shared-memory view consumed by later compute stages.
  **CN:** 当 FMHA 流水中的某个全局张量 tile 需要转换成后续计算阶段可消费的共享内存视图时，就会用到这种 staging 工具。
- **EN:** Even though this file is small, it captures the memory-movement pattern that underpins fused attention performance.
  **CN:** 虽然文件很小，但它体现了支撑融合 attention 性能的核心数据搬运模式。
## Dependencies / 依赖项
- `<cutlass/cutlass.h>` — CUTLASS core definitions and portability macros / CUTLASS 核心定义与可移植性宏
- `"cutlass/aligned_buffer.h"` — aligned shared-memory/storage buffers / 对齐的共享内存 / 存储缓冲区
- `"cutlass/array.h"` — fixed-size array containers used for fragments / 用于 fragment 的定长数组容器
- `"cutlass/layout/matrix.h"` — matrix layout tags and coordinate helpers / 矩阵布局标签与坐标辅助类型
- `"cutlass/layout/pitch_linear.h"` — pitch-linear layout support for threadblock iterators / 面向 threadblock iterator 的 pitch-linear 布局支持
- `"cutlass/numeric_types.h"` — CUTLASS scalar and numeric utility types / CUTLASS 标量与数值工具类型
- `"cutlass/transform/pitch_linear_thread_map.h"` — thread maps that distribute vector accesses across threads / 在线程间分配向量访问的 thread map
- `"cutlass/transform/threadblock/predicated_tile_iterator.h"` — predicated iterators for safe edge-tile loads/stores / 用于边界 tile 安全读写的 predicated iterator
- `"cutlass/transform/threadblock/regular_tile_iterator.h"` — threadblock iterators for regular shared-memory tile access / 用于常规共享内存 tile 访问的 threadblock iterator
