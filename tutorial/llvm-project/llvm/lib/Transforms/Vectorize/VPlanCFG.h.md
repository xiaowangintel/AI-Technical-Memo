# VPlanCFG.h — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Vectorize/VPlanCFG.h` | `llvm/lib/Transforms/Vectorize/VPlanCFG.h` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This header defines the public interface for graphTraits for VP blocks within LLVM's vectorization and VPlan infrastructure layer. | 该头文件定义了对外接口 LLVM 向量化与 VPlan 基础设施层中的 VPlanCFG 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-37

```cpp
//===- VPlanCFG.h - GraphTraits for VP blocks -------------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
/// Specializations of GraphTraits that allow VPBlockBase graphs to be
/// treated as proper graphs for generic algorithms;
//===----------------------------------------------------------------------===//

#ifndef LLVM_TRANSFORMS_VECTORIZE_VPLANCFG_H
#define LLVM_TRANSFORMS_VECTORIZE_VPLANCFG_H

#include "VPlan.h"
#include "VPlanUtils.h"
#include "llvm/ADT/DepthFirstIterator.h"
#include "llvm/ADT/GraphTraits.h"
#include "llvm/ADT/PostOrderIterator.h"
#include "llvm/ADT/SmallVector.h"

namespace llvm {

//===----------------------------------------------------------------------===//
// GraphTraits specializations for VPlan Hierarchical Control-Flow Graphs     //
//===----------------------------------------------------------------------===//

/// Iterator to traverse all successors/predecessors of a VPBlockBase node,
/// including its hierarchical successors/predecessors:
///
///     A
///     |
///  +-----+ <- Region R
///  |  b  |
///  |     |
///  | ... |
///  |     |
```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. The header guard ensures the declarations are safe to include from multiple compilation units. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 头文件保护宏保证这些声明可以被多个编译单元安全地重复包含。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。

### Lines 38-80

```cpp
///  |  e  |
///  +-----+
///     |
///     B
///
///  Forward == true:
///    Region blocks themselves traverse only their entries directly.
///    Region's successor is implictly traversed when processing its exiting
///    block.
///    children(A) == {R}
///    children(R) == {b}
///    children(e) == {B}
///
///  Forward == false:
///    Region blocks themselves traverse only their exiting blocks directly.
///    Region's predecessor is implicitly traversed when processing its entry
///    block.
///    children(B) == {R}
///    children(R) == {e}
///    children(b) == {A}
///
/// The scheme described above ensures that all blocks of the region are visited
/// before continuing traversal outside the region when doing a reverse
/// post-order traversal of the VPlan.
template <typename BlockPtrTy, bool Forward = true>
class VPHierarchicalChildrenIterator
    : public iterator_facade_base<
          VPHierarchicalChildrenIterator<BlockPtrTy, Forward>,
          std::bidirectional_iterator_tag, VPBlockBase> {
  BlockPtrTy Block;
  /// Index of the current successor/predecessor. For VPBasicBlock nodes, this
  /// simply is the index for the successors/predecessors array. For
  /// VPRegionBlock, EdgeIdx == 0 is used for the region's entry/exiting block,
  /// and EdgeIdx - 1 are the indices for the successors/predecessors array.
  size_t EdgeIdx;

  static size_t getNumOutgoingEdges(BlockPtrTy Current) {
    if constexpr (Forward)
      return Current->getNumSuccessors();
    else
      return Current->getNumPredecessors();
  }

```
- EN: Core entities appearing here include VPHierarchicalChildrenIterator, getNumOutgoingEdges, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 VPHierarchicalChildrenIterator, getNumOutgoingEdges，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 81-115

```cpp
  static ArrayRef<BlockPtrTy> getOutgoingEdges(BlockPtrTy Current) {
    if constexpr (Forward)
      return Current->getSuccessors();
    else
      return Current->getPredecessors();
  }

  static BlockPtrTy getBlockWithOutgoingEdges(BlockPtrTy Current) {
    while (Current && getNumOutgoingEdges(Current) == 0)
      Current = Current->getParent();
    return Current;
  }

  /// Templated helper to dereference successor/predecessor \p EdgeIdx of \p
  /// Block. Used by both the const and non-const operator* implementations.
  template <typename T1> static T1 deref(T1 Block, unsigned EdgeIdx) {
    if (auto *R = dyn_cast<VPRegionBlock>(Block)) {
      assert(EdgeIdx == 0);
      if constexpr (Forward)
        return R->getEntry();
      else
        return R->getExiting();
    }

    // For exit blocks, use the next parent region with successors.
    return getOutgoingEdges(getBlockWithOutgoingEdges(Block))[EdgeIdx];
  }

public:
  /// Used by iterator_facade_base with bidirectional_iterator_tag.
  using reference = BlockPtrTy;

  VPHierarchicalChildrenIterator(BlockPtrTy Block, size_t Idx = 0)
      : Block(Block), EdgeIdx(Idx) {}

```
- EN: Core entities appearing here include getOutgoingEdges, getBlockWithOutgoingEdges, f, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 getOutgoingEdges, getBlockWithOutgoingEdges, f，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 116-156

```cpp
  static VPHierarchicalChildrenIterator end(BlockPtrTy Block) {
    if (auto *R = dyn_cast<VPRegionBlock>(Block)) {
      // Traverse through the region's entry/exiting (based on Forward) node.
      return {R, 1};
    }
    BlockPtrTy ParentWithOutgoingEdges = getBlockWithOutgoingEdges(Block);
    unsigned NumOutgoingEdges =
        ParentWithOutgoingEdges ? getNumOutgoingEdges(ParentWithOutgoingEdges)
                                : 0;
    return {Block, NumOutgoingEdges};
  }

  bool operator==(const VPHierarchicalChildrenIterator &R) const {
    return Block == R.Block && EdgeIdx == R.EdgeIdx;
  }

  const VPBlockBase *operator*() const { return deref(Block, EdgeIdx); }

  BlockPtrTy operator*() { return deref(Block, EdgeIdx); }

  VPHierarchicalChildrenIterator &operator++() {
    EdgeIdx++;
    return *this;
  }

  VPHierarchicalChildrenIterator &operator--() {
    EdgeIdx--;
    return *this;
  }

  VPHierarchicalChildrenIterator operator++(int X) {
    VPHierarchicalChildrenIterator Orig = *this;
    EdgeIdx++;
    return Orig;
  }
};

/// Helper for GraphTraits specialization that traverses through VPRegionBlocks.
template <typename BlockTy> class VPBlockDeepTraversalWrapper {
  BlockTy Entry;

```
- EN: Core entities appearing here include end, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 end，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 157-197

```cpp
public:
  VPBlockDeepTraversalWrapper(BlockTy Entry) : Entry(Entry) {}
  BlockTy getEntry() { return Entry; }
};

/// GraphTraits specialization to recursively traverse VPBlockBase nodes,
/// including traversing through VPRegionBlocks.  Exit blocks of a region
/// implicitly have their parent region's successors. This ensures all blocks in
/// a region are visited before any blocks in a successor region when doing a
/// reverse post-order traversal of the graph.
template <> struct GraphTraits<VPBlockDeepTraversalWrapper<VPBlockBase *>> {
  using NodeRef = VPBlockBase *;
  using ChildIteratorType = VPHierarchicalChildrenIterator<VPBlockBase *>;

  static NodeRef getEntryNode(VPBlockDeepTraversalWrapper<VPBlockBase *> N) {
    return N.getEntry();
  }

  static inline ChildIteratorType child_begin(NodeRef N) {
    return ChildIteratorType(N);
  }

  static inline ChildIteratorType child_end(NodeRef N) {
    return ChildIteratorType::end(N);
  }
};

template <>
struct GraphTraits<VPBlockDeepTraversalWrapper<const VPBlockBase *>> {
  using NodeRef = const VPBlockBase *;
  using ChildIteratorType = VPHierarchicalChildrenIterator<const VPBlockBase *>;

  static NodeRef
  getEntryNode(VPBlockDeepTraversalWrapper<const VPBlockBase *> N) {
    return N.getEntry();
  }

  static inline ChildIteratorType child_begin(NodeRef N) {
    return ChildIteratorType(N);
  }

```
- EN: Core entities appearing here include getEntryNode, child_begin, child_end, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 getEntryNode, child_begin, child_end，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 198-239

```cpp
  static inline ChildIteratorType child_end(NodeRef N) {
    return ChildIteratorType::end(N);
  }
};

/// Helper for GraphTraits specialization that does not traverses through
/// VPRegionBlocks.
template <typename BlockTy> class VPBlockShallowTraversalWrapper {
  BlockTy Entry;

public:
  VPBlockShallowTraversalWrapper(BlockTy Entry) : Entry(Entry) {}
  BlockTy getEntry() { return Entry; }
};

template <> struct GraphTraits<VPBlockShallowTraversalWrapper<VPBlockBase *>> {
  using NodeRef = VPBlockBase *;
  using ChildIteratorType = SmallVectorImpl<VPBlockBase *>::iterator;

  static NodeRef getEntryNode(VPBlockShallowTraversalWrapper<VPBlockBase *> N) {
    return N.getEntry();
  }

  static inline ChildIteratorType child_begin(NodeRef N) {
    return N->getSuccessors().begin();
  }

  static inline ChildIteratorType child_end(NodeRef N) {
    return N->getSuccessors().end();
  }
};

template <>
struct GraphTraits<VPBlockShallowTraversalWrapper<const VPBlockBase *>> {
  using NodeRef = const VPBlockBase *;
  using ChildIteratorType = SmallVectorImpl<VPBlockBase *>::const_iterator;

  static NodeRef
  getEntryNode(VPBlockShallowTraversalWrapper<const VPBlockBase *> N) {
    return N.getEntry();
  }

```
- EN: Core entities appearing here include child_end, getEntryNode, child_begin, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 child_end, getEntryNode, child_begin，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 240-276

```cpp
  static inline ChildIteratorType child_begin(NodeRef N) {
    return N->getSuccessors().begin();
  }

  static inline ChildIteratorType child_end(NodeRef N) {
    return N->getSuccessors().end();
  }
};

/// Returns an iterator range to traverse the graph starting at \p G in
/// depth-first order. The iterator won't traverse through region blocks.
inline iterator_range<
    df_iterator<VPBlockShallowTraversalWrapper<VPBlockBase *>>>
vp_depth_first_shallow(VPBlockBase *G) {
  return depth_first(VPBlockShallowTraversalWrapper<VPBlockBase *>(G));
}
inline iterator_range<
    df_iterator<VPBlockShallowTraversalWrapper<const VPBlockBase *>>>
vp_depth_first_shallow(const VPBlockBase *G) {
  return depth_first(VPBlockShallowTraversalWrapper<const VPBlockBase *>(G));
}

/// Returns the VPBasicBlocks forming the loop body of a plain (pre-region)
/// VPlan in reverse post-order starting from \p Header.
inline SmallVector<VPBasicBlock *>
vp_rpo_plain_cfg_loop_body(VPBasicBlock *Header) {
  assert(!Header->getParent() && "Header must not be inside a region");
  VPBlockBase *Middle = Header->getPredecessors()[1]->getSuccessors()[0];
  SmallVector<VPBasicBlock *> Result;
  ReversePostOrderTraversal<VPBlockShallowTraversalWrapper<VPBlockBase *>> RPOT(
      Header);
  for (VPBasicBlock *VPBB : VPBlockUtils::blocksAs<VPBasicBlock>(RPOT)) {
    if (VPBB == Middle)
      break;
    // Skip exit blocks.
    if (isa<VPIRBasicBlock>(VPBB)) {
      assert(is_contained(Header->getPlan()->getExitBlocks(), VPBB) &&
```
- EN: Core entities appearing here include child_begin, child_end, vp_depth_first_shallow, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 child_begin, child_end, vp_depth_first_shallow，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 277-317

```cpp
             "skipped VPIRBBs must be exit blocks");
      continue;
    }
    Result.push_back(VPBB);
  }
  return Result;
}

/// Returns an iterator range to traverse the graph starting at \p G in
/// depth-first order while traversing through region blocks.
inline iterator_range<df_iterator<VPBlockDeepTraversalWrapper<VPBlockBase *>>>
vp_depth_first_deep(VPBlockBase *G) {
  return depth_first(VPBlockDeepTraversalWrapper<VPBlockBase *>(G));
}
inline iterator_range<
    df_iterator<VPBlockDeepTraversalWrapper<const VPBlockBase *>>>
vp_depth_first_deep(const VPBlockBase *G) {
  return depth_first(VPBlockDeepTraversalWrapper<const VPBlockBase *>(G));
}

// The following set of template specializations implement GraphTraits to treat
// any VPBlockBase as a node in a graph of VPBlockBases. It's important to note
// that VPBlockBase traits don't recurse into VPRegioBlocks, i.e., if the
// VPBlockBase is a VPRegionBlock, this specialization provides access to its
// successors/predecessors but not to the blocks inside the region.

template <> struct GraphTraits<VPBlockBase *> {
  using NodeRef = VPBlockBase *;
  using ChildIteratorType = VPHierarchicalChildrenIterator<VPBlockBase *>;

  static NodeRef getEntryNode(NodeRef N) { return N; }

  static inline ChildIteratorType child_begin(NodeRef N) {
    return ChildIteratorType(N);
  }

  static inline ChildIteratorType child_end(NodeRef N) {
    return ChildIteratorType::end(N);
  }
};

```
- EN: Core entities appearing here include vp_depth_first_deep, child_begin, child_end, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 vp_depth_first_deep, child_begin, child_end，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 318-355

```cpp
template <> struct GraphTraits<const VPBlockBase *> {
  using NodeRef = const VPBlockBase *;
  using ChildIteratorType = VPHierarchicalChildrenIterator<const VPBlockBase *>;

  static NodeRef getEntryNode(NodeRef N) { return N; }

  static inline ChildIteratorType child_begin(NodeRef N) {
    return ChildIteratorType(N);
  }

  static inline ChildIteratorType child_end(NodeRef N) {
    return ChildIteratorType::end(N);
  }
};

template <> struct GraphTraits<Inverse<VPBlockBase *>> {
  using NodeRef = VPBlockBase *;
  using ChildIteratorType =
      VPHierarchicalChildrenIterator<VPBlockBase *, /*Forward=*/false>;

  static NodeRef getEntryNode(Inverse<NodeRef> B) { return B.Graph; }

  static inline ChildIteratorType child_begin(NodeRef N) {
    return ChildIteratorType(N);
  }

  static inline ChildIteratorType child_end(NodeRef N) {
    return ChildIteratorType::end(N);
  }
};

template <> struct GraphTraits<VPlan *> {
  using GraphRef = VPlan *;
  using NodeRef = VPBlockBase *;
  using nodes_iterator = df_iterator<NodeRef>;

  static NodeRef getEntryNode(GraphRef N) { return N->getEntry(); }

```
- EN: Core entities appearing here include child_begin, child_end, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 child_begin, child_end，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 356-369

```cpp
  static nodes_iterator nodes_begin(GraphRef N) {
    return nodes_iterator::begin(N->getEntry());
  }

  static nodes_iterator nodes_end(GraphRef N) {
    // df_iterator::end() returns an empty iterator so the node used doesn't
    // matter.
    return nodes_iterator::end(N->getEntry());
  }
};

} // namespace llvm

#endif // LLVM_TRANSFORMS_VECTORIZE_VPLANCFG_H
```
- EN: Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. Core entities appearing here include nodes_begin, nodes_end, which carry the local state, declarations, or transformation entry points for this region.
- CN: 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 此处出现的核心实体包括 nodes_begin, nodes_end，它们承载了该区间涉及的局部状态、声明或变换入口。

## Key Concepts / 关键概念

- EN: Domain: `Vectorize` focuses on vectorization and VPlan infrastructure.
  - CN: 领域：`Vectorize` 主要处理 向量化与 VPlan 基础设施。
- EN: Primary entities: `VPHierarchicalChildrenIterator, GraphTraits, getNumOutgoingEdges, getOutgoingEdges, getBlockWithOutgoingEdges, f, end, getEntryNode` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`VPHierarchicalChildrenIterator, GraphTraits, getNumOutgoingEdges, getOutgoingEdges, getBlockWithOutgoingEdges, f, end, getEntryNode` 构成该文件对外 API 与主要实现挂钩。
- EN: Header role: it mostly exposes declarations, contracts, inline helpers, and reusable utilities rather than driving a standalone transform by itself.
  - CN: 头文件角色：它主要暴露声明、约定、内联辅助函数和可复用工具，而不是独立执行某个完整变换。

## Dependencies / 依赖关系

- EN: Utility headers: `llvm/ADT/DepthFirstIterator.h`, `llvm/ADT/GraphTraits.h`, `llvm/ADT/PostOrderIterator.h`, `llvm/ADT/SmallVector.h` contribute containers, diagnostics, casting, hashing, and other infrastructure services.
  - CN: 工具头文件：`llvm/ADT/DepthFirstIterator.h`, `llvm/ADT/GraphTraits.h`, `llvm/ADT/PostOrderIterator.h`, `llvm/ADT/SmallVector.h` 提供了容器、诊断、类型转换、哈希等基础设施能力。
- EN: Standard/other headers: `VPlan.h`, `VPlanUtils.h` supply C++ language utilities used alongside LLVM APIs.
  - CN: 标准库/其他头文件：`VPlan.h`, `VPlanUtils.h` 为 LLVM API 之外的 C++ 语言工具提供支持。
