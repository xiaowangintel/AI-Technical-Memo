# predicated_tile_access_iterator_residual_last.h — Code Analysis / 代码分析
**Source / 源文件**: `examples/41_fused_multi_head_attention/iterators/predicated_tile_access_iterator_residual_last.h`
**Purpose / 用途**: This file implements the low-level address-and-predicate iterator used by the fused multi-head attention example when the residual tile should be handled explicitly at the end of traversal. It is the engine beneath the higher-level tile iterator: it knows how to compute per-access pointers, pack predicate bits, and walk through vector/contiguous/strided sub-iterations. The file contains two true “core” implementations—pitch-linear and affine-rank-2—and many layout adapters that remap coordinates to those cores. / 这个文件实现了融合多头注意力示例使用的底层“地址 + 谓词”迭代器，适用于把 residual 尾块显式放到遍历末尾处理的场景。 它是更高层 tile iterator 的底层引擎：负责计算每次访问的指针、打包谓词位，并在 vector / contiguous / strided 子迭代之间推进。 文件里真正承担核心实现的只有两类：pitch-linear 与 affine-rank-2；其余大部分特化都是把坐标重映射到这两个核心实现上的适配层。
---
## Line-by-Line Analysis / 逐行分析

### Lines 1-56
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
    \brief Templates calculating the address and predicates to the load of tiles
    from pitch-linear rank=2 tensors.

    This iterator uses masks to guard out-of-bounds accesses. The first tile
   this iterator visits maybe partial, then the remaining tiles are complete.
   So, we only need to compute the predicates twice, once before the first tile
   and once for the remaining full tiles which can share the same predicates.

    A precomputed "Params" object minimizes the amount of state that must be
    stored in registers, and integer addition is used to advance the pointer
    through memory.
*/

#pragma once

#include "cutlass/array.h"
#include "cutlass/coord.h"
#include "cutlass/cutlass.h"
#include "cutlass/layout/matrix.h"
#include "cutlass/layout/pitch_linear.h"
#include "cutlass/matrix_shape.h"
#include "cutlass/predicate_vector.h"
#include "cutlass/tensor_ref.h"
#include "cutlass/tensor_view.h"
#include "cutlass/transform/threadblock/predicated_tile_access_iterator_params.h"
```
**EN**: L1-L56 — File prologue, includes, and the note that only two predicate computations are needed: one for the tail shape and one for steady state.
**CN**: `L1-L56` —— 文件前言、依赖包含，以及“只需计算两次谓词：一次针对尾块，一次针对稳态”的说明。

### Lines 57-78
```cpp

////////////////////////////////////////////////////////////////////////////////

////////////////////////////////////////////////////////////////////////////////

namespace cutlass {
namespace transform {
namespace threadblock {

////////////////////////////////////////////////////////////////////////////////

/// PredicatedTileAccessIteratorResidualLast
///
template <
    typename Shape,
    typename Element,
    typename Layout,
    int AdvanceRank,
    typename ThreadMap,
    typename AccessType,
    bool Gather = false>
class PredicatedTileAccessIteratorResidualLast;
```
**EN**: L62-L78 — Namespace opening and primary template declaration.
**CN**: `L62-L78` —— 命名空间展开与主模板声明。

### Lines 79-432
```cpp

////////////////////////////////////////////////////////////////////////////////

/// Specialization of PredicatedTileAccessIteratorResidualLast for pitch-linear
/// data.
///
template <
    typename Shape_,
    typename Element_,
    int AdvanceRank,
    typename ThreadMap_,
    typename AccessType_,
    bool Gather>
class PredicatedTileAccessIteratorResidualLast<
    Shape_,
    Element_,
    layout::PitchLinear,
    AdvanceRank,
    ThreadMap_,
    AccessType_,
    Gather> {
 public:
  static_assert(
      AdvanceRank == 0 || AdvanceRank == 1,
      "Specialization for pitch-linear iterator may along advance along the "
      "contiguous(rank=0) or strided(rank=1) dimension.");

  using Shape = Shape_;
  using Element = Element_;
  using Layout = layout::PitchLinear;
  static int const kAdvanceRank = AdvanceRank;
  using ThreadMap = ThreadMap_;
  using AccessType = AccessType_;

  using Index = typename Layout::Index;
  using LongIndex = typename Layout::LongIndex;

  using TensorRef = TensorRef<Element, Layout>;
  using TensorView = TensorView<Element, Layout>;
  using TensorCoord = typename Layout::TensorCoord;

  using Pointer = Element*;
  using NonConstPointer = typename platform::remove_const<Element>::type*;

  using UnderlyingPredicates = PredicatedTileAccessIteratorPredicates<
      Shape,
      Element,
      Layout,
      AdvanceRank,
      ThreadMap,
      AccessType>;

  static int const kAccessesPerVector =
      ThreadMap::kElementsPerAccess / AccessType::kElements;

  static_assert(
      !(ThreadMap::kElementsPerAccess % AccessType::kElements),
      "Vectors implied by the thread map must be divisible by the access type.");

  using Mask = typename UnderlyingPredicates::Mask;

  /// Uses a non-template class
  struct Params : PredicatedTileAccessIteratorParams {
    using Base = PredicatedTileAccessIteratorParams;

    // Default ctor
    CUTLASS_HOST_DEVICE
    Params() {}

    /// Construct the Params object given a pitch-linear tensor's layout
    CUTLASS_HOST_DEVICE
    Params(Layout const& layout)
        : Base(
              layout.stride(0),
              MakePredicatedTileAccessIteratorDesc<
                  Shape,
                  Element,
                  Layout,
                  kAdvanceRank,
                  ThreadMap>()()) {}

    CUTLASS_HOST_DEVICE
    Params(Base const& base) : Base(base) {}
  };

 private:
  /// Internal pointer type permits fast address arithmetic
  using BytePointer = char*;

 private:
  //
  // Data members
  //

  UnderlyingPredicates the_predicates;
  Mask residual_tile_mask;

  /// Parameters object with precomputed internal state
  Params params_;

  /// Internal pointer to first access of tile
  BytePointer pointer_;

  /// Below is used when Gather is turned on.  We need to record strided_offset
  /// and contiguous_offset separated to compute the offset by using
  ///
  /// offset = contiguous_offset + indices[strided_offset]
  ///

  /// Gather indices
  int const* indices_;

  Index gather_offset_strided;

 private:
  /// Computes predicates based on internally tracked per-thread offset.
  CUTLASS_DEVICE
  void compute_predicates_(
      /// Extent of the matrix window
      TensorCoord extent,
      /// optionally, simplify predicate calculation during 'steady state' phase
      bool is_steady_state = false) {
    the_predicates.compute_predicates_(extent, is_steady_state);
  }

 public:
  /// Constructs a TileIterator from its precomputed state, threadblock offset,
  /// and thread ID
  CUTLASS_HOST_DEVICE
  PredicatedTileAccessIteratorResidualLast(
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
      : params_(params),
        pointer_(reinterpret_cast<BytePointer>(
            const_cast<NonConstPointer>(pointer))),
        the_predicates(extent),
        indices_(indices) {
    the_predicates.set_predicates(thread_id, threadblock_offset);
    the_predicates.get_mask(residual_tile_mask);

    // Working around a weird compiler bug happening on P100 for the backward.
    // I've seen together: the_predicates.predicates_[0] = 14 (instead of 15)
    // residual_tile_mask[0] = 15 (correct)
    //
    // Adding prints when the value is calculated (in `compute_predicates_`)
    // sometimes removes the bug. The consequence is that we skip some
    // element of a tensor, leading to wrong results
    // Setting `compute_predicates_`'s second argument (`is_steady_state`) to
    // true also seems to get rid of the bug - at the cost of twice as many
    // comparisons.
#if !defined(__CUDA_ARCH__) || (__CUDA_ARCH__ >= 700)
    constexpr bool kWorkAroundCompilerBug = false;
#else
    constexpr bool kWorkAroundCompilerBug = true;
#endif
    the_predicates.compute_predicates_(extent, true && !kWorkAroundCompilerBug);

    // update internal pointers
    Layout layout(params_.stride_);

    if (!Gather) {
      add_pointer_offset(layout(the_predicates.thread_offset_));
    } else {
      gather_offset_strided = the_predicates.thread_offset_.strided();
      add_pointer_offset(
          layout(make_Coord(the_predicates.thread_offset_.contiguous(), 0)));
    }
  }

  /// Construct a PredicatedTileAccessIteratorResidualLast with zero threadblock
  /// offset
  CUTLASS_HOST_DEVICE
  PredicatedTileAccessIteratorResidualLast(
      /// Precomputed parameters object
      Params const& params,
      /// Pointer to start of tensor
      Pointer pointer,
      /// Extent of tensor
      TensorCoord extent,
      ///< ID of each participating thread
      int thread_id)
      : PredicatedTileAccessIteratorResidualLast(
            params,
            pointer,
            extent,
            thread_id,
            make_Coord(0, 0)) {}

  /// Overrides the internal iteration index
  CUTLASS_HOST_DEVICE
  void set_iteration_index(int index) {
    the_predicates.set_iteration_index(index);
  }

  CUTLASS_HOST_DEVICE
  void set_residual_tile(bool is_residual_tile) {
    if (is_residual_tile) {
      the_predicates.set_mask(residual_tile_mask);
    }
  }

  /// Adds a pointer offset in units of Element
  CUTLASS_HOST_DEVICE
  void add_pointer_offset(LongIndex pointer_offset) {
    pointer_ += sizeof_bits<Element>::value * pointer_offset / 8;
  }

  /// Advances an iterator along logical dimensions of matrix in units of whole
  /// tiles
  CUTLASS_DEVICE
  void add_tile_offset(TensorCoord const& tile_offset) {
    if (!Gather) {
      if (kAdvanceRank) {
        pointer_ += params_.inc_advance_ * LongIndex(tile_offset.strided());
        pointer_ += Shape::kContiguous * tile_offset.contiguous();
      } else {
        pointer_ += params_.inc_advance_ * LongIndex(tile_offset.contiguous());
        pointer_ += Shape::kStrided * tile_offset.strided();
      }
    } else {
      add_pointer_offset(Shape::kContiguous * tile_offset.contiguous());
      gather_offset_strided += Shape::kStrided * tile_offset.strided();
    }
  }

  /// Returns a pointer
  CUTLASS_HOST_DEVICE
  AccessType* get() const {
    if (Gather) {
      assert(indices_);

      if (!valid()) {
        return nullptr;
      }

      LongIndex contiguous_offset = the_predicates.iteration_contiguous_ *
              (ThreadMap::Delta::kContiguous * sizeof_bits<Element>::value /
               8) +
          the_predicates.iteration_vector_;
      int strided_index = gather_offset_strided +
          the_predicates.iteration_strided_ * ThreadMap::Delta::kStrided;

      LongIndex strided_offset = indices_[strided_index] *
          LongIndex(params_.stride_) * sizeof_bits<Element>::value / 8;

      return reinterpret_cast<AccessType*>(
          pointer_ + contiguous_offset + strided_offset);
    }

    return reinterpret_cast<AccessType*>(
               pointer_ +
               the_predicates.iteration_contiguous_ *
                   (ThreadMap::Delta::kContiguous *
                    sizeof_bits<Element>::value) /
                   8) +
        the_predicates.iteration_vector_;
  }

  /// Increment and return an instance to self.
  CUTLASS_HOST_DEVICE
  PredicatedTileAccessIteratorResidualLast& operator++() {
    the_predicates.operator++();

    ++the_predicates.iteration_vector_;
    if (the_predicates.iteration_vector_ < kAccessesPerVector) {
      return *this;
    }

    the_predicates.iteration_vector_ = 0;
    ++the_predicates.iteration_contiguous_;

    if (the_predicates.iteration_contiguous_ <
        ThreadMap::Iterations::kContiguous) {
      return *this;
    }

    // Enter here only if (iteration_contiguous_ ==
    // ThreadMap::Iteration::kContiguous)
    the_predicates.iteration_contiguous_ = 0;
    ++the_predicates.iteration_strided_;

    if (the_predicates.iteration_strided_ < ThreadMap::Iterations::kStrided) {
      if (!Gather) {
        pointer_ += params_.inc_strided_;
      }

      return *this;
    }

    // Enter here only if (iteration_stride_ == ThreadMap::Iteration::kStrided)
    // which means we enter the next tile.
    the_predicates.iteration_strided_ = 0;

    if (!Gather) {
      // advance to next tile
      pointer_ += params_.inc_next_;

      // now return to start tile - if the iterator is subsequently advanced,
      // this subtraction as well as the subsequent integer addition are both
      // elided by the compiler.
      pointer_ -= params_.inc_advance_;
    }

    return *this;
  }

  /// Increment and return an instance to self.
  CUTLASS_HOST_DEVICE
  PredicatedTileAccessIteratorResidualLast operator++(int) {
    PredicatedTileAccessIteratorResidualLast self(*this);
    operator++();
    return self;
  }

  /// Clears the predicate set efficiently
  CUTLASS_HOST_DEVICE
  void clear_mask(bool enable = true) {
    the_predicates.clear_mask(enable);
  }

  /// Clears the predicate set efficiently
  CUTLASS_HOST_DEVICE
  void enable_mask() {
    the_predicates.enable_mask();
  }

  /// Sets the predicate mask, overriding value stored in predicate iterator
  CUTLASS_HOST_DEVICE
  void set_mask(Mask const& mask) {
    the_predicates.set_mask(mask);
  }

  /// Gets the mask
  CUTLASS_HOST_DEVICE
  void get_mask(Mask& mask) {
    the_predicates.get_mask(mask);
  }

  /// Returns whether access is valid or not
  CUTLASS_HOST_DEVICE
  bool valid() const {
    return the_predicates.valid();
  }
};
```
**EN**: L82-L432 — Pitch-linear core specialization: owns the real residual-last predicate caching, optional gather path, byte pointer math, and per-access increment logic.
**CN**: `L82-L432` —— pitch-linear 核心特化：真正实现 residual-last 谓词缓存、可选 gather 路径、字节指针运算与逐访问递增逻辑。

### Lines 433-658
```cpp

////////////////////////////////////////////////////////////////////////////////

/// Specialization of PredicatedTileAccessIteratorResidualLast for column-major
/// data.
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
    typename AccessType_,
    bool Gather>
class PredicatedTileAccessIteratorResidualLast<
    Shape_,
    Element_,
    layout::ColumnMajor,
    AdvanceRank,
    ThreadMap_,
    AccessType_,
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
  using AccessType = AccessType_;

  using Index = typename Layout::Index;
  using LongIndex = typename Layout::LongIndex;

  using TensorRef = TensorRef<Element, Layout>;
  using TensorView = TensorView<Element, Layout>;
  using TensorCoord = typename Layout::TensorCoord;

  using Pointer = Element*;
  using NonConstPointer = typename platform::remove_const<Element>::type*;

  using UnderlyingIterator = PredicatedTileAccessIteratorResidualLast<
      layout::PitchLinearShape<Shape::kRow, Shape::kColumn>,
      Element,
      layout::PitchLinear,
      (kAdvanceRank == 0 ? 0 : 1),
      ThreadMap,
      AccessType,
      Gather>;

  /// Predicate vector stores mask to guard accesses
  using Mask = typename UnderlyingIterator::Mask;

  static int const kAccessesPerVector = UnderlyingIterator::kAccessesPerVector;

  /// Parameters object is precomputed state and is host-constructible
  class Params {
   private:
    friend PredicatedTileAccessIteratorResidualLast;

    /// Parameters object
    typename UnderlyingIterator::Params params_;

   public:
    /// Default ctor
    CUTLASS_HOST_DEVICE
    Params() {}

    /// Construct the Params object given a pitch-linear tensor's layout
    CUTLASS_HOST_DEVICE
    Params(Layout const& layout)
        : params_(layout::PitchLinear(layout.stride(0))){};

    /// Construct the Params object given a pitch-linear tensor's layout
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
  PredicatedTileAccessIteratorResidualLast(
      ///< Precomputed parameters object
      Params const& params,
      ///< Pointer to start of tensor
      Pointer pointer,
      ///< Extent of tensor
      TensorCoord extent,
      ///< ID of each participating thread
      int thread_id,
      ///< Initial offset of threadblock
      TensorCoord const& threadblock_offset,
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

  /// Construct a PredicatedTileAccessIteratorResidualLast with zero threadblock
  /// offset
  CUTLASS_HOST_DEVICE
  PredicatedTileAccessIteratorResidualLast(
      Params const& params, ///< Precomputed parameters object
      Pointer pointer, ///< Pointer to start of tensor
      TensorCoord extent, ///< Extent of tensor
      int thread_id ///< ID of each participating thread
      )
      : PredicatedTileAccessIteratorResidualLast(
            params,
            pointer,
            extent,
            thread_id,
            make_Coord(0, 0)) {}

  /// Overrides the internal iteration index
  CUTLASS_HOST_DEVICE
  void set_iteration_index(int index) {
    iterator_.set_iteration_index(index);
  }

  CUTLASS_HOST_DEVICE
  void set_residual_tile(bool enable) {
    iterator_.set_residual_tile(enable);
  }

  /// Adds a pointer offset in units of Element
  CUTLASS_HOST_DEVICE
  void add_pointer_offset(LongIndex pointer_offset) {
    iterator_.add_pointer_offset(pointer_offset);
  }

  /// Advances an iterator along logical dimensions of matrix in units of whole
  /// tiles
  CUTLASS_HOST_DEVICE
  void add_tile_offset(TensorCoord const& tile_offset) {
    iterator_.add_tile_offset({tile_offset.row(), tile_offset.column()});
  }

  /// Returns a pointer
  CUTLASS_HOST_DEVICE
  AccessType* get() const {
    return reinterpret_cast<AccessType*>(iterator_.get());
  }

  /// Advances to the next tile in memory.
  ///
  /// The first time this method is called, predicates are updated, and the
  /// iterator's internal pointer is reverted to the first "steady state" tile.
  /// Subsequent calls are lightweight and must only update the internal
  /// pointer.
  CUTLASS_HOST_DEVICE
  PredicatedTileAccessIteratorResidualLast& operator++() {
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
  PredicatedTileAccessIteratorResidualLast operator++(int) {
    PredicatedTileAccessIteratorResidualLast self(*this);
    operator++();
    return self;
  }

  /// Clears the predicate set efficiently
  CUTLASS_HOST_DEVICE
  void clear_mask(bool enable = true) {
    iterator_.clear_mask(enable);
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

  /// Returns whether access is valid or not
  CUTLASS_HOST_DEVICE
  bool valid() {
    return iterator_.valid();
  }
};
```
**EN**: L436-L658 — Column-major adapter: remaps row/column coordinates to a pitch-linear underlying iterator with the same residual-last semantics.
**CN**: `L436-L658` —— column-major 适配器：把行列坐标映射到底层 pitch-linear 迭代器，并继承同样的 residual-last 语义。

### Lines 659-880
```cpp
////////////////////////////////////////////////////////////////////////////////

/// Specialization of PredicatedTileAccessIteratorResidualLast for row-major
/// data.
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
    typename AccessType_,
    bool Gather>
class PredicatedTileAccessIteratorResidualLast<
    Shape_,
    Element_,
    layout::RowMajor,
    AdvanceRank,
    ThreadMap_,
    AccessType_,
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
  using AccessType = AccessType_;

  using Index = typename Layout::Index;
  using LongIndex = typename Layout::LongIndex;

  using TensorRef = TensorRef<Element, Layout>;
  using TensorView = TensorView<Element, Layout>;
  using TensorCoord = typename Layout::TensorCoord;

  using Pointer = Element*;
  using NonConstPointer = typename platform::remove_const<Element>::type*;

  using UnderlyingIterator = PredicatedTileAccessIteratorResidualLast<
      layout::PitchLinearShape<Shape::kColumn, Shape::kRow>,
      Element,
      layout::PitchLinear,
      (kAdvanceRank == 0 ? 1 : 0),
      ThreadMap,
      AccessType,
      Gather>;

  static int const kAccessesPerVector = UnderlyingIterator::kAccessesPerVector;

  /// Predicate vector stores mask to guard accesses
  using Mask = typename UnderlyingIterator::Mask;

  /// Parameters object is precomputed state and is host-constructible
  class Params {
   private:
    friend PredicatedTileAccessIteratorResidualLast;

    /// Parameters object
    typename UnderlyingIterator::Params params_;

   public:
    /// Default ctor
    CUTLASS_HOST_DEVICE
    Params() {}

    /// Construct the Params object given a pitch-linear tensor's layout
    CUTLASS_HOST_DEVICE
    Params(Layout const& layout)
        : params_(layout::PitchLinear(layout.stride(0))){};

    /// Construct the Params object given a pitch-linear tensor's layout
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
  PredicatedTileAccessIteratorResidualLast(
      ///< Precomputed parameters object
      Params const& params,
      ///< Pointer to start of tensor
      Pointer pointer,
      ///< Extent of tensor
      TensorCoord extent,
      ///< ID of each participating thread
      int thread_id,
      ///< Initial offset of threadblock
      TensorCoord const& threadblock_offset,
      /// Gather indices
      int const* indices = nullptr)
      : iterator_(
            params.params_,
            pointer,
            layout::PitchLinearCoord(extent.column(), extent.row()),
            thread_id,
            layout::PitchLinearCoord(
                threadblock_offset.column(),
                threadblock_offset.row()),
            indices) {}

  /// Construct a PredicatedTileAccessIteratorResidualLast with zero threadblock
  /// offset
  CUTLASS_HOST_DEVICE
  PredicatedTileAccessIteratorResidualLast(
      Params const& params, ///< Precomputed parameters object
      Pointer pointer, ///< Pointer to start of tensor
      TensorCoord extent, ///< Extent of tensor
      int thread_id ///< ID of each participating thread
      )
      : PredicatedTileAccessIteratorResidualLast(
            params,
            pointer,
            extent,
            thread_id,
            make_Coord(0, 0)) {}

  /// Overrides the internal iteration index
  CUTLASS_HOST_DEVICE
  void set_iteration_index(int index) {
    iterator_.set_iteration_index(index);
  }

  CUTLASS_HOST_DEVICE
  void set_residual_tile(bool enable) {
    iterator_.set_residual_tile(enable);
  }

  /// Adds a pointer offset in units of Element
  CUTLASS_HOST_DEVICE
  void add_pointer_offset(LongIndex pointer_offset) {
    iterator_.add_pointer_offset(pointer_offset);
  }

  /// Advances an iterator along logical dimensions of matrix in units of whole
  /// tiles
  CUTLASS_HOST_DEVICE
  void add_tile_offset(TensorCoord const& tile_offset) {
    iterator_.add_tile_offset({tile_offset.column(), tile_offset.row()});
  }

  /// Returns a pointer
  CUTLASS_HOST_DEVICE
  AccessType* get() const {
    return reinterpret_cast<AccessType*>(iterator_.get());
  }

  /// Advances to the next tile in memory.
  ///
  /// The first time this method is called, predicates are updated, and the
  /// iterator's internal pointer is reverted to the first "steady state" tile.
  /// Subsequent calls are lightweight and must only update the internal
  /// pointer.
  CUTLASS_HOST_DEVICE
  PredicatedTileAccessIteratorResidualLast& operator++() {
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
  PredicatedTileAccessIteratorResidualLast operator++(int) {
    PredicatedTileAccessIteratorResidualLast self(*this);
    operator++();
    return self;
  }

  /// Clears the predicate set efficiently
  CUTLASS_HOST_DEVICE
  void clear_mask(bool enable = true) {
    iterator_.clear_mask(enable);
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

  /// Returns whether access is valid or not
  CUTLASS_HOST_DEVICE
  bool valid() {
    return iterator_.valid();
  }
};
```
**EN**: L661-L880 — Row-major adapter: similar forwarding layer, but it swaps row/column roles when building pitch-linear coordinates and tile offsets.
**CN**: `L661-L880` —— row-major 适配器：也是转发层，但在构造 pitch-linear 坐标与 tile offset 时交换了行/列角色。

### Lines 881-1214
```cpp

////////////////////////////////////////////////////////////////////////////////

/// Specialization of PredicatedTileAccessIteratorResidualLast for affine rank 2
/// data.
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
    typename AccessType_>
class PredicatedTileAccessIteratorResidualLast<
    Shape_,
    Element_,
    layout::AffineRankN<2>,
    AdvanceRank,
    ThreadMap_,
    AccessType_,
    false> {
 public:
  static_assert(
      AdvanceRank == 0 || AdvanceRank == 1,
      "Specialization for pitch-linear iterator may along advance along the "
      "contiguous(rank=0) or strided(rank=1) dimension.");

  using Shape = Shape_;
  using Element = Element_;
  using Layout = layout::AffineRankN<2>;
  static int const kAdvanceRank = AdvanceRank;
  using ThreadMap = ThreadMap_;
  using AccessType = AccessType_;

  using Index = typename Layout::Index;
  using LongIndex = typename Layout::LongIndex;

  using TensorRef = TensorRef<Element, Layout>;
  using TensorView = TensorView<Element, Layout>;
  using TensorCoord = typename Layout::TensorCoord;

  using Pointer = Element*;
  using NonConstPointer = typename platform::remove_const<Element>::type*;

  using UnderlyingPredicates = PredicatedTileAccessIteratorPredicates<
      Shape,
      Element,
      layout::PitchLinear,
      AdvanceRank,
      ThreadMap,
      AccessType>;

  static int const kAccessesPerVector =
      ThreadMap::kElementsPerAccess / AccessType::kElements;

  static_assert(
      !(ThreadMap::kElementsPerAccess % AccessType::kElements),
      "Vectors implied by the thread map must be divisible by the access type.");

  /// Predicate vector stores mask to guard accesses
  using Mask = typename UnderlyingPredicates::Mask;

  /// Parameters object is precomputed state and is host-constructible
  class Params {
   public:
    friend PredicatedTileAccessIteratorResidualLast;

   private:
    /// stride of pitch-linear layout (units of Element)
    Coord<Layout::kStrideRank, Layout::LongIndex> stride_;
    /// amount (in byte) to increment pointer to move to next access along
    /// contiguous dimension
    LongIndex inc_contiguous_;
    /// amount (in byte) to increment pointer from first access of current
    /// contiguous dimension to first access of next one.
    LongIndex inc_strided_;
    /// amount (in byte) to increment pointer from last access of current
    /// contiguous dimension to first access of next one.
    LongIndex inc_next_strided_;
    /// amount (in byte) to increment pointer from last access to first access
    /// of next tile
    LongIndex inc_next_;
    /// amount (in byte) to increment pointer from first access of current tile
    /// to first access of next tile
    LongIndex inc_advance_;

   public:
    // Default ctor
    CUTLASS_HOST_DEVICE
    Params()
        : stride_(0),
          inc_contiguous_(0),
          inc_strided_(0),
          inc_next_(0),
          inc_advance_(0) {}

    /// Construct the Params object given a pitch-linear tensor's layout
    CUTLASS_HOST_DEVICE
    Params(Layout const& layout)
        : stride_({layout.stride(0), layout.stride(1)}) {
      inc_contiguous_ =
          (LongIndex(stride_[0]) * ThreadMap::Delta::kContiguous) *
          sizeof_bits<Element>::value / 8;

      inc_strided_ = (LongIndex(stride_[1]) * ThreadMap::Delta::kStrided) *
          sizeof_bits<Element>::value / 8;

      inc_next_strided_ = inc_strided_ -
          LongIndex(ThreadMap::Iterations::kContiguous - 1) * inc_contiguous_;

      if (kAdvanceRank) {
        // advance along strided dimension
        inc_advance_ = Shape::kStrided * LongIndex(stride_[1]) *
            sizeof_bits<Element>::value / 8;
      } else {
        // advance along contiguous dimension
        inc_advance_ =
            Shape::kContiguous * stride_[0] * sizeof_bits<Element>::value / 8;
      }

      inc_next_ = inc_advance_ -
          LongIndex(ThreadMap::Iterations::kContiguous - 1) * inc_contiguous_ -
          LongIndex(ThreadMap::Iterations::kStrided - 1) * inc_strided_;
    };
  };

 private:
  /// Internal pointer type permits fast address arithmetic
  using BytePointer = char*;

  //
  // Data members
  //

  /// Parameters object with precomputed internal state
  Params params_;

  /// Internal pointer to first access of tile
  BytePointer pointer_;

  UnderlyingPredicates the_predicates;
  Mask residual_tile_mask;

 private:
  /// Computes predicates based on internally tracked per-thread offset.
  CUTLASS_DEVICE
  void compute_predicates_(
      /// Extent of the matrix window
      TensorCoord extent,
      /// optionally, simplify predicate calculation during 'steady state' phase
      bool is_steady_state = false) {
    the_predicates.compute_predicates_(extent, is_steady_state);
  }

 public:
  /// Constructs a TileIterator from its precomputed state, threadblock offset,
  /// and thread ID
  CUTLASS_HOST_DEVICE
  PredicatedTileAccessIteratorResidualLast(
      ///< Precomputed parameters object
      Params const& params,
      ///< Pointer to start of tensor
      Pointer pointer,
      ///< Extent of tensor
      TensorCoord extent,
      ///< ID of each participating thread
      int thread_id,
      ///< Initial offset of threadblock
      TensorCoord const& threadblock_offset,
      int const* indices =
          nullptr ///< gather/scatter indices, note no support for
                  ///< gather/scatter at this specialization
      )
      : params_(params),
        pointer_(reinterpret_cast<BytePointer>(
            const_cast<NonConstPointer>(pointer))),
        the_predicates(extent) {
    the_predicates.set_predicates(thread_id, threadblock_offset);

    // update internal pointers
    Layout layout(params_.stride_);
    add_pointer_offset(layout(the_predicates.thread_offset_));
  }

  /// Construct a PredicatedTileAccessIteratorResidualLast with zero threadblock
  /// offset
  CUTLASS_HOST_DEVICE
  PredicatedTileAccessIteratorResidualLast(
      Params const& params, ///< Precomputed parameters object
      Pointer pointer, ///< Pointer to start of tensor
      TensorCoord extent, ///< Extent of tensor
      int thread_id ///< ID of each participating thread
      )
      : PredicatedTileAccessIteratorResidualLast(
            params,
            pointer,
            extent,
            thread_id,
            make_Coord(0, 0)) {}

  /// Overrides the internal iteration index
  CUTLASS_HOST_DEVICE
  void set_iteration_index(int index) {
    the_predicates.set_iteration_index(index);
  }

  CUTLASS_HOST_DEVICE
  void set_residual_tile(bool is_residual_tile) {
    if (is_residual_tile) {
      the_predicates.set_mask(residual_tile_mask);
    }
  }

  /// Adds a pointer offset in units of Element
  CUTLASS_HOST_DEVICE
  void add_pointer_offset(LongIndex pointer_offset) {
    pointer_ += sizeof_bits<Element>::value * pointer_offset / 8;
  }

  /// Advances an iterator along logical dimensions of matrix in units of whole
  /// tiles
  CUTLASS_HOST_DEVICE
  void add_tile_offset(TensorCoord const& tile_offset) {
    if (kAdvanceRank) {
      pointer_ += params_.inc_advance_ * LongIndex(tile_offset[1]);
      pointer_ += Shape::kContiguous * tile_offset[0];
    } else {
      pointer_ += params_.inc_advance_ * LongIndex(tile_offset[0]);
      pointer_ += Shape::kStrided * tile_offset[1];
    }
  }

  /// Returns a pointer
  CUTLASS_HOST_DEVICE
  AccessType* get() const {
    return reinterpret_cast<AccessType*>(pointer_) +
        the_predicates.iteration_vector_;
  }

  /// Advances to the next tile in memory.
  ///
  /// The first time this method is called, predicates are updated, and the
  /// iterator's internal pointer is reverted to the first "steady state" tile.
  /// Subsequent calls are lightweight and must only update the internal
  /// pointer.
  CUTLASS_HOST_DEVICE
  PredicatedTileAccessIteratorResidualLast& operator++() {
    the_predicates.operator++();
    ++the_predicates.iteration_vector_;
    if (the_predicates.iteration_vector_ < kAccessesPerVector) {
      return *this;
    }

    the_predicates.iteration_vector_ = 0;
    ++the_predicates.iteration_contiguous_;

    if (the_predicates.iteration_contiguous_ <
        ThreadMap::Iterations::kContiguous) {
      pointer_ += params_.inc_contiguous_;
      return *this;
    }

    // Enter here only if (iteration_contiguous_ ==
    // ThreadMap::Iteration::kContiguous)
    the_predicates.iteration_contiguous_ = 0;
    ++the_predicates.iteration_strided_;

    if (the_predicates.iteration_strided_ < ThreadMap::Iterations::kStrided) {
      pointer_ += params_.inc_next_strided_;
      return *this;
    }

    // Enter here only if (iteration_stride_ == ThreadMap::Iteration::kStrided)
    // which means we enter the next tile.
    the_predicates.iteration_strided_ = 0;

    // advance to next tile
    pointer_ += params_.inc_next_;

    // now return to start tile - if the iterator is subsequently advanced, this
    // subtraction as well as the subsequent integer addition are both elided by
    // the compiler.
    pointer_ -= params_.inc_advance_;

    return *this;
  }

  /// Advances to the next tile in memory.
  ///
  /// The first time this method is called, predicates are updated, and the
  /// iterator's internal pointer is reverted to the first "steady state" tile.
  /// Subsequent calls are lightweight and must only update the internal
  /// pointer.
  CUTLASS_HOST_DEVICE
  PredicatedTileAccessIteratorResidualLast operator++(int) {
    PredicatedTileAccessIteratorResidualLast self(*this);
    operator++();
    return self;
  }

  /// Clears the predicate set efficiently
  CUTLASS_HOST_DEVICE
  void clear_mask(bool enable = true) {
    the_predicates.clear_mask(enable);
  }

  /// Clears the predicate set efficiently
  CUTLASS_HOST_DEVICE
  void enable_mask() {
    the_predicates.enable_mask();
  }

  /// Sets the predicate mask, overriding value stored in predicate iterator
  CUTLASS_HOST_DEVICE
  void set_mask(Mask const& mask) {
    the_predicates.set_mask(mask);
  }

  /// Gets the mask
  CUTLASS_HOST_DEVICE
  void get_mask(Mask& mask) {
    the_predicates.get_mask(mask);
  }

  /// Returns whether access is valid or not
  CUTLASS_HOST_DEVICE
  bool valid() {
    return the_predicates.valid();
  }
};
```
**EN**: L884-L1214 — Affine-rank-2 core specialization: implements generic 2-stride pointer increments, predicate state, and per-access traversal for layout::AffineRankN<2>.
**CN**: `L884-L1214` —— affine-rank-2 核心特化：为 `layout::AffineRankN<2>` 实现通用双 stride 指针增量、谓词状态和逐访问遍历。

### Lines 1215-1433
```cpp

////////////////////////////////////////////////////////////////////////////////

/// Specialization of PredicatedTileAccessIteratorResidualLast for affine rank 2
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
    typename AccessType_>
class PredicatedTileAccessIteratorResidualLast<
    Shape_,
    Element_,
    layout::AffineRank2ColumnMajor,
    AdvanceRank,
    ThreadMap_,
    AccessType_,
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
  using AccessType = AccessType_;

  using Index = typename Layout::Index;
  using LongIndex = typename Layout::LongIndex;

  using TensorRef = TensorRef<Element, Layout>;
  using TensorView = TensorView<Element, Layout>;
  using TensorCoord = typename Layout::TensorCoord;

  using Pointer = Element*;
  using NonConstPointer = typename platform::remove_const<Element>::type*;

  // Map to the underlying AffineRankN<2> layout
  using UnderlyingIterator = PredicatedTileAccessIteratorResidualLast<
      layout::PitchLinearShape<Shape::kRow, Shape::kColumn>,
      Element,
      layout::AffineRankN<2>,
      (kAdvanceRank == 0 ? 0 : 1),
      ThreadMap,
      AccessType>;

  static int const kAccessesPerVector = UnderlyingIterator::kAccessesPerVector;

  /// Predicate vector stores mask to guard accesses
  using Mask = typename UnderlyingIterator::Mask;

  /// Parameters object is precomputed state and is host-constructible
  class Params {
   private:
    friend PredicatedTileAccessIteratorResidualLast;

    /// Parameters object
    typename UnderlyingIterator::Params params_;

   public:
    /// Default ctor
    CUTLASS_HOST_DEVICE
    Params() {}

    /// Construct the Params object given an AffineRankN<2> tensor's layout
    CUTLASS_HOST_DEVICE
    Params(Layout const& layout)
        : params_(layout::AffineRankN<2>(layout.stride(0), layout.stride(1))){};
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
  PredicatedTileAccessIteratorResidualLast(
      ///< Precomputed parameters object
      Params const& params,
      ///< Pointer to start of tensor
      Pointer pointer,
      ///< Extent of tensor
      TensorCoord extent,
      ///< ID of each participating thread
      int thread_id,
      ///< Initial offset of threadblock
      TensorCoord const& threadblock_offset,
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

  /// Construct a PredicatedTileAccessIteratorResidualLast with zero threadblock
  /// offset
  CUTLASS_HOST_DEVICE
  PredicatedTileAccessIteratorResidualLast(
      Params const& params, ///< Precomputed parameters object
      Pointer pointer, ///< Pointer to start of tensor
      TensorCoord extent, ///< Extent of tensor
      int thread_id ///< ID of each participating thread
      )
      : PredicatedTileAccessIteratorResidualLast(
            params,
            pointer,
            extent,
            thread_id,
            make_Coord(0, 0)) {}

  /// Overrides the internal iteration index
  CUTLASS_HOST_DEVICE
  void set_iteration_index(int index) {
    iterator_.set_iteration_index(index);
  }

  CUTLASS_HOST_DEVICE
  void set_residual_tile(bool enable) {
    iterator_.set_residual_tile(enable);
  }

  /// Adds a pointer offset in units of Element
  CUTLASS_HOST_DEVICE
  void add_pointer_offset(LongIndex pointer_offset) {
    iterator_.add_pointer_offset(pointer_offset);
  }

  /// Advances an iterator along logical dimensions of matrix in units of whole
  /// tiles
  CUTLASS_HOST_DEVICE
  void add_tile_offset(TensorCoord const& tile_offset) {
    iterator_.add_tile_offset(
        make_Coord(tile_offset.row(), tile_offset.column()));
  }

  /// Returns a pointer
  CUTLASS_HOST_DEVICE
  AccessType* get() const {
    return reinterpret_cast<AccessType*>(iterator_.get());
  }

  /// Advances to the next tile in memory.
  ///
  /// The first time this method is called, predicates are updated, and the
  /// iterator's internal pointer is reverted to the first "steady state" tile.
  /// Subsequent calls are lightweight and must only update the internal
  /// pointer.
  CUTLASS_HOST_DEVICE
  PredicatedTileAccessIteratorResidualLast& operator++() {
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
  PredicatedTileAccessIteratorResidualLast operator++(int) {
    PredicatedTileAccessIteratorResidualLast self(*this);
    operator++();
    return self;
  }

  /// Clears the predicate set efficiently
  CUTLASS_HOST_DEVICE
  void clear_mask(bool enable = true) {
    iterator_.clear_mask(enable);
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

  /// Returns whether access is valid or not
  CUTLASS_HOST_DEVICE
  bool valid() {
    return iterator_.valid();
  }
};
```
**EN**: L1218-L1433 — Affine rank-2 column-major adapter: packages AffineRank2ColumnMajor as an AffineRankN<2> core iterator.
**CN**: `L1218-L1433` —— affine rank-2 列主适配器：把 `AffineRank2ColumnMajor` 封装到底层 `AffineRankN<2>` 核心迭代器上。

### Lines 1434-1652
```cpp

////////////////////////////////////////////////////////////////////////////////

/// Specialization of PredicatedTileAccessIteratorResidualLast for affine rank-2
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
    typename AccessType_>
class PredicatedTileAccessIteratorResidualLast<
    Shape_,
    Element_,
    layout::AffineRank2RowMajor,
    AdvanceRank,
    ThreadMap_,
    AccessType_,
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
  using AccessType = AccessType_;

  using Index = typename Layout::Index;
  using LongIndex = typename Layout::LongIndex;

  using TensorRef = TensorRef<Element, Layout>;
  using TensorView = TensorView<Element, Layout>;
  using TensorCoord = typename Layout::TensorCoord;

  using Pointer = Element*;
  using NonConstPointer = typename platform::remove_const<Element>::type*;

  // Map to the underlying AffineRankN<2> layout
  using UnderlyingIterator = PredicatedTileAccessIteratorResidualLast<
      layout::PitchLinearShape<Shape::kColumn, Shape::kRow>,
      Element,
      layout::AffineRankN<2>,
      (kAdvanceRank == 0 ? 1 : 0),
      ThreadMap,
      AccessType>;

  static int const kAccessesPerVector = UnderlyingIterator::kAccessesPerVector;

  /// Predicate vector stores mask to guard accesses
  using Mask = typename UnderlyingIterator::Mask;

  /// Parameters object is precomputed state and is host-constructible
  class Params {
   private:
    friend PredicatedTileAccessIteratorResidualLast;

    /// Parameters object
    typename UnderlyingIterator::Params params_;

   public:
    /// Default ctor
    CUTLASS_HOST_DEVICE
    Params() {}

    /// Construct the Params object given an AffineRankN<2> tensor's layout
    CUTLASS_HOST_DEVICE
    Params(Layout const& layout)
        : params_(layout::AffineRankN<2>(layout.stride(1), layout.stride(0))){};
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
  PredicatedTileAccessIteratorResidualLast(
      ///< Precomputed parameters object
      Params const& params,
      ///< Pointer to start of tensor
      Pointer pointer,
      ///< Extent of tensor
      TensorCoord extent,
      ///< ID of each participating thread
      int thread_id,
      ///< Initial offset of threadblock
      TensorCoord const& threadblock_offset,
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

  /// Construct a PredicatedTileAccessIteratorResidualLast with zero threadblock
  /// offset
  CUTLASS_HOST_DEVICE
  PredicatedTileAccessIteratorResidualLast(
      Params const& params, ///< Precomputed parameters object
      Pointer pointer, ///< Pointer to start of tensor
      TensorCoord extent, ///< Extent of tensor
      int thread_id ///< ID of each participating thread
      )
      : PredicatedTileAccessIteratorResidualLast(
            params,
            pointer,
            extent,
            thread_id,
            make_Coord(0, 0)) {}

  /// Overrides the internal iteration index
  CUTLASS_HOST_DEVICE
  void set_iteration_index(int index) {
    iterator_.set_iteration_index(index);
  }

  CUTLASS_HOST_DEVICE
  void set_residual_tile(bool enable) {
    iterator_.set_residual_tile(enable);
  }

  /// Adds a pointer offset in units of Element
  CUTLASS_HOST_DEVICE
  void add_pointer_offset(LongIndex pointer_offset) {
    iterator_.add_pointer_offset(pointer_offset);
  }

  /// Advances an iterator along logical dimensions of matrix in units of whole
  /// tiles
  CUTLASS_HOST_DEVICE
  void add_tile_offset(TensorCoord const& tile_offset) {
    iterator_.add_tile_offset(
        make_Coord(tile_offset.column(), tile_offset.row()));
  }

  /// Returns a pointer
  CUTLASS_HOST_DEVICE
  AccessType* get() const {
    return reinterpret_cast<AccessType*>(iterator_.get());
  }

  /// Advances to the next tile in memory.
  ///
  /// The first time this method is called, predicates are updated, and the
  /// iterator's internal pointer is reverted to the first "steady state" tile.
  /// Subsequent calls are lightweight and must only update the internal
  /// pointer.
  CUTLASS_HOST_DEVICE
  PredicatedTileAccessIteratorResidualLast& operator++() {
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
  PredicatedTileAccessIteratorResidualLast operator++(int) {
    PredicatedTileAccessIteratorResidualLast self(*this);
    operator++();
    return self;
  }

  /// Clears the predicate set efficiently
  CUTLASS_HOST_DEVICE
  void clear_mask(bool enable = true) {
    iterator_.clear_mask(enable);
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

  /// Returns whether access is valid or not
  CUTLASS_HOST_DEVICE
  bool valid() {
    return iterator_.valid();
  }
};
```
**EN**: L1437-L1652 — Affine rank-2 row-major adapter: same idea, with row/column swapped in the mapping.
**CN**: `L1437-L1652` —— affine rank-2 行主适配器：思路相同，但映射时会交换行/列。

### Lines 1653-1879
```cpp

////////////////////////////////////////////////////////////////////////////////

/// Specialization of PredicatedTileAccessIteratorResidualLast for column-major
/// interleaved data. It is mapped to the congruous layout.
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
    typename AccessType_,
    int InterleavedK>
class PredicatedTileAccessIteratorResidualLast<
    Shape_,
    Element_,
    layout::ColumnMajorInterleaved<InterleavedK>,
    AdvanceRank,
    ThreadMap_,
    AccessType_,
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
  using AccessType = AccessType_;

  using Index = typename Layout::Index;
  using LongIndex = typename Layout::LongIndex;

  using TensorRef = TensorRef<Element, Layout>;
  using TensorView = TensorView<Element, Layout>;
  using TensorCoord = typename Layout::TensorCoord;

  using Pointer = Element*;
  using NonConstPointer = typename platform::remove_const<Element>::type*;

  using UnderlyingIterator = PredicatedTileAccessIteratorResidualLast<
      layout::PitchLinearShape<
          Shape::kRow * kInterleavedK,
          Shape::kColumn / kInterleavedK>,
      Element,
      layout::PitchLinear,
      (kAdvanceRank == 0 ? 0 : 1),
      ThreadMap,
      AccessType>;

  static int const kAccessesPerVector = UnderlyingIterator::kAccessesPerVector;

  /// Predicate vector stores mask to guard accesses
  using Mask = typename UnderlyingIterator::Mask;

  /// Parameters object is precomputed state and is host-constructible
  class Params {
   private:
    friend PredicatedTileAccessIteratorResidualLast;

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
  PredicatedTileAccessIteratorResidualLast(
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

  /// Construct a PredicatedTileAccessIteratorResidualLast with zero threadblock
  /// offset
  CUTLASS_HOST_DEVICE
  PredicatedTileAccessIteratorResidualLast(
      Params const& params, ///< Precomputed parameters object
      Pointer pointer, ///< Pointer to start of tensor
      TensorCoord extent, ///< Extent of tensor
      int thread_id ///< ID of each participating thread
      )
      : PredicatedTileAccessIteratorResidualLast(
            params,
            pointer,
            extent,
            thread_id,
            make_Coord(0, 0)) {}

  /// Overrides the internal iteration index
  CUTLASS_HOST_DEVICE
  void set_iteration_index(int index) {
    iterator_.set_iteration_index(index);
  }

  CUTLASS_HOST_DEVICE
  void set_residual_tile(bool enable) {
    iterator_.set_residual_tile(enable);
  }

  /// Adds a pointer offset in units of Element
  CUTLASS_HOST_DEVICE
  void add_pointer_offset(LongIndex pointer_offset) {
    iterator_.add_pointer_offset(pointer_offset);
  }

  /// Advances an iterator along logical dimensions of matrix in units of whole
  /// tiles
  CUTLASS_HOST_DEVICE
  void add_tile_offset(TensorCoord const& tile_offset) {
    iterator_.add_tile_offset({tile_offset.row(), tile_offset.column()});
  }

  /// Returns a pointer
  CUTLASS_HOST_DEVICE
  AccessType* get() const {
    return reinterpret_cast<AccessType*>(iterator_.get());
  }

  /// Advances to the next tile in memory.
  ///
  /// The first time this method is called, predicates are updated, and the
  /// iterator's internal pointer is reverted to the first "steady state" tile.
  /// Subsequent calls are lightweight and must only update the internal
  /// pointer.
  CUTLASS_HOST_DEVICE
  PredicatedTileAccessIteratorResidualLast& operator++() {
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
  PredicatedTileAccessIteratorResidualLast operator++(int) {
    PredicatedTileAccessIteratorResidualLast self(*this);
    operator++();
    return self;
  }

  /// Clears the predicate set efficiently
  CUTLASS_HOST_DEVICE
  void clear_mask(bool enable = true) {
    iterator_.clear_mask(enable);
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

  /// Returns whether access is valid or not
  CUTLASS_HOST_DEVICE
  bool valid() {
    return iterator_.valid();
  }
};
```
**EN**: L1656-L1879 — Column-major interleaved adapter: maps interleaved coordinates to a congruous pitch-linear shape scaled by InterleavedK.
**CN**: `L1656-L1879` —— 列主交错布局适配器：把交错坐标映射为按 `InterleavedK` 缩放后的 congruous pitch-linear 形状。

### Lines 1880-2106
```cpp

////////////////////////////////////////////////////////////////////////////////

/// Specialization of PredicatedTileAccessIteratorResidualLast for row-major
/// interleaved data.
//  It is mapped to the congruous layout.
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
    typename AccessType_,
    int InterleavedK>
class PredicatedTileAccessIteratorResidualLast<
    Shape_,
    Element_,
    layout::RowMajorInterleaved<InterleavedK>,
    AdvanceRank,
    ThreadMap_,
    AccessType_,
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
  using AccessType = AccessType_;

  using Index = typename Layout::Index;
  using LongIndex = typename Layout::LongIndex;

  using TensorRef = TensorRef<Element, Layout>;
  using TensorView = TensorView<Element, Layout>;
  using TensorCoord = typename Layout::TensorCoord;

  using Pointer = Element*;
  using NonConstPointer = typename platform::remove_const<Element>::type*;

  using UnderlyingIterator = PredicatedTileAccessIteratorResidualLast<
      layout::PitchLinearShape<
          Shape::kColumn * kInterleavedK,
          Shape::kRow / kInterleavedK>,
      Element,
      layout::PitchLinear,
      (kAdvanceRank == 0 ? 1 : 0),
      ThreadMap,
      AccessType>;

  static int const kAccessesPerVector = UnderlyingIterator::kAccessesPerVector;

  /// Predicate vector stores mask to guard accesses
  using Mask = typename UnderlyingIterator::Mask;

  /// Parameters object is precomputed state and is host-constructible
  class Params {
   private:
    friend PredicatedTileAccessIteratorResidualLast;

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
  PredicatedTileAccessIteratorResidualLast(
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

  /// Construct a PredicatedTileAccessIteratorResidualLast with zero threadblock
  /// offset
  CUTLASS_HOST_DEVICE
  PredicatedTileAccessIteratorResidualLast(
      Params const& params, ///< Precomputed parameters object
      Pointer pointer, ///< Pointer to start of tensor
      TensorCoord extent, ///< Extent of tensor
      int thread_id ///< ID of each participating thread
      )
      : PredicatedTileAccessIteratorResidualLast(
            params,
            pointer,
            extent,
            thread_id,
            make_Coord(0, 0)) {}

  /// Overrides the internal iteration index
  CUTLASS_HOST_DEVICE
  void set_iteration_index(int index) {
    iterator_.set_iteration_index(index);
  }

  CUTLASS_HOST_DEVICE
  void set_residual_tile(bool enable) {
    iterator_.set_residual_tile(enable);
  }

  /// Adds a pointer offset in units of Element
  CUTLASS_HOST_DEVICE
  void add_pointer_offset(LongIndex pointer_offset) {
    iterator_.add_pointer_offset(pointer_offset);
  }

  /// Advances an iterator along logical dimensions of matrix in units of whole
  /// tiles
  CUTLASS_HOST_DEVICE
  void add_tile_offset(TensorCoord const& tile_offset) {
    iterator_.add_tile_offset({tile_offset.column(), tile_offset.row()});
  }

  /// Returns a pointer
  CUTLASS_HOST_DEVICE
  AccessType* get() const {
    return reinterpret_cast<AccessType*>(iterator_.get());
  }

  /// Advances to the next tile in memory.
  ///
  /// The first time this method is called, predicates are updated, and the
  /// iterator's internal pointer is reverted to the first "steady state" tile.
  /// Subsequent calls are lightweight and must only update the internal
  /// pointer.
  CUTLASS_HOST_DEVICE
  PredicatedTileAccessIteratorResidualLast& operator++() {
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
  PredicatedTileAccessIteratorResidualLast operator++(int) {
    PredicatedTileAccessIteratorResidualLast self(*this);
    operator++();
    return self;
  }

  /// Clears the predicate set efficiently
  CUTLASS_HOST_DEVICE
  void clear_mask(bool enable = true) {
    iterator_.clear_mask(enable);
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

  /// Returns whether access is valid or not
  CUTLASS_HOST_DEVICE
  bool valid() {
    return iterator_.valid();
  }
};
```
**EN**: L1883-L2106 — Row-major interleaved adapter: the row-major interleaved counterpart with analogous coordinate reshaping.
**CN**: `L1883-L2106` —— 行主交错布局适配器：与上面对应，只是针对 row-major interleaved 做类似的坐标重塑。

### Lines 2107-2114
```cpp

////////////////////////////////////////////////////////////////////////////////

} // namespace threadblock
} // namespace transform
} // namespace cutlass

////////////////////////////////////////////////////////////////////////////////
```
**EN**: L2110-L2114 — Namespace closing.
**CN**: `L2110-L2114` —— 命名空间结束。

---
## Key Concepts / 关键概念
- **EN**: The primary template is `PredicatedTileAccessIteratorResidualLast<Shape, Element, Layout, AdvanceRank, ThreadMap, AccessType, Gather>`.
  **CN**: 主模板是 `PredicatedTileAccessIteratorResidualLast<Shape, Element, Layout, AdvanceRank, ThreadMap, AccessType, Gather>`。
- **EN**: `Shape` gives the logical tile size, `AdvanceRank` chooses which tensor dimension is the tile-to-tile marching direction, and `ThreadMap` defines each thread’s access pattern inside the tile.
  **CN**: `Shape` 给出逻辑 tile 尺寸，`AdvanceRank` 指定 tile 级推进维度，`ThreadMap` 定义线程在 tile 内部的访问分布。
- **EN**: `AccessType` is the vectorized transaction type, while `Gather` toggles the special indexed-strided path that uses `indices_` instead of a dense strided offset.
  **CN**: `AccessType` 是向量化事务类型，`Gather` 控制特殊的索引化 strided 路径：此时会使用 `indices_`，而不是规则的连续 stride 偏移。
- **EN**: The pitch-linear core reuses `PredicatedTileAccessIteratorParams` and `PredicatedTileAccessIteratorPredicates`; affine rank-2 defines its own richer `Params` because both dimensions may have arbitrary strides.
  **CN**: pitch-linear 核心直接复用 `PredicatedTileAccessIteratorParams` 与 `PredicatedTileAccessIteratorPredicates`；而 affine rank-2 因为两个维度都可能是任意 stride，所以定义了更丰富的 `Params`。
- **EN**: In the pitch-linear core (`L92-L432`), the important state is `the_predicates`, `residual_tile_mask`, `params_`, `pointer_`, `indices_`, and `gather_offset_strided`.
  **CN**: 在 pitch-linear 核心实现（`L92-L432`）中，关键状态包括 `the_predicates`、`residual_tile_mask`、`params_`、`pointer_`、`indices_` 与 `gather_offset_strided`。
- **EN**: `the_predicates` owns iteration counters (`iteration_vector_`, `iteration_contiguous_`, `iteration_strided_`) and packed validity bits; `residual_tile_mask` snapshots the tail-tile predicate state.
  **CN**: `the_predicates` 持有迭代计数器（`iteration_vector_`、`iteration_contiguous_`、`iteration_strided_`）以及压缩后的有效性位；`residual_tile_mask` 则保存尾块的谓词快照。
- **EN**: The affine-rank-2 core (`L898-L1214`) keeps similar logical state, but its `Params` also stores `stride_[2]`, `inc_contiguous_`, `inc_strided_`, `inc_next_strided_`, `inc_next_`, and `inc_advance_` to support general 2D affine layouts.
  **CN**: affine-rank-2 核心（`L898-L1214`）保留了类似的逻辑状态，但其 `Params` 额外保存 `stride_[2]`、`inc_contiguous_`、`inc_strided_`、`inc_next_strided_`、`inc_next_` 与 `inc_advance_`，以支持通用二维 affine 布局。
- **EN**: All adapter specializations only store `UnderlyingIterator iterator_`; they translate extents, threadblock offsets, and tile offsets, then forward every operation.
  **CN**: 所有适配型特化都只保存 `UnderlyingIterator iterator_`；它们负责转换 extent、threadblock offset 和 tile offset，然后把所有操作转发下去。
## Dependencies / 依赖项
- `cutlass/array.h` — Direct include used by this file / 本文件直接包含并依赖的头文件
- `cutlass/coord.h` — Direct include used by this file / 本文件直接包含并依赖的头文件
- `cutlass/cutlass.h` — Direct include used by this file / 本文件直接包含并依赖的头文件
- `cutlass/layout/matrix.h` — Direct include used by this file / 本文件直接包含并依赖的头文件
- `cutlass/layout/pitch_linear.h` — Direct include used by this file / 本文件直接包含并依赖的头文件
- `cutlass/matrix_shape.h` — Direct include used by this file / 本文件直接包含并依赖的头文件
- `cutlass/predicate_vector.h` — Direct include used by this file / 本文件直接包含并依赖的头文件
- `cutlass/tensor_ref.h` — Direct include used by this file / 本文件直接包含并依赖的头文件
- `cutlass/tensor_view.h` — Direct include used by this file / 本文件直接包含并依赖的头文件
- `cutlass/transform/threadblock/predicated_tile_access_iterator_params.h` — Direct include used by this file / 本文件直接包含并依赖的头文件
