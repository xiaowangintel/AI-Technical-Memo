# predicated_tile_iterator_residual_last.h — Code Analysis / 代码分析
**Source / 源文件**: `examples/41_fused_multi_head_attention/iterators/predicated_tile_iterator_residual_last.h`
**Purpose / 用途**: This file builds fragment-level load/store tile iterators on top of the lower-level residual-last access iterators. It is the interface most GEMM-style pipelines use directly. Its job is to turn many guarded vector accesses into one logical `Fragment` load or store while preserving the same residual-last tail-tile control policy. As in the access-iterator file, only two families are “core” wrappers with explicit fragment loops; the remaining specializations mostly remap layouts and forward behavior. / 这个文件在更底层的 residual-last access iterator 之上构建了 fragment 级的 load/store tile iterator，是 GEMM 风格流水线更直接使用的接口层。 它的职责是把许多受谓词保护的向量访问组合成一次逻辑上的 `Fragment` 加载或存储，同时保留相同的 residual-last 尾块控制策略。 和 access-iterator 文件类似，真正带有显式 fragment 循环的“核心”只有两类；其余特化大多只是做 layout 重映射并转发行为。
---
## Line-by-Line Analysis / 逐行分析

### Lines 1-48
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
    \brief Templates implementing loading of tiles from pitch-linear rank=2
   tensors.

    This iterator uses masks to guard out-of-bounds accesses. The first tile
   this iterator visits maybe partial, then the remaining tiles are complete.
   So, we only need to compute the predicates twice, once before the first tile
   and once for the remaining full tiles which can share the same predicates.

    A precomputed "Params" object minimizes the amount of state that must be
   stored in registers, and integer addition is used to advance the pointer
   through memory.
*/

#pragma once

#include "cutlass/arch/memory.h"
#include "cutlass/transform/threadblock/predicated_tile_access_iterator.h"
```
**EN**: L1-L48 — File header and includes; the comments explain the two-phase predicate idea inherited from the underlying access iterator.
**CN**: `L1-L48` —— 文件头与依赖；注释说明了从底层 access iterator 继承而来的“两阶段谓词”思想。

### Lines 49-150
```cpp

////////////////////////////////////////////////////////////////////////////////

namespace cutlass {
namespace transform {
namespace threadblock {

////////////////////////////////////////////////////////////////////////////////

/// PredicatedTileIteratorResidualLast
///
/// Satisfies: ForwardTileIteratorConcept |
///            ReadableContiguousTileIteratorConcept |
///            WriteableContiguousTileIteratorConcept |
///            MaskedTileIteratorConcept
///
/// Regular tile iterator using a precomputed control structure to minimize
/// register liveness and integer arithmetic.
///
/// Layout is assumed to be invariant at the time the precomputed "Params"
/// object is constructed.
///
/// Base pointer and tensor extents may be specified at the time the iterator is
/// constructed. Subsequently, they are assumed to be immutable.
///
/// Adding a logical coordinate offset may be performed at the time the iterator
/// is constructed. Subsequent additions to logical coordinate offset may be
/// performed but are relatively expensive.
///
/// Visitation order is intended to first visit a "residual" tile that may be
/// partially full in both the advance dimension and the steady-state dimension.
/// This is assumed to be the last tile in the iteration sequence. Advancing an
/// iterator that has just been constructed moves to the first tile that is full
/// in the advance dimension and recomputes predicates. Subsequent accesses may
/// be performed without updating internal predicates and are efficient in terms
/// of live register state and pointer arithmetic instructions.
///
/// To be efficient, this assumes the iterator will be dereferenced and advanced
/// at least once outside any looping structure to minimize integer arithmetic.
///
/// Accesses out of bounds are safe so long as `clear_mask()` is called prior to
/// dereferencing the iterator.
///
///
/// Example:
///
/// An efficient pipeline structure may be constructed as follows:
///
// template <typename Iterator>
// __global__ void kernel(
//   typename Iterator::Params params,
//   typename Iterator::Element *ptr,
//   TensorCoord extent) {
//
//   typename Iterator::Fragment fragment;
//
//   TensorCoord threadblock_offset(0, 0);
//
//   Iterator iter(params, ptr, extent, threadIdx.x, threadblock_offsets);
//
//
//   fragment = *iter;        // load "residue" tile first
//   ++iter;                  // advance to first "steady state" tile and update
//   internal masks
//
//
//   #pragma unroll
//   for (int i = Remaining - 1; i >= 0; --i) {
//
//     f(fragment);
//
//     if (!i) {
//       iter.clear_mask();   // light-weight operation to clear masks -
//       subsequent loads become NO-OPs.
//     }
//
//     fragment = *iter;      // load tile during "steady state" phase
//     ++iter;                // advance to next tile - lightweight due to
//     steady-state masks
//   }
// }
//
// void host(TensorView<Element, 2, layout::PitchLinear> view) {
//
//   using Iterator =
//   transform::threadblock::PredicatedTileIteratorResidualLast;
//
//   typename Iterator::Params params(view.layout());
//
//   kernel<Iterator>(params, view.data());
// }
///
///
template <
    typename Shape,
    typename Element,
    typename Layout,
    int AdvanceRank,
    typename ThreadMap,
    int AccessSize = ThreadMap::kElementsPerAccess,
    bool Gather = false>
class PredicatedTileIteratorResidualLast;
```
**EN**: L52-L150 — Namespace opening, primary template declaration, and a long usage comment showing a pipelined load/advance sequence.
**CN**: `L52-L150` —— 命名空间展开、主模板声明，以及展示流水线式 load/advance 序列的长注释示例。

### Lines 151-446
```cpp

////////////////////////////////////////////////////////////////////////////////

/// Specialization of PredicatedTileIteratorResidualLast for pitch-linear data.
///
/// Satisfies: ForwardTileIteratorConcept |
///            ReadableContiguousTileIteratorConcept |
///            WriteableContiguousTileIteratorConcept |
///            MaskedTileIteratorConcept
///
template <
    typename Shape_,
    typename Element_,
    int AdvanceRank,
    typename ThreadMap_,
    int AccessSize,
    bool Gather>
class PredicatedTileIteratorResidualLast<
    Shape_,
    Element_,
    layout::PitchLinear,
    AdvanceRank,
    ThreadMap_,
    AccessSize,
    Gather> {
 public:
  static_assert(
      AdvanceRank == 0 || AdvanceRank == 1,
      "Specialization for pitch-linear iterator may advance along the "
      "contiguous(rank=0) or strided(rank=1) dimension.");

  using Shape = Shape_;
  using Element = Element_;
  using Layout = layout::PitchLinear;
  static int const kAdvanceRank = AdvanceRank;
  using ThreadMap = ThreadMap_;

  using Index = typename Layout::Index;
  using LongIndex = typename Layout::LongIndex;

  using TensorRef = TensorRef<Element, Layout>;
  using TensorView = TensorView<Element, Layout>;
  using TensorCoord = typename Layout::TensorCoord;

  using Pointer = Element*;
  using NonConstPointer = typename platform::remove_const<Element>::type*;

  /// Type used for internal memory accesses
  using AccessType = AlignedArray<
      Element,
      AccessSize,
      (AccessSize * sizeof_bits<Element>::value / 8)>;

  /// Underlying iterator to compute the addresses
  using TileAccessIterator = PredicatedTileAccessIteratorResidualLast<
      Shape,
      Element,
      Layout,
      kAdvanceRank,
      ThreadMap,
      AccessType,
      Gather>;

  static int const kAccessesPerVector = TileAccessIterator::kAccessesPerVector;

  /// Fragment object to be loaded or stored
  using Fragment = cutlass::Array<
      Element,
      ThreadMap::Iterations::kCount * ThreadMap::kElementsPerAccess>;

  /// Predicate vector stores mask to guard accesses
  using Mask = typename TileAccessIterator::Mask;

  /// Parameters object is precomputed state and is host-constructible
  class Params {
   public:
    using Base = typename TileAccessIterator::Params::Base;

    friend PredicatedTileIteratorResidualLast;

   private:
    /// Parameters object
    typename TileAccessIterator::Params params_;

   public:
    /// Construct the Params object given a pitch-linear tensor's layout
    CUTLASS_HOST_DEVICE
    Params(Layout const& layout) : params_(layout) {}

    CUTLASS_HOST_DEVICE
    Params() {}

    CUTLASS_HOST_DEVICE
    Params(Base const& base) : params_(base) {}
  };

 private:
  /// Internal pointer type permits fast address arithmetic
  using BytePointer = char*;

 private:
  //
  // Data members
  //

  /// Data member to the tile access iterator
  TileAccessIterator address_iterator_;

 public:
  /// Constructs a TileIterator from its precomputed state, threadblock offset,
  /// and thread ID
  CUTLASS_HOST_DEVICE
  PredicatedTileIteratorResidualLast(
      /// Precomputed parameters object
      Params const& params,
      /// Pointer to start of tensor
      Pointer pointer,
      /// Extent of tensor
      TensorCoord extent,
      /// ID of each participating thread
      int thread_id,
      /// Initial offset of threadblock
      TensorCoord const& threadblock_offset,
      /// Gather indices
      int const* indices = nullptr)
      : address_iterator_(
            params.params_,
            pointer,
            extent,
            thread_id,
            threadblock_offset,
            indices) {}

  /// Construct a PredicatedTileIteratorResidualLast with zero threadblock
  /// offset
  CUTLASS_HOST_DEVICE
  PredicatedTileIteratorResidualLast(
      Params const& params, ///< Precomputed parameters object
      Pointer pointer, ///< Pointer to start of tensor
      TensorCoord extent, ///< Extent of tensor
      int thread_id ///< ID of each participating thread
      )
      : PredicatedTileIteratorResidualLast(
            params,
            pointer,
            extent,
            thread_id,
            make_Coord(0, 0)) {}

  /// Adds a pointer offset in units of Element
  CUTLASS_HOST_DEVICE
  void add_pointer_offset(LongIndex pointer_offset) {
    address_iterator_.add_pointer_offset(pointer_offset);
  }

  /// Advances to the next tile in memory.
  ///
  /// The first time this method is called, predicates are updated, and the
  /// iterator's internal pointer is reverted to the first "steady state" tile.
  /// Subsequent calls are lightweight and must only update the internal
  /// pointer.
  CUTLASS_HOST_DEVICE
  PredicatedTileIteratorResidualLast& operator++() {
    if (kAdvanceRank)
      address_iterator_.add_tile_offset({0, 1});
    else
      address_iterator_.add_tile_offset({1, 0});

    return *this;
  }

  /// Advances to the next tile in memory.
  ///
  /// The first time this method is called, predicates are updated, and the
  /// iterator's internal pointer is reverted to the first "steady state" tile.
  /// Subsequent calls are lightweight and must only update the internal
  /// pointer.
  CUTLASS_HOST_DEVICE
  PredicatedTileIteratorResidualLast operator++(int) {
    PredicatedTileIteratorResidualLast self(*this);
    operator++();
    return self;
  }

  /// Clears the predicate set efficiently
  CUTLASS_HOST_DEVICE
  void clear_mask(bool enable = true) {
    address_iterator_.clear_mask(enable);
  }

  CUTLASS_HOST_DEVICE
  void set_residual_tile(bool enable) {
    address_iterator_.set_residual_tile(enable);
  }

  /// Clears the predicate set efficiently
  CUTLASS_HOST_DEVICE
  void enable_mask() {
    address_iterator_.enable_mask();
  }

  /// Sets the predicate mask, overriding value stored in predicate iterator
  CUTLASS_HOST_DEVICE
  void set_mask(Mask const& mask) {
    address_iterator_.set_mask(mask);
  }

  /// Gets the mask
  CUTLASS_HOST_DEVICE
  void get_mask(Mask& mask) {
    address_iterator_.get_mask(mask);
  }

  CUTLASS_DEVICE
  void load_with_pointer_offset(Fragment& frag, Index pointer_offset) {
    load_with_byte_offset(
        frag, pointer_offset * sizeof_bits<Element>::value / 8);
  }

  CUTLASS_DEVICE
  void load_with_byte_offset(Fragment& frag, LongIndex byte_offset) {
    AccessType* frag_ptr = reinterpret_cast<AccessType*>(&frag);

    CUTLASS_PRAGMA_UNROLL
    for (int s = 0; s < ThreadMap::Iterations::kStrided; ++s) {
      CUTLASS_PRAGMA_UNROLL
      for (int c = 0; c < ThreadMap::Iterations::kContiguous; ++c) {
        CUTLASS_PRAGMA_UNROLL
        for (int v = 0; v < kAccessesPerVector; ++v) {
          int idx = v +
              kAccessesPerVector * (c + s * ThreadMap::Iterations::kContiguous);

          address_iterator_.set_iteration_index(idx);
          char const* byte_ptr =
              reinterpret_cast<char const*>(address_iterator_.get()) +
              byte_offset;

          AccessType const* access_ptr =
              reinterpret_cast<AccessType const*>(byte_ptr);

          cutlass::arch::global_load<AccessType, sizeof(AccessType)>(
              frag_ptr[idx], access_ptr, address_iterator_.valid());

          ++address_iterator_;
        }
      }
    }
  }

  /// Loads a fragment from memory
  CUTLASS_DEVICE
  void load(Fragment& frag) {
    load_with_byte_offset(frag, 0);
  }

  /// Store a fragment to memory
  CUTLASS_DEVICE
  void store_with_pointer_offset(Fragment const& frag, Index pointer_offset) {
    store_with_byte_offset(
        frag, pointer_offset * sizeof_bits<Element>::value / 8);
  }

  /// Store a fragment to memory
  CUTLASS_DEVICE
  void store_with_byte_offset(Fragment const& frag, LongIndex byte_offset) {
    address_iterator_.set_iteration_index(0);
    AccessType const* frag_ptr = reinterpret_cast<AccessType const*>(&frag);

    CUTLASS_PRAGMA_UNROLL
    for (int s = 0; s < ThreadMap::Iterations::kStrided; ++s) {
      CUTLASS_PRAGMA_UNROLL
      for (int c = 0; c < ThreadMap::Iterations::kContiguous; ++c) {
        CUTLASS_PRAGMA_UNROLL
        for (int v = 0; v < kAccessesPerVector; ++v) {
          int idx = v +
              kAccessesPerVector * (c + s * ThreadMap::Iterations::kContiguous);

          char* byte_ptr =
              reinterpret_cast<char*>(address_iterator_.get()) + byte_offset;
          AccessType* access_ptr = reinterpret_cast<AccessType*>(byte_ptr);

          if (address_iterator_.valid()) {
            *access_ptr = frag_ptr[idx];
          }
          ++address_iterator_;
        }
      }
    }
  }

  /// Store a fragment to memory
  CUTLASS_DEVICE
  void store(Fragment const& frag) {
    store_with_byte_offset(frag, 0);
  }
};
```
**EN**: L154-L446 — Pitch-linear core specialization: defines AccessType, TileAccessIterator, Fragment, and the explicit load/store loops.
**CN**: `L154-L446` —— pitch-linear 核心特化：定义 `AccessType`、`TileAccessIterator`、`Fragment`，并实现显式的 `load/store` 循环。

### Lines 447-678
```cpp

////////////////////////////////////////////////////////////////////////////////

/// Specialization of PredicatedTileIteratorResidualLast for pitch-linear data.
///
/// Satisfies: ForwardTileIteratorConcept |
///            ReadableContiguousTileIteratorConcept |
///            WriteableContiguousTileIteratorConcept |
///            MaskedTileIteratorConcept
///
template <
    typename Shape_,
    typename Element_,
    int AdvanceRank,
    typename ThreadMap_,
    int AccessSize,
    bool Gather>
class PredicatedTileIteratorResidualLast<
    Shape_,
    Element_,
    layout::ColumnMajor,
    AdvanceRank,
    ThreadMap_,
    AccessSize,
    Gather> {
 public:
  static_assert(
      AdvanceRank == 0 || AdvanceRank == 1,
      "Specialization for pitch-linear iterator may along advance along the "
      "contiguous(rank=0) or strided(rank=1) dimension.");

  using Shape = Shape_;
  using Element = Element_;
  using Layout = layout::ColumnMajor;
  static int const kAdvanceRank = AdvanceRank;
  using ThreadMap = ThreadMap_;

  using Index = typename Layout::Index;
  using LongIndex = typename Layout::LongIndex;

  using TensorRef = TensorRef<Element, Layout>;
  using TensorView = TensorView<Element, Layout>;
  using TensorCoord = typename Layout::TensorCoord;

  using Pointer = Element*;
  using NonConstPointer = typename platform::remove_const<Element>::type*;

  using UnderlyingIterator = PredicatedTileIteratorResidualLast<
      layout::PitchLinearShape<Shape::kRow, Shape::kColumn>,
      Element,
      layout::PitchLinear,
      (kAdvanceRank == 0 ? 0 : 1),
      ThreadMap,
      AccessSize,
      Gather>;

  using AccessType = typename UnderlyingIterator::AccessType;

  /// Fragment object to be loaded or stored
  using Fragment = cutlass::Array<
      Element,
      ThreadMap::Iterations::kCount * ThreadMap::kElementsPerAccess>;

  /// Predicate vector stores mask to guard accesses
  using Mask = typename UnderlyingIterator::Mask;

  /// Parameters object is precomputed state and is host-constructible
  class Params {
   private:
    friend PredicatedTileIteratorResidualLast;

    /// Parameters object
    typename UnderlyingIterator::Params params_;

   public:
    CUTLASS_HOST_DEVICE
    Params() {}

    /// Construct the Params object given a pitch-linear tensor's layout
    CUTLASS_HOST_DEVICE
    Params(Layout const& layout)
        : params_(layout::PitchLinear(layout.stride(0))) {}

    CUTLASS_HOST_DEVICE
    Params(typename UnderlyingIterator::Params::Base const& base)
        : params_(base) {}
  };

 private:
  //
  // Data members
  //

  /// Underlying pitch-linear tile iterator
  UnderlyingIterator iterator_;

 public:
  /// Constructs a TileIterator from its precomputed state, threadblock offset,
  /// and thread ID
  CUTLASS_HOST_DEVICE
  PredicatedTileIteratorResidualLast(
      Params const& params, ///< Precomputed parameters object
      Pointer pointer, ///< Pointer to start of tensor
      TensorCoord extent, ///< Extent of tensor
      int thread_id, ///< ID of each participating thread
      TensorCoord const& threadblock_offset, ///< Initial offset of threadblock
      int const* indices =
          nullptr ///< gather/scatter indices, note no support for
                  ///< gather/scatter at this specialization
      )
      : iterator_(
            params.params_,
            pointer,
            layout::PitchLinearCoord(extent.row(), extent.column()),
            thread_id,
            layout::PitchLinearCoord(
                threadblock_offset.row(),
                threadblock_offset.column()),
            indices) {}

  /// Construct a PredicatedTileIteratorResidualLast with zero threadblock
  /// offset
  CUTLASS_HOST_DEVICE
  PredicatedTileIteratorResidualLast(
      Params const& params, ///< Precomputed parameters object
      Pointer pointer, ///< Pointer to start of tensor
      TensorCoord extent, ///< Extent of tensor
      int thread_id ///< ID of each participating thread
      )
      : PredicatedTileIteratorResidualLast(
            params,
            pointer,
            extent,
            thread_id,
            make_Coord(0, 0)) {}

  /// Adds a pointer offset in units of Element
  CUTLASS_HOST_DEVICE
  void add_pointer_offset(LongIndex pointer_offset) {
    iterator_.add_pointer_offset(pointer_offset);
  }

  /// Advances to the next tile in memory.
  ///
  /// The first time this method is called, predicates are updated, and the
  /// iterator's internal pointer is reverted to the first "steady state" tile.
  /// Subsequent calls are lightweight and must only update the internal
  /// pointer.
  CUTLASS_HOST_DEVICE
  PredicatedTileIteratorResidualLast& operator++() {
    ++iterator_;
    return *this;
  }

  /// Advances to the next tile in memory.
  ///
  /// The first time this method is called, predicates are updated, and the
  /// iterator's internal pointer is reverted to the first "steady state" tile.
  /// Subsequent calls are lightweight and must only update the internal
  /// pointer.
  CUTLASS_HOST_DEVICE
  PredicatedTileIteratorResidualLast operator++(int) {
    PredicatedTileIteratorResidualLast self(*this);
    operator++();
    return self;
  }

  /// Clears the predicate set efficiently
  CUTLASS_HOST_DEVICE
  void clear_mask(bool enable = true) {
    iterator_.clear_mask(enable);
  }

  CUTLASS_HOST_DEVICE
  void set_residual_tile(bool enable) {
    iterator_.set_residual_tile(enable);
  }

  /// Clears the predicate set efficiently
  CUTLASS_HOST_DEVICE
  void enable_mask() {
    iterator_.enable_mask();
  }

  /// Sets the predicate mask, overriding value stored in predicate iterator
  CUTLASS_HOST_DEVICE
  void set_mask(Mask const& mask) {
    iterator_.set_mask(mask);
  }

  /// Gets the mask
  CUTLASS_HOST_DEVICE
  void get_mask(Mask& mask) {
    iterator_.get_mask(mask);
  }

  /// Loads a fragment from memory
  CUTLASS_DEVICE
  void load_with_pointer_offset(Fragment& frag, Index pointer_offset) {
    iterator_.load_with_pointer_offset(frag, pointer_offset);
  }

  /// Loads a fragment from memory
  CUTLASS_DEVICE
  void load_with_byte_offset(Fragment& frag, LongIndex byte_offset) {
    iterator_.load_with_byte_offset(frag, byte_offset);
  }

  /// Loads a fragment from memory
  CUTLASS_DEVICE
  void load(Fragment& frag) {
    load_with_pointer_offset(frag, 0);
  }

  /// Store a fragment to memory
  CUTLASS_DEVICE
  void store_with_pointer_offset(Fragment const& frag, Index pointer_offset) {
    iterator_.store_with_pointer_offset(frag, pointer_offset);
  }

  /// Store a fragment to memory
  CUTLASS_DEVICE
  void store_with_byte_offset(Fragment const& frag, LongIndex byte_offset) {
    iterator_.store_with_byte_offset(frag, byte_offset);
  }

  /// Store a fragment to memory
  CUTLASS_DEVICE
  void store(Fragment const& frag) {
    store_with_pointer_offset(frag, 0);
  }
};
```
**EN**: L450-L678 — Column-major adapter that remaps coordinates to the pitch-linear core and forwards mask/load/store operations.
**CN**: `L450-L678` —— column-major 适配器：把坐标映射到 pitch-linear 核心，并转发 mask/load/store 操作。

### Lines 679-908
```cpp

////////////////////////////////////////////////////////////////////////////////

/// Specialization of PredicatedTileIteratorResidualLast for pitch-linear data.
///
/// Satisfies: ForwardTileIteratorConcept |
///            ReadableContiguousTileIteratorConcept |
///            WriteableContiguousTileIteratorConcept |
///            MaskedTileIteratorConcept
///
template <
    typename Shape_,
    typename Element_,
    int AdvanceRank,
    typename ThreadMap_,
    int AccessSize,
    bool Gather>
class PredicatedTileIteratorResidualLast<
    Shape_,
    Element_,
    layout::RowMajor,
    AdvanceRank,
    ThreadMap_,
    AccessSize,
    Gather> {
 public:
  static_assert(
      AdvanceRank == 0 || AdvanceRank == 1,
      "Specialization for pitch-linear iterator may along advance along the "
      "contiguous(rank=0) or strided(rank=1) dimension.");

  using Shape = Shape_;
  using Element = Element_;
  using Layout = layout::RowMajor;
  static int const kAdvanceRank = AdvanceRank;
  using ThreadMap = ThreadMap_;

  using Index = typename Layout::Index;
  using LongIndex = typename Layout::LongIndex;

  using TensorRef = TensorRef<Element, Layout>;
  using TensorView = TensorView<Element, Layout>;
  using TensorCoord = typename Layout::TensorCoord;

  using Pointer = Element*;
  using NonConstPointer = typename platform::remove_const<Element>::type*;

  using UnderlyingIterator = PredicatedTileIteratorResidualLast<
      layout::PitchLinearShape<Shape::kColumn, Shape::kRow>,
      Element,
      layout::PitchLinear,
      (kAdvanceRank == 0 ? 1 : 0),
      ThreadMap,
      AccessSize,
      Gather>;

  using AccessType = typename UnderlyingIterator::AccessType;

  /// Fragment object to be loaded or stored
  using Fragment = cutlass::Array<
      Element,
      ThreadMap::Iterations::kCount * ThreadMap::kElementsPerAccess>;

  /// Predicate vector stores mask to guard accesses
  using Mask = typename UnderlyingIterator::Mask;

  /// Parameters object is precomputed state and is host-constructible
  class Params {
   private:
    friend PredicatedTileIteratorResidualLast;

    /// Parameters object
    typename UnderlyingIterator::Params params_;

   public:
    CUTLASS_HOST_DEVICE
    Params() {}

    /// Construct the Params object given a pitch-linear tensor's layout
    CUTLASS_HOST_DEVICE
    Params(Layout const& layout)
        : params_(layout::PitchLinear(layout.stride(0))) {}

    CUTLASS_HOST_DEVICE
    Params(typename UnderlyingIterator::Params::Base const& base)
        : params_(base) {}
  };

 private:
  //
  // Data members
  //

  /// Underlying pitch-linear tile iterator
  UnderlyingIterator iterator_;

 public:
  /// Constructs a TileIterator from its precomputed state, threadblock offset,
  /// and thread ID
  CUTLASS_HOST_DEVICE
  PredicatedTileIteratorResidualLast(
      Params const& params, ///< Precomputed parameters object
      Pointer pointer, ///< Pointer to start of tensor
      TensorCoord extent, ///< Extent of tensor
      int thread_id, ///< ID of each participating thread
      TensorCoord const& threadblock_offset, ///< Initial offset of threadblock
      int const* indices = nullptr ///< Gather indices
      )
      : iterator_(
            params.params_,
            pointer,
            layout::PitchLinearCoord(extent.column(), extent.row()),
            thread_id,
            layout::PitchLinearCoord(
                threadblock_offset.column(),
                threadblock_offset.row()),
            indices) {}

  /// Construct a PredicatedTileIteratorResidualLast with zero threadblock
  /// offset
  CUTLASS_HOST_DEVICE
  PredicatedTileIteratorResidualLast(
      Params const& params, ///< Precomputed parameters object
      Pointer pointer, ///< Pointer to start of tensor
      TensorCoord extent, ///< Extent of tensor
      int thread_id ///< ID of each participating thread
      )
      : PredicatedTileIteratorResidualLast(
            params,
            pointer,
            extent,
            thread_id,
            make_Coord(0, 0)) {}

  /// Adds a pointer offset in units of Element
  CUTLASS_HOST_DEVICE
  void add_pointer_offset(LongIndex pointer_offset) {
    iterator_.add_pointer_offset(pointer_offset);
  }

  /// Advances to the next tile in memory.
  ///
  /// The first time this method is called, predicates are updated, and the
  /// iterator's internal pointer is reverted to the first "steady state" tile.
  /// Subsequent calls are lightweight and must only update the internal
  /// pointer.
  CUTLASS_HOST_DEVICE
  PredicatedTileIteratorResidualLast& operator++() {
    ++iterator_;
    return *this;
  }

  /// Advances to the next tile in memory.
  ///
  /// The first time this method is called, predicates are updated, and the
  /// iterator's internal pointer is reverted to the first "steady state" tile.
  /// Subsequent calls are lightweight and must only update the internal
  /// pointer.
  CUTLASS_HOST_DEVICE
  PredicatedTileIteratorResidualLast operator++(int) {
    PredicatedTileIteratorResidualLast self(*this);
    operator++();
    return self;
  }

  /// Clears the predicate set efficiently
  CUTLASS_HOST_DEVICE
  void clear_mask(bool enable = true) {
    iterator_.clear_mask(enable);
  }

  CUTLASS_HOST_DEVICE
  void set_residual_tile(bool enable) {
    iterator_.set_residual_tile(enable);
  }

  /// Clears the predicate set efficiently
  CUTLASS_HOST_DEVICE
  void enable_mask() {
    iterator_.enable_mask();
  }

  /// Sets the predicate mask, overriding value stored in predicate iterator
  CUTLASS_HOST_DEVICE
  void set_mask(Mask const& mask) {
    iterator_.set_mask(mask);
  }

  /// Gets the mask
  CUTLASS_HOST_DEVICE
  void get_mask(Mask& mask) {
    iterator_.get_mask(mask);
  }

  /// Loads a fragment from memory
  CUTLASS_DEVICE
  void load_with_pointer_offset(Fragment& frag, Index pointer_offset) {
    iterator_.load_with_pointer_offset(frag, pointer_offset);
  }

  /// Loads a fragment from memory
  CUTLASS_DEVICE
  void load_with_byte_offset(Fragment& frag, LongIndex byte_offset) {
    iterator_.load_with_byte_offset(frag, byte_offset);
  }

  /// Loads a fragment from memory
  CUTLASS_DEVICE
  void load(Fragment& frag) {
    load_with_pointer_offset(frag, 0);
  }

  /// Store a fragment to memory
  CUTLASS_DEVICE
  void store_with_pointer_offset(Fragment const& frag, Index pointer_offset) {
    iterator_.store_with_pointer_offset(frag, pointer_offset);
  }

  /// Store a fragment to memory
  CUTLASS_DEVICE
  void store_with_byte_offset(Fragment const& frag, LongIndex byte_offset) {
    iterator_.store_with_byte_offset(frag, byte_offset);
  }

  /// Store a fragment to memory
  CUTLASS_DEVICE
  void store(Fragment const& frag) {
    store_with_pointer_offset(frag, 0);
  }
};
```
**EN**: L682-L908 — Row-major adapter with row/column-swapped mapping into the same core behavior.
**CN**: `L682-L908` —— row-major 适配器：通过交换行列映射复用同样的核心行为。

### Lines 909-1198
```cpp

////////////////////////////////////////////////////////////////////////////////

/// Specialization of PredicatedTileIteratorResidualLast for affine rank-2 data.
///
/// Satisfies: ForwardTileIteratorConcept |
///            ReadableContiguousTileIteratorConcept |
///            WriteableContiguousTileIteratorConcept |
///            MaskedTileIteratorConcept
///
template <
    typename Shape_,
    typename Element_,
    int AdvanceRank,
    typename ThreadMap_,
    int AccessSize>
class PredicatedTileIteratorResidualLast<
    Shape_,
    Element_,
    layout::AffineRankN<2>,
    AdvanceRank,
    ThreadMap_,
    AccessSize,
    false> {
 public:
  static_assert(
      AdvanceRank == 0 || AdvanceRank == 1,
      "Specialization for pitch-linear iterator may advance along the "
      "contiguous(rank=0) or strided(rank=1) dimension.");

  using Shape = Shape_;
  using Element = Element_;
  using Layout = layout::AffineRankN<2>;
  static int const kAdvanceRank = AdvanceRank;
  using ThreadMap = ThreadMap_;

  using Index = typename Layout::Index;
  using LongIndex = typename Layout::LongIndex;

  using TensorRef = TensorRef<Element, Layout>;
  using TensorView = TensorView<Element, Layout>;
  using TensorCoord = typename Layout::TensorCoord;

  using Pointer = Element*;
  using NonConstPointer = typename platform::remove_const<Element>::type*;

  /// Type used for internal memory accesses
  using AccessType = AlignedArray<
      Element,
      AccessSize,
      (AccessSize * sizeof_bits<Element>::value / 8)>;

  /// Underlying iterator to compute the addresses
  using TileAccessIterator = PredicatedTileAccessIteratorResidualLast<
      Shape,
      Element,
      Layout,
      kAdvanceRank,
      ThreadMap,
      AccessType>;

  static int const kAccessesPerVector = TileAccessIterator::kAccessesPerVector;

  /// Fragment object to be loaded or stored
  using Fragment = cutlass::Array<
      Element,
      ThreadMap::Iterations::kCount * ThreadMap::kElementsPerAccess>;

  /// Predicate vector stores mask to guard accesses
  using Mask = typename TileAccessIterator::Mask;

  /// Parameters object is precomputed state and is host-constructible
  class Params {
   public:
    friend PredicatedTileIteratorResidualLast;

   private:
    /// Parameters object
    typename TileAccessIterator::Params params_;

   public:
    /// Construct the Params object given a pitch-linear tensor's layout
    CUTLASS_HOST_DEVICE
    Params(Layout const& layout) : params_(layout) {}

    CUTLASS_HOST_DEVICE
    Params() {}
  };

 private:
  /// Internal pointer type permits fast address arithmetic
  using BytePointer = char*;

 private:
  //
  // Data members
  //

  /// Data member to the tile access iterator
  TileAccessIterator address_iterator_;

 public:
  /// Constructs a TileIterator from its precomputed state, threadblock offset,
  /// and thread ID
  CUTLASS_HOST_DEVICE
  PredicatedTileIteratorResidualLast(
      /// Precomputed parameters object
      Params const& params,
      /// Pointer to start of tensor
      Pointer pointer,
      /// Extent of tensor
      TensorCoord extent,
      /// ID of each participating thread
      int thread_id,
      /// Initial offset of threadblock
      TensorCoord const& threadblock_offset,
      int const* indices =
          nullptr ///< gather/scatter indices, note no support for
                  ///< gather/scatter at this specialization
      )
      : address_iterator_(
            params.params_,
            pointer,
            extent,
            thread_id,
            threadblock_offset) {}

  /// Construct a PredicatedTileIteratorResidualLast with zero threadblock
  /// offset
  CUTLASS_HOST_DEVICE
  PredicatedTileIteratorResidualLast(
      Params const& params, ///< Precomputed parameters object
      Pointer pointer, ///< Pointer to start of tensor
      TensorCoord extent, ///< Extent of tensor
      int thread_id ///< ID of each participating thread
      )
      : PredicatedTileIteratorResidualLast(
            params,
            pointer,
            extent,
            thread_id,
            make_Coord(0, 0)) {}

  /// Adds a pointer offset in units of Element
  CUTLASS_HOST_DEVICE
  void add_pointer_offset(LongIndex pointer_offset) {
    address_iterator_.add_pointer_offset(pointer_offset);
  }

  /// Advances to the next tile in memory.
  ///
  /// The first time this method is called, predicates are updated, and the
  /// iterator's internal pointer is reverted to the first "steady state" tile.
  /// Subsequent calls are lightweight and must only update the internal
  /// pointer.
  CUTLASS_HOST_DEVICE
  PredicatedTileIteratorResidualLast& operator++() {
    if (kAdvanceRank)
      address_iterator_.add_tile_offset(make_Coord(0, 1));
    else
      address_iterator_.add_tile_offset(make_Coord(1, 0));

    return *this;
  }

  /// Advances to the next tile in memory.
  ///
  /// The first time this method is called, predicates are updated, and the
  /// iterator's internal pointer is reverted to the first "steady state" tile.
  /// Subsequent calls are lightweight and must only update the internal
  /// pointer.
  CUTLASS_HOST_DEVICE
  PredicatedTileIteratorResidualLast operator++(int) {
    PredicatedTileIteratorResidualLast self(*this);
    operator++();
    return self;
  }

  /// Clears the predicate set efficiently
  CUTLASS_HOST_DEVICE
  void clear_mask(bool enable = true) {
    address_iterator_.clear_mask(enable);
  }

  CUTLASS_HOST_DEVICE
  void set_residual_tile(bool enable) {
    address_iterator_.set_residual_tile(enable);
  }

  /// Clears the predicate set efficiently
  CUTLASS_HOST_DEVICE
  void enable_mask() {
    address_iterator_.enable_mask();
  }

  /// Sets the predicate mask, overriding value stored in predicate iterator
  CUTLASS_HOST_DEVICE
  void set_mask(Mask const& mask) {
    address_iterator_.set_mask(mask);
  }

  /// Gets the mask
  CUTLASS_HOST_DEVICE
  void get_mask(Mask& mask) {
    address_iterator_.get_mask(mask);
  }

  CUTLASS_DEVICE
  void load_with_pointer_offset(Fragment& frag, Index pointer_offset) {
    load_with_byte_offset(
        frag, pointer_offset * sizeof_bits<Element>::value / 8);
  }

  CUTLASS_DEVICE
  void load_with_byte_offset(Fragment& frag, LongIndex byte_offset) {
    AccessType* frag_ptr = reinterpret_cast<AccessType*>(&frag);

    CUTLASS_PRAGMA_UNROLL
    for (int s = 0; s < ThreadMap::Iterations::kStrided; ++s) {
      CUTLASS_PRAGMA_UNROLL
      for (int c = 0; c < ThreadMap::Iterations::kContiguous; ++c) {
        CUTLASS_PRAGMA_UNROLL
        for (int v = 0; v < kAccessesPerVector; ++v) {
          int idx = v +
              kAccessesPerVector * (c + s * ThreadMap::Iterations::kContiguous);

          address_iterator_.set_iteration_index(idx);
          char const* byte_ptr =
              reinterpret_cast<char const*>(address_iterator_.get()) +
              byte_offset;

          AccessType const* access_ptr =
              reinterpret_cast<AccessType const*>(byte_ptr);

          cutlass::arch::global_load<AccessType, sizeof(AccessType)>(
              frag_ptr[idx], access_ptr, address_iterator_.valid());

          ++address_iterator_;
        }
      }
    }
  }

  /// Loads a fragment from memory
  CUTLASS_DEVICE
  void load(Fragment& frag) {
    load_with_byte_offset(frag, 0);
  }

  /// Store a fragment to memory
  CUTLASS_DEVICE
  void store_with_pointer_offset(Fragment const& frag, Index pointer_offset) {
    store_with_byte_offset(
        frag, pointer_offset * sizeof_bits<Element>::value / 8);
  }

  /// Store a fragment to memory
  CUTLASS_DEVICE
  void store_with_byte_offset(Fragment const& frag, LongIndex byte_offset) {
    address_iterator_.set_iteration_index(0);
    AccessType const* frag_ptr = reinterpret_cast<AccessType const*>(&frag);

    CUTLASS_PRAGMA_UNROLL
    for (int s = 0; s < ThreadMap::Iterations::kStrided; ++s) {
      CUTLASS_PRAGMA_UNROLL
      for (int c = 0; c < ThreadMap::Iterations::kContiguous; ++c) {
        CUTLASS_PRAGMA_UNROLL
        for (int v = 0; v < kAccessesPerVector; ++v) {
          int idx = v +
              kAccessesPerVector * (c + s * ThreadMap::Iterations::kContiguous);

          char* byte_ptr =
              reinterpret_cast<char*>(address_iterator_.get()) + byte_offset;
          AccessType* access_ptr = reinterpret_cast<AccessType*>(byte_ptr);

          if (address_iterator_.valid()) {
            *access_ptr = frag_ptr[idx];
          }
          ++address_iterator_;
        }
      }
    }
  }

  /// Store a fragment to memory
  CUTLASS_DEVICE
  void store(Fragment const& frag) {
    store_with_byte_offset(frag, 0);
  }
};
```
**EN**: L912-L1198 — Affine-rank-2 core specialization: same fragment loop structure, but with an affine-rank-2 access iterator underneath.
**CN**: `L912-L1198` —— affine-rank-2 核心特化：fragment 循环结构相同，只是下层换成 affine-rank-2 access iterator。

### Lines 1199-1425
```cpp

////////////////////////////////////////////////////////////////////////////////

/// Specialization of PredicatedTileIteratorResidualLast for affine rank 2
/// column-major data.
///
/// Satisfies: ForwardTileIteratorConcept |
///            ReadableContiguousTileIteratorConcept |
///            WriteableContiguousTileIteratorConcept |
///            MaskedTileIteratorConcept
///
template <
    typename Shape_,
    typename Element_,
    int AdvanceRank,
    typename ThreadMap_,
    int AccessSize>
class PredicatedTileIteratorResidualLast<
    Shape_,
    Element_,
    layout::AffineRank2ColumnMajor,
    AdvanceRank,
    ThreadMap_,
    AccessSize,
    false> {
 public:
  static_assert(
      AdvanceRank == 0 || AdvanceRank == 1,
      "Specialization for pitch-linear iterator may along advance along the "
      "contiguous(rank=0) or strided(rank=1) dimension.");

  using Shape = Shape_;
  using Element = Element_;
  using Layout = layout::AffineRank2ColumnMajor;
  static int const kAdvanceRank = AdvanceRank;
  using ThreadMap = ThreadMap_;

  using Index = typename Layout::Index;
  using LongIndex = typename Layout::LongIndex;

  using TensorRef = TensorRef<Element, Layout>;
  using TensorView = TensorView<Element, Layout>;
  using TensorCoord = typename Layout::TensorCoord;

  using Pointer = Element*;
  using NonConstPointer = typename platform::remove_const<Element>::type*;

  // Map to the underlying AffineRankN<2> layout
  using UnderlyingIterator = PredicatedTileIteratorResidualLast<
      layout::PitchLinearShape<Shape::kRow, Shape::kColumn>,
      Element,
      layout::AffineRankN<2>,
      (kAdvanceRank == 0 ? 0 : 1),
      ThreadMap,
      AccessSize>;

  using AccessType = typename UnderlyingIterator::AccessType;

  /// Fragment object to be loaded or stored
  using Fragment = cutlass::Array<
      Element,
      ThreadMap::Iterations::kCount * ThreadMap::kElementsPerAccess>;

  /// Predicate vector stores mask to guard accesses
  using Mask = typename UnderlyingIterator::Mask;

  /// Parameters object is precomputed state and is host-constructible
  class Params {
   private:
    friend PredicatedTileIteratorResidualLast;

    /// Parameters object
    typename UnderlyingIterator::Params params_;

   public:
    CUTLASS_HOST_DEVICE
    Params() {}

    /// Construct the Params object given an AffineRankN<2> tensor's layout
    CUTLASS_HOST_DEVICE
    Params(Layout const& layout)
        : params_(layout::AffineRankN<2>(layout.stride(0), layout.stride(1))) {}
  };

 private:
  //
  // Data members
  //

  /// Underlying AffineRankN<2> tile iterator
  UnderlyingIterator iterator_;

 public:
  /// Constructs a TileIterator from its precomputed state, threadblock offset,
  /// and thread ID
  CUTLASS_HOST_DEVICE
  PredicatedTileIteratorResidualLast(
      Params const& params, ///< Precomputed parameters object
      Pointer pointer, ///< Pointer to start of tensor
      TensorCoord extent, ///< Extent of tensor
      int thread_id, ///< ID of each participating thread
      TensorCoord const& threadblock_offset, ///< Initial offset of threadblock
      int const* indices =
          nullptr ///< gather/scatter indices, note no support for
                  ///< gather/scatter at this specialization
      )
      : iterator_(
            params.params_,
            pointer,
            layout::PitchLinearCoord(extent.row(), extent.column()),
            thread_id,
            layout::PitchLinearCoord(
                threadblock_offset.row(),
                threadblock_offset.column())) {}

  /// Construct a PredicatedTileIteratorResidualLast with zero threadblock
  /// offset
  CUTLASS_HOST_DEVICE
  PredicatedTileIteratorResidualLast(
      Params const& params, ///< Precomputed parameters object
      Pointer pointer, ///< Pointer to start of tensor
      TensorCoord extent, ///< Extent of tensor
      int thread_id ///< ID of each participating thread
      )
      : PredicatedTileIteratorResidualLast(
            params,
            pointer,
            extent,
            thread_id,
            make_Coord(0, 0)) {}

  /// Adds a pointer offset in units of Element
  CUTLASS_HOST_DEVICE
  void add_pointer_offset(LongIndex pointer_offset) {
    iterator_.add_pointer_offset(pointer_offset);
  }

  /// Advances to the next tile in memory.
  ///
  /// The first time this method is called, predicates are updated, and the
  /// iterator's internal pointer is reverted to the first "steady state" tile.
  /// Subsequent calls are lightweight and must only update the internal
  /// pointer.
  CUTLASS_HOST_DEVICE
  PredicatedTileIteratorResidualLast& operator++() {
    ++iterator_;
    return *this;
  }

  /// Advances to the next tile in memory.
  ///
  /// The first time this method is called, predicates are updated, and the
  /// iterator's internal pointer is reverted to the first "steady state" tile.
  /// Subsequent calls are lightweight and must only update the internal
  /// pointer.
  CUTLASS_HOST_DEVICE
  PredicatedTileIteratorResidualLast operator++(int) {
    PredicatedTileIteratorResidualLast self(*this);
    operator++();
    return self;
  }

  /// Clears the predicate set efficiently
  CUTLASS_HOST_DEVICE
  void clear_mask(bool enable = true) {
    iterator_.clear_mask(enable);
  }

  CUTLASS_HOST_DEVICE
  void set_residual_tile(bool enable) {
    iterator_.set_residual_tile(enable);
  }

  /// Clears the predicate set efficiently
  CUTLASS_HOST_DEVICE
  void enable_mask() {
    iterator_.enable_mask();
  }

  /// Sets the predicate mask, overriding value stored in predicate iterator
  CUTLASS_HOST_DEVICE
  void set_mask(Mask const& mask) {
    iterator_.set_mask(mask);
  }

  /// Gets the mask
  CUTLASS_HOST_DEVICE
  void get_mask(Mask& mask) {
    iterator_.get_mask(mask);
  }

  /// Loads a fragment from memory
  CUTLASS_DEVICE
  void load_with_pointer_offset(Fragment& frag, Index pointer_offset) {
    iterator_.load_with_pointer_offset(frag, pointer_offset);
  }

  /// Loads a fragment from memory
  CUTLASS_DEVICE
  void load_with_byte_offset(Fragment& frag, LongIndex byte_offset) {
    iterator_.load_with_byte_offset(frag, byte_offset);
  }

  /// Loads a fragment from memory
  CUTLASS_DEVICE
  void load(Fragment& frag) {
    load_with_pointer_offset(frag, 0);
  }

  /// Store a fragment to memory
  CUTLASS_DEVICE
  void store_with_pointer_offset(Fragment const& frag, Index pointer_offset) {
    iterator_.store_with_pointer_offset(frag, pointer_offset);
  }

  /// Store a fragment to memory
  CUTLASS_DEVICE
  void store_with_byte_offset(Fragment const& frag, LongIndex byte_offset) {
    iterator_.store_with_byte_offset(frag, byte_offset);
  }

  /// Store a fragment to memory
  CUTLASS_DEVICE
  void store(Fragment const& frag) {
    store_with_pointer_offset(frag, 0);
  }
};
```
**EN**: L1202-L1425 — Affine rank-2 column-major adapter.
**CN**: `L1202-L1425` —— affine rank-2 列主适配器。

### Lines 1426-1652
```cpp

////////////////////////////////////////////////////////////////////////////////

/// Specialization of PredicatedTileIteratorResidualLast for affine rank 2
/// row-major data.
///
/// Satisfies: ForwardTileIteratorConcept |
///            ReadableContiguousTileIteratorConcept |
///            WriteableContiguousTileIteratorConcept |
///            MaskedTileIteratorConcept
///
template <
    typename Shape_,
    typename Element_,
    int AdvanceRank,
    typename ThreadMap_,
    int AccessSize>
class PredicatedTileIteratorResidualLast<
    Shape_,
    Element_,
    layout::AffineRank2RowMajor,
    AdvanceRank,
    ThreadMap_,
    AccessSize,
    false> {
 public:
  static_assert(
      AdvanceRank == 0 || AdvanceRank == 1,
      "Specialization for pitch-linear iterator may along advance along the "
      "contiguous(rank=0) or strided(rank=1) dimension.");

  using Shape = Shape_;
  using Element = Element_;
  using Layout = layout::AffineRank2RowMajor;
  static int const kAdvanceRank = AdvanceRank;
  using ThreadMap = ThreadMap_;

  using Index = typename Layout::Index;
  using LongIndex = typename Layout::LongIndex;

  using TensorRef = TensorRef<Element, Layout>;
  using TensorView = TensorView<Element, Layout>;
  using TensorCoord = typename Layout::TensorCoord;

  using Pointer = Element*;
  using NonConstPointer = typename platform::remove_const<Element>::type*;

  // Map to the underlying AffineRankN<2> layout
  using UnderlyingIterator = PredicatedTileIteratorResidualLast<
      layout::PitchLinearShape<Shape::kColumn, Shape::kRow>,
      Element,
      layout::AffineRankN<2>,
      (kAdvanceRank == 0 ? 1 : 0),
      ThreadMap,
      AccessSize>;

  using AccessType = typename UnderlyingIterator::AccessType;

  /// Fragment object to be loaded or stored
  using Fragment = cutlass::Array<
      Element,
      ThreadMap::Iterations::kCount * ThreadMap::kElementsPerAccess>;

  /// Predicate vector stores mask to guard accesses
  using Mask = typename UnderlyingIterator::Mask;

  /// Parameters object is precomputed state and is host-constructible
  class Params {
   private:
    friend PredicatedTileIteratorResidualLast;

    /// Parameters object
    typename UnderlyingIterator::Params params_;

   public:
    CUTLASS_HOST_DEVICE
    Params() {}

    /// Construct the Params object given an AffineRankN<2> tensor's layout
    CUTLASS_HOST_DEVICE
    Params(Layout const& layout)
        : params_(layout::AffineRankN<2>(layout.stride(1), layout.stride(0))) {}
  };

 private:
  //
  // Data members
  //

  /// Underlying AffineRankN<2> tile iterator
  UnderlyingIterator iterator_;

 public:
  /// Constructs a TileIterator from its precomputed state, threadblock offset,
  /// and thread ID
  CUTLASS_HOST_DEVICE
  PredicatedTileIteratorResidualLast(
      Params const& params, ///< Precomputed parameters object
      Pointer pointer, ///< Pointer to start of tensor
      TensorCoord extent, ///< Extent of tensor
      int thread_id, ///< ID of each participating thread
      TensorCoord const& threadblock_offset, ///< Initial offset of threadblock
      int const* indices =
          nullptr ///< gather/scatter indices, note no support for
                  ///< gather/scatter at this specialization
      )
      : iterator_(
            params.params_,
            pointer,
            layout::PitchLinearCoord(extent.column(), extent.row()),
            thread_id,
            layout::PitchLinearCoord(
                threadblock_offset.column(),
                threadblock_offset.row())) {}

  /// Construct a PredicatedTileIteratorResidualLast with zero threadblock
  /// offset
  CUTLASS_HOST_DEVICE
  PredicatedTileIteratorResidualLast(
      Params const& params, ///< Precomputed parameters object
      Pointer pointer, ///< Pointer to start of tensor
      TensorCoord extent, ///< Extent of tensor
      int thread_id ///< ID of each participating thread
      )
      : PredicatedTileIteratorResidualLast(
            params,
            pointer,
            extent,
            thread_id,
            make_Coord(0, 0)) {}

  /// Adds a pointer offset in units of Element
  CUTLASS_HOST_DEVICE
  void add_pointer_offset(LongIndex pointer_offset) {
    iterator_.add_pointer_offset(pointer_offset);
  }

  /// Advances to the next tile in memory.
  ///
  /// The first time this method is called, predicates are updated, and the
  /// iterator's internal pointer is reverted to the first "steady state" tile.
  /// Subsequent calls are lightweight and must only update the internal
  /// pointer.
  CUTLASS_HOST_DEVICE
  PredicatedTileIteratorResidualLast& operator++() {
    ++iterator_;
    return *this;
  }

  /// Advances to the next tile in memory.
  ///
  /// The first time this method is called, predicates are updated, and the
  /// iterator's internal pointer is reverted to the first "steady state" tile.
  /// Subsequent calls are lightweight and must only update the internal
  /// pointer.
  CUTLASS_HOST_DEVICE
  PredicatedTileIteratorResidualLast operator++(int) {
    PredicatedTileIteratorResidualLast self(*this);
    operator++();
    return self;
  }

  /// Clears the predicate set efficiently
  CUTLASS_HOST_DEVICE
  void clear_mask(bool enable = true) {
    iterator_.clear_mask(enable);
  }

  CUTLASS_HOST_DEVICE
  void set_residual_tile(bool enable) {
    iterator_.set_residual_tile(enable);
  }

  /// Clears the predicate set efficiently
  CUTLASS_HOST_DEVICE
  void enable_mask() {
    iterator_.enable_mask();
  }

  /// Sets the predicate mask, overriding value stored in predicate iterator
  CUTLASS_HOST_DEVICE
  void set_mask(Mask const& mask) {
    iterator_.set_mask(mask);
  }

  /// Gets the mask
  CUTLASS_HOST_DEVICE
  void get_mask(Mask& mask) {
    iterator_.get_mask(mask);
  }

  /// Loads a fragment from memory
  CUTLASS_DEVICE
  void load_with_pointer_offset(Fragment& frag, Index pointer_offset) {
    iterator_.load_with_pointer_offset(frag, pointer_offset);
  }

  /// Loads a fragment from memory
  CUTLASS_DEVICE
  void load_with_byte_offset(Fragment& frag, LongIndex byte_offset) {
    iterator_.load_with_byte_offset(frag, byte_offset);
  }

  /// Loads a fragment from memory
  CUTLASS_DEVICE
  void load(Fragment& frag) {
    load_with_pointer_offset(frag, 0);
  }

  /// Store a fragment to memory
  CUTLASS_DEVICE
  void store_with_pointer_offset(Fragment const& frag, Index pointer_offset) {
    iterator_.store_with_pointer_offset(frag, pointer_offset);
  }

  /// Store a fragment to memory
  CUTLASS_DEVICE
  void store_with_byte_offset(Fragment const& frag, LongIndex byte_offset) {
    iterator_.store_with_byte_offset(frag, byte_offset);
  }

  /// Store a fragment to memory
  CUTLASS_DEVICE
  void store(Fragment const& frag) {
    store_with_pointer_offset(frag, 0);
  }
};
```
**EN**: L1429-L1652 — Affine rank-2 row-major adapter.
**CN**: `L1429-L1652` —— affine rank-2 行主适配器。

### Lines 1653-1882
```cpp

////////////////////////////////////////////////////////////////////////////////

/// Specialization of PredicatedTileIteratorResidualLast for interleaved data.
/// It is mapped to the congruous layout.
///
/// Satisfies: ForwardTileIteratorConcept |
///            ReadableContiguousTileIteratorConcept |
///            WriteableContiguousTileIteratorConcept |
///            MaskedTileIteratorConcept
///

template <
    typename Shape_,
    typename Element_,
    int AdvanceRank,
    typename ThreadMap_,
    int AccessSize,
    int InterleavedK>
class PredicatedTileIteratorResidualLast<
    Shape_,
    Element_,
    layout::ColumnMajorInterleaved<InterleavedK>,
    AdvanceRank,
    ThreadMap_,
    AccessSize,
    false> {
 public:
  static_assert(
      AdvanceRank == 0 || AdvanceRank == 1,
      "Specialization for pitch-linear iterator may along advance along the "
      "contiguous(rank=0) or strided(rank=1) dimension.");

  using Shape = Shape_;
  using Element = Element_;
  static int const kInterleavedK = InterleavedK;
  using Layout = layout::ColumnMajorInterleaved<kInterleavedK>;
  static int const kAdvanceRank = AdvanceRank;
  using ThreadMap = ThreadMap_;

  using Index = typename Layout::Index;
  using LongIndex = typename Layout::LongIndex;

  using TensorRef = TensorRef<Element, Layout>;
  using TensorView = TensorView<Element, Layout>;
  using TensorCoord = typename Layout::TensorCoord;

  using Pointer = Element*;
  using NonConstPointer = typename platform::remove_const<Element>::type*;

  using UnderlyingIterator = PredicatedTileIteratorResidualLast<
      layout::PitchLinearShape<
          Shape::kRow * kInterleavedK,
          Shape::kColumn / kInterleavedK>,
      Element,
      layout::PitchLinear,
      (kAdvanceRank == 0 ? 0 : 1),
      ThreadMap,
      AccessSize>;

  using AccessType = typename UnderlyingIterator::AccessType;

  /// Fragment object to be loaded or stored
  using Fragment = cutlass::Array<
      Element,
      ThreadMap::Iterations::kCount * ThreadMap::kElementsPerAccess>;

  /// Predicate vector stores mask to guard accesses
  using Mask = typename UnderlyingIterator::Mask;

  /// Parameters object is precomputed state and is host-constructible
  class Params {
   private:
    friend PredicatedTileIteratorResidualLast;

    /// Parameters object
    typename UnderlyingIterator::Params params_;

   public:
    CUTLASS_HOST_DEVICE
    Params() {}

    /// Construct the Params object given a pitch-linear tensor's layout
    CUTLASS_HOST_DEVICE
    Params(Layout const& layout)
        : params_(layout::PitchLinear(layout.stride(0))) {}

    CUTLASS_HOST_DEVICE
    Params(typename UnderlyingIterator::Params::Base const& base)
        : params_(base) {}
  };

 private:
  //
  // Data members
  //

  /// Underlying pitch-linear tile iterator
  UnderlyingIterator iterator_;

 public:
  /// Constructs a TileIterator from its precomputed state, threadblock offset,
  /// and thread ID
  CUTLASS_HOST_DEVICE
  PredicatedTileIteratorResidualLast(
      /// Precomputed parameters object
      Params const& params,
      /// Pointer to start of tensor
      Pointer pointer,
      /// Extent of tensor
      TensorCoord extent,
      /// ID of each participating thread
      int thread_id,
      /// Initial offset of threadblock
      TensorCoord const& threadblock_offset,
      int const* indices =
          nullptr ///< gather/scatter indices, note no support for
                  ///< gather/scatter at this specialization
      )
      : iterator_(
            params.params_,
            pointer,
            layout::PitchLinearCoord(
                extent.row() * kInterleavedK,
                extent.column() / kInterleavedK),
            thread_id,
            layout::PitchLinearCoord(
                threadblock_offset.row() * kInterleavedK,
                threadblock_offset.column() / kInterleavedK)) {}

  /// Construct a PredicatedTileIteratorResidualLast with zero threadblock
  /// offset
  CUTLASS_HOST_DEVICE
  PredicatedTileIteratorResidualLast(
      Params const& params, ///< Precomputed parameters object
      Pointer pointer, ///< Pointer to start of tensor
      TensorCoord extent, ///< Extent of tensor
      int thread_id ///< ID of each participating thread
      )
      : PredicatedTileIteratorResidualLast(
            params,
            pointer,
            extent,
            thread_id,
            make_Coord(0, 0)) {}

  /// Adds a pointer offset in units of Element
  CUTLASS_HOST_DEVICE
  void add_pointer_offset(LongIndex pointer_offset) {
    iterator_.add_pointer_offset(pointer_offset);
  }

  /// Advances to the next tile in memory.
  ///
  /// The first time this method is called, predicates are updated, and the
  /// iterator's internal pointer is reverted to the first "steady state" tile.
  /// Subsequent calls are lightweight and must only update the internal
  /// pointer.
  CUTLASS_HOST_DEVICE
  PredicatedTileIteratorResidualLast& operator++() {
    ++iterator_;
    return *this;
  }

  /// Advances to the next tile in memory.
  ///
  /// The first time this method is called, predicates are updated, and the
  /// iterator's internal pointer is reverted to the first "steady state" tile.
  /// Subsequent calls are lightweight and must only update the internal
  /// pointer.
  CUTLASS_HOST_DEVICE
  PredicatedTileIteratorResidualLast operator++(int) {
    PredicatedTileIteratorResidualLast self(*this);
    operator++();
    return self;
  }

  /// Clears the predicate set efficiently
  CUTLASS_HOST_DEVICE
  void clear_mask(bool enable = true) {
    iterator_.clear_mask(enable);
  }

  CUTLASS_HOST_DEVICE
  void set_residual_tile(bool enable) {
    iterator_.set_residual_tile(enable);
  }

  /// Clears the predicate set efficiently
  CUTLASS_HOST_DEVICE
  void enable_mask() {
    iterator_.enable_mask();
  }

  /// Sets the predicate mask, overriding value stored in predicate iterator
  CUTLASS_HOST_DEVICE
  void set_mask(Mask const& mask) {
    iterator_.set_mask(mask);
  }

  /// Gets the mask
  CUTLASS_HOST_DEVICE
  void get_mask(Mask& mask) {
    iterator_.get_mask(mask);
  }

  /// Loads a fragment from memory
  CUTLASS_DEVICE
  void load_with_pointer_offset(Fragment& frag, Index pointer_offset) {
    iterator_.load_with_pointer_offset(frag, pointer_offset);
  }

  /// Loads a fragment from memory
  CUTLASS_DEVICE
  void load(Fragment& frag) {
    load_with_pointer_offset(frag, 0);
  }

  /// Store a fragment to memory
  CUTLASS_DEVICE
  void store_with_pointer_offset(Fragment const& frag, Index pointer_offset) {
    iterator_.store_with_pointer_offset(frag, pointer_offset);
  }

  /// Store a fragment to memory
  CUTLASS_DEVICE
  void store(Fragment const& frag) {
    store_with_pointer_offset(frag, 0);
  }
};
```
**EN**: L1656-L1882 — Column-major interleaved adapter mapped to a congruous pitch-linear iterator.
**CN**: `L1656-L1882` —— 列主交错布局适配器，映射到底层 congruous pitch-linear 迭代器。

### Lines 1883-2111
```cpp

////////////////////////////////////////////////////////////////////////////////

/// Specialization of PredicatedTileIteratorResidualLast for interleaved-32
/// data.  It is mapped to the congruous layout.
///
/// Satisfies: ForwardTileIteratorConcept |
///            ReadableContiguousTileIteratorConcept |
///            WriteableContiguousTileIteratorConcept |
///            MaskedTileIteratorConcept
///
template <
    typename Shape_,
    typename Element_,
    int AdvanceRank,
    typename ThreadMap_,
    int AccessSize,
    int InterleavedK>
class PredicatedTileIteratorResidualLast<
    Shape_,
    Element_,
    layout::RowMajorInterleaved<InterleavedK>,
    AdvanceRank,
    ThreadMap_,
    AccessSize,
    false> {
 public:
  static_assert(
      AdvanceRank == 0 || AdvanceRank == 1,
      "Specialization for pitch-linear iterator may along advance along the "
      "contiguous(rank=0) or strided(rank=1) dimension.");

  using Shape = Shape_;
  using Element = Element_;
  static int const kInterleavedK = InterleavedK;
  using Layout = layout::RowMajorInterleaved<kInterleavedK>;
  static int const kAdvanceRank = AdvanceRank;
  using ThreadMap = ThreadMap_;

  using Index = typename Layout::Index;
  using LongIndex = typename Layout::LongIndex;

  using TensorRef = TensorRef<Element, Layout>;
  using TensorView = TensorView<Element, Layout>;
  using TensorCoord = typename Layout::TensorCoord;

  using Pointer = Element*;
  using NonConstPointer = typename platform::remove_const<Element>::type*;

  using UnderlyingIterator = PredicatedTileIteratorResidualLast<
      layout::PitchLinearShape<
          Shape::kColumn * kInterleavedK,
          Shape::kRow / kInterleavedK>,
      Element,
      layout::PitchLinear,
      (kAdvanceRank == 0 ? 1 : 0),
      ThreadMap,
      AccessSize>;

  using AccessType = typename UnderlyingIterator::AccessType;

  /// Fragment object to be loaded or stored
  using Fragment = cutlass::Array<
      Element,
      ThreadMap::Iterations::kCount * ThreadMap::kElementsPerAccess>;

  /// Predicate vector stores mask to guard accesses
  using Mask = typename UnderlyingIterator::Mask;

  /// Parameters object is precomputed state and is host-constructible
  class Params {
   private:
    friend PredicatedTileIteratorResidualLast;

    /// Parameters object
    typename UnderlyingIterator::Params params_;

   public:
    CUTLASS_HOST_DEVICE
    Params() {}

    /// Construct the Params object given a pitch-linear tensor's layout
    CUTLASS_HOST_DEVICE
    Params(Layout const& layout)
        : params_(layout::PitchLinear(layout.stride(0))) {}

    CUTLASS_HOST_DEVICE
    Params(typename UnderlyingIterator::Params::Base const& base)
        : params_(base) {}
  };

 private:
  //
  // Data members
  //

  /// Underlying pitch-linear tile iterator
  UnderlyingIterator iterator_;

 public:
  /// Constructs a TileIterator from its precomputed state, threadblock offset,
  /// and thread ID
  CUTLASS_HOST_DEVICE
  PredicatedTileIteratorResidualLast(
      /// Precomputed parameters object
      Params const& params,
      /// Pointer to start of tensor
      Pointer pointer,
      /// Extent of tensor
      TensorCoord extent,
      /// ID of each participating thread
      int thread_id,
      /// Initial offset of threadblock
      TensorCoord const& threadblock_offset,
      int const* indices =
          nullptr ///< gather/scatter indices, note no support for
                  ///< gather/scatter at this specialization
      )
      : iterator_(
            params.params_,
            pointer,
            layout::PitchLinearCoord(
                extent.column() * kInterleavedK,
                extent.row() / kInterleavedK),
            thread_id,
            layout::PitchLinearCoord(
                threadblock_offset.column() * kInterleavedK,
                threadblock_offset.row() / kInterleavedK)) {}

  /// Construct a PredicatedTileIteratorResidualLast with zero threadblock
  /// offset
  CUTLASS_HOST_DEVICE
  PredicatedTileIteratorResidualLast(
      Params const& params, ///< Precomputed parameters object
      Pointer pointer, ///< Pointer to start of tensor
      TensorCoord extent, ///< Extent of tensor
      int thread_id ///< ID of each participating thread
      )
      : PredicatedTileIteratorResidualLast(
            params,
            pointer,
            extent,
            thread_id,
            make_Coord(0, 0)) {}

  /// Adds a pointer offset in units of Element
  CUTLASS_HOST_DEVICE
  void add_pointer_offset(LongIndex pointer_offset) {
    iterator_.add_pointer_offset(pointer_offset);
  }

  /// Advances to the next tile in memory.
  ///
  /// The first time this method is called, predicates are updated, and the
  /// iterator's internal pointer is reverted to the first "steady state" tile.
  /// Subsequent calls are lightweight and must only update the internal
  /// pointer.
  CUTLASS_HOST_DEVICE
  PredicatedTileIteratorResidualLast& operator++() {
    ++iterator_;
    return *this;
  }

  /// Advances to the next tile in memory.
  ///
  /// The first time this method is called, predicates are updated, and the
  /// iterator's internal pointer is reverted to the first "steady state" tile.
  /// Subsequent calls are lightweight and must only update the internal
  /// pointer.
  CUTLASS_HOST_DEVICE
  PredicatedTileIteratorResidualLast operator++(int) {
    PredicatedTileIteratorResidualLast self(*this);
    operator++();
    return self;
  }

  /// Clears the predicate set efficiently
  CUTLASS_HOST_DEVICE
  void clear_mask(bool enable = true) {
    iterator_.clear_mask(enable);
  }

  CUTLASS_HOST_DEVICE
  void set_residual_tile(bool enable) {
    iterator_.set_residual_tile(enable);
  }

  /// Clears the predicate set efficiently
  CUTLASS_HOST_DEVICE
  void enable_mask() {
    iterator_.enable_mask();
  }

  /// Sets the predicate mask, overriding value stored in predicate iterator
  CUTLASS_HOST_DEVICE
  void set_mask(Mask const& mask) {
    iterator_.set_mask(mask);
  }

  /// Gets the mask
  CUTLASS_HOST_DEVICE
  void get_mask(Mask& mask) {
    iterator_.get_mask(mask);
  }

  /// Loads a fragment from memory
  CUTLASS_DEVICE
  void load_with_pointer_offset(Fragment& frag, Index pointer_offset) {
    iterator_.load_with_pointer_offset(frag, pointer_offset);
  }

  /// Loads a fragment from memory
  CUTLASS_DEVICE
  void load(Fragment& frag) {
    load_with_pointer_offset(frag, 0);
  }

  /// Store a fragment to memory
  CUTLASS_DEVICE
  void store_with_pointer_offset(Fragment const& frag, Index pointer_offset) {
    iterator_.store_with_pointer_offset(frag, pointer_offset);
  }

  /// Store a fragment to memory
  CUTLASS_DEVICE
  void store(Fragment const& frag) {
    store_with_pointer_offset(frag, 0);
  }
};
```
**EN**: L1886-L2111 — Row-major interleaved adapter with analogous remapping.
**CN**: `L1886-L2111` —— 行主交错布局适配器，做对称的重映射。

### Lines 2112-2119
```cpp

////////////////////////////////////////////////////////////////////////////////

} // namespace threadblock
} // namespace transform
} // namespace cutlass

////////////////////////////////////////////////////////////////////////////////
```
**EN**: L2115-L2119 — Namespace closing.
**CN**: `L2115-L2119` —— 命名空间结束。

---
## Key Concepts / 关键概念
- **EN**: The primary template is `PredicatedTileIteratorResidualLast<Shape, Element, Layout, AdvanceRank, ThreadMap, AccessSize, Gather>`.
  **CN**: 主模板是 `PredicatedTileIteratorResidualLast<Shape, Element, Layout, AdvanceRank, ThreadMap, AccessSize, Gather>`。
- **EN**: `AccessSize` is turned into an `AlignedArray<Element, AccessSize, ...>` `AccessType` inside the core specializations, so fragment loads/stores are expressed as vector transactions.
  **CN**: 在核心特化中，`AccessSize` 会被封装成 `AlignedArray<Element, AccessSize, ...>` 形式的 `AccessType`，因此 fragment 的读写最终表现为向量事务。
- **EN**: `TileAccessIterator` (or `UnderlyingIterator`) is the most important policy-dependent type: it provides address generation, mask control, and per-access increment behavior.
  **CN**: `TileAccessIterator`（或 `UnderlyingIterator`）是最关键的策略相关类型：它负责地址生成、掩码控制以及逐访问的递增行为。
- **EN**: `Fragment` is always `Array<Element, ThreadMap::Iterations::kCount * ThreadMap::kElementsPerAccess>`, matching the full per-thread tile footprint.
  **CN**: `Fragment` 始终是 `Array<Element, ThreadMap::Iterations::kCount * ThreadMap::kElementsPerAccess>`，与每个线程负责的完整 tile 足迹相匹配。
- **EN**: The pitch-linear core (`L168-L446`) stores a single `TileAccessIterator address_iterator_`; all predicate state, pointer state, and residual mask state live below this layer.
  **CN**: pitch-linear 核心（`L168-L446`）只保存一个 `TileAccessIterator address_iterator_`；所有谓词状态、指针状态以及 residual 掩码状态都位于更底层。
- **EN**: The affine-rank-2 core (`L925-L1198`) has the same structure: one `address_iterator_`, no duplicate mask storage, and fragment loops that call the lower-level iterator repeatedly.
  **CN**: affine-rank-2 核心（`L925-L1198`）结构相同：同样只有一个 `address_iterator_`，不重复保存掩码，而是通过 fragment 循环反复调用下层 access iterator。
- **EN**: Adapter specializations for column-major, row-major, affine rank-2 named layouts, and interleaved layouts all store `UnderlyingIterator iterator_` and forward the public API.
  **CN**: 列主、行主、具名 affine rank-2 布局以及交错布局的适配特化都只保存 `UnderlyingIterator iterator_`，并把公共接口全部转发出去。
- **EN**: `Params` is always a thin wrapper around the corresponding lower-level iterator `Params`, which keeps host-side precomputation consistent across layers.
  **CN**: `Params` 始终只是对对应底层迭代器 `Params` 的一层薄封装，从而保证 host 侧预计算在各层之间保持一致。
## Dependencies / 依赖项
- `cutlass/arch/memory.h` — Direct include used by this file / 本文件直接包含并依赖的头文件
- `cutlass/transform/threadblock/predicated_tile_access_iterator.h` — Direct include used by this file / 本文件直接包含并依赖的头文件
