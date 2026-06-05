# epilogue_predicated_tile_iterator.h — Code Analysis / 代码分析
**Source / 源文件**: `examples/41_fused_multi_head_attention/iterators/epilogue_predicated_tile_iterator.h`
**Purpose / 用途**: This class moves epilogue fragments between registers and global memory for a row-major output tile. It keeps CUTLASS-style predicate guarding, supports optional scatter addressing for D, adds explicit `prefetch()` / `prefetch_all()`, and includes two convolution-oriented helper loads for downsample and upsample cases. / 这个类负责在寄存器片段与全局内存之间搬运 epilogue 输出 tile（行主序）。它保留了 CUTLASS 风格的谓词保护，支持可选的 D 散列写回地址（scatter），增加了显式的 `prefetch()` / `prefetch_all()`，并提供了面向卷积上下采样场景的两个辅助加载函数。
---
## Line-by-Line Analysis / 逐行分析

### Lines 1-61
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
/*! \file
  \brief Epilogue iterator that supports prefetching

  Mostly copied from "cutlass/epilogue/threadblock/predicated_tile_iterator.h"
*/

#pragma once

#include "cutlass/arch/arch.h"
#include "cutlass/arch/memory.h"
#include "cutlass/array.h"
#include "cutlass/cutlass.h"
#include "cutlass/epilogue/threadblock/output_tile_thread_map.h"
#include "cutlass/epilogue/threadblock/predicated_tile_iterator_params.h"
#include "cutlass/layout/matrix.h"
#include "cutlass/layout/tensor.h"
#include "cutlass/matrix_shape.h"
#include "cutlass/numeric_types.h"
#include "cutlass/tensor_ref.h"
#include "cutlass/transform/pitch_linear_thread_map.h"

////////////////////////////////////////////////////////////////////////////////

namespace cutlass {

////////////////////////////////////////////////////////////////////////////////

namespace epilogue {
namespace threadblock {

////////////////////////////////////////////////////////////////////////////////
```
**EN**: File header, includes, namespaces, and a note that the iterator is mostly copied from CUTLASS' standard predicated epilogue iterator.
**CN**: 文件头、依赖包含、命名空间，以及注释说明：该实现大体上复制自 CUTLASS 标准的 epilogue predicated iterator。

### Lines 62-115
```cpp

/// Tile iterator used to load and store output tile from global memory in
/// epilogue.
///
/// Satisfies: ReadableTileIterator | PredicatedTileIterator |
/// ForwardTileIterator
///
template <
    typename ThreadMap_, ///< Thread map (conept: OutputTileThreadMap)
    typename Element_, ///< Element data type
    bool ScatterD = false, ///< Scatter D operand or not
    bool UseCUDAStore = false>
class PredicatedTileIteratorPrefetch {
 public:
  using ThreadMap = ThreadMap_;
  using Shape = typename ThreadMap::Shape;

  using Element = Element_;

  using Layout = layout::RowMajor;
  using TensorRef = TensorRef<Element, Layout>;
  using ConstTensorRef = typename TensorRef::ConstTensorRef;

  using Index = typename Layout::Index;
  using LongIndex = typename Layout::LongIndex;
  using TensorCoord = MatrixCoord;

  static int const kElementsPerAccess = ThreadMap::kElementsPerAccess;
  static int const kThreads = ThreadMap::kThreads;
  static int const kIterations = ThreadMap::Count::kTile;

  static_assert(
      ThreadMap::Iterations::kRow > 0,
      "ThreadMap::Iterations::kRow must be > 0");
  static_assert(
      ThreadMap::Iterations::kGroup > 0,
      "ThreadMap::Iterations::kGroup must be > 0");
  static_assert(
      ThreadMap::Iterations::kCluster > 0,
      "ThreadMap::Iterations::kCluster must be > 0");
  static_assert(
      ThreadMap::Iterations::kColumn > 0,
      "ThreadMap::Iterations::kColumn must be > 0");

  /// Fragment object
  using Fragment = Array<
      Element,
      ThreadMap::Iterations::kColumn * ThreadMap::Iterations::kRow *
          ThreadMap::Iterations::kGroup * ThreadMap::Iterations::kCluster *
          ThreadMap::kElementsPerAccess>;

  /// Memory access size
  using AccessType = AlignedArray<Element, ThreadMap::kElementsPerAccess>;
```
**EN**: Class template declaration, major aliases, static constants, and fragment/access vector definitions.
**CN**: 类模板声明、主要类型别名、静态常量，以及片段/向量访问类型定义。

### Lines 116-167
```cpp
  //
  // Parameters struct
  //

  /// Uses a non-template class
  struct Params : PredicatedTileIteratorParams {
    using Base = PredicatedTileIteratorParams;

    CUTLASS_HOST_DEVICE
    Params() {}

    CUTLASS_HOST_DEVICE
    Params(Layout const& layout)
        : PredicatedTileIteratorParams(
              layout.stride(0) * int(sizeof(AccessType)) / kElementsPerAccess,
              make_OutputTileThreadMapDesc<ThreadMap>()) {}

    CUTLASS_HOST_DEVICE
    Params(Base const& base) : Base(base) {}
  };

  /// Mask object
  struct Mask {
    static int const kCount = ThreadMap::Iterations::kColumn;

    /// Predicate state
    bool predicates[kCount];

    //
    // Mask
    //
    CUTLASS_HOST_DEVICE
    Mask() {
      enable();
    }

    ///< Efficiently disables all accesses guarded by mask
    CUTLASS_HOST_DEVICE void clear() {
      CUTLASS_PRAGMA_UNROLL
      for (int i = 0; i < kCount; ++i) {
        predicates[i] = false;
      }
    }

    ///< CUTLASS_HOST_DEVICE enables all accesses guarded by mask
    CUTLASS_DEVICE void enable() {
      CUTLASS_PRAGMA_UNROLL
      for (int i = 0; i < kCount; ++i) {
        predicates[i] = true;
      }
    }
  };
```
**EN**: `Params` wrapper and `Mask` helper. `Params` converts layout stride plus thread-map metadata into byte increments; `Mask` can clear or enable all column predicates.
**CN**: `Params` 包装层与 `Mask` 辅助结构。`Params` 将布局 stride 和 thread map 元数据转换成字节级增量；`Mask` 支持一次性清空或启用全部列谓词。

### Lines 168-221
```cpp

 private:
  //
  // Data members
  //

  /// Parameters structure containing reference and precomputed state.
  PredicatedTileIteratorParams params_;

  /// Byte-level pointer
  uint8_t* byte_pointer_;

  /// Array of boolean values to contain steady-state predicates
  Mask mask_;

  /// Extent of the matrix tile in rows
  Index extent_row_;

  /// Extent of the matrix tile in rows
  Index extent_column_;

  /// A thread's starting row position (assuming steady-state predicates have
  /// been computed)
  Index thread_start_row_;

  /// A thread's starting column
  Index thread_start_column_;

  /// Internal state counter
  int state_[3];

  /// Scatter indices
  int const* indices_;

  //
  // Static asserts about internal strides
  //

  static_assert(sizeof(extent_row_) == 4, "Expected 32b extents");
  static_assert(sizeof(thread_start_row_) == 4, "Expected 32b extents");
  static_assert(
      sizeof(PredicatedTileIteratorParams::stride) == 8,
      "Expected 64b strides");

 private:
  //
  // Methods
  //

 public:
  //
  // Methods
  //
```
**EN**: Private state and static assertions. The code assumes 32-bit extents and 64-bit strides, which matches CUTLASS iterator parameter conventions.
**CN**: 私有状态与静态断言。代码假定 extent 为 32 位、stride 为 64 位，这与 CUTLASS 迭代器参数约定一致。

### Lines 222-272
```cpp
  /// Constructor
  CUTLASS_DEVICE
  PredicatedTileIteratorPrefetch(
      PredicatedTileIteratorParams const& params,
      Element* pointer,
      TensorCoord extent,
      int thread_idx,
      TensorCoord threadblock_offset = TensorCoord(),
      int const* indices = nullptr)
      : params_(params), indices_(indices) {
    TensorCoord thread_offset =
        ThreadMap::initial_offset(thread_idx) + threadblock_offset;

    extent_row_ = extent.row();
    extent_column_ = extent.column();

    thread_start_row_ = thread_offset.row();
    thread_start_column_ = thread_offset.column();

    // Initialize predicates
    CUTLASS_PRAGMA_UNROLL
    for (int c = 0; c < ThreadMap::Iterations::kColumn; ++c) {
      mask_.predicates[c] =
          ((thread_offset.column() + ThreadMap::Delta::kColumn * c) <
           extent.column());
    }

    // Null pointer performs no accesses
    if (!pointer) {
      mask_.clear();
    }

    if (ScatterD && !indices) {
      mask_.clear();
    }

    // Initialize pointer
    byte_pointer_ = reinterpret_cast<uint8_t*>(pointer) +
        LongIndex(thread_offset.row()) * LongIndex(params_.stride) +
        LongIndex(thread_offset.column()) * sizeof(AccessType) /
            kElementsPerAccess;

    if (ScatterD) {
      byte_pointer_ = reinterpret_cast<uint8_t*>(pointer) +
          LongIndex(thread_offset.column()) * sizeof(AccessType) /
              kElementsPerAccess;
    }

    // Initialize internal state counter
    state_[0] = state_[1] = state_[2] = 0;
  }
```
**EN**: Constructor. It computes `thread_offset = ThreadMap::initial_offset(thread_idx) + threadblock_offset`, caches the thread start row/column, precomputes steady-state column predicates, and initializes the byte pointer either as linear row-major or scatter-aware column-only base.
**CN**: 构造函数。它先计算 `thread_offset = ThreadMap::initial_offset(thread_idx) + threadblock_offset`，缓存线程起始行列位置，预计算稳态列谓词，并初始化字节指针：普通路径按行主序线性定位，scatter 路径则只保留列基址、把行定位推迟到后续索引计算。

### Lines 273-278
```cpp

  /// Adds a pointer offset in units of Element
  CUTLASS_HOST_DEVICE
  void add_pointer_offset(LongIndex pointer_offset) {
    byte_pointer_ += pointer_offset * sizeof_bits<Element>::value / 8;
  }
```
**EN**: `add_pointer_offset()` applies a raw element-count offset to the byte pointer.
**CN**: `add_pointer_offset()` 以元素数为单位，对底层字节指针做原始偏移。

### Lines 279-335
```cpp

  CUTLASS_DEVICE
  void prefetch_all() {
    CUTLASS_PRAGMA_UNROLL
    for (int iter = 0; iter < kIterations; ++iter) {
      prefetch();
      ++(*this);
    }
  }

  CUTLASS_DEVICE
  void prefetch() {
    uint8_t* byte_pointer = byte_pointer_;

    CUTLASS_PRAGMA_UNROLL
    for (int cluster = 0; cluster < ThreadMap::Iterations::kCluster;
         ++cluster) {
      CUTLASS_PRAGMA_UNROLL
      for (int group = 0; group < ThreadMap::Iterations::kGroup; ++group) {
        CUTLASS_PRAGMA_UNROLL
        for (int row = 0; row < ThreadMap::Iterations::kRow; ++row) {
          int row_offset = row * ThreadMap::Delta::kRow +
              group * ThreadMap::Delta::kGroup +
              cluster * ThreadMap::Delta::kCluster;

          AccessType* memory_pointer =
              reinterpret_cast<AccessType*>(byte_pointer);

          CUTLASS_PRAGMA_UNROLL
          for (int column = 0; column < ThreadMap::Iterations::kColumn;
               ++column) {
            // on windows using unsigned long here gives the error
            // error: asm operand type size(4) does not match
            // type/size implied by constraint 'l'
            uint64_t addr = (uint64_t)((void*)&memory_pointer
                                           [column * ThreadMap::Delta::kColumn /
                                            kElementsPerAccess]);
            asm volatile("prefetch.global.L1 [ %1 ];" : "=l"(addr) : "l"(addr));
          }

          if (row + 1 < ThreadMap::Iterations::kRow) {
            if (!ScatterD) {
              byte_pointer += params_.increment_row;
            }
          }
        }

        if (group + 1 < ThreadMap::Iterations::kGroup) {
          byte_pointer += params_.increment_group;
        }
      }

      if (cluster + 1 < ThreadMap::Iterations::kCluster) {
        byte_pointer += params_.increment_cluster;
      }
    }
  }
```
**EN**: `prefetch_all()` and `prefetch()`. They walk the same cluster/group/row structure as normal accesses but issue `prefetch.global.L1` on each column access location.
**CN**: `prefetch_all()` 与 `prefetch()`。它们沿着与正常访问相同的 cluster/group/row 结构遍历，并对每个列访问位置发出 `prefetch.global.L1` 指令。

### Lines 336-408
```cpp

  /// Loads a fragment from memory
  CUTLASS_DEVICE
  void load_with_byte_offset(Fragment& frag, int64_t byte_offset) const {
    uint8_t* byte_pointer = byte_pointer_;
    AccessType* frag_ptr = reinterpret_cast<AccessType*>(&frag);

    CUTLASS_PRAGMA_UNROLL
    for (int cluster = 0; cluster < ThreadMap::Iterations::kCluster;
         ++cluster) {
      CUTLASS_PRAGMA_UNROLL
      for (int group = 0; group < ThreadMap::Iterations::kGroup; ++group) {
        CUTLASS_PRAGMA_UNROLL
        for (int row = 0; row < ThreadMap::Iterations::kRow; ++row) {
          int frag_row_idx =
              (row +
               ThreadMap::Iterations::kRow *
                   (group + ThreadMap::Iterations::kGroup * cluster));

          int row_offset = row * ThreadMap::Delta::kRow +
              group * ThreadMap::Delta::kGroup +
              cluster * ThreadMap::Delta::kCluster;

          bool row_guard = ((row_offset + thread_start_row_) < extent_row_);

          AccessType* memory_pointer =
              reinterpret_cast<AccessType*>(byte_pointer + byte_offset);

          if (ScatterD && row_guard) {
            assert(indices_);

            memory_pointer = reinterpret_cast<AccessType*>(
                byte_pointer + byte_offset +
                LongIndex(indices_[row_offset + thread_start_row_]) *
                    LongIndex(params_.stride));
          }

          CUTLASS_PRAGMA_UNROLL
          for (int column = 0; column < ThreadMap::Iterations::kColumn;
               ++column) {
            bool guard = row_guard && mask_.predicates[column];

            cutlass::arch::global_load<AccessType, sizeof(AccessType)>(
                frag_ptr
                    [frag_row_idx * ThreadMap::Iterations::kColumn + column],
                (void*)&memory_pointer
                    [column * ThreadMap::Delta::kColumn / kElementsPerAccess],
                guard);
          }

          if (row + 1 < ThreadMap::Iterations::kRow) {
            if (!ScatterD) {
              byte_pointer += params_.increment_row;
            }
          }
        }

        if (group + 1 < ThreadMap::Iterations::kGroup) {
          byte_pointer += params_.increment_group;
        }
      }

      if (cluster + 1 < ThreadMap::Iterations::kCluster) {
        byte_pointer += params_.increment_cluster;
      }
    }
  }

  /// Loads a fragment from memory
  CUTLASS_DEVICE
  void load(Fragment& frag) const {
    load_with_byte_offset(frag, 0);
  }
```
**EN**: Regular loads. The iterator computes fragment row indices, row offsets, row guards, optional scatter row addresses, and finally guarded `global_load` operations per column vector.
**CN**: 常规加载。迭代器会计算 fragment 行索引、行偏移、行 guard、可选的 scatter 行地址，并对每个列向量执行带 guard 的 `global_load`。

### Lines 409-491
```cpp

  /// Stores a fragment to memory
  CUTLASS_DEVICE
  void store_with_byte_offset(Fragment const& frag, int64_t byte_offset) const {
    uint8_t* byte_pointer = byte_pointer_;
    AccessType const* frag_ptr = reinterpret_cast<AccessType const*>(&frag);

    CUTLASS_PRAGMA_UNROLL
    for (int cluster = 0; cluster < ThreadMap::Iterations::kCluster;
         ++cluster) {
      CUTLASS_PRAGMA_UNROLL
      for (int group = 0; group < ThreadMap::Iterations::kGroup; ++group) {
        CUTLASS_PRAGMA_UNROLL
        for (int row = 0; row < ThreadMap::Iterations::kRow; ++row) {
          int frag_row_idx =
              (row +
               ThreadMap::Iterations::kRow *
                   (group + ThreadMap::Iterations::kGroup * cluster));

          int row_offset = row * ThreadMap::Delta::kRow +
              group * ThreadMap::Delta::kGroup +
              cluster * ThreadMap::Delta::kCluster;

          bool row_guard = ((row_offset + thread_start_row_) < extent_row_);

          AccessType* memory_pointer =
              reinterpret_cast<AccessType*>(byte_pointer + byte_offset);

          if (ScatterD && row_guard) {
            assert(indices_);

            memory_pointer = reinterpret_cast<AccessType*>(
                byte_pointer + byte_offset +
                LongIndex(indices_[row_offset + thread_start_row_]) *
                    LongIndex(params_.stride));
          }

          CUTLASS_PRAGMA_UNROLL
          for (int column = 0; column < ThreadMap::Iterations::kColumn;
               ++column) {
            bool guard = row_guard && mask_.predicates[column];

            if (UseCUDAStore) {
              if (guard) {
                memory_pointer
                    [column * ThreadMap::Delta::kColumn / kElementsPerAccess] =
                        frag_ptr
                            [frag_row_idx * ThreadMap::Iterations::kColumn +
                             column];
              }
            } else {
              cutlass::arch::global_store<AccessType, sizeof(AccessType)>(
                  frag_ptr
                      [frag_row_idx * ThreadMap::Iterations::kColumn + column],
                  (void*)&memory_pointer
                      [column * ThreadMap::Delta::kColumn / kElementsPerAccess],
                  guard);
            }
          }

          if (row + 1 < ThreadMap::Iterations::kRow) {
            if (!ScatterD) {
              byte_pointer += params_.increment_row;
            }
          }
        }

        if (group + 1 < ThreadMap::Iterations::kGroup) {
          byte_pointer += params_.increment_group;
        }
      }

      if (cluster + 1 < ThreadMap::Iterations::kCluster) {
        byte_pointer += params_.increment_cluster;
      }
    }
  }

  /// Stores a fragment to memory
  CUTLASS_DEVICE
  void store(Fragment const& frag) const {
    store_with_byte_offset(frag, 0);
  }
```
**EN**: Regular stores. The control flow mirrors load logic, but can either write directly with native CUDA stores or use predicated `global_store`.
**CN**: 常规存储。控制流与加载基本镜像对称，但可以选择直接使用 CUDA 原生写回，或者使用带谓词的 `global_store`。

### Lines 492-566
```cpp

  /// Loads a fragment from memory
  CUTLASS_DEVICE
  void downsample_load_with_byte_offset(
      Fragment& frag,
      int64_t byte_offset,
      int convolution_P,
      int convolution_Q,
      int add_P,
      int add_Q,
      int problem_N) const {
    uint8_t* byte_pointer = byte_pointer_;
    AccessType* frag_ptr = reinterpret_cast<AccessType*>(&frag);

    CUTLASS_PRAGMA_UNROLL
    for (int cluster = 0; cluster < ThreadMap::Iterations::kCluster;
         ++cluster) {
      CUTLASS_PRAGMA_UNROLL
      for (int group = 0; group < ThreadMap::Iterations::kGroup; ++group) {
        CUTLASS_PRAGMA_UNROLL
        for (int row = 0; row < ThreadMap::Iterations::kRow; ++row) {
          int frag_row_idx =
              (row +
               ThreadMap::Iterations::kRow *
                   (group + ThreadMap::Iterations::kGroup * cluster));

          int row_offset = row * ThreadMap::Delta::kRow +
              group * ThreadMap::Delta::kGroup +
              cluster * ThreadMap::Delta::kCluster;

          bool row_guard = ((row_offset + thread_start_row_) < extent_row_);

          int output_row = row_offset + thread_start_row_;
          int output_N = output_row / (convolution_P * convolution_Q);
          int output_PQ = output_row % (convolution_P * convolution_Q);
          int output_P = output_PQ / convolution_Q;
          int output_Q = output_PQ % convolution_Q;

          int input_row = output_N * 2 * convolution_P * 2 * convolution_Q +
              (2 * output_P + add_P) * 2 * convolution_Q + 2 * output_Q + add_Q;

          int64_t byte_offset =
              (input_row - output_row) * problem_N * sizeof(float);

          AccessType* memory_pointer =
              reinterpret_cast<AccessType*>(byte_pointer + byte_offset);

          CUTLASS_PRAGMA_UNROLL
          for (int column = 0; column < ThreadMap::Iterations::kColumn;
               ++column) {
            bool guard = row_guard && mask_.predicates[column];

            cutlass::arch::global_load<AccessType, sizeof(AccessType)>(
                frag_ptr
                    [frag_row_idx * ThreadMap::Iterations::kColumn + column],
                (void*)&memory_pointer
                    [column * ThreadMap::Delta::kColumn / kElementsPerAccess],
                guard);
          }

          if (row + 1 < ThreadMap::Iterations::kRow) {
            byte_pointer += params_.increment_row;
          }
        }

        if (group + 1 < ThreadMap::Iterations::kGroup) {
          byte_pointer += params_.increment_group;
        }
      }

      if (cluster + 1 < ThreadMap::Iterations::kCluster) {
        byte_pointer += params_.increment_cluster;
      }
    }
  }
```
**EN**: `downsample_load_with_byte_offset()`. It remaps each logical output row to a larger input image by expanding `N/P/Q` coordinates and adding per-row byte displacement before loading.
**CN**: `downsample_load_with_byte_offset()`。它把逻辑输出行重新映射回更大的输入图像：先展开 `N/P/Q` 坐标，再为每一行追加额外的字节位移后执行加载。

### Lines 567-648
```cpp

  /// Loads a fragment from memory
  CUTLASS_DEVICE
  void upsample_load_with_byte_offset(
      Fragment& frag,
      int64_t byte_offset,
      int convolution_P,
      int convolution_Q,
      int add_P,
      int add_Q,
      int problem_N) const {
    uint8_t* byte_pointer = byte_pointer_;
    AccessType* frag_ptr = reinterpret_cast<AccessType*>(&frag);

    CUTLASS_PRAGMA_UNROLL
    for (int cluster = 0; cluster < ThreadMap::Iterations::kCluster;
         ++cluster) {
      CUTLASS_PRAGMA_UNROLL
      for (int group = 0; group < ThreadMap::Iterations::kGroup; ++group) {
        CUTLASS_PRAGMA_UNROLL
        for (int row = 0; row < ThreadMap::Iterations::kRow; ++row) {
          int frag_row_idx =
              (row +
               ThreadMap::Iterations::kRow *
                   (group + ThreadMap::Iterations::kGroup * cluster));

          int row_offset = row * ThreadMap::Delta::kRow +
              group * ThreadMap::Delta::kGroup +
              cluster * ThreadMap::Delta::kCluster;

          bool row_guard = ((row_offset + thread_start_row_) < extent_row_);

          int output_row = row_offset + thread_start_row_;
          int output_N = output_row / (convolution_P * convolution_Q);
          int output_PQ = output_row % (convolution_P * convolution_Q);
          int output_P = output_PQ / convolution_Q;
          int output_Q = output_PQ % convolution_Q;
          int row_add_P = add_P;
          int row_add_Q = add_Q;
          if (output_P > convolution_P - 2)
            row_add_P = 0;
          if (output_Q > convolution_Q - 2)
            row_add_Q = 0;

          int input_row = output_N * (convolution_P / 2) * (convolution_Q / 2) +
              ((output_P + row_add_P) / 2) * (convolution_Q / 2) +
              (output_Q + row_add_Q) / 2;

          int64_t byte_offset =
              (input_row - output_row) * problem_N * sizeof(float);

          AccessType* memory_pointer =
              reinterpret_cast<AccessType*>(byte_pointer + byte_offset);

          CUTLASS_PRAGMA_UNROLL
          for (int column = 0; column < ThreadMap::Iterations::kColumn;
               ++column) {
            bool guard = row_guard && mask_.predicates[column];

            cutlass::arch::global_load<AccessType, sizeof(AccessType)>(
                frag_ptr
                    [frag_row_idx * ThreadMap::Iterations::kColumn + column],
                (void*)&memory_pointer
                    [column * ThreadMap::Delta::kColumn / kElementsPerAccess],
                guard);
          }

          if (row + 1 < ThreadMap::Iterations::kRow) {
            byte_pointer += params_.increment_row;
          }
        }

        if (group + 1 < ThreadMap::Iterations::kGroup) {
          byte_pointer += params_.increment_group;
        }
      }

      if (cluster + 1 < ThreadMap::Iterations::kCluster) {
        byte_pointer += params_.increment_cluster;
      }
    }
  }
```
**EN**: `upsample_load_with_byte_offset()`. It maps an output row back into a reduced-resolution input domain, with boundary corrections when the last `P/Q` positions cannot add the requested offset.
**CN**: `upsample_load_with_byte_offset()`。它将输出行映射到较低分辨率输入域；当 `P/Q` 落在尾部边界、无法继续加偏移时，会先修正 `add_P/add_Q` 再计算源行。

### Lines 649-677
```cpp

  CUTLASS_DEVICE
  MatrixCoord thread_start() const {
    return MatrixCoord(thread_start_row_, thread_start_column_);
  }

  /// Need to get the thread start row from the tile iterator
  CUTLASS_DEVICE
  int32_t thread_start_row() const {
    return thread_start_row_;
  }

  /// Need to get the thread start row from the tile iterator
  CUTLASS_DEVICE
  int32_t thread_start_column() const {
    return thread_start_column_;
  }

  /// Extent of the matrix in rows
  CUTLASS_DEVICE
  Index extent_row() const {
    return extent_row_;
  }

  /// Extent of the matrix in columns
  CUTLASS_DEVICE
  Index extent_column() const {
    return extent_column_;
  }
```
**EN**: Lightweight inspectors exposing thread start coordinates and extents.
**CN**: 轻量级访问器，用于读取线程起始坐标与矩阵范围。

### Lines 678-715
```cpp

  /// Advances to the next position to load or store
  CUTLASS_HOST_DEVICE
  PredicatedTileIteratorPrefetch& operator++() {
    ++state_[0];

    if (!ScatterD) {
      byte_pointer_ += params_.advance_row;
    }

    thread_start_row_ += ThreadMap::Shape::kRow;

    if (state_[0] == ThreadMap::Count::kRow) {
      state_[0] = 0;
      ++state_[1];
      byte_pointer_ += params_.advance_group;

      thread_start_row_ += (ThreadMap::Shape::kGroup - 1) *
          ThreadMap::Shape::kRow * ThreadMap::Count::kRow;

      if (state_[1] == ThreadMap::Count::kGroup) {
        state_[1] = 0;
        ++state_[2];
        byte_pointer_ += params_.advance_cluster;

        thread_start_row_ += ThreadMap::Count::kGroup *
            ThreadMap::Shape::kGroup * ThreadMap::Count::kRow *
            ThreadMap::Shape::kRow;

        if (state_[2] == ThreadMap::Count::kCluster) {
          state_[2] = 0;
          byte_pointer_ += params_.advance_tile;
        }
      }
    }

    return *this;
  }
```
**EN**: `operator++()` advances the iterator over epilogue tile positions using a row/group/cluster state machine and the precomputed byte advances from `params_`.
**CN**: `operator++()` 通过 row/group/cluster 状态机推进 epilogue tile 位置，并使用 `params_` 中预计算好的字节前进量。

### Lines 716-735
```cpp

  ///< Efficiently disables all accesses guarded by mask
  CUTLASS_DEVICE void clear_mask() {
    mask_.clear();
  }

  ///< Efficiently enables all accesses guarded by mask
  CUTLASS_DEVICE void enable_mask() {
    mask_.enable();
  }

  ///< Sets the mask
  CUTLASS_DEVICE void get_mask(Mask& mask) const {
    mask = mask_;
  }

  ///< Sets the mask
  CUTLASS_DEVICE void set_mask(Mask const& mask) {
    mask_ = mask;
  }
```
**EN**: Mask utilities: clear, enable, copy out, and replace the stored mask.
**CN**: 掩码辅助接口：清空、启用、导出和替换当前保存的掩码。

### Lines 736-751
```cpp
};

template <typename IT>
struct MakePrefetchableIterator {
  using Iterator = PredicatedTileIteratorPrefetch<
      typename IT::ThreadMap,
      typename IT::Element>;
};

///////////////////////////////////////////////////////////////////////////////

} // namespace threadblock
} // namespace epilogue
} // namespace cutlass

////////////////////////////////////////////////////////////////////////////////
```
**EN**: Small adapter `MakePrefetchableIterator` plus namespace closing boilerplate.
**CN**: 小型适配器 `MakePrefetchableIterator` 与命名空间结束代码。

---
## Key Concepts / 关键概念
- **EN**: Template parameters are `ThreadMap_`, `Element_`, `ScatterD`, and `UseCUDAStore`. `ThreadMap` defines tile shape, per-thread iteration counts, deltas, and the initial per-thread coordinate; `ScatterD` switches row addressing from linear row stride to `indices_`; `UseCUDAStore` selects direct stores versus `cutlass::arch::global_store`.
  **CN**: 模板参数包括 `ThreadMap_`、`Element_`、`ScatterD` 和 `UseCUDAStore`。其中 `ThreadMap` 定义 tile 形状、每线程迭代次数、各维步长以及线程初始坐标；`ScatterD` 控制行地址计算是普通线性步进还是通过 `indices_` 做散列；`UseCUDAStore` 决定使用直接写回还是 `cutlass::arch::global_store`。
- **EN**: `Layout` is fixed to `layout::RowMajor`, `Fragment` packs one thread's entire tile slice, `AccessType` is an aligned vector of `ThreadMap::kElementsPerAccess`, and `Params` inherits `PredicatedTileIteratorParams` to precompute byte increments and tile advances.
  **CN**: `Layout` 固定为 `layout::RowMajor`；`Fragment` 打包一个线程负责的完整片段；`AccessType` 是长度为 `ThreadMap::kElementsPerAccess` 的对齐向量；`Params` 继承 `PredicatedTileIteratorParams`，用于预先计算按字节计的行/组/簇增量与 tile 前进量。
- **EN**: Core aliases are `Shape`, `TensorRef`, `TensorCoord`, `Fragment`, and `AccessType`. The iterator stores `params_`, a byte pointer `byte_pointer_`, a column predicate mask `mask_`, matrix extents `extent_row_` / `extent_column_`, thread-local origin `thread_start_row_` / `thread_start_column_`, three traversal counters `state_[3]`, and optional scatter indices `indices_`.
  **CN**: 核心别名包括 `Shape`、`TensorRef`、`TensorCoord`、`Fragment` 和 `AccessType`。运行时状态包含 `params_`、字节指针 `byte_pointer_`、列谓词掩码 `mask_`、矩阵范围 `extent_row_` / `extent_column_`、线程局部起点 `thread_start_row_` / `thread_start_column_`、三个遍历状态计数器 `state_[3]`，以及可选的散列索引 `indices_`。
- **EN**: `Mask` contains one boolean per column iteration only. Row legality is checked dynamically during load/store, so the final guard is a conjunction of a row guard and a precomputed column predicate.
  **CN**: `Mask` 只保存“列迭代”这一维的布尔谓词。行是否合法并不预存，而是在 load/store 时动态判断，因此最终 guard 是“行 guard”和“列谓词”的合取。
## Dependencies / 依赖项
- `cutlass/arch/arch.h` — Direct include used by this file / 本文件直接包含并依赖的头文件
- `cutlass/arch/memory.h` — Direct include used by this file / 本文件直接包含并依赖的头文件
- `cutlass/array.h` — Direct include used by this file / 本文件直接包含并依赖的头文件
- `cutlass/cutlass.h` — Direct include used by this file / 本文件直接包含并依赖的头文件
- `cutlass/epilogue/threadblock/output_tile_thread_map.h` — Direct include used by this file / 本文件直接包含并依赖的头文件
- `cutlass/epilogue/threadblock/predicated_tile_iterator_params.h` — Direct include used by this file / 本文件直接包含并依赖的头文件
- `cutlass/layout/matrix.h` — Direct include used by this file / 本文件直接包含并依赖的头文件
- `cutlass/layout/tensor.h` — Direct include used by this file / 本文件直接包含并依赖的头文件
- `cutlass/matrix_shape.h` — Direct include used by this file / 本文件直接包含并依赖的头文件
- `cutlass/numeric_types.h` — Direct include used by this file / 本文件直接包含并依赖的头文件
- `cutlass/tensor_ref.h` — Direct include used by this file / 本文件直接包含并依赖的头文件
- `cutlass/transform/pitch_linear_thread_map.h` — Direct include used by this file / 本文件直接包含并依赖的头文件
