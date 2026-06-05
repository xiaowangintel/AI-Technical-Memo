# MemRefUtils.h — Code Analysis / 代码分析

## Source / 来源

- **File**: `mlir/include/mlir/ExecutionEngine/MemRefUtils.h`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Declares C++ interfaces and helper types for the MLIR MemRefUtils component. The leading comments describe it as: Utils for MLIR ABI interfacing with frameworks.
- **用途（CN）**: 声明 MLIR MemRefUtils 组件相关的 C++ 接口与辅助类型。 文件开头的注释还对该职责进行了补充说明。

## Line-by-Line Analysis / 逐行分析

### Lines 1-31
````cpp
//===- MemRefUtils.h - Memref helpers to invoke MLIR JIT code ---*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// Utils for MLIR ABI interfacing with frameworks.
//
// The templated free functions below make it possible to allocate dense
// contiguous buffers with shapes that interoperate properly with the MLIR
// codegen ABI.
//
//===----------------------------------------------------------------------===//

#include "mlir/ExecutionEngine/CRunnerUtils.h"
#include "mlir/Support/LLVM.h"
#include "llvm/ADT/ArrayRef.h"
#include "llvm/ADT/STLExtras.h"

#include "llvm/Support/raw_ostream.h"

#include <algorithm>
#include <array>
#include <cassert>
#include <climits>
#include <functional>
#include <initializer_list>
#include <memory>
#include <optional>
````
- **EN**: This block pulls in the MLIR/LLVM/TableGen dependencies required by later declarations and helps establish the compilation boundary of the file.
- **CN**: 该代码块引入后续声明所依赖的 MLIR/LLVM/TableGen 头文件或记录文件，用于建立本文件的编译边界。

### Lines 33-34
````cpp
#ifndef MLIR_EXECUTIONENGINE_MEMREFUTILS_H_
#define MLIR_EXECUTIONENGINE_MEMREFUTILS_H_
````
- **EN**: This block manages the file guard so the header or TableGen fragment is only processed once per translation or inclusion path.
- **CN**: 该代码块管理文件保护宏，确保头文件或 TableGen 片段在一次编译/包含路径中只被处理一次。

### Lines 36-96
````cpp
namespace mlir {
using AllocFunType = llvm::function_ref<void *(size_t)>;

namespace detail {

/// Given a shape with sizes greater than 0 along all dimensions, returns the
/// distance, in number of elements, between a slice in a dimension and the next
/// slice in the same dimension.
///    e.g. shape[3, 4, 5] -> strides[20, 5, 1]
template <size_t N>
inline std::array<int64_t, N> makeStrides(ArrayRef<int64_t> shape) {
  assert(shape.size() == N && "expect shape specification to match rank");
  std::array<int64_t, N> res;
  int64_t running = 1;
  for (int64_t idx = N - 1; idx >= 0; --idx) {
    assert(shape[idx] >= 0 &&
           "size must be non-negative for all shape dimensions");
    res[idx] = running;
    running *= shape[idx];
  }
  return res;
}

/// Build a `StridedMemRefDescriptor<T, N>` that matches the MLIR ABI.
/// This is an implementation detail that is kept in sync with MLIR codegen
/// conventions.  Additionally takes a `shapeAlloc` array which
/// is used instead of `shape` to allocate "more aligned" data and compute the
/// corresponding strides.
template <int N, typename T>
typename std::enable_if<(N >= 1), StridedMemRefType<T, N>>::type
makeStridedMemRefDescriptor(T *ptr, T *alignedPtr, ArrayRef<int64_t> shape,
                            ArrayRef<int64_t> shapeAlloc) {
  assert(shape.size() == N);
  assert(shapeAlloc.size() == N);
  StridedMemRefType<T, N> descriptor;
  descriptor.basePtr = ptr;
  descriptor.data = alignedPtr;
  descriptor.offset = 0;
  std::copy(shape.begin(), shape.end(), descriptor.sizes);
  auto strides = makeStrides<N>(shapeAlloc);
  std::copy(strides.begin(), strides.end(), descriptor.strides);
  return descriptor;
}

/// Build a `StridedMemRefDescriptor<T, 0>` that matches the MLIR ABI.
/// This is an implementation detail that is kept in sync with MLIR codegen
/// conventions.  Additionally takes a `shapeAlloc` array which
/// is used instead of `shape` to allocate "more aligned" data and compute the
/// corresponding strides.
template <int N, typename T>
typename std::enable_if<(N == 0), StridedMemRefType<T, 0>>::type
makeStridedMemRefDescriptor(T *ptr, T *alignedPtr, ArrayRef<int64_t> shape = {},
                            ArrayRef<int64_t> shapeAlloc = {}) {
  assert(shape.size() == N);
  assert(shapeAlloc.size() == N);
  StridedMemRefType<T, 0> descriptor;
  descriptor.basePtr = ptr;
  descriptor.data = alignedPtr;
  descriptor.offset = 0;
  return descriptor;
}
````
- **EN**: This C++ declaration introduces `mlir` and establishes part of the API surface for `MemRefUtils`. Representative entry points here include `makeStrides`, `size`, `makeStridedMemRefDescriptor`, `copy`.
- **CN**: 该 C++ 声明引入了 `mlir`，并构成 `MemRefUtils` API 表面的一部分。 这一段可见的代表性接口包括 `makeStrides`, `size`, `makeStridedMemRefDescriptor`, `copy`。

### Lines 97-120
````cpp
/// Align `nElements` of type T with an optional `alignment`.
/// This replaces a portable `posix_memalign`.
/// `alignment` must be a power of 2 and greater than the size of T. By default
/// the alignment is sizeof(T).
template <typename T>
std::pair<T *, T *>
allocAligned(size_t nElements, AllocFunType allocFun = &::malloc,
             std::optional<uint64_t> alignment = std::optional<uint64_t>()) {
  assert(sizeof(T) <= UINT_MAX && "Elemental type overflows");
  auto size = nElements * sizeof(T);
  auto desiredAlignment = alignment.value_or(nextPowerOf2(sizeof(T)));
  assert((desiredAlignment & (desiredAlignment - 1)) == 0);
  assert(desiredAlignment >= sizeof(T));
  T *data = reinterpret_cast<T *>(allocFun(size + desiredAlignment));
  uintptr_t addr = reinterpret_cast<uintptr_t>(data);
  uintptr_t rem = addr % desiredAlignment;
  T *alignedData = (rem == 0)
                       ? data
                       : reinterpret_cast<T *>(addr + (desiredAlignment - rem));
  assert(reinterpret_cast<uintptr_t>(alignedData) % desiredAlignment == 0);
  return std::make_pair(data, alignedData);
}

} // namespace detail
````
- **EN**: This block groups callable interfaces such as `allocAligned`, `value_or`, `nextPowerOf2`, `allocFun`, indicating how `MemRefUtils` is queried or updated.
- **CN**: 该代码块聚合了 `allocAligned`, `value_or`, `nextPowerOf2`, `allocFun` 等可调用接口，展示了如何查询或更新 `MemRefUtils`。

### Lines 123-195
````cpp
//===----------------------------------------------------------------------===//
// Public API
//===----------------------------------------------------------------------===//

/// Convenient callback to "visit" a memref element by element.
/// This takes a reference to an individual element as well as the coordinates.
/// It can be used in conjuction with a StridedMemrefIterator.
template <typename T>
using ElementWiseVisitor = llvm::function_ref<void(T &ptr, ArrayRef<int64_t>)>;

/// Owning MemRef type that abstracts over the runtime type for ranked strided
/// memref.
template <typename T, unsigned Rank>
class OwningMemRef {
public:
  using DescriptorType = StridedMemRefType<T, Rank>;
  using FreeFunType = std::function<void(DescriptorType)>;

  /// Allocate a new dense StridedMemrefRef with a given `shape`. An optional
  /// `shapeAlloc` array can be supplied to "pad" every dimension individually.
  /// If an ElementWiseVisitor is provided, it will be used to initialize the
  /// data, else the memory will be zero-initialized. The alloc and free method
  /// used to manage the data allocation can be optionally provided, and default
  /// to malloc/free.
  OwningMemRef(
      ArrayRef<int64_t> shape, ArrayRef<int64_t> shapeAlloc = {},
      ElementWiseVisitor<T> init = {},
      std::optional<uint64_t> alignment = std::optional<uint64_t>(),
      AllocFunType allocFun = &::malloc,
      std::function<void(StridedMemRefType<T, Rank>)> freeFun =
          [](StridedMemRefType<T, Rank> descriptor) {
            ::free(descriptor.basePtr);
          })
      : freeFunc(freeFun) {
    if (shapeAlloc.empty())
      shapeAlloc = shape;
    assert(shape.size() == Rank);
    assert(shapeAlloc.size() == Rank);
    for (unsigned i = 0; i < Rank; ++i)
      assert(shape[i] <= shapeAlloc[i] &&
             "shapeAlloc must be greater than or equal to shape");
    int64_t nElements = 1;
    for (int64_t s : shapeAlloc)
      nElements *= s;
    auto [allocatedPtr, alignedData] =
        detail::allocAligned<T>(nElements, allocFun, alignment);
    descriptor = detail::makeStridedMemRefDescriptor<Rank>(
        allocatedPtr, alignedData, shape, shapeAlloc);
    if (init) {
      for (StridedMemrefIterator<T, Rank> it = descriptor.begin(),
                                          end = descriptor.end();
           it != end; ++it)
        init(*it, it.getIndices());
    } else {
      memset(alignedData, 0, nElements * sizeof(T));
    }
  }
  /// Take ownership of an existing descriptor with a custom deleter.
  OwningMemRef(DescriptorType descriptor, FreeFunType freeFunc)
      : freeFunc(freeFunc), descriptor(descriptor) {}
  ~OwningMemRef() {
    if (freeFunc)
      freeFunc(descriptor);
  }
  OwningMemRef(const OwningMemRef &) = delete;
  OwningMemRef &operator=(const OwningMemRef &) = delete;
  OwningMemRef &operator=(const OwningMemRef &&other) {
    freeFunc = other.freeFunc;
    descriptor = other.descriptor;
    other.freeFunc = nullptr;
    memset(&other.descriptor, 0, sizeof(other.descriptor));
  }
  OwningMemRef(OwningMemRef &&other) { *this = std::move(other); }
````
- **EN**: This C++ declaration introduces `OwningMemRef` and establishes part of the API surface for `MemRefUtils`. Representative entry points here include `void`, `OwningMemRef`, `free`, `freeFunc`.
- **CN**: 该 C++ 声明引入了 `OwningMemRef`，并构成 `MemRefUtils` API 表面的一部分。 这一段可见的代表性接口包括 `void`, `OwningMemRef`, `free`, `freeFunc`。

### Lines 196-210
````cpp
  DescriptorType &operator*() { return descriptor; }
  DescriptorType *operator->() { return &descriptor; }
  T &operator[](std::initializer_list<int64_t> indices) {
    return descriptor[indices];
  }

private:
  /// Custom deleter used to release the data buffer manager with the descriptor
  /// below.
  FreeFunType freeFunc;
  /// The descriptor is an instance of StridedMemRefType<T, rank>.
  DescriptorType descriptor;
};

} // namespace mlir
````
- **EN**: This section focuses on descriptortype &operator*() { return descriptor; }, grouping the declarations and helpers needed for that concern.
- **CN**: 本节聚焦于“DescriptorType &operator*() { return descriptor; }”这一主题，把相关声明与辅助接口组织在一起。

### Lines 213-213
````cpp
#endif // MLIR_EXECUTIONENGINE_MEMREFUTILS_H_
````
- **EN**: This block manages the file guard so the header or TableGen fragment is only processed once per translation or inclusion path.
- **CN**: 该代码块管理文件保护宏，确保头文件或 TableGen 片段在一次编译/包含路径中只被处理一次。

## Key Concepts / 关键概念

- **EN**: C++ declaration surface for MLIR infrastructure
  **CN**: MLIR 基础设施的 C++ 声明层
- **EN**: Type or attribute abstraction
  **CN**: 类型或属性抽象

## Dependencies / 依赖关系

- mlir/ExecutionEngine/CRunnerUtils.h
- mlir/Support/LLVM.h
- llvm/ADT/ArrayRef.h
- llvm/ADT/STLExtras.h
- llvm/Support/raw_ostream.h
