# RaggedArray.h — Code Analysis / 代码分析

## Source / 来源

- **File**: `mlir/include/mlir/Dialect/Transform/Utils/RaggedArray.h`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Declares C++ interfaces and helper types for the MLIR RaggedArray component. The leading comments describe it as: A 2D array where each row may have different length. Elements of each row.
- **用途（CN）**: 声明 MLIR RaggedArray 组件相关的 C++ 接口与辅助类型。 文件开头的注释还对该职责进行了补充说明。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12
````cpp
//===- RaggedArray.h - 2D array with different inner lengths ----*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "mlir/Support/LLVM.h"
#include "llvm/ADT/STLExtras.h"
#include "llvm/ADT/SmallVector.h"
#include <iterator>
````
- **EN**: This block pulls in the MLIR/LLVM/TableGen dependencies required by later declarations and helps establish the compilation boundary of the file.
- **CN**: 该代码块引入后续声明所依赖的 MLIR/LLVM/TableGen 头文件或记录文件，用于建立本文件的编译边界。

### Lines 14-62
````cpp
namespace mlir {
/// A 2D array where each row may have different length. Elements of each row
/// are stored contiguously, but rows don't have a fixed order in the storage.
template <typename T>
class RaggedArray {
public:
  /// Returns the number of rows in the 2D array.
  size_t size() const { return slices.size(); }

  /// Returns true if the are no rows in the 2D array. Note that an array with a
  /// non-zero number of empty rows is *NOT* empty.
  bool empty() const { return slices.empty(); }

  /// Accesses `pos`-th row.
  ArrayRef<T> operator[](size_t pos) const { return at(pos); }
  ArrayRef<T> at(size_t pos) const {
    if (slices[pos].first == static_cast<size_t>(-1))
      return ArrayRef<T>();
    return ArrayRef<T>(storage).slice(slices[pos].first, slices[pos].second);
  }
  MutableArrayRef<T> operator[](size_t pos) { return at(pos); }
  MutableArrayRef<T> at(size_t pos) {
    if (slices[pos].first == static_cast<size_t>(-1))
      return MutableArrayRef<T>();
    return MutableArrayRef<T>(storage).slice(slices[pos].first,
                                             slices[pos].second);
  }

  /// Iterator over the rows.
  class iterator
      : public llvm::iterator_facade_base<
            iterator, std::forward_iterator_tag, MutableArrayRef<T>,
            std::ptrdiff_t, MutableArrayRef<T> *, MutableArrayRef<T>> {
  public:
    /// Creates the start iterator.
    explicit iterator(RaggedArray &ragged) : ragged(ragged), pos(0) {}

    /// Creates the end iterator.
    iterator(RaggedArray &ragged, size_t pos) : ragged(ragged), pos(pos) {}

    /// Dereferences the current iterator. Assumes in-bounds.
    MutableArrayRef<T> operator*() const { return ragged[pos]; }

    /// Increments the iterator.
    iterator &operator++() {
      if (pos < ragged.slices.size())
        ++pos;
      return *this;
    }
````
- **EN**: This C++ declaration introduces `RaggedArray` and establishes part of the API surface for `RaggedArray`. Representative entry points here include `size`, `empty`, `at`, `slice`.
- **CN**: 该 C++ 声明引入了 `RaggedArray`，并构成 `RaggedArray` API 表面的一部分。 这一段可见的代表性接口包括 `size`, `empty`, `at`, `slice`。

### Lines 63-107
````cpp
    /// Compares the two iterators. Iterators into different ragged arrays
    /// compare not equal.
    bool operator==(const iterator &other) const {
      return &ragged == &other.ragged && pos == other.pos;
    }

  private:
    RaggedArray &ragged;
    size_t pos;
  };

  /// Constant iterator over the rows.
  class const_iterator
      : public llvm::iterator_facade_base<
            const_iterator, std::forward_iterator_tag, ArrayRef<T>,
            std::ptrdiff_t, ArrayRef<T> *, ArrayRef<T>> {
  public:
    /// Creates the start iterator.
    explicit const_iterator(const RaggedArray &ragged)
        : ragged(ragged), pos(0) {}

    /// Creates the end iterator.
    const_iterator(const RaggedArray &ragged, size_t pos)
        : ragged(ragged), pos(pos) {}

    /// Dereferences the current iterator. Assumes in-bounds.
    ArrayRef<T> operator*() const { return ragged[pos]; }

    /// Increments the iterator.
    const_iterator &operator++() {
      if (pos < ragged.slices.size())
        ++pos;
      return *this;
    }

    /// Compares the two iterators. Iterators into different ragged arrays
    /// compare not equal.
    bool operator==(const const_iterator &other) const {
      return &ragged == &other.ragged && pos == other.pos;
    }

  private:
    const RaggedArray &ragged;
    size_t pos;
  };
````
- **EN**: This C++ declaration introduces `const_iterator` and establishes part of the API surface for `RaggedArray`. Representative entry points here include `const_iterator`, `ragged`, `pos`, `size`.
- **CN**: 该 C++ 声明引入了 `const_iterator`，并构成 `RaggedArray` API 表面的一部分。 这一段可见的代表性接口包括 `const_iterator`, `ragged`, `pos`, `size`。

### Lines 109-153
````cpp
  /// Iterator over rows.
  const_iterator begin() const { return const_iterator(*this); }
  const_iterator end() const { return const_iterator(*this, slices.size()); }
  iterator begin() { return iterator(*this); }
  iterator end() { return iterator(*this, slices.size()); }

  /// Reserve space to store `size` rows with `nestedSize` elements each.
  void reserve(size_t size, size_t nestedSize = 0) {
    slices.reserve(size);
    storage.reserve(size * nestedSize);
  }

  /// Appends the given range of elements as a new row to the 2D array. May
  /// invalidate the end iterator.
  template <typename Range>
  void push_back(Range &&elements) {
    slices.push_back(appendToStorage(std::forward<Range>(elements)));
  }

  /// Replaces the `pos`-th row in the 2D array with the given range of
  /// elements. Invalidates iterators and references to `pos`-th and all
  /// succeeding rows.
  template <typename Range>
  void replace(size_t pos, Range &&elements) {
    if (slices[pos].first != static_cast<size_t>(-1)) {
      auto from = std::next(storage.begin(), slices[pos].first);
      auto to = std::next(from, slices[pos].second);
      auto newFrom = storage.erase(from, to);
      // Update the array refs after the underlying storage was shifted.
      for (size_t i = pos + 1, e = size(); i < e; ++i) {
        slices[i] = std::make_pair(std::distance(storage.begin(), newFrom),
                                   slices[i].second);
        std::advance(newFrom, slices[i].second);
      }
    }
    slices[pos] = appendToStorage(std::forward<Range>(elements));
  }

  /// Appends `num` empty rows to the array.
  void appendEmptyRows(size_t num) {
    slices.resize(slices.size() + num, std::pair<size_t, size_t>(-1, 0));
  }

  /// Removes the first subarray in-place. Invalidates iterators to all rows.
  void removeFront() { slices.erase(slices.begin()); }
````
- **EN**: This block groups callable interfaces such as `begin`, `const_iterator`, `end`, `size`, indicating how `RaggedArray` is queried or updated.
- **CN**: 该代码块聚合了 `begin`, `const_iterator`, `end`, `size` 等可调用接口，展示了如何查询或更新 `RaggedArray`。

### Lines 155-175
````cpp
private:
  /// Appends the given elements to the storage and returns an ArrayRef
  /// pointing to them in the storage.
  template <typename Range>
  std::pair<size_t, size_t> appendToStorage(Range &&elements) {
    size_t start = storage.size();
    llvm::append_range(storage, std::forward<Range>(elements));
    return std::make_pair(start, storage.size() - start);
  }

  /// Outer elements of the ragged array. Each entry is an (offset, length)
  /// pair identifying a contiguous segment in the `storage` list that
  /// contains the actual elements. This allows for elements to be stored
  /// contiguously without nested vectors and for different segments to be set
  /// or replaced in any order.
  SmallVector<std::pair<size_t, size_t>> slices;

  /// Dense storage for ragged array elements.
  SmallVector<T> storage;
};
} // namespace mlir
````
- **EN**: This block groups callable interfaces such as `appendToStorage`, `size`, `append_range`, `make_pair`, indicating how `RaggedArray` is queried or updated.
- **CN**: 该代码块聚合了 `appendToStorage`, `size`, `append_range`, `make_pair` 等可调用接口，展示了如何查询或更新 `RaggedArray`。

## Key Concepts / 关键概念

- **EN**: C++ declaration surface for MLIR infrastructure
  **CN**: MLIR 基础设施的 C++ 声明层
- **EN**: Iterator-based traversal APIs
  **CN**: 基于迭代器的遍历接口

## Dependencies / 依赖关系

- mlir/Support/LLVM.h
- llvm/ADT/STLExtras.h
- llvm/ADT/SmallVector.h
- iterator inherits from public llvm::iterator_facade_base< iterator, std::forward_iterator_tag, MutableArrayRef<T>, std::ptrdiff_t, MutableArrayRef<T> *, MutableArrayRef<T>>
- const_iterator inherits from public llvm::iterator_facade_base< const_iterator, std::forward_iterator_tag, ArrayRef<T>, std::ptrdiff_t, ArrayRef<T> *, ArrayRef<T>>
- iterator builds on public llvm::iterator_facade_base<
- const_iterator builds on public llvm::iterator_facade_base<
