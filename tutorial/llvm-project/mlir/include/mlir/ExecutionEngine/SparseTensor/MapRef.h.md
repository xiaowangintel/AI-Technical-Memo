# MapRef.h — Code Analysis / 代码分析

## Source / 来源

- **File**: `mlir/include/mlir/ExecutionEngine/SparseTensor/MapRef.h`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Declares C++ interfaces and helper types for the MLIR MapRef component. The leading comments describe it as: A dim2lvl/lvl2dim map encoding class, with utility methods.
- **用途（CN）**: 声明 MLIR MapRef 组件相关的 C++ 接口与辅助类型。 文件开头的注释还对该职责进行了补充说明。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18
````cpp
//===- MapRef.h - A dim2lvl/lvl2dim map encoding ----------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// A dim2lvl/lvl2dim map encoding class, with utility methods.
//
//===----------------------------------------------------------------------===//

#ifndef MLIR_EXECUTIONENGINE_SPARSETENSOR_MAPREF_H
#define MLIR_EXECUTIONENGINE_SPARSETENSOR_MAPREF_H

#include <cinttypes>

#include <cassert>
````
- **EN**: This block establishes the file guard and imports the MLIR/LLVM/TableGen dependencies that the rest of the file builds on.
- **CN**: 该代码块同时建立文件保护宏，并引入后续实现所依赖的 MLIR/LLVM/TableGen 头文件或记录文件。

### Lines 20-79
````cpp
namespace mlir {
namespace sparse_tensor {

/// A class for capturing the sparse tensor type map with a compact encoding.
///
/// Currently, the following situations are supported:
///   (1) map is a permutation
///   (2) map has affine ops (restricted set)
///
/// The pushforward/backward operations are fast for (1) but incur some obvious
/// overhead for situation (2).
///
class MapRef final {
public:
  MapRef(uint64_t d, uint64_t l, const uint64_t *d2l, const uint64_t *l2d);

  //
  // Push forward maps from dimensions to levels.
  //

  // Map from dimRank in to lvlRank out.
  template <typename T>
  inline void pushforward(const T *in, T *out) const {
    if (isPermutation) {
      for (uint64_t l = 0; l < lvlRank; l++) {
        out[l] = in[dim2lvl[l]];
      }
    } else {
      uint64_t i, c;
      for (uint64_t l = 0; l < lvlRank; l++)
        if (isFloor(l, i, c)) {
          out[l] = in[i] / c;
        } else if (isMod(l, i, c)) {
          out[l] = in[i] % c;
        } else {
          out[l] = in[dim2lvl[l]];
        }
    }
  }

  //
  // Push backward maps from levels to dimensions.
  //

  // Map from lvlRank in to dimRank out.
  template <typename T>
  inline void pushbackward(const T *in, T *out) const {
    if (isPermutation) {
      for (uint64_t d = 0; d < dimRank; d++)
        out[d] = in[lvl2dim[d]];
    } else {
      uint64_t i, c, ii;
      for (uint64_t d = 0; d < dimRank; d++)
        if (isMul(d, i, c, ii)) {
          out[d] = in[i] + c * in[ii];
        } else {
          out[d] = in[lvl2dim[d]];
        }
    }
  }
````
- **EN**: This C++ declaration introduces `MapRef` and establishes part of the API surface for `MapRef`. Representative entry points here include `MapRef`, `pushforward`, `isFloor`, `isMod`.
- **CN**: 该 C++ 声明引入了 `MapRef`，并构成 `MapRef` API 表面的一部分。 这一段可见的代表性接口包括 `MapRef`, `pushforward`, `isFloor`, `isMod`。

### Lines 80-98
````cpp
  uint64_t getDimRank() const { return dimRank; }
  uint64_t getLvlRank() const { return lvlRank; }

private:
  bool isPermutationMap() const;

  bool isFloor(uint64_t l, uint64_t &i, uint64_t &c) const;
  bool isMod(uint64_t l, uint64_t &i, uint64_t &c) const;
  bool isMul(uint64_t d, uint64_t &i, uint64_t &c, uint64_t &ii) const;

  const uint64_t dimRank;
  const uint64_t lvlRank;
  const uint64_t *const dim2lvl; // non-owning pointer
  const uint64_t *const lvl2dim; // non-owning pointer
  const bool isPermutation;
};

} // namespace sparse_tensor
} // namespace mlir
````
- **EN**: This block groups callable interfaces such as `getDimRank`, `getLvlRank`, `isPermutationMap`, `isFloor`, indicating how `MapRef` is queried or updated.
- **CN**: 该代码块聚合了 `getDimRank`, `getLvlRank`, `isPermutationMap`, `isFloor` 等可调用接口，展示了如何查询或更新 `MapRef`。

### Lines 101-101
````cpp
#endif //  MLIR_EXECUTIONENGINE_SPARSETENSOR_MAPREF_H
````
- **EN**: This block manages the file guard so the header or TableGen fragment is only processed once per translation or inclusion path.
- **CN**: 该代码块管理文件保护宏，确保头文件或 TableGen 片段在一次编译/包含路径中只被处理一次。

## Key Concepts / 关键概念

- **EN**: C++ declaration surface for MLIR infrastructure
  **CN**: MLIR 基础设施的 C++ 声明层

## Dependencies / 依赖关系

- Primarily relies on nearby MLIR declarations surrounding MapRef.h.
