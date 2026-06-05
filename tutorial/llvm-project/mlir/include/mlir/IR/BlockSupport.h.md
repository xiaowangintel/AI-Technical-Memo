# BlockSupport.h — Code Analysis / 代码分析

## Source / 来源

- **File**: `mlir/include/mlir/IR/BlockSupport.h`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Declares C++ interfaces and helper types for the MLIR BlockSupport component. The leading comments describe it as: This file defines a number of support types for the Block class.
- **用途（CN）**: 声明 MLIR BlockSupport 组件相关的 C++ 接口与辅助类型。 文件开头的注释还对该职责进行了补充说明。

## Line-by-Line Analysis / 逐行分析

### Lines 1-19
````cpp
//===- BlockSupport.h -------------------------------------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file defines a number of support types for the Block class.
//
//===----------------------------------------------------------------------===//

#ifndef MLIR_IR_BLOCKSUPPORT_H
#define MLIR_IR_BLOCKSUPPORT_H

#include "mlir/IR/Value.h"
#include "llvm/ADT/PointerUnion.h"
#include "llvm/ADT/ilist.h"
#include "llvm/ADT/ilist_node.h"
````
- **EN**: This block establishes the file guard and imports the MLIR/LLVM/TableGen dependencies that the rest of the file builds on.
- **CN**: 该代码块同时建立文件保护宏，并引入后续实现所依赖的 MLIR/LLVM/TableGen 头文件或记录文件。

### Lines 21-22
````cpp
namespace mlir {
class Block;
````
- **EN**: This C++ declaration introduces `Block` and establishes part of the API surface for `BlockSupport`.
- **CN**: 该 C++ 声明引入了 `Block`，并构成 `BlockSupport` API 表面的一部分。

### Lines 24-39
````cpp
//===----------------------------------------------------------------------===//
// BlockOperand
//===----------------------------------------------------------------------===//

/// A block operand represents an operand that holds a reference to a Block,
/// e.g. for terminator operations.
class BlockOperand : public IROperand<BlockOperand, Block *> {
public:
  using IROperand<BlockOperand, Block *>::IROperand;

  /// Provide the use list that is attached to the given block.
  static IRObjectWithUseList<BlockOperand> *getUseList(Block *value);

  /// Return which operand this is in the BlockOperand list of the Operation.
  unsigned getOperandNumber() const;
};
````
- **EN**: This C++ declaration introduces `BlockOperand` and establishes part of the API surface for `BlockSupport`. Representative entry points here include `getUseList`, `getOperandNumber`.
- **CN**: 该 C++ 声明引入了 `BlockOperand`，并构成 `BlockSupport` API 表面的一部分。 这一段可见的代表性接口包括 `getUseList`, `getOperandNumber`。

### Lines 41-64
````cpp
//===----------------------------------------------------------------------===//
// Predecessors
//===----------------------------------------------------------------------===//

/// Implement a predecessor iterator for blocks. This works by walking the use
/// lists of the blocks. The entries on this list are the BlockOperands that
/// are embedded into terminator operations. From the operand, we can get the
/// terminator that contains it, and its parent block is the predecessor.
class PredecessorIterator final
    : public llvm::mapped_iterator<ValueUseIterator<BlockOperand>,
                                   Block *(*)(BlockOperand &)> {
  static Block *unwrap(BlockOperand &value);

public:
  /// Initializes the operand type iterator to the specified operand iterator.
  PredecessorIterator(ValueUseIterator<BlockOperand> it)
      : llvm::mapped_iterator<ValueUseIterator<BlockOperand>,
                              Block *(*)(BlockOperand &)>(it, &unwrap) {}
  explicit PredecessorIterator(BlockOperand *operand)
      : PredecessorIterator(ValueUseIterator<BlockOperand>(operand)) {}

  /// Get the successor number in the predecessor terminator.
  unsigned getSuccessorIndex() const;
};
````
- **EN**: This C++ declaration introduces `PredecessorIterator` and establishes part of the API surface for `BlockSupport`. Representative entry points here include `unwrap`, `PredecessorIterator`, `getSuccessorIndex`.
- **CN**: 该 C++ 声明引入了 `PredecessorIterator`，并构成 `BlockSupport` API 表面的一部分。 这一段可见的代表性接口包括 `unwrap`, `PredecessorIterator`, `getSuccessorIndex`。

### Lines 66-92
````cpp
//===----------------------------------------------------------------------===//
// Successors
//===----------------------------------------------------------------------===//

/// This class implements the successor iterators for Block.
class SuccessorRange final
    : public llvm::detail::indexed_accessor_range_base<
          SuccessorRange, BlockOperand *, Block *, Block *, Block *> {
public:
  using RangeBaseT::RangeBaseT;
  SuccessorRange();
  SuccessorRange(Block *block);
  SuccessorRange(Operation *term);

private:
  /// See `llvm::detail::indexed_accessor_range_base` for details.
  static BlockOperand *offset_base(BlockOperand *object, ptrdiff_t index) {
    return object + index;
  }
  /// See `llvm::detail::indexed_accessor_range_base` for details.
  static Block *dereference_iterator(BlockOperand *object, ptrdiff_t index) {
    return object[index].get();
  }

  /// Allow access to `offset_base` and `dereference_iterator`.
  friend RangeBaseT;
};
````
- **EN**: This C++ declaration introduces `SuccessorRange` and establishes part of the API surface for `BlockSupport`. Representative entry points here include `SuccessorRange`, `offset_base`, `dereference_iterator`, `get`.
- **CN**: 该 C++ 声明引入了 `SuccessorRange`，并构成 `BlockSupport` API 表面的一部分。 这一段可见的代表性接口包括 `SuccessorRange`, `offset_base`, `dereference_iterator`, `get`。

### Lines 94-132
````cpp
//===----------------------------------------------------------------------===//
// BlockRange
//===----------------------------------------------------------------------===//

/// This class provides an abstraction over the different types of ranges over
/// Blocks. In many cases, this prevents the need to explicitly materialize a
/// SmallVector/std::vector. This class should be used in places that are not
/// suitable for a more derived type (e.g. ArrayRef) or a template range
/// parameter.
class BlockRange final
    : public llvm::detail::indexed_accessor_range_base<
          BlockRange, llvm::PointerUnion<BlockOperand *, Block *const *>,
          Block *, Block *, Block *> {
public:
  using RangeBaseT::RangeBaseT;
  BlockRange(ArrayRef<Block *> blocks = {});
  BlockRange(SuccessorRange successors);
  template <typename Arg, typename = std::enable_if_t<std::is_constructible<
                              ArrayRef<Block *>, Arg>::value>>
  BlockRange(Arg &&arg LLVM_LIFETIME_BOUND)
      : BlockRange(ArrayRef<Block *>(std::forward<Arg>(arg))) {}
  BlockRange(std::initializer_list<Block *> blocks LLVM_LIFETIME_BOUND)
      : BlockRange(ArrayRef<Block *>(blocks)) {}

private:
  /// The owner of the range is either:
  /// * A pointer to the first element of an array of block operands.
  /// * A pointer to the first element of an array of Block *.
  using OwnerT = llvm::PointerUnion<BlockOperand *, Block *const *>;

  /// See `llvm::detail::indexed_accessor_range_base` for details.
  static OwnerT offset_base(OwnerT object, ptrdiff_t index);

  /// See `llvm::detail::indexed_accessor_range_base` for details.
  static Block *dereference_iterator(OwnerT object, ptrdiff_t index);

  /// Allow access to `offset_base` and `dereference_iterator`.
  friend RangeBaseT;
};
````
- **EN**: This C++ declaration introduces `BlockRange` and establishes part of the API surface for `BlockSupport`. Representative entry points here include `BlockRange`, `offset_base`, `dereference_iterator`.
- **CN**: 该 C++ 声明引入了 `BlockRange`，并构成 `BlockSupport` API 表面的一部分。 这一段可见的代表性接口包括 `BlockRange`, `offset_base`, `dereference_iterator`。

### Lines 134-172
````cpp
//===----------------------------------------------------------------------===//
// Operation Iterators
//===----------------------------------------------------------------------===//

namespace detail {
/// A utility iterator that filters out operations that are not 'OpT'.
template <typename OpT, typename IteratorT>
class op_filter_iterator
    : public llvm::filter_iterator<IteratorT, bool (*)(Operation &)> {
  static bool filter(Operation &op) { return llvm::isa<OpT>(op); }

public:
  op_filter_iterator(IteratorT it, IteratorT end)
      : llvm::filter_iterator<IteratorT, bool (*)(Operation &)>(it, end,
                                                                &filter) {}

  /// Allow implicit conversion to the underlying iterator.
  operator const IteratorT &() const { return this->wrapped(); }
};

/// This class provides iteration over the held operations of a block for a
/// specific operation type.
template <typename OpT, typename IteratorT>
class op_iterator
    : public llvm::mapped_iterator<op_filter_iterator<OpT, IteratorT>,
                                   OpT (*)(Operation &)> {
  static OpT unwrap(Operation &op) { return cast<OpT>(op); }

public:
  /// Initializes the iterator to the specified filter iterator.
  op_iterator(op_filter_iterator<OpT, IteratorT> it)
      : llvm::mapped_iterator<op_filter_iterator<OpT, IteratorT>,
                              OpT (*)(Operation &)>(it, &unwrap) {}

  /// Allow implicit conversion to the underlying block iterator.
  operator const IteratorT &() const { return this->wrapped(); }
};
} // namespace detail
} // namespace mlir
````
- **EN**: This C++ declaration introduces `op_filter_iterator` and establishes part of the API surface for `BlockSupport`. Representative entry points here include `bool`, `filter`, `op_filter_iterator`, `wrapped`.
- **CN**: 该 C++ 声明引入了 `op_filter_iterator`，并构成 `BlockSupport` API 表面的一部分。 这一段可见的代表性接口包括 `bool`, `filter`, `op_filter_iterator`, `wrapped`。

### Lines 174-197
````cpp
namespace llvm {

/// Provide support for hashing successor ranges.
template <>
struct DenseMapInfo<mlir::SuccessorRange> {
  static mlir::SuccessorRange getEmptyKey() {
    auto *pointer = llvm::DenseMapInfo<mlir::BlockOperand *>::getEmptyKey();
    return mlir::SuccessorRange(pointer, 0);
  }
  static mlir::SuccessorRange getTombstoneKey() {
    auto *pointer = llvm::DenseMapInfo<mlir::BlockOperand *>::getTombstoneKey();
    return mlir::SuccessorRange(pointer, 0);
  }
  static unsigned getHashValue(mlir::SuccessorRange value) {
    return llvm::hash_combine_range(value);
  }
  static bool isEqual(mlir::SuccessorRange lhs, mlir::SuccessorRange rhs) {
    if (rhs.getBase() == getEmptyKey().getBase())
      return lhs.getBase() == getEmptyKey().getBase();
    if (rhs.getBase() == getTombstoneKey().getBase())
      return lhs.getBase() == getTombstoneKey().getBase();
    return lhs == rhs;
  }
};
````
- **EN**: This C++ declaration introduces `DenseMapInfo` and establishes part of the API surface for `BlockSupport`. Representative entry points here include `getEmptyKey`, `SuccessorRange`, `getTombstoneKey`, `getHashValue`.
- **CN**: 该 C++ 声明引入了 `DenseMapInfo`，并构成 `BlockSupport` API 表面的一部分。 这一段可见的代表性接口包括 `getEmptyKey`, `SuccessorRange`, `getTombstoneKey`, `getHashValue`。

### Lines 199-238
````cpp
//===----------------------------------------------------------------------===//
// ilist_traits for Operation
//===----------------------------------------------------------------------===//

namespace ilist_detail {
// Explicitly define the node access for the operation list so that we can
// break the dependence on the Operation class in this header. This allows for
// operations to have trailing Regions without a circular include
// dependence.
template <>
struct SpecificNodeAccess<compute_node_options<::mlir::Operation>::type>
    : NodeAccess {
protected:
  using OptionsT = compute_node_options<mlir::Operation>::type;
  using pointer = OptionsT::pointer;
  using const_pointer = OptionsT::const_pointer;
  using node_type = ilist_node_impl<OptionsT>;

  static node_type *getNodePtr(pointer N);
  static const node_type *getNodePtr(const_pointer N);

  static pointer getValuePtr(node_type *N);
  static const_pointer getValuePtr(const node_type *N);
};
} // namespace ilist_detail

template <>
struct ilist_traits<::mlir::Operation> {
  using Operation = ::mlir::Operation;
  using op_iterator = simple_ilist<Operation>::iterator;

  static void deleteNode(Operation *op);
  void addNodeToList(Operation *op);
  void removeNodeFromList(Operation *op);
  void transferNodesFromList(ilist_traits<Operation> &otherList,
                             op_iterator first, op_iterator last);

private:
  mlir::Block *getContainingBlock();
};
````
- **EN**: This C++ declaration introduces `SpecificNodeAccess` and establishes part of the API surface for `BlockSupport`. Representative entry points here include `getNodePtr`, `getValuePtr`, `deleteNode`, `addNodeToList`.
- **CN**: 该 C++ 声明引入了 `SpecificNodeAccess`，并构成 `BlockSupport` API 表面的一部分。 这一段可见的代表性接口包括 `getNodePtr`, `getValuePtr`, `deleteNode`, `addNodeToList`。

### Lines 240-258
````cpp
//===----------------------------------------------------------------------===//
// ilist_traits for Block
//===----------------------------------------------------------------------===//

template <>
struct ilist_traits<::mlir::Block> : public ilist_alloc_traits<::mlir::Block> {
  using Block = ::mlir::Block;
  using block_iterator = simple_ilist<::mlir::Block>::iterator;

  void addNodeToList(Block *block);
  void removeNodeFromList(Block *block);
  void transferNodesFromList(ilist_traits<Block> &otherList,
                             block_iterator first, block_iterator last);

private:
  mlir::Region *getParentRegion();
};

} // namespace llvm
````
- **EN**: This C++ declaration introduces `ilist_traits` and establishes part of the API surface for `BlockSupport`. Representative entry points here include `addNodeToList`, `removeNodeFromList`, `transferNodesFromList`, `getParentRegion`.
- **CN**: 该 C++ 声明引入了 `ilist_traits`，并构成 `BlockSupport` API 表面的一部分。 这一段可见的代表性接口包括 `addNodeToList`, `removeNodeFromList`, `transferNodesFromList`, `getParentRegion`。

### Lines 260-260
````cpp
#endif // MLIR_IR_BLOCKSUPPORT_H
````
- **EN**: This block manages the file guard so the header or TableGen fragment is only processed once per translation or inclusion path.
- **CN**: 该代码块管理文件保护宏，确保头文件或 TableGen 片段在一次编译/包含路径中只被处理一次。

## Key Concepts / 关键概念

- **EN**: C++ declaration surface for MLIR infrastructure
  **CN**: MLIR 基础设施的 C++ 声明层
- **EN**: Iterator-based traversal APIs
  **CN**: 基于迭代器的遍历接口
- **EN**: Region/block ownership and nesting
  **CN**: Region/Block 的所有权与嵌套关系

## Dependencies / 依赖关系

- mlir/IR/Value.h
- llvm/ADT/PointerUnion.h
- llvm/ADT/ilist.h
- llvm/ADT/ilist_node.h
- BlockOperand inherits from public IROperand<BlockOperand, Block *>
- op_filter_iterator inherits from public llvm::filter_iterator<IteratorT, bool (*)(Operation &)>
- op_iterator inherits from public llvm::mapped_iterator<op_filter_iterator<OpT, IteratorT>, OpT (*)(Operation &)>
- BlockOperand builds on public IROperand<BlockOperand, Block *>
- op_filter_iterator builds on public llvm::filter_iterator<IteratorT, bool (*)(Operation &)>
- op_iterator builds on public llvm::mapped_iterator<op_filter_iterator<OpT, IteratorT>,
