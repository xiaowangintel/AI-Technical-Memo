# Block.h — Code Analysis / 代码分析

## Source / 来源

- **File**: `mlir/include/mlir/IR/Block.h`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Declares the MLIR Block class and its APIs for arguments, operations, control-flow relationships, and mutation. The leading comments describe it as: This file defines the Block class.
- **用途（CN）**: 声明 MLIR 的 Block 类，以及其关于参数、操作链表、控制流关系与变更的接口。 文件开头的注释还对该职责进行了补充说明。

## Line-by-Line Analysis / 逐行分析

### Lines 1-19
````cpp
//===- Block.h - MLIR Block Class -------------------------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file defines the Block class.
//
//===----------------------------------------------------------------------===//

#ifndef MLIR_IR_BLOCK_H
#define MLIR_IR_BLOCK_H

#include "mlir/IR/BlockSupport.h"
#include "mlir/IR/Visitors.h"

#include "llvm/ADT/SmallPtrSet.h"
````
- **EN**: This block establishes the file guard and imports the MLIR/LLVM/TableGen dependencies that the rest of the file builds on.
- **CN**: 该代码块同时建立文件保护宏，并引入后续实现所依赖的 MLIR/LLVM/TableGen 头文件或记录文件。

### Lines 21-24
````cpp
namespace llvm {
class BitVector;
class raw_ostream;
} // namespace llvm
````
- **EN**: This C++ declaration introduces `BitVector` and establishes part of the API surface for `Block`.
- **CN**: 该 C++ 声明引入了 `BitVector`，并构成 `Block` API 表面的一部分。

### Lines 26-71
````cpp
namespace mlir {
class TypeRange;
template <typename ValueRangeT>
class ValueTypeRange;

/// `Block` represents an ordered list of `Operation`s.
class alignas(8) Block : public IRObjectWithUseList<BlockOperand>,
                         public llvm::ilist_node_with_parent<Block, Region> {
public:
  explicit Block() = default;
  ~Block();

  void clear() {
    // Drop all references from within this block.
    dropAllReferences();

    // Clear operations in the reverse order so that uses are destroyed
    // before their defs.
    while (!empty())
      operations.pop_back();
  }

  /// Provide a 'getParent' method for ilist_node_with_parent methods.
  /// We mark it as a const function because ilist_node_with_parent specifically
  /// requires a 'getParent() const' method. Once ilist_node removes this
  /// constraint, we should drop the const to fit the rest of the MLIR const
  /// model.
  Region *getParent() const;

  /// Returns the closest surrounding operation that contains this block.
  Operation *getParentOp();

  /// Return if this block is the entry block in the parent region.
  bool isEntryBlock();

  /// Insert this block (which must not already be in a region) right before
  /// the specified block.
  void insertBefore(Block *block);

  /// Insert this block (which must not already be in a region) right after
  /// the specified block.
  void insertAfter(Block *block);

  /// Unlink this block from its current region and insert it right before the
  /// specific block.
  void moveBefore(Block *block);
````
- **EN**: This C++ declaration introduces `TypeRange` and establishes part of the API surface for `Block`. Representative entry points here include `Block`, `clear`, `dropAllReferences`, `empty`.
- **CN**: 该 C++ 声明引入了 `TypeRange`，并构成 `Block` API 表面的一部分。 这一段可见的代表性接口包括 `Block`, `clear`, `dropAllReferences`, `empty`。

### Lines 72-87
````cpp
  /// Unlink this block from its current region and insert it right before the
  /// block that the given iterator points to in the region region.
  void moveBefore(Region *region, llvm::iplist<Block>::iterator iterator);

  /// Unlink this Block from its parent region and delete it.
  void erase();

  /// Compute the position of this block within its parent region using an O(N)
  /// linear scan.
  ///
  /// Note: There is no semantic meaning to a block number. Blocks are used for
  /// unstructured control flow and relying on block numbers for functional
  /// purposes may indicate a design flaw. (You can give semantic meaning to
  /// region numbers instead.) Block numbers are useful for debugging purposes
  /// and for error messages.
  unsigned computeBlockNumber();
````
- **EN**: This block groups callable interfaces such as `moveBefore`, `erase`, `computeBlockNumber`, indicating how `Block` is queried or updated.
- **CN**: 该代码块聚合了 `moveBefore`, `erase`, `computeBlockNumber` 等可调用接口，展示了如何查询或更新 `Block`。

### Lines 90-136
````cpp
  //===--------------------------------------------------------------------===//
  // Block argument management
  //===--------------------------------------------------------------------===//

  // This is the list of arguments to the block.
  using BlockArgListType = MutableArrayRef<BlockArgument>;

  BlockArgListType getArguments() { return arguments; }

  /// Return a range containing the types of the arguments for this block.
  ValueTypeRange<BlockArgListType> getArgumentTypes();

  using args_iterator = BlockArgListType::iterator;
  using reverse_args_iterator = BlockArgListType::reverse_iterator;
  args_iterator args_begin() { return getArguments().begin(); }
  args_iterator args_end() { return getArguments().end(); }
  reverse_args_iterator args_rbegin() { return getArguments().rbegin(); }
  reverse_args_iterator args_rend() { return getArguments().rend(); }

  bool args_empty() { return arguments.empty(); }

  /// Add one value to the argument list.
  BlockArgument addArgument(Type type, Location loc);

  /// Insert one value to the position in the argument list indicated by the
  /// given iterator. The existing arguments are shifted. The block is expected
  /// not to have predecessors.
  BlockArgument insertArgument(args_iterator it, Type type, Location loc);

  /// Add one argument to the argument list for each type specified in the list.
  /// `locs` is required to have the same number of elements as `types`.
  iterator_range<args_iterator> addArguments(TypeRange types,
                                             ArrayRef<Location> locs);

  /// Add one value to the argument list at the specified position.
  BlockArgument insertArgument(unsigned index, Type type, Location loc);

  /// Erase the argument at 'index' and remove it from the argument list.
  void eraseArgument(unsigned index);
  /// Erases 'num' arguments from the index 'start'.
  void eraseArguments(unsigned start, unsigned num);
  /// Erases the arguments that have their corresponding bit set in
  /// `eraseIndices` and removes them from the argument list.
  void eraseArguments(const BitVector &eraseIndices);
  /// Erases arguments using the given predicate. If the predicate returns true,
  /// that argument is erased.
  void eraseArguments(function_ref<bool(BlockArgument)> shouldEraseFn);
````
- **EN**: This block groups callable interfaces such as `getArguments`, `getArgumentTypes`, `args_begin`, `begin`, indicating how `Block` is queried or updated.
- **CN**: 该代码块聚合了 `getArguments`, `getArgumentTypes`, `args_begin`, `begin` 等可调用接口，展示了如何查询或更新 `Block`。

### Lines 137-138
````cpp
  unsigned getNumArguments() { return arguments.size(); }
  BlockArgument getArgument(unsigned i) { return arguments[i]; }
````
- **EN**: This block groups callable interfaces such as `getNumArguments`, `size`, `getArgument`, indicating how `Block` is queried or updated.
- **CN**: 该代码块聚合了 `getNumArguments`, `size`, `getArgument` 等可调用接口，展示了如何查询或更新 `Block`。

### Lines 141-186
````cpp
  //===--------------------------------------------------------------------===//
  // Operation list management
  //===--------------------------------------------------------------------===//

  /// This is the list of operations in the block.
  using OpListType = llvm::iplist<Operation>;
  OpListType &getOperations() { return operations; }

  // Iteration over the operations in the block.
  using iterator = OpListType::iterator;
  using reverse_iterator = OpListType::reverse_iterator;

  iterator begin() { return operations.begin(); }
  iterator end() { return operations.end(); }
  reverse_iterator rbegin() { return operations.rbegin(); }
  reverse_iterator rend() { return operations.rend(); }

  bool empty() { return operations.empty(); }
  void push_back(Operation *op) { operations.push_back(op); }
  void push_front(Operation *op) { operations.push_front(op); }

  Operation &back() { return operations.back(); }
  Operation &front() { return operations.front(); }

  /// Returns 'op' if 'op' lies in this block, or otherwise finds the
  /// ancestor operation of 'op' that lies in this block. Returns nullptr if
  /// the latter fails.
  /// TODO: This is very specific functionality that should live somewhere else,
  /// probably in Dominance.cpp.
  Operation *findAncestorOpInBlock(Operation &op);

  /// This drops all operand uses from operations within this block, which is
  /// an essential step in breaking cyclic dependences between references when
  /// they are to be deleted.
  void dropAllReferences();

  /// This drops all uses of values defined in this block or in the blocks of
  /// nested regions wherever the uses are located.
  void dropAllDefinedValueUses();

  /// Returns true if the ordering of the child operations is valid, false
  /// otherwise.
  bool isOpOrderValid();

  /// Invalidates the current ordering of operations.
  void invalidateOpOrder();
````
- **EN**: This block groups callable interfaces such as `getOperations`, `begin`, `end`, `rbegin`, indicating how `Block` is queried or updated.
- **CN**: 该代码块聚合了 `getOperations`, `begin`, `end`, `rbegin` 等可调用接口，展示了如何查询或更新 `Block`。

### Lines 187-225
````cpp
  /// Verifies the current ordering of child operations matches the
  /// validOpOrder flag. Returns false if the order is valid, true otherwise.
  bool verifyOpOrder();

  /// Recomputes the ordering of child operations within the block.
  void recomputeOpOrder();

  /// This class provides iteration over the held operations of a block for a
  /// specific operation type.
  template <typename OpT>
  using op_iterator = detail::op_iterator<OpT, iterator>;

  /// Return an iterator range over the operations within this block that are of
  /// 'OpT'.
  template <typename OpT>
  iterator_range<op_iterator<OpT>> getOps() {
    auto endIt = end();
    return {detail::op_filter_iterator<OpT, iterator>(begin(), endIt),
            detail::op_filter_iterator<OpT, iterator>(endIt, endIt)};
  }
  template <typename OpT>
  op_iterator<OpT> op_begin() {
    return detail::op_filter_iterator<OpT, iterator>(begin(), end());
  }
  template <typename OpT>
  op_iterator<OpT> op_end() {
    return detail::op_filter_iterator<OpT, iterator>(end(), end());
  }

  /// Return an iterator range over the operation within this block excluding
  /// the terminator operation at the end. If the block has no terminator,
  /// return an iterator range over the entire block. If it is unknown if the
  /// block has a terminator (i.e., last block operation is unregistered), also
  /// return an iterator range over the entire block.
  iterator_range<iterator> without_terminator() {
    if (begin() == end())
      return {begin(), end()};
    return without_terminator_impl();
  }
````
- **EN**: This block groups callable interfaces such as `verifyOpOrder`, `recomputeOpOrder`, `getOps`, `end`, indicating how `Block` is queried or updated.
- **CN**: 该代码块聚合了 `verifyOpOrder`, `recomputeOpOrder`, `getOps`, `end` 等可调用接口，展示了如何查询或更新 `Block`。

### Lines 228-238
````cpp
  //===--------------------------------------------------------------------===//
  // Terminator management
  //===--------------------------------------------------------------------===//

  /// Get the terminator operation of this block. This function asserts that
  /// the block might have a valid terminator operation.
  Operation *getTerminator();

  /// Return "true" if this block might have a terminator. Return "true" if
  /// the last operation is unregistered.
  bool mightHaveTerminator();
````
- **EN**: This block groups callable interfaces such as `getTerminator`, `mightHaveTerminator`, indicating how `Block` is queried or updated.
- **CN**: 该代码块聚合了 `getTerminator`, `mightHaveTerminator` 等可调用接口，展示了如何查询或更新 `Block`。

### Lines 240-293
````cpp
  //===--------------------------------------------------------------------===//
  // Predecessors and successors.
  //===--------------------------------------------------------------------===//

  // Predecessor iteration.
  using pred_iterator = PredecessorIterator;
  pred_iterator pred_begin() {
    return pred_iterator((BlockOperand *)getFirstUse());
  }
  pred_iterator pred_end() { return pred_iterator(nullptr); }
  iterator_range<pred_iterator> getPredecessors() {
    return {pred_begin(), pred_end()};
  }

  /// Return true if this block has no predecessors.
  bool hasNoPredecessors() { return pred_begin() == pred_end(); }

  /// Returns true if this blocks has no successors.
  bool hasNoSuccessors() { return succ_begin() == succ_end(); }

  /// If this block has exactly one predecessor, return it.  Otherwise, return
  /// null.
  ///
  /// Note that if a block has duplicate predecessors from a single block (e.g.
  /// if you have a conditional branch with the same block as the true/false
  /// destinations) is not considered to be a single predecessor.
  Block *getSinglePredecessor();

  /// If this block has a unique predecessor, i.e., all incoming edges originate
  /// from one block, return it. Otherwise, return null.
  Block *getUniquePredecessor();

  // Indexed successor access.
  unsigned getNumSuccessors();
  Block *getSuccessor(unsigned i);

  // Successor iteration.
  using succ_iterator = SuccessorRange::iterator;
  succ_iterator succ_begin() { return getSuccessors().begin(); }
  succ_iterator succ_end() { return getSuccessors().end(); }
  SuccessorRange getSuccessors() { return SuccessorRange(this); }

  /// Return "true" if there is a path from this block to the given block
  /// (according to the successors relationship). Both blocks must be in the
  /// same region. Paths that contain a block from `except` do not count.
  /// This function returns "false" if `other` is in `except`.
  ///
  /// Note: This function performs a block graph traversal and its complexity
  /// linear in the number of blocks in the parent region.
  ///
  /// Note: Reachability is a necessary but insufficient condition for
  /// dominance. Do not use this function in places where you need to check for
  /// dominance.
  bool isReachable(Block *other, SmallPtrSet<Block *, 16> &&except = {});
````
- **EN**: This block groups callable interfaces such as `pred_begin`, `pred_iterator`, `getFirstUse`, `pred_end`, indicating how `Block` is queried or updated.
- **CN**: 该代码块聚合了 `pred_begin`, `pred_iterator`, `getFirstUse`, `pred_end` 等可调用接口，展示了如何查询或更新 `Block`。

### Lines 294-339
````cpp
  //===--------------------------------------------------------------------===//
  // Walkers
  //===--------------------------------------------------------------------===//

  /// Walk all nested operations, blocks (including this block) or regions,
  /// depending on the type of callback.
  ///
  /// The order in which operations, blocks or regions at the same nesting
  /// level are visited (e.g., lexicographical or reverse lexicographical order)
  /// is determined by `Iterator`. The walk order for enclosing operations,
  /// blocks or regions with respect to their nested ones is specified by
  /// `Order` (post-order by default).
  ///
  /// A callback on a operation or block is allowed to erase that operation or
  /// block if either:
  ///   * the walk is in post-order, or
  ///   * the walk is in pre-order and the walk is skipped after the erasure.
  ///
  /// See Operation::walk for more details.
  template <WalkOrder Order = WalkOrder::PostOrder,
            typename Iterator = ForwardIterator, typename FnT,
            typename ArgT = detail::first_argument<FnT>,
            typename RetT = detail::walkResultType<FnT>>
  RetT walk(FnT &&callback) {
    if constexpr (std::is_same<ArgT, Block *>::value &&
                  Order == WalkOrder::PreOrder) {
      // Pre-order walk on blocks: invoke the callback on this block.
      if constexpr (std::is_same<RetT, void>::value) {
        callback(this);
      } else {
        RetT result = callback(this);
        if (result.wasSkipped())
          return WalkResult::advance();
        if (result.wasInterrupted())
          return WalkResult::interrupt();
      }
    }

    // Walk nested operations, blocks or regions.
    if constexpr (std::is_same<RetT, void>::value) {
      walk<Order, Iterator>(begin(), end(), std::forward<FnT>(callback));
    } else {
      if (walk<Order, Iterator>(begin(), end(), std::forward<FnT>(callback))
              .wasInterrupted())
        return WalkResult::interrupt();
    }
````
- **EN**: This block groups callable interfaces such as `walk`, `constexpr`, `callback`, `wasSkipped`, indicating how `Block` is queried or updated.
- **CN**: 该代码块聚合了 `walk`, `constexpr`, `callback`, `wasSkipped` 等可调用接口，展示了如何查询或更新 `Block`。

### Lines 341-380
````cpp
    if constexpr (std::is_same<ArgT, Block *>::value &&
                  Order == WalkOrder::PostOrder) {
      // Post-order walk on blocks: invoke the callback on this block.
      return callback(this);
    }
    if constexpr (!std::is_same<RetT, void>::value)
      return WalkResult::advance();
  }

  /// Walk all nested operations, blocks (excluding this block) or regions,
  /// depending on the type of callback, in the specified [begin, end) range of
  /// this block.
  ///
  /// The order in which operations, blocks or regions at the same nesting
  /// level are visited (e.g., lexicographical or reverse lexicographical order)
  /// is determined by `Iterator`. The walk order for enclosing operations,
  /// blocks or regions with respect to their nested ones is specified by
  /// `Order` (post-order by default).
  ///
  /// A callback on a operation or block is allowed to erase that operation or
  /// block if either:
  ///   * the walk is in post-order, or
  ///   * the walk is in pre-order and the walk is skipped after the erasure.
  ///
  /// See Operation::walk for more details.
  template <WalkOrder Order = WalkOrder::PostOrder,
            typename Iterator = ForwardIterator, typename FnT,
            typename RetT = detail::walkResultType<FnT>>
  RetT walk(Block::iterator begin, Block::iterator end, FnT &&callback) {
    for (auto &op : llvm::make_early_inc_range(llvm::make_range(begin, end))) {
      if constexpr (std::is_same<RetT, WalkResult>::value) {
        if (detail::walk<Order, Iterator>(&op, callback).wasInterrupted())
          return WalkResult::interrupt();
      } else {
        detail::walk<Order, Iterator>(&op, callback);
      }
    }
    if constexpr (std::is_same<RetT, WalkResult>::value)
      return WalkResult::advance();
  }
````
- **EN**: This block groups callable interfaces such as `constexpr`, `callback`, `advance`, `walk`, indicating how `Block` is queried or updated.
- **CN**: 该代码块聚合了 `constexpr`, `callback`, `advance`, `walk` 等可调用接口，展示了如何查询或更新 `Block`。

### Lines 383-429
````cpp
  //===--------------------------------------------------------------------===//
  // Other
  //===--------------------------------------------------------------------===//

  /// Split the block into two blocks before the specified operation or
  /// iterator.
  ///
  /// Note that all operations BEFORE the specified iterator stay as part of
  /// the original basic block, and the rest of the operations in the original
  /// block are moved to the new block, including the old terminator.  The
  /// original block is left without a terminator.
  ///
  /// The newly formed Block is returned, and the specified iterator is
  /// invalidated.
  Block *splitBlock(iterator splitBefore);
  Block *splitBlock(Operation *splitBeforeOp) {
    return splitBlock(iterator(splitBeforeOp));
  }

  /// Returns pointer to member of operation list.
  static OpListType Block::*getSublistAccess(Operation *) {
    return &Block::operations;
  }

  void print(raw_ostream &os);
  void print(raw_ostream &os, AsmState &state);
  void dump();

  /// Print out the name of the block without printing its body.
  /// NOTE: The printType argument is ignored.  We keep it for compatibility
  /// with LLVM dominator machinery that expects it to exist.
  void printAsOperand(raw_ostream &os, bool printType = true);
  void printAsOperand(raw_ostream &os, AsmState &state);

private:
  /// Same as `without_terminator`, but assumes that the block is not empty.
  iterator_range<iterator> without_terminator_impl();

  /// Pair of the parent object that owns this block and a bit that signifies if
  /// the operations within this block have a valid ordering.
  llvm::PointerIntPair<Region *, /*IntBits=*/1, bool> parentValidOpOrderPair;

  /// This is the list of operations in the block.
  OpListType operations;

  /// This is the list of arguments to the block.
  std::vector<BlockArgument> arguments;
````
- **EN**: This block groups callable interfaces such as `splitBlock`, `iterator`, `getSublistAccess`, `print`, indicating how `Block` is queried or updated.
- **CN**: 该代码块聚合了 `splitBlock`, `iterator`, `getSublistAccess`, `print` 等可调用接口，展示了如何查询或更新 `Block`。

### Lines 430-437
````cpp
  Block(Block &) = delete;
  void operator=(Block &) = delete;

  friend struct llvm::ilist_traits<Block>;
};

raw_ostream &operator<<(raw_ostream &, Block &);
} // namespace mlir
````
- **EN**: This block groups callable interfaces such as `Block`, indicating how `Block` is queried or updated.
- **CN**: 该代码块聚合了 `Block` 等可调用接口，展示了如何查询或更新 `Block`。

### Lines 440-459
````cpp
namespace llvm {
template <>
struct DenseMapInfo<mlir::Block::iterator> {
  static mlir::Block::iterator getEmptyKey() {
    void *pointer = llvm::DenseMapInfo<void *>::getEmptyKey();
    return mlir::Block::iterator((mlir::Operation *)pointer);
  }
  static mlir::Block::iterator getTombstoneKey() {
    void *pointer = llvm::DenseMapInfo<void *>::getTombstoneKey();
    return mlir::Block::iterator((mlir::Operation *)pointer);
  }
  static unsigned getHashValue(mlir::Block::iterator iter) {
    return hash_value(iter.getNodePtr());
  }
  static bool isEqual(mlir::Block::iterator lhs, mlir::Block::iterator rhs) {
    return lhs == rhs;
  }
};

} // end namespace llvm
````
- **EN**: This C++ declaration introduces `DenseMapInfo` and establishes part of the API surface for `Block`. Representative entry points here include `getEmptyKey`, `iterator`, `getTombstoneKey`, `getHashValue`.
- **CN**: 该 C++ 声明引入了 `DenseMapInfo`，并构成 `Block` API 表面的一部分。 这一段可见的代表性接口包括 `getEmptyKey`, `iterator`, `getTombstoneKey`, `getHashValue`。

### Lines 461-461
````cpp
#endif // MLIR_IR_BLOCK_H
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
- **EN**: IR construction and mutation helpers
  **CN**: IR 构造与变更辅助接口
- **EN**: Type or attribute abstraction
  **CN**: 类型或属性抽象

## Dependencies / 依赖关系

- mlir/IR/BlockSupport.h
- mlir/IR/Visitors.h
- llvm/ADT/SmallPtrSet.h
