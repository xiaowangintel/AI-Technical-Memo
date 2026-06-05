# Utils.h — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/include/mlir/Dialect/Affine/Analysis/Utils.h`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This header file defines prototypes for various transformation utilities for memref's and non-loop IR structures. These are not passes by themselves but are used either by passes, optimization sequences, or in turn by other transformation utilities.
  - **CN**: 该文件位于 `mlir/include/mlir/Dialect/Affine/Analysis`，围绕 Affine 方言公开 `Utils` 相关的接口、规则或生成式定义。

## Line-by-Line Analysis / 逐行分析

### Lines 1-14
```cpp
//===- Utils.h - General analysis utilities ---------------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This header file defines prototypes for various transformation utilities for
// memref's and non-loop IR structures. These are not passes by themselves but
// are used either by passes, optimization sequences, or in turn by other
// transformation utilities.
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and file-level summary.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件级摘要。

### Lines 15-23
```cpp

#ifndef MLIR_DIALECT_AFFINE_ANALYSIS_UTILS_H
#define MLIR_DIALECT_AFFINE_ANALYSIS_UTILS_H

#include "mlir/Dialect/Affine/Analysis/AffineStructures.h"
#include "mlir/Dialect/Affine/IR/AffineOps.h"
#include <memory>
#include <optional>

```
- **EN**: Pulls in the headers needed by this declaration unit, including `mlir/Dialect/Affine/Analysis/AffineStructures.h`, `mlir/Dialect/Affine/IR/AffineOps.h`, `memory`, `optional`.
- **CN**: 引入该声明单元所需的头文件，其中包括 `mlir/Dialect/Affine/Analysis/AffineStructures.h`, `mlir/Dialect/Affine/IR/AffineOps.h`, `memory`, `optional`。

### Lines 24-31
```cpp
namespace mlir {
class Block;
class Location;
class Operation;
class Value;

namespace affine {
class AffineForOp;
```
- **EN**: Introduces declarations for `mlir`, `Block`, `Location`, `Operation`, and 3 more symbols, establishing the public symbols or declarative records used later.
- **CN**: 引入 `mlir`, `Block`, `Location`, `Operation`, and 3 more symbols 等声明，建立后续使用的公共符号或声明式记录。

### Lines 32-47
```cpp
class AffineValueMap;
struct MemRefAccess;

// LoopNestStateCollector walks loop nests and collects load and store
// operations, and whether or not a region holding op other than ForOp and IfOp
// was encountered in the loop nest.
struct LoopNestStateCollector {
  SmallVector<AffineForOp, 4> forOps;
  // Affine loads.
  SmallVector<Operation *, 4> loadOpInsts;
  // Affine stores.
  SmallVector<Operation *, 4> storeOpInsts;
  // Non-affine loads.
  SmallVector<Operation *, 4> memrefLoads;
  // Non-affine stores.
  SmallVector<Operation *, 4> memrefStores;
```
- **EN**: Introduces declarations for `AffineValueMap`, `MemRefAccess`, `LoopNestStateCollector`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `AffineValueMap`, `MemRefAccess`, `LoopNestStateCollector` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 48-55
```cpp
  // Free operations.
  SmallVector<Operation *, 4> memrefFrees;

  // Collects load and store operations, and whether or not a region holding op
  // other than ForOp and IfOp was encountered in the loop nest.
  void collect(Operation *opToWalk);
};

```
- **EN**: Declares APIs or declarative rules around `collect`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 声明与 `collect` 相关的 API 或声明式规则；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 56-63
```cpp
// MemRefDependenceGraph is a graph data structure where graph nodes are
// top-level operations in a `Block` and edges are memref dependences or SSA
// dependences (on memrefs) between the nodes. Nodes are created for all
// top-level operations except in certain cases (see `init` method). Edges are
// created between nodes with a dependence (see `Edge` documentation). Edges
// aren't created from/to nodes that have no memory effects.
struct MemRefDependenceGraph {
public:
```
- **EN**: Introduces declarations for `MemRefDependenceGraph`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `MemRefDependenceGraph` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 64-79
```cpp
  // Node represents a node in the graph. A Node is either an entire loop nest
  // rooted at the top level which contains loads/stores, or a top level
  // load/store.
  struct Node {
    // The unique identifier of this node in the graph.
    unsigned id;
    // The top-level statement which is (or contains) a load/store.
    Operation *op;
    // List of affine loads.
    SmallVector<Operation *, 4> loads;
    // List of non-affine loads.
    SmallVector<Operation *, 4> memrefLoads;
    // List of affine store ops.
    SmallVector<Operation *, 4> stores;
    // List of non-affine stores.
    SmallVector<Operation *, 4> memrefStores;
```
- **EN**: Introduces declarations for `Node`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `Node` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 80-89
```cpp
    // List of free operations.
    SmallVector<Operation *, 4> memrefFrees;
    // Set of private memrefs used in this node.
    DenseSet<Value> privateMemrefs;

    Node(unsigned id, Operation *op) : id(id), op(op) {}

    // Returns the load op count for 'memref'.
    unsigned getLoadOpCount(Value memref) const;

```
- **EN**: Implements logic around `Node`, `getLoadOpCount`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `Node`, `getLoadOpCount` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 90-99
```cpp
    // Returns the store op count for 'memref'.
    unsigned getStoreOpCount(Value memref) const;

    /// Returns true if there exists an operation with a write memory effect to
    /// `memref` in this node.
    unsigned hasStore(Value memref) const;

    // Returns true if the node has a free op on `memref`.
    unsigned hasFree(Value memref) const;

```
- **EN**: Declares APIs or declarative rules around `getStoreOpCount`, `hasStore`, `hasFree`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `getStoreOpCount`, `hasStore`, `hasFree` 相关的 API 或声明式规则；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 100-107
```cpp
    // Returns all store ops in 'storeOps' which access 'memref'.
    void getStoreOpsForMemref(Value memref,
                              SmallVectorImpl<Operation *> *storeOps) const;

    // Returns all load ops in 'loadOps' which access 'memref'.
    void getLoadOpsForMemref(Value memref,
                             SmallVectorImpl<Operation *> *loadOps) const;

```
- **EN**: Declares APIs or declarative rules around `getStoreOpsForMemref`, `getLoadOpsForMemref`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `getStoreOpsForMemref`, `getLoadOpsForMemref` 相关的 API 或声明式规则；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 108-123
```cpp
    // Returns all memrefs in 'loadAndStoreMemrefSet' for which this node
    // has at least one load and store operation.
    void getLoadAndStoreMemrefSet(DenseSet<Value> *loadAndStoreMemrefSet) const;
  };

  // Edge represents a data dependence between nodes in the graph. It can either
  // be a memory dependence or an SSA dependence. In the former case, it
  // corresponds to a pair of memory accesses to the same memref or aliasing
  // memrefs where at least one of them has a write or free memory effect. The
  // memory accesses need not be affine load/store operations. Operations are
  // checked for read/write effects and edges may be added conservatively. Edges
  // are not created to/from nodes that have no memory effect. An exception to
  // this are SSA dependences between operations that define memrefs (like
  // alloc's, view-like ops) and their memory-effecting users that are enclosed
  // in loops.
  struct Edge {
```
- **EN**: Introduces declarations for `Edge`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `Edge` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 124-139
```cpp
    // The id of the node at the other end of the edge.
    // If this edge is stored in Edge = Node.inEdges[i], then
    // 'Node.inEdges[i].id' is the identifier of the source node of the edge.
    // If this edge is stored in Edge = Node.outEdges[i], then
    // 'Node.outEdges[i].id' is the identifier of the dest node of the edge.
    unsigned id;
    // The SSA value on which this edge represents a dependence.
    // If the value is a memref, then the dependence is between graph nodes
    // which contain accesses to the same memref 'value'. If the value is a
    // non-memref value, then the dependence is between a graph node which
    // defines an SSA value and another graph node which uses the SSA value
    // (e.g. a constant or load operation defining a value which is used inside
    // a loop nest).
    Value value;
  };

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 140-153
```cpp
  // Map from node id to Node.
  DenseMap<unsigned, Node> nodes;
  // Map from node id to list of input edges. The absence of an entry for a key
  // is also equivalent to the absence of any edges.
  DenseMap<unsigned, SmallVector<Edge, 2>> inEdges;
  // Map from node id to list of output edges. The absence of an entry for a
  // node is also equivalent to the absence of any edges.
  DenseMap<unsigned, SmallVector<Edge, 2>> outEdges;
  // Map from memref to a count on the dependence edges associated with that
  // memref.
  DenseMap<Value, unsigned> memrefEdgeCount;
  // The next unique identifier to use for newly created graph nodes.
  unsigned nextNodeId = 0;

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 154-167
```cpp
  MemRefDependenceGraph(Block &block) : block(block) {}

  // Initializes the data dependence graph by iterating over the operations of
  // the MDG's `block`. A `Node` is created for every top-level op except for
  // side-effect-free operations with zero results and no regions. Assigns each
  // node in the graph a node id based on the order in block. Fails if certain
  // kinds of operations, for which `Node` creation isn't supported, are
  // encountered (unknown region holding ops). If `fullAffineDependences` is
  // set, affine memory dependence analysis is performed before concluding that
  // conflicting affine memory accesses lead to a dependence check; otherwise, a
  // pair of conflicting affine memory accesses (where one of them is a store
  // and they are to the same memref) always leads to an edge (conservatively).
  bool init(bool fullAffineDependences = true);

```
- **EN**: Implements logic around `MemRefDependenceGraph`, `init`; this block works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `MemRefDependenceGraph`, `init` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 168-177
```cpp
  // Returns the graph node for 'id'.
  const Node *getNode(unsigned id) const;
  Node *getNode(unsigned id) {
    return const_cast<Node *>(
        static_cast<const MemRefDependenceGraph *>(this)->getNode(id));
  }

  // Returns true if the graph has node with ID `id`.
  bool hasNode(unsigned id) const { return nodes.contains(id); }

```
- **EN**: Implements logic around `getNode`, `hasNode`.
- **CN**: 围绕 `getNode`, `hasNode` 实现具体逻辑。

### Lines 178-187
```cpp
  // Returns the graph node for 'forOp'.
  const Node *getForOpNode(AffineForOp forOp) const;
  Node *getForOpNode(AffineForOp forOp) {
    return const_cast<Node *>(
        static_cast<const MemRefDependenceGraph *>(this)->getForOpNode(forOp));
  }

  // Adds a node with 'op' to the graph and returns its unique identifier.
  unsigned addNode(Operation *op);

```
- **EN**: Implements logic around `getForOpNode`, `addNode`.
- **CN**: 围绕 `getForOpNode`, `addNode` 实现具体逻辑。

### Lines 188-199
```cpp
  // Remove node 'id' (and its associated edges) from graph.
  void removeNode(unsigned id);

  // Returns true if node 'id' writes to any memref which escapes (or is an
  // argument to) the block. Returns false otherwise.
  bool writesToLiveInOrEscapingMemrefs(unsigned id) const;

  // Returns true iff there is an edge from node 'srcId' to node 'dstId' which
  // is for 'value' if non-null, or for any value otherwise. Returns false
  // otherwise.
  bool hasEdge(unsigned srcId, unsigned dstId, Value value = nullptr) const;

```
- **EN**: Declares APIs or declarative rules around `removeNode`, `writesToLiveInOrEscapingMemrefs`, `hasEdge`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `removeNode`, `writesToLiveInOrEscapingMemrefs`, `hasEdge` 相关的 API 或声明式规则；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 200-210
```cpp
  // Adds an edge from node 'srcId' to node 'dstId' for 'value'.
  void addEdge(unsigned srcId, unsigned dstId, Value value);

  // Removes an edge from node 'srcId' to node 'dstId' for 'value'.
  void removeEdge(unsigned srcId, unsigned dstId, Value value);

  // Returns true if there is a path in the dependence graph from node 'srcId'
  // to node 'dstId'. Returns false otherwise. `srcId`, `dstId`, and the
  // operations that the edges connected are expected to be from the same block.
  bool hasDependencePath(unsigned srcId, unsigned dstId) const;

```
- **EN**: Declares APIs or declarative rules around `addEdge`, `removeEdge`, `hasDependencePath`.
- **CN**: 声明与 `addEdge`, `removeEdge`, `hasDependencePath` 相关的 API 或声明式规则。

### Lines 211-218
```cpp
  // Returns the input edge count for node 'id' and 'memref' from src nodes
  // which access 'memref' with a store operation.
  unsigned getIncomingMemRefAccesses(unsigned id, Value memref) const;

  // Returns the output edge count for node 'id' and 'memref' (if non-null),
  // otherwise returns the total output edge count from node 'id'.
  unsigned getOutEdgeCount(unsigned id, Value memref = nullptr) const;

```
- **EN**: Declares APIs or declarative rules around `getIncomingMemRefAccesses`, `getOutEdgeCount`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `getIncomingMemRefAccesses`, `getOutEdgeCount` 相关的 API 或声明式规则；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 219-228
```cpp
  /// Return all nodes which define SSA values used in node 'id'.
  void gatherDefiningNodes(unsigned id,
                           DenseSet<unsigned> &definingNodes) const;

  // Computes and returns an insertion point operation, before which the
  // the fused <srcId, dstId> loop nest can be inserted while preserving
  // dependences. Returns nullptr if no such insertion point is found.
  Operation *getFusedLoopNestInsertionPoint(unsigned srcId,
                                            unsigned dstId) const;

```
- **EN**: Declares APIs or declarative rules around `gatherDefiningNodes`, `getFusedLoopNestInsertionPoint`.
- **CN**: 声明与 `gatherDefiningNodes`, `getFusedLoopNestInsertionPoint` 相关的 API 或声明式规则。

### Lines 229-236
```cpp
  // Updates edge mappings from node 'srcId' to node 'dstId' after fusing them,
  // taking into account that:
  //   *) if 'removeSrcId' is true, 'srcId' will be removed after fusion,
  //   *) memrefs in 'privateMemRefs' has been replaced in node at 'dstId' by a
  //      private memref.
  void updateEdges(unsigned srcId, unsigned dstId,
                   const DenseSet<Value> &privateMemRefs, bool removeSrcId);

```
- **EN**: Declares APIs or declarative rules around `updateEdges`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `updateEdges` 相关的 API 或声明式规则；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 237-247
```cpp
  // Update edge mappings for nodes 'sibId' and 'dstId' to reflect fusion
  // of sibling node 'sibId' into node 'dstId'.
  void updateEdges(unsigned sibId, unsigned dstId);

  // Adds the specified ops to lists of node at 'id'.
  void addToNode(unsigned id, ArrayRef<Operation *> loads,
                 ArrayRef<Operation *> stores,
                 ArrayRef<Operation *> memrefLoads,
                 ArrayRef<Operation *> memrefStores,
                 ArrayRef<Operation *> memrefFrees);

```
- **EN**: Declares APIs or declarative rules around `updateEdges`, `addToNode`.
- **CN**: 声明与 `updateEdges`, `addToNode` 相关的 API 或声明式规则。

### Lines 248-259
```cpp
  void clearNodeLoadAndStores(unsigned id);

  // Calls 'callback' for each input edge incident to node 'id' which carries a
  // memref dependence.
  void forEachMemRefInputEdge(unsigned id,
                              const std::function<void(Edge)> &callback);

  // Calls 'callback' for each output edge from node 'id' which carries a
  // memref dependence.
  void forEachMemRefOutputEdge(unsigned id,
                               const std::function<void(Edge)> &callback);

```
- **EN**: Declares APIs or declarative rules around `clearNodeLoadAndStores`, `forEachMemRefInputEdge`, `function`, `forEachMemRefOutputEdge`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `clearNodeLoadAndStores`, `forEachMemRefInputEdge`, `function`, `forEachMemRefOutputEdge` 相关的 API 或声明式规则；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 260-268
```cpp
  // Calls 'callback' for each edge in 'edges' which carries a memref
  // dependence.
  void forEachMemRefEdge(ArrayRef<Edge> edges,
                         const std::function<void(Edge)> &callback);

  void print(raw_ostream &os) const;

  void dump() const { print(llvm::errs()); }

```
- **EN**: Implements logic around `forEachMemRefEdge`, `function`, `print`, `dump`; this block handles textual assembly parsing or printing concerns; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `forEachMemRefEdge`, `function`, `print`, `dump` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并协调核心结构化 MLIR 方言之间的行为。

### Lines 269-277
```cpp
  /// The block for which this graph is created to perform fusion.
  Block &block;
};

/// Populates 'loops' with IVs of the affine.for ops surrounding 'op' ordered
/// from the outermost 'affine.for' operation to the innermost one while not
/// traversing outside of the surrounding affine scope.
void getAffineForIVs(Operation &op, SmallVectorImpl<AffineForOp> *loops);

```
- **EN**: Declares APIs or declarative rules around `getAffineForIVs`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `getAffineForIVs` 相关的 API 或声明式规则；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 278-285
```cpp
/// Populates 'ivs' with IVs of the surrounding affine.for and affine.parallel
/// ops ordered from the outermost one to the innermost while not traversing
/// outside of the surrounding affine scope.
void getAffineIVs(Operation &op, SmallVectorImpl<Value> &ivs);

/// Populates 'ops' with affine operations enclosing `op` ordered from outermost
/// to innermost while stopping at the boundary of the affine scope. affine.for,
/// affine.if, or affine.parallel ops comprise such surrounding affine ops.
```
- **EN**: Declares APIs or declarative rules around `getAffineIVs`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `getAffineIVs` 相关的 API 或声明式规则；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 286-293
```cpp
/// `ops` is guaranteed by design to have a successive chain of affine parent
/// ops.
void getEnclosingAffineOps(Operation &op, SmallVectorImpl<Operation *> *ops);

/// Returns the nesting depth of this operation, i.e., the number of loops
/// surrounding this operation.
unsigned getNestingDepth(Operation *op);

```
- **EN**: Declares APIs or declarative rules around `getEnclosingAffineOps`, `getNestingDepth`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `getEnclosingAffineOps`, `getNestingDepth` 相关的 API 或声明式规则；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 294-301
```cpp
/// Returns whether a loop is a parallel loop and contains a reduction loop.
bool isLoopParallelAndContainsReduction(AffineForOp forOp);

/// Returns in 'sequentialLoops' all sequential loops in loop nest rooted
/// at 'forOp'.
void getSequentialLoops(AffineForOp forOp,
                        llvm::SmallDenseSet<Value, 8> *sequentialLoops);

```
- **EN**: Declares APIs or declarative rules around `isLoopParallelAndContainsReduction`, `getSequentialLoops`.
- **CN**: 声明与 `isLoopParallelAndContainsReduction`, `getSequentialLoops` 相关的 API 或声明式规则。

### Lines 302-313
```cpp
/// Enumerates different result statuses of slice computation by
/// `computeSliceUnion`
// TODO: Identify and add different kinds of failures during slice computation.
struct SliceComputationResult {
  enum ResultEnum {
    Success,
    IncorrectSliceFailure, // Slice is computed, but it is incorrect.
    GenericFailure,        // Unable to compute src loop computation slice.
  } value;
  SliceComputationResult(ResultEnum v) : value(v) {}
};

```
- **EN**: Introduces declarations for `SliceComputationResult`, `ResultEnum`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `SliceComputationResult`, `ResultEnum` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 314-329
```cpp
/// ComputationSliceState aggregates loop IVs, loop bound AffineMaps and their
/// associated operands for a set of loops within a loop nest (typically the
/// set of loops surrounding a store operation). Loop bound AffineMaps which
/// are non-null represent slices of that loop's iteration space.
struct ComputationSliceState {
  // List of sliced loop IVs (ordered from outermost to innermost).
  // EX: 'ivs[i]' has lower bound 'lbs[i]' and upper bound 'ubs[i]'.
  SmallVector<Value, 4> ivs;
  // List of lower bound AffineMaps.
  SmallVector<AffineMap, 4> lbs;
  // List of upper bound AffineMaps.
  SmallVector<AffineMap, 4> ubs;
  // List of lower bound operands (lbOperands[i] are used by 'lbs[i]').
  std::vector<SmallVector<Value, 4>> lbOperands;
  // List of upper bound operands (ubOperands[i] are used by 'ubs[i]').
  std::vector<SmallVector<Value, 4>> ubOperands;
```
- **EN**: Introduces declarations for `ComputationSliceState`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `ComputationSliceState` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 330-339
```cpp
  // Slice loop nest insertion point in target loop nest.
  Block::iterator insertPoint;
  // Adds to 'cst' with constraints which represent the slice bounds on 'ivs'
  // in 'this'. Specifically, the values in 'ivs' are added to 'cst' as dim
  // variables and the values in 'lb/ubOperands' are added as symbols.
  // Constraints are added for all loop IV bounds (dim or symbol), and
  // constraints are added for slice bounds in 'lbs'/'ubs'.
  // Returns failure if we cannot add loop bounds because of unsupported cases.
  LogicalResult getAsConstraints(FlatAffineValueConstraints *cst) const;

```
- **EN**: Declares APIs or declarative rules around `getAsConstraints`.
- **CN**: 声明与 `getAsConstraints` 相关的 API 或声明式规则。

### Lines 340-348
```cpp
  /// Adds to 'cst' constraints which represent the original loop bounds on
  /// 'ivs' in 'this'. This corresponds to the original domain of the loop nest
  /// from which the slice is being computed. Returns failure if we cannot add
  /// loop bounds because of unsupported cases.
  LogicalResult getSourceAsConstraints(FlatAffineValueConstraints &cst) const;

  // Clears all bounds and operands in slice state.
  void clearBounds();

```
- **EN**: Declares APIs or declarative rules around `getSourceAsConstraints`, `clearBounds`.
- **CN**: 声明与 `getSourceAsConstraints`, `clearBounds` 相关的 API 或声明式规则。

### Lines 349-358
```cpp
  /// Returns true if the computation slice is empty.
  bool isEmpty() const { return ivs.empty(); }

  /// Returns true if the computation slice encloses all the iterations of the
  /// sliced loop nest. Returns false if it does not. Returns std::nullopt if it
  /// cannot determine if the slice is maximal or not.
  // TODO: Cache 'isMaximal' so that we don't recompute it when the slice
  // information hasn't changed.
  std::optional<bool> isMaximal() const;

```
- **EN**: Implements logic around `isEmpty`, `isMaximal`.
- **CN**: 围绕 `isEmpty`, `isMaximal` 实现具体逻辑。

### Lines 359-366
```cpp
  /// Checks the validity of the slice computed. This is done using the
  /// following steps:
  /// 1. Get the new domain of the slice that would be created if fusion
  /// succeeds. This domain gets constructed with source loop IVS and
  /// destination loop IVS as dimensions.
  /// 2. Project out the dimensions of the destination loop from the domain
  /// above calculated in step(1) to express it purely in terms of the source
  /// loop IVs.
```
- **EN**: Documents the next declarations, design intent, or usage constraints for this file.
- **CN**: 为后续声明、设计意图或使用约束提供说明。

### Lines 367-374
```cpp
  /// 3. Calculate a set difference between the iterations of the new domain and
  /// the original domain of the source loop.
  /// If this difference is empty, the slice is declared to be valid. Otherwise,
  /// return false as it implies that the effective fusion results in at least
  /// one iteration of the slice that was not originally in the source's domain.
  /// If the validity cannot be determined, returns std::nullopt.
  std::optional<bool> isSliceValid() const;

```
- **EN**: Declares APIs or declarative rules around `isSliceValid`.
- **CN**: 声明与 `isSliceValid` 相关的 API 或声明式规则。

### Lines 375-382
```cpp
  void dump() const;

private:
  /// Fast check to determine if the computation slice is maximal. Returns true
  /// if each slice dimension maps to an existing dst dimension and both the src
  /// and the dst loops for those dimensions have the same bounds. Returns false
  /// if both the src and the dst loops don't have the same bounds. Returns
  /// std::nullopt if none of the above can be proven.
```
- **EN**: Declares APIs or declarative rules around `dump`.
- **CN**: 声明与 `dump` 相关的 API 或声明式规则。

### Lines 383-390
```cpp
  std::optional<bool> isSliceMaximalFastCheck() const;
};

/// Computes the computation slice loop bounds for one loop nest as affine maps
/// of the other loop nest's IVs and symbols, using 'dependenceConstraints'
/// computed between 'depSourceAccess' and 'depSinkAccess'.
/// If 'isBackwardSlice' is true, a backwards slice is computed in which the
/// slice bounds of loop nest surrounding 'depSourceAccess' are computed in
```
- **EN**: Declares APIs or declarative rules around `isSliceMaximalFastCheck`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `isSliceMaximalFastCheck` 相关的 API 或声明式规则；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 391-406
```cpp
/// terms of loop IVs and symbols of the loop nest surrounding 'depSinkAccess'
/// at 'loopDepth'.
/// If 'isBackwardSlice' is false, a forward slice is computed in which the
/// slice bounds of loop nest surrounding 'depSinkAccess' are computed in terms
/// of loop IVs and symbols of the loop nest surrounding 'depSourceAccess' at
/// 'loopDepth'.
/// The slice loop bounds and associated operands are returned in 'sliceState'.
//
//  Backward slice example:
//
//    affine.for %i0 = 0 to 10 {
//      affine.store %cst, %0[%i0] : memref<100xf32>  // 'depSourceAccess'
//    }
//    affine.for %i1 = 0 to 10 {
//      %v = affine.load %0[%i1] : memref<100xf32>    // 'depSinkAccess'
//    }
```
- **EN**: Documents the next declarations, design intent, or usage constraints for this file.
- **CN**: 为后续声明、设计意图或使用约束提供说明。

### Lines 407-422
```cpp
//
//    // Backward computation slice of loop nest '%i0'.
//    affine.for %i0 = (d0) -> (d0)(%i1) to (d0) -> (d0 + 1)(%i1) {
//      affine.store %cst, %0[%i0] : memref<100xf32>  // 'depSourceAccess'
//    }
//
//  Forward slice example:
//
//    affine.for %i0 = 0 to 10 {
//      affine.store %cst, %0[%i0] : memref<100xf32>  // 'depSourceAccess'
//    }
//    affine.for %i1 = 0 to 10 {
//      %v = affine.load %0[%i1] : memref<100xf32>    // 'depSinkAccess'
//    }
//
//    // Forward computation slice of loop nest '%i1'.
```
- **EN**: Documents the next declarations, design intent, or usage constraints for this file.
- **CN**: 为后续声明、设计意图或使用约束提供说明。

### Lines 423-431
```cpp
//    affine.for %i1 = (d0) -> (d0)(%i0) to (d0) -> (d0 + 1)(%i0) {
//      %v = affine.load %0[%i1] : memref<100xf32>    // 'depSinkAccess'
//    }
//
void getComputationSliceState(
    Operation *depSourceOp, Operation *depSinkOp,
    const FlatAffineValueConstraints &dependenceConstraints, unsigned loopDepth,
    bool isBackwardSlice, ComputationSliceState *sliceState);

```
- **EN**: Implements logic around `getComputationSliceState`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getComputationSliceState` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 432-439
```cpp
/// Return the number of iterations for the `slicetripCountMap` provided.
uint64_t getSliceIterationCount(
    const llvm::SmallDenseMap<Operation *, uint64_t, 8> &sliceTripCountMap);

/// Builds a map 'tripCountMap' from AffineForOp to constant trip count for
/// loop nest surrounding represented by slice loop bounds in 'slice'. Returns
/// true on success, false otherwise (if a non-constant trip count was
/// encountered).
```
- **EN**: Declares APIs or declarative rules around `getSliceIterationCount`.
- **CN**: 声明与 `getSliceIterationCount` 相关的 API 或声明式规则。

### Lines 440-447
```cpp
bool buildSliceTripCountMap(
    const ComputationSliceState &slice,
    llvm::SmallDenseMap<Operation *, uint64_t, 8> *tripCountMap);

/// Computes in 'sliceUnion' the union of all slice bounds computed at
/// 'loopDepth' between all dependent pairs of ops in 'opsA' and 'opsB', and
/// then verifies if it is valid. The parameter 'numCommonLoops' is the number
/// of loops common to the operations in 'opsA' and 'opsB'. If 'isBackwardSlice'
```
- **EN**: Declares APIs or declarative rules around `buildSliceTripCountMap`.
- **CN**: 声明与 `buildSliceTripCountMap` 相关的 API 或声明式规则。

### Lines 448-459
```cpp
/// is true, computes slice bounds for loop nest surrounding ops in 'opsA', as a
/// function of IVs and symbols of loop nest surrounding ops in 'opsB' at
/// 'loopDepth'. If 'isBackwardSlice' is false, computes slice bounds for loop
/// nest surrounding ops in 'opsB', as a function of IVs and symbols of loop
/// nest surrounding ops in 'opsA' at 'loopDepth'. Returns
/// 'SliceComputationResult::Success' if union was computed correctly, an
/// appropriate 'failure' otherwise.
SliceComputationResult
computeSliceUnion(ArrayRef<Operation *> opsA, ArrayRef<Operation *> opsB,
                  unsigned loopDepth, unsigned numCommonLoops,
                  bool isBackwardSlice, ComputationSliceState *sliceUnion);

```
- **EN**: Declares APIs or declarative rules around `computeSliceUnion`.
- **CN**: 声明与 `computeSliceUnion` 相关的 API 或声明式规则。

### Lines 460-474
```cpp
/// Creates a clone of the computation contained in the loop nest surrounding
/// 'srcOpInst', slices the iteration space of src loop based on slice bounds
/// in 'sliceState', and inserts the computation slice at the beginning of the
/// operation block of the loop at 'dstLoopDepth' in the loop nest surrounding
/// 'dstOpInst'. Returns the top-level loop of the computation slice on
/// success, returns nullptr otherwise.
// Loop depth is a crucial optimization choice that determines where to
// materialize the results of the backward slice - presenting a trade-off b/w
// storage and redundant computation in several cases.
// TODO: Support computation slices with common surrounding loops.
AffineForOp insertBackwardComputationSlice(Operation *srcOpInst,
                                           Operation *dstOpInst,
                                           unsigned dstLoopDepth,
                                           ComputationSliceState *sliceState);

```
- **EN**: Declares APIs or declarative rules around `insertBackwardComputationSlice`.
- **CN**: 声明与 `insertBackwardComputationSlice` 相关的 API 或声明式规则。

### Lines 475-489
```cpp
/// A region of a memref's data space; this is typically constructed by
/// analyzing load/store op's on this memref and the index space of loops
/// surrounding such op's.
// For example, the memref region for a load operation at loop depth = 1:
//
//    affine.for %i = 0 to 32 {
//      affine.for %ii = %i to (d0) -> (d0 + 8) (%i) {
//        affine.load %A[%ii]
//      }
//    }
//
// Region:  {memref = %A, write = false, {%i <= m0 <= %i + 7} }
// The last field is a 2-d FlatAffineValueConstraints symbolic in %i.
//
struct MemRefRegion {
```
- **EN**: Introduces declarations for `MemRefRegion`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `MemRefRegion` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 490-497
```cpp
  explicit MemRefRegion(Location loc) : loc(loc) {}

  /// Computes the memory region accessed by this memref with the region
  /// represented as constraints symbolic/parametric in 'loopDepth' loops
  /// surrounding opInst. The computed region's 'cst' field has exactly as many
  /// dimensional variables as the rank of the memref, and *potentially*
  /// additional symbolic variables which could include any of the loop IVs
  /// surrounding opInst up until 'loopDepth' and another additional Function
```
- **EN**: Implements logic around `MemRefRegion`; this block works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `MemRefRegion` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 498-505
```cpp
  /// symbols involved with the access (for eg., those appear in affine.apply's,
  /// loop bounds, etc.). If 'sliceState' is non-null, operands from
  /// 'sliceState' are added as symbols, and the following constraints are added
  /// to the system:
  /// *) Inequality constraints which represent loop bounds for 'sliceState'
  ///    operands which are loop IVS (these represent the destination loop IVs
  ///    of the slice, and are added as symbols to MemRefRegion's constraint
  ///    system).
```
- **EN**: Documents the next declarations, design intent, or usage constraints for this file.
- **CN**: 为后续声明、设计意图或使用约束提供说明。

### Lines 506-513
```cpp
  /// *) Inequality constraints for the slice bounds in 'sliceState', which
  ///    represent the bounds on the loop IVs in this constraint system w.r.t
  ///    to slice operands (which correspond to symbols).
  /// If 'addMemRefDimBounds' is true, constant upper/lower bounds
  /// [0, memref.getDimSize(i)) are added for each MemRef dimension 'i'.
  /// If `dropLocalVars` is true, all local variables in `cst` are projected
  /// out.
  ///
```
- **EN**: Documents the next declarations, design intent, or usage constraints for this file.
- **CN**: 为后续声明、设计意图或使用约束提供说明。

### Lines 514-521
```cpp
  ///  For example, the memref region for this operation at loopDepth = 1 will
  ///  be:
  ///
  ///    affine.for %i = 0 to 32 {
  ///      affine.for %ii = %i to (d0) -> (d0 + 8) (%i) {
  ///        load %A[%ii]
  ///      }
  ///    }
```
- **EN**: Documents the next declarations, design intent, or usage constraints for this file.
- **CN**: 为后续声明、设计意图或使用约束提供说明。

### Lines 522-529
```cpp
  ///
  ///   {memref = %A, write = false, {%i <= m0 <= %i + 7} }
  /// The last field is a 2-d FlatAffineValueConstraints symbolic in %i.
  ///
  /// If `dropOuterIVs` is true, project out any IVs other than those among
  /// `loopDepth` surrounding IVs, which would be symbols. If `dropOuterIVs`
  /// is false, the IVs would be turned into local variables instead of being
  /// projected out.
```
- **EN**: Documents the next declarations, design intent, or usage constraints for this file.
- **CN**: 为后续声明、设计意图或使用约束提供说明。

### Lines 530-539
```cpp
  LogicalResult compute(Operation *op, unsigned loopDepth,
                        const ComputationSliceState *sliceState = nullptr,
                        bool addMemRefDimBounds = true,
                        bool dropLocalVars = true, bool dropOuterIVs = true);

  FlatAffineValueConstraints *getConstraints() { return &cst; }
  const FlatAffineValueConstraints *getConstraints() const { return &cst; }
  bool isWrite() const { return write; }
  void setWrite(bool flag) { write = flag; }

```
- **EN**: Implements logic around `compute`, `getConstraints`, `isWrite`, `setWrite`.
- **CN**: 围绕 `compute`, `getConstraints`, `isWrite`, `setWrite` 实现具体逻辑。

### Lines 540-547
```cpp
  /// Returns a constant upper bound on the number of elements in this region if
  /// bounded by a known constant (always possible for static shapes),
  /// std::nullopt otherwise. Note that the symbols of the region are treated
  /// specially, i.e., the returned bounding constant holds for *any given*
  /// value of the symbol variables. The 'shape' vector is set to the
  /// corresponding dimension-wise bounds major to minor. The number of elements
  /// and all the dimension-wise bounds are guaranteed to be non-negative. We
  /// use int64_t instead of uint64_t since index types can be at most
```
- **EN**: Documents the next declarations, design intent, or usage constraints for this file.
- **CN**: 为后续声明、设计意图或使用约束提供说明。

### Lines 548-555
```cpp
  /// int64_t. `lbs` are set to the lower bound maps for each of the rank
  /// dimensions where each of these maps is purely symbolic in the constraints
  /// set's symbols.
  std::optional<int64_t> getConstantBoundingSizeAndShape(
      SmallVectorImpl<int64_t> *shape = nullptr,
      SmallVectorImpl<AffineMap> *lbs = nullptr) const;

  /// Gets the lower and upper bound map for the dimensional variable at
```
- **EN**: Declares APIs or declarative rules around `getConstantBoundingSizeAndShape`.
- **CN**: 声明与 `getConstantBoundingSizeAndShape` 相关的 API 或声明式规则。

### Lines 556-565
```cpp
  /// `pos`.
  void getLowerAndUpperBound(unsigned pos, AffineMap &lbMap,
                             AffineMap &ubMap) const;

  /// Returns the size of this MemRefRegion in bytes.
  std::optional<int64_t> getRegionSize();

  // Wrapper around FlatAffineValueConstraints::unionBoundingBox.
  LogicalResult unionBoundingBox(const MemRefRegion &other);

```
- **EN**: Declares APIs or declarative rules around `getLowerAndUpperBound`, `getRegionSize`, `unionBoundingBox`.
- **CN**: 声明与 `getLowerAndUpperBound`, `getRegionSize`, `unionBoundingBox` 相关的 API 或声明式规则。

### Lines 566-574
```cpp
  /// Returns the rank of the memref that this region corresponds to.
  unsigned getRank() const;

  /// Memref that this region corresponds to.
  Value memref;

  /// Read or write.
  bool write = false;

```
- **EN**: Declares APIs or declarative rules around `getRank`; this block works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `getRank` 相关的 API 或声明式规则；该代码块处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 575-582
```cpp
  /// If there is more than one load/store op associated with the region, the
  /// location information would correspond to one of those op's.
  Location loc;

  /// Region (data space) of the memref accessed. This set will thus have at
  /// least as many dimensional variables as the shape dimensionality of the
  /// memref, and these are the leading dimensions of the set appearing in that
  /// order (major to minor / outermost to innermost). There may be additional
```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 583-590
```cpp
  /// variables since getMemRefRegion() is called with a specific loop depth,
  /// and thus the region is symbolic in the outer surrounding loops at that
  /// depth.
  FlatAffineValueConstraints cst;
};

/// Returns the size of a memref with element type int or float in bytes if it's
/// statically shaped, std::nullopt otherwise.
```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 591-599
```cpp
std::optional<uint64_t> getIntOrFloatMemRefSizeInBytes(MemRefType memRefType);

/// Checks a load or store op for an out of bound access; returns failure if the
/// access is out of bounds along any of the dimensions, success otherwise.
/// Emits a diagnostic error (with location information) if emitError is true.
template <typename LoadOrStoreOpPointer>
LogicalResult boundCheckLoadOrStoreOp(LoadOrStoreOpPointer loadOrStoreOp,
                                      bool emitError = true);

```
- **EN**: Declares APIs or declarative rules around `getIntOrFloatMemRefSizeInBytes`, `boundCheckLoadOrStoreOp`.
- **CN**: 声明与 `getIntOrFloatMemRefSizeInBytes`, `boundCheckLoadOrStoreOp` 相关的 API 或声明式规则。

### Lines 600-607
```cpp
/// Returns the number of surrounding loops common to both A and B.
unsigned getNumCommonSurroundingLoops(Operation &a, Operation &b);

/// Gets the memory footprint of all data touched in the specified memory space
/// in bytes; if the memory space is unspecified, considers all memory spaces.
std::optional<int64_t> getMemoryFootprintBytes(AffineForOp forOp,
                                               int memorySpace = -1);

```
- **EN**: Declares APIs or declarative rules around `getNumCommonSurroundingLoops`, `getMemoryFootprintBytes`.
- **CN**: 声明与 `getNumCommonSurroundingLoops`, `getMemoryFootprintBytes` 相关的 API 或声明式规则。

### Lines 608-615
```cpp
/// Returns the memref's element type's size in bytes where the elemental type
/// is an int or float or a vector of such types.
std::optional<int64_t> getMemRefIntOrFloatEltSizeInBytes(MemRefType memRefType);

/// Simplify the integer set by simplifying the underlying affine expressions by
/// flattening and some simple inference. Also, drop any duplicate constraints.
/// Returns the simplified integer set. This method runs in time linear in the
/// number of constraints.
```
- **EN**: Declares APIs or declarative rules around `getMemRefIntOrFloatEltSizeInBytes`; this block works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `getMemRefIntOrFloatEltSizeInBytes` 相关的 API 或声明式规则；该代码块处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 616-623
```cpp
IntegerSet simplifyIntegerSet(IntegerSet set);

/// Returns the innermost common loop depth for the set of operations in 'ops'.
unsigned getInnermostCommonLoopDepth(
    ArrayRef<Operation *> ops,
    SmallVectorImpl<AffineForOp> *surroundingLoops = nullptr);

/// Try to simplify the given affine.min or affine.max op to an affine map with
```
- **EN**: Declares APIs or declarative rules around `simplifyIntegerSet`, `getInnermostCommonLoopDepth`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `simplifyIntegerSet`, `getInnermostCommonLoopDepth` 相关的 API 或声明式规则；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 624-631
```cpp
/// a single result and operands, taking into account the specified constraint
/// set. Return failure if no simplified version could be found.
FailureOr<AffineValueMap>
simplifyConstrainedMinMaxOp(Operation *op,
                            FlatAffineValueConstraints constraints);

/// Find the innermost common `Block` of `a` and `b` in the affine scope
/// that `a` and `b` are part of. Return nullptr if they belong to different
```
- **EN**: Declares APIs or declarative rules around `simplifyConstrainedMinMaxOp`; this block works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `simplifyConstrainedMinMaxOp` 相关的 API 或声明式规则；该代码块处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 632-640
```cpp
/// affine scopes. Also, return nullptr if they do not have a common `Block`
/// ancestor (for eg., when they are part of the `then` and `else` regions
/// of an op that itself starts an affine scope.
mlir::Block *findInnermostCommonBlockInScope(mlir::Operation *a,
                                             mlir::Operation *b);

} // namespace affine
} // namespace mlir

```
- **EN**: Introduces declarations for `affine`, `mlir`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `affine`, `mlir` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 641-641
```cpp
#endif // MLIR_DIALECT_AFFINE_ANALYSIS_UTILS_H
```
- **EN**: Defines preprocessor-controlled structure, include guards, generated hook points, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、头文件保护、生成式钩子或编译期常量。

## Key Concepts / 关键概念

- **Dialect declarations / 方言声明**:
  - **EN**: Defines the public include-surface for a dialect, exposing operations, attributes, types, or interfaces.
  - **CN**: 定义方言的公共头文件表面，暴露操作、属性、类型或接口。
- **Textual assembly syntax / 文本汇编语法**:
  - **EN**: Implements dialect-specific parsing and printing hooks for MLIR assembly.
  - **CN**: 实现 MLIR 汇编的方言专用解析与打印钩子。
- **Structured IR coordination / 结构化 IR 协同**:
  - **EN**: Interacts with structured MLIR dialects that model loops, tensors, memory, and vector semantics.
  - **CN**: 与建模循环、张量、内存和向量语义的结构化 MLIR 方言交互。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/Affine/Analysis/AffineStructures.h`, `mlir/Dialect/Affine/IR/AffineOps.h`
- **Standard-library headers / 标准库头文件**: `<memory>`, `<optional>`
- **Subsystem categories / 子系统类别**: dialect-specific operations, attributes, types, transforms, or interface declarations / 方言相关的操作、属性、类型、变换或接口声明 (2)
