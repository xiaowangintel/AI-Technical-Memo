# fmha_fusion.hpp — Code Analysis / 代码分析

**Source / 源文件**: `examples/77_blackwell_fmha/collective/fmha_fusion.hpp`  
**Purpose / 用途**: Defines mask and variable-length helpers that let FMHA kernels share one masking interface across normal, residual-tail, causal, backward, and varlen paths. / 定义 mask 与变长辅助，使 FMHA kernel 在普通、残块尾部、因果、反向和变长路径上共享统一的掩码接口。

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
```

**EN**: This code block advances the file's control flow or type system in a focused step. In CUTLASS examples, even seemingly local code usually exists to reconcile runtime problem metadata with a heavily templated compile-time kernel configuration. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一代码块以一个较集中的步骤推进文件的控制流或类型系统。在 CUTLASS 示例中，即便看似局部的代码，通常也是为了把运行期问题元数据和高度模板化的编译期内核配置对齐起来。属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

```cpp
#include "cutlass/cutlass.h"
#include "cute/tensor.hpp"

namespace cutlass::fmha::collective {

using namespace cute;

struct NoMask {
  template<class BlkCoord, class TileShape, class ProblemSize>
  CUTLASS_DEVICE
  int get_trip_count(
      BlkCoord const& blk_coord,
      TileShape const& tile_shape,
      ProblemSize const& problem_size) {

    return ceil_div(get<1>(problem_size), get<1>(tile_shape));
  }

  template<class BlkCoord, class TileShape, class ProblemSize>
  CUTLASS_DEVICE
  int get_masked_trip_count(
      BlkCoord const& blk_coord,
      TileShape const& tile_shape,
      ProblemSize const& problem_size) {

    return 0;
  }

  template<class BlkCoord, class TileShape, class ProblemSize>
  CUTLASS_DEVICE
  int get_unmasked_trip_count(
      BlkCoord const& blk_coord,
      TileShape const& tile_shape,
      ProblemSize const& problem_size) {

    return get_trip_count(blk_coord, tile_shape, problem_size);
  }
```

**EN**: This code block advances the file's control flow or type system in a focused step. In CUTLASS examples, even seemingly local code usually exists to reconcile runtime problem metadata with a heavily templated compile-time kernel configuration. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一代码块以一个较集中的步骤推进文件的控制流或类型系统。在 CUTLASS 示例中，即便看似局部的代码，通常也是为了把运行期问题元数据和高度模板化的编译期内核配置对齐起来。属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

```cpp
  template<class AccQK, class IndexQK, class ProblemSize>
  CUTLASS_DEVICE
  void apply_mask(
      AccQK& acc_qk,
      IndexQK const& index_qk,
      ProblemSize const& problem_size) {

    return;
  }
};

struct ResidualMask : NoMask {

  using Base = NoMask;

  template <class BlkCoord, class TileShape, class ProblemSize>
  CUTLASS_DEVICE int get_masked_trip_count(
      BlkCoord const& blk_coord,
      TileShape const& tile_shape,
      ProblemSize const& problem_size) {

    if (get<1>(problem_size) % get<1>(tile_shape) != 0) {
      return 1;
    }
    return 0;
  }

  template<class BlkCoord, class TileShape, class ProblemSize>
  CUTLASS_DEVICE
  int get_unmasked_trip_count(
      BlkCoord const& blk_coord,
      TileShape const& tile_shape,
      ProblemSize const& problem_size) {

    // if the sequence length does not divide the tile size evenly
    if (get<1>(problem_size) % get<1>(tile_shape) != 0) {
      return get_trip_count(blk_coord, tile_shape, problem_size) - 1;
    }
    return get_trip_count(blk_coord, tile_shape, problem_size);
  }
```

**EN**: This code block advances the file's control flow or type system in a focused step. In CUTLASS examples, even seemingly local code usually exists to reconcile runtime problem metadata with a heavily templated compile-time kernel configuration. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一代码块以一个较集中的步骤推进文件的控制流或类型系统。在 CUTLASS 示例中，即便看似局部的代码，通常也是为了把运行期问题元数据和高度模板化的编译期内核配置对齐起来。属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

```cpp
  template<class AccQK, class IndexQK, class ProblemSize>
  CUTLASS_DEVICE
  void apply_mask(
      AccQK& acc_qk,
      IndexQK const& index_qk,
      ProblemSize const& problem_size) {

    // This is useful is seqlen_k % kBlockN != 0 since it masks
    // the remaining elements out from softmax.
    // d % kHeadDim != 0 or seqlen_q % kBlockM do not suffer from similar
    // issues as they are transparently taken care of by TMA and the
    // epilogue, if it is instantiated with predication support.
    CUTLASS_PRAGMA_UNROLL
    for (int i = 0; i < size(acc_qk); i++) {
      auto pos = index_qk(i);
      if (get<1>(pos) >= get<1>(problem_size)) {
        acc_qk(i) = -INFINITY;
      }
    }
  }
};

struct ResidualMaskForBackward : NoMask {

  using Base = NoMask;

  template <class BlkCoord, class TileShape, class ProblemSize>
  CUTLASS_DEVICE int get_masked_trip_count(
      BlkCoord const& blk_coord,
      TileShape const& tile_shape,
      ProblemSize const& problem_size) {

    if (get<1>(problem_size) % get<1>(tile_shape) != 0) {
      return 1;
    }
    return 0;
  }
```

**EN**: This code block advances the file's control flow or type system in a focused step. In CUTLASS examples, even seemingly local code usually exists to reconcile runtime problem metadata with a heavily templated compile-time kernel configuration. Relies on tma-style bulk movement or descriptor handling. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一代码块以一个较集中的步骤推进文件的控制流或类型系统。在 CUTLASS 示例中，即便看似局部的代码，通常也是为了把运行期问题元数据和高度模板化的编译期内核配置对齐起来。依赖 TMA 风格的批量搬运或描述符处理。 属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

```cpp
  template<class BlkCoord, class TileShape, class ProblemSize>
  CUTLASS_DEVICE
  int get_unmasked_trip_count(
      BlkCoord const& blk_coord,
      TileShape const& tile_shape,
      ProblemSize const& problem_size) {

    // if the sequence length does not divide the tile size evenly
    if (get<1>(problem_size) % get<1>(tile_shape) != 0) {
      return get_trip_count(blk_coord, tile_shape, problem_size) - 1;
    }
    return get_trip_count(blk_coord, tile_shape, problem_size);
  }

  template<class AccQK, class IndexQK, class ProblemSize>
  CUTLASS_DEVICE
  void apply_mask(
      AccQK& acc_qk,
      IndexQK const& index_qk,
      ProblemSize const& problem_size) {

    // This is useful is seqlen_k % kBlockN != 0 since it masks
    // the remaining elements out from softmax.
    // d % kHeadDim != 0 or seqlen_q % kBlockM do not suffer from similar
    // issues as they are transparently taken care of by TMA and the
    // epilogue, if it is instantiated with predication support.
    CUTLASS_PRAGMA_UNROLL
    for (int i = 0; i < size(acc_qk); i++) {
      auto pos = index_qk(i);
      if (! elem_less(pos, select<0,1>(problem_size))) {
        acc_qk(i) = -INFINITY;
      }
    }
  }
};

// There are two ways to do causal if N_Q != N_K
// (1) The Q is at the beginning of the matrix
// (2) The Q is at the end of the matrix
template<bool kIsQBegin = true>
struct CausalMask : NoMask {
```

**EN**: This code block advances the file's control flow or type system in a focused step. In CUTLASS examples, even seemingly local code usually exists to reconcile runtime problem metadata with a heavily templated compile-time kernel configuration. Relies on tma-style bulk movement or descriptor handling. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一代码块以一个较集中的步骤推进文件的控制流或类型系统。在 CUTLASS 示例中，即便看似局部的代码，通常也是为了把运行期问题元数据和高度模板化的编译期内核配置对齐起来。依赖 TMA 风格的批量搬运或描述符处理。 属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

```cpp
  using Base = NoMask;

  static constexpr bool IsQBegin = kIsQBegin;

  template<class BlkCoord, class TileShape, class ProblemSize>
  CUTLASS_DEVICE
  int get_trip_count(
      BlkCoord const& blk_coord,
      TileShape const& tile_shape,
      ProblemSize const& problem_size) {

    // See note below on different ways to think about causal attention
    // Again, we'd add the offset_q into the max_blocks_q calculation
    int max_blocks_k = Base::get_trip_count(blk_coord, tile_shape, problem_size);
    if constexpr (IsQBegin) {
      int max_blocks_q = ceil_div((get<0>(blk_coord) + 1) * get<0>(tile_shape), get<1>(tile_shape));
      return std::min(max_blocks_k, max_blocks_q);
    } else {
      const int offset_q = get<1>(problem_size) - get<0>(problem_size);
      int max_blocks_q = ceil_div((get<0>(blk_coord) + 1) * get<0>(tile_shape) + offset_q, get<1>(tile_shape));
      return std::min(max_blocks_k, max_blocks_q);
    }
  }

  template<class BlkCoord, class TileShape, class ProblemSize>
  CUTLASS_DEVICE
  int get_masked_trip_count(
      BlkCoord const& blk_coord,
      TileShape const& tile_shape,
      ProblemSize const& problem_size) {
        
    int trip_count = get_trip_count(blk_coord, tile_shape, problem_size);
    if constexpr (IsQBegin) {
      return std::min(trip_count, int(ceil_div(size<0>(tile_shape), size<1>(tile_shape))));
    } else {
      const int offset_tile_q = (get<1>(problem_size) - get<0>(problem_size)) % get<1>(tile_shape);
      return std::min(trip_count, int(ceil_div(get<0>(tile_shape) + offset_tile_q, get<1>(tile_shape))));
    }
  }
```

**EN**: This code block advances the file's control flow or type system in a focused step. In CUTLASS examples, even seemingly local code usually exists to reconcile runtime problem metadata with a heavily templated compile-time kernel configuration. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一代码块以一个较集中的步骤推进文件的控制流或类型系统。在 CUTLASS 示例中，即便看似局部的代码，通常也是为了把运行期问题元数据和高度模板化的编译期内核配置对齐起来。属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

```cpp
  template<class BlkCoord, class TileShape, class ProblemSize>
  CUTLASS_DEVICE
  int get_unmasked_trip_count(
      BlkCoord const& blk_coord,
      TileShape const& tile_shape,
      ProblemSize const& problem_size) {

    return get_trip_count(blk_coord, tile_shape, problem_size) - get_masked_trip_count(blk_coord, tile_shape, problem_size);
  }

  template<class AccQK, class IndexQK, class ProblemSize>
  CUTLASS_DEVICE
  void apply_mask(
      AccQK& acc_qk,
      IndexQK const& index_qk,
      ProblemSize const& problem_size) {

    // There are two ways to do causal if N_Q != N_K
    // (1) is to assume that the Q is at the beginning of the matrix
    //    - this is the default setting.
    // (2) is that it is at the end of the matrix
    //    - this is usually what we want for inference settings
    //      where we only compute the next row and use cache for the rest
    //    - if you'd like this, you only need to set kIsQBegin=false
```

**EN**: This code block advances the file's control flow or type system in a focused step. In CUTLASS examples, even seemingly local code usually exists to reconcile runtime problem metadata with a heavily templated compile-time kernel configuration. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一代码块以一个较集中的步骤推进文件的控制流或类型系统。在 CUTLASS 示例中，即便看似局部的代码，通常也是为了把运行期问题元数据和高度模板化的编译期内核配置对齐起来。属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

```cpp
    if constexpr (IsQBegin) {
      CUTLASS_PRAGMA_UNROLL
      for (int i = 0; i < size(acc_qk); i++) {
        auto pos = index_qk(i);
        if ((get<0>(pos) < get<1>(pos)) || (get<1>(pos) >= get<1>(problem_size))) {
          acc_qk(i) = -INFINITY;
        }
      }
    } else {
      const auto offset_q = get<1>(problem_size) - get<0>(problem_size);
      CUTLASS_PRAGMA_UNROLL
      for (int i = 0; i < size(acc_qk); i++) {
        auto pos = index_qk(i);
        if ((get<0>(pos) + offset_q < get<1>(pos)) || (get<1>(pos) >= get<1>(problem_size))) {
          acc_qk(i) = -INFINITY;
        }
      }
    }
  }
};

template<bool kIsQBegin = true>
struct CausalForBackwardMask : CausalMask<kIsQBegin>, ResidualMaskForBackward {

  using Base = CausalMask<kIsQBegin>;

  template<class AccQK, class IndexQK, class ProblemSize>
  CUTLASS_DEVICE
  void apply_mask(
      AccQK& acc_qk,
      IndexQK const& index_qk,
      ProblemSize const& problem_size) {

    // There are two ways to do causal if N_Q != N_K
    // (1) is to assume that the Q is at the beginning of the matrix
    //    - this is what we demonstrate here
    // (2) is that it is at the end of the matrix
    //    - this is usually what we want for inference settings
    //      where we only compute the next row and use cache for the rest
    //    - if you'd like this, you only need to add an offset like so:
    //      get<0>(pos) + offset_q < get<1>(pos)
```

**EN**: This code block advances the file's control flow or type system in a focused step. In CUTLASS examples, even seemingly local code usually exists to reconcile runtime problem metadata with a heavily templated compile-time kernel configuration. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一代码块以一个较集中的步骤推进文件的控制流或类型系统。在 CUTLASS 示例中，即便看似局部的代码，通常也是为了把运行期问题元数据和高度模板化的编译期内核配置对齐起来。属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

```cpp
    int offset_q = 0;
    if constexpr (!kIsQBegin) {
      offset_q = get<1>(problem_size) - get<0>(problem_size);
    }

    CUTLASS_PRAGMA_UNROLL
    for (int i = 0; i < size(acc_qk); i++) {
      auto pos = index_qk(i);
      bool masked = (get<0>(pos) + offset_q < get<1>(pos)) || !elem_less(pos, problem_size);
      if (masked) {
        acc_qk(i) = -INFINITY;
      }
    }
  }

};

struct VariableLength {
  int max_length;
  int* cumulative_length = nullptr;
  int total_length = -1;

  CUTE_HOST_DEVICE operator int() const {
    return max_length;
  }
};

template<class T> struct is_variable_length_impl : std::false_type {};
template<> struct is_variable_length_impl<VariableLength> : std::true_type {};
template<class T> constexpr bool is_variable_length_v = is_variable_length_impl<remove_cvref_t<T>>::value;
```

**EN**: This code block advances the file's control flow or type system in a focused step. In CUTLASS examples, even seemingly local code usually exists to reconcile runtime problem metadata with a heavily templated compile-time kernel configuration. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一代码块以一个较集中的步骤推进文件的控制流或类型系统。在 CUTLASS 示例中，即便看似局部的代码，通常也是为了把运行期问题元数据和高度模板化的编译期内核配置对齐起来。属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

```cpp
template<class Shape, class Idx>
CUTE_HOST_DEVICE
constexpr auto
apply_variable_length(Shape const& shape, Idx const& idx) {
  return transform_leaf(shape, [&](auto const& s) {
    if constexpr (is_variable_length_v<decltype(s)>) {
      return s.cumulative_length[idx+1] - s.cumulative_length[idx];
    }
    else {
      return s;
    }
  });
}

template<class Shape, class Coord, class Idx>
CUTE_HOST_DEVICE
constexpr auto
apply_variable_length(Shape const& shape, Coord const& coord, Idx const& idx) {
  auto new_shape = apply_variable_length(shape, idx);
  auto new_coord = transform_leaf(shape, coord, [&](auto const& s, auto const& c) {
    if constexpr (is_variable_length_v<decltype(s)>) {
      return cute::make_tuple(c, s.cumulative_length[idx]);
    }
    else {
      return c;
    }
  });
  return cute::make_tuple(new_shape, new_coord);
}
```

**EN**: This code block advances the file's control flow or type system in a focused step. In CUTLASS examples, even seemingly local code usually exists to reconcile runtime problem metadata with a heavily templated compile-time kernel configuration. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一代码块以一个较集中的步骤推进文件的控制流或类型系统。在 CUTLASS 示例中，即便看似局部的代码，通常也是为了把运行期问题元数据和高度模板化的编译期内核配置对齐起来。属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

```cpp
template<class Shape, class Coord>
CUTE_HOST_DEVICE
constexpr auto
apply_variable_length_offset(Shape const& shape, Coord const& coord) {
  auto idx = back(back(coord));
  auto result_shape = transform_leaf(shape, [&](auto const& s) {
    if constexpr (is_variable_length_v<decltype(s)>) {
      return s.cumulative_length[idx+1] - s.cumulative_length[idx];
    }
    else {
      return s;
    }
  });
  auto result_offset = transform_leaf(coord, shape, [&](auto const& c, auto const& s) {
    if constexpr (is_variable_length_v<decltype(s)>) {
      return s.cumulative_length[idx];
    }
    else {
      return _0{};
    }
  });
  return cute::make_tuple(result_shape, result_offset);
}

}  // namespace cutlass::fmha::collective

namespace cute {

template<>
struct is_integral<cutlass::fmha::collective::VariableLength> : true_type {};

CUTE_HOST_DEVICE
void print(cutlass::fmha::collective::VariableLength a) {
  printf("Varlen<%d, %p>", a.max_length, a.cumulative_length);
}
```

**EN**: This code block advances the file's control flow or type system in a focused step. In CUTLASS examples, even seemingly local code usually exists to reconcile runtime problem metadata with a heavily templated compile-time kernel configuration. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一代码块以一个较集中的步骤推进文件的控制流或类型系统。在 CUTLASS 示例中，即便看似局部的代码，通常也是为了把运行期问题元数据和高度模板化的编译期内核配置对齐起来。属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

```cpp
}
```

**EN**: This code block advances the file's control flow or type system in a focused step. In CUTLASS examples, even seemingly local code usually exists to reconcile runtime problem metadata with a heavily templated compile-time kernel configuration. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一代码块以一个较集中的步骤推进文件的控制流或类型系统。在 CUTLASS 示例中，即便看似局部的代码，通常也是为了把运行期问题元数据和高度模板化的编译期内核配置对齐起来。属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

## Key Concepts / 关键概念

- Architecture-tagged specialization (`Sm100`/`Sm103`/`Sm120`) / 基于架构标签的特化（`Sm100`/`Sm103`/`Sm120`）
- TMA-driven data movement / 基于 TMA 的数据搬运
- Warp-specialized FMHA layering across collective, kernel, and device wrappers / 跨 collective、kernel 与 device wrapper 的 warp-specialized FMHA 分层

## Dependencies / 依赖项

- `cutlass/cutlass.h` — core CUTLASS types, architecture tags, and utilities / CUTLASS 核心类型、架构标签与工具
- `cute/tensor.hpp` — CuTe tensor and layout primitives / CuTe 张量与布局原语
