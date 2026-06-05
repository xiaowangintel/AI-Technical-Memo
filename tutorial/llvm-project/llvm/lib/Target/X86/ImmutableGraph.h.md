# ImmutableGraph.h — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/X86/ImmutableGraph.h`
- **Repository**: llvm/llvm-project
- **Purpose**: Declares interfaces and shared data structures for the immutable graph component in the core X86 backend. / 为X86 后端核心中的ImmutableGraph 对应组件声明接口与共享数据结构。

## Line-by-Line Analysis / 逐行分析
### Lines 1-50: File header, licensing, and opening context / 文件头、许可证与开场上下文
```cpp
//==========-- ImmutableGraph.h - A fast DAG implementation ---------=========//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
/// \file
/// Description: ImmutableGraph is a fast DAG implementation that cannot be
/// modified, except by creating a new ImmutableGraph. ImmutableGraph is
/// implemented as two arrays: one containing nodes, and one containing edges.
/// The advantages to this implementation are two-fold:
/// 1. Iteration and traversal operations benefit from cache locality.
/// 2. Operations on sets of nodes/edges are efficient, and representations of
///    those sets in memory are compact. For instance, a set of edges is
///    implemented as a bit vector, wherein each bit corresponds to one edge in
///    the edge array. This implies a lower bound of 64x spatial improvement
///    over, e.g., an llvm::DenseSet or llvm::SmallSet. It also means that
///    insert/erase/contains operations complete in negligible constant time:
///    insert and erase require one load and one store, and contains requires
///    just one load.
///
//===----------------------------------------------------------------------===//

#ifndef LLVM_LIB_TARGET_X86_IMMUTABLEGRAPH_H
#define LLVM_LIB_TARGET_X86_IMMUTABLEGRAPH_H

#include "llvm/ADT/BitVector.h"
#include "llvm/ADT/GraphTraits.h"
#include "llvm/ADT/STLExtras.h"
#include <algorithm>
#include <iterator>
#include <utility>
#include <vector>

namespace llvm {

template <typename NodeValueT, typename EdgeValueT> class ImmutableGraph {
  using Traits = GraphTraits<ImmutableGraph<NodeValueT, EdgeValueT> *>;
  template <typename> friend class ImmutableGraphBuilder;

public:
  using node_value_type = NodeValueT;
  using edge_value_type = EdgeValueT;
  using size_type = int;
  class Node;
  class Edge {
    friend class ImmutableGraph;
    template <typename> friend class ImmutableGraphBuilder;

```
**EN:** This section declares APIs, helper types, or inline logic related to the core X86 backend. Key symbols include Node, Edge. It defines contracts that implementation files depend on.

**CN:** 本段声明与X86 后端核心相关的 API、辅助类型或内联逻辑。关键符号包括 Node, Edge。这些内容定义了实现文件所依赖的契约。

### Lines 51-100: Type declarations and shared abstractions / 类型声明与共享抽象
```cpp
    const Node *Dest;
    edge_value_type Value;

  public:
    const Node *getDest() const { return Dest; };
    const edge_value_type &getValue() const { return Value; }
  };
  class Node {
    friend class ImmutableGraph;
    template <typename> friend class ImmutableGraphBuilder;

    const Edge *Edges;
    node_value_type Value;

  public:
    const node_value_type &getValue() const { return Value; }

    const Edge *edges_begin() const { return Edges; }
    // Nodes are allocated sequentially. Edges for a node are stored together.
    // The end of this Node's edges is the beginning of the next node's edges.
    // An extra node was allocated to hold the end pointer for the last real
    // node.
    const Edge *edges_end() const { return (this + 1)->Edges; }
    ArrayRef<Edge> edges() const {
      return ArrayRef(edges_begin(), edges_end());
    }
  };

protected:
  ImmutableGraph(std::unique_ptr<Node[]> Nodes, std::unique_ptr<Edge[]> Edges,
                 size_type NodesSize, size_type EdgesSize)
      : Nodes(std::move(Nodes)), Edges(std::move(Edges)), NodesSize(NodesSize),
        EdgesSize(EdgesSize) {}
  ImmutableGraph(const ImmutableGraph &) = delete;
  ImmutableGraph(ImmutableGraph &&) = delete;
  ImmutableGraph &operator=(const ImmutableGraph &) = delete;
  ImmutableGraph &operator=(ImmutableGraph &&) = delete;

public:
  ArrayRef<Node> nodes() const { return ArrayRef(Nodes.get(), NodesSize); }
  const Node *nodes_begin() const { return nodes().begin(); }
  const Node *nodes_end() const { return nodes().end(); }

  ArrayRef<Edge> edges() const { return ArrayRef(Edges.get(), EdgesSize); }
  const Edge *edges_begin() const { return edges().begin(); }
  const Edge *edges_end() const { return edges().end(); }

  size_type nodes_size() const { return NodesSize; }
  size_type edges_size() const { return EdgesSize; }

```
**EN:** This section declares APIs, helper types, or inline logic related to the core X86 backend. Key symbols include Node. It defines contracts that implementation files depend on.

**CN:** 本段声明与X86 后端核心相关的 API、辅助类型或内联逻辑。关键符号包括 Node。这些内容定义了实现文件所依赖的契约。

### Lines 101-150: Type declarations and shared abstractions / 类型声明与共享抽象
```cpp
  // Node N must belong to this ImmutableGraph.
  size_type getNodeIndex(const Node &N) const {
    return std::distance(nodes_begin(), &N);
  }
  // Edge E must belong to this ImmutableGraph.
  size_type getEdgeIndex(const Edge &E) const {
    return std::distance(edges_begin(), &E);
  }

  // FIXME: Could NodeSet and EdgeSet be templated to share code?
  class NodeSet {
    const ImmutableGraph &G;
    BitVector V;

  public:
    NodeSet(const ImmutableGraph &G, bool ContainsAll = false)
        : G{G}, V{static_cast<unsigned>(G.nodes_size()), ContainsAll} {}
    bool insert(const Node &N) {
      size_type Idx = G.getNodeIndex(N);
      bool AlreadyExists = V.test(Idx);
      V.set(Idx);
      return !AlreadyExists;
    }
    void erase(const Node &N) {
      size_type Idx = G.getNodeIndex(N);
      V.reset(Idx);
    }
    bool contains(const Node &N) const {
      size_type Idx = G.getNodeIndex(N);
      return V.test(Idx);
    }
    void clear() { V.reset(); }
    size_type empty() const { return V.none(); }
    /// Return the number of elements in the set
    size_type count() const { return V.count(); }
    /// Return the size of the set's domain
    size_type size() const { return V.size(); }
    /// Set union
    NodeSet &operator|=(const NodeSet &RHS) {
      assert(&this->G == &RHS.G);
      V |= RHS.V;
      return *this;
    }
    /// Set intersection
    NodeSet &operator&=(const NodeSet &RHS) {
      assert(&this->G == &RHS.G);
      V &= RHS.V;
      return *this;
    }
    /// Set disjoint union
```
**EN:** This section declares APIs, helper types, or inline logic related to the core X86 backend. Key symbols include NodeSet. It defines contracts that implementation files depend on.

**CN:** 本段声明与X86 后端核心相关的 API、辅助类型或内联逻辑。关键符号包括 NodeSet。这些内容定义了实现文件所依赖的契约。

### Lines 151-200: Type declarations and shared abstractions / 类型声明与共享抽象
```cpp
    NodeSet &operator^=(const NodeSet &RHS) {
      assert(&this->G == &RHS.G);
      V ^= RHS.V;
      return *this;
    }

    using index_iterator = typename BitVector::const_set_bits_iterator;
    index_iterator index_begin() const { return V.set_bits_begin(); }
    index_iterator index_end() const { return V.set_bits_end(); }
    void set(size_type Idx) { V.set(Idx); }
    void reset(size_type Idx) { V.reset(Idx); }

    class iterator {
      const NodeSet &Set;
      size_type Current;

      void advance() {
        assert(Current != -1);
        Current = Set.V.find_next(Current);
      }

    public:
      iterator(const NodeSet &Set, size_type Begin)
          : Set{Set}, Current{Begin} {}
      iterator operator++(int) {
        iterator Tmp = *this;
        advance();
        return Tmp;
      }
      iterator &operator++() {
        advance();
        return *this;
      }
      Node *operator*() const {
        assert(Current != -1);
        return Set.G.nodes_begin() + Current;
      }
      bool operator==(const iterator &other) const {
        assert(&this->Set == &other.Set);
        return this->Current == other.Current;
      }
      bool operator!=(const iterator &other) const { return !(*this == other); }
    };

    iterator begin() const { return iterator{*this, V.find_first()}; }
    iterator end() const { return iterator{*this, -1}; }
  };

  class EdgeSet {
    const ImmutableGraph &G;
```
**EN:** This section declares APIs, helper types, or inline logic related to the core X86 backend. Key symbols include iterator, EdgeSet. It defines contracts that implementation files depend on.

**CN:** 本段声明与X86 后端核心相关的 API、辅助类型或内联逻辑。关键符号包括 iterator, EdgeSet。这些内容定义了实现文件所依赖的契约。

### Lines 201-250: Function implementation and target-specific behavior / 函数实现与目标特定行为
```cpp
    BitVector V;

  public:
    EdgeSet(const ImmutableGraph &G, bool ContainsAll = false)
        : G{G}, V{static_cast<unsigned>(G.edges_size()), ContainsAll} {}
    bool insert(const Edge &E) {
      size_type Idx = G.getEdgeIndex(E);
      bool AlreadyExists = V.test(Idx);
      V.set(Idx);
      return !AlreadyExists;
    }
    void erase(const Edge &E) {
      size_type Idx = G.getEdgeIndex(E);
      V.reset(Idx);
    }
    bool contains(const Edge &E) const {
      size_type Idx = G.getEdgeIndex(E);
      return V.test(Idx);
    }
    void clear() { V.reset(); }
    bool empty() const { return V.none(); }
    /// Return the number of elements in the set
    size_type count() const { return V.count(); }
    /// Return the size of the set's domain
    size_type size() const { return V.size(); }
    /// Set union
    EdgeSet &operator|=(const EdgeSet &RHS) {
      assert(&this->G == &RHS.G);
      V |= RHS.V;
      return *this;
    }
    /// Set intersection
    EdgeSet &operator&=(const EdgeSet &RHS) {
      assert(&this->G == &RHS.G);
      V &= RHS.V;
      return *this;
    }
    /// Set disjoint union
    EdgeSet &operator^=(const EdgeSet &RHS) {
      assert(&this->G == &RHS.G);
      V ^= RHS.V;
      return *this;
    }

    using index_iterator = typename BitVector::const_set_bits_iterator;
    index_iterator index_begin() const { return V.set_bits_begin(); }
    index_iterator index_end() const { return V.set_bits_end(); }
    void set(size_type Idx) { V.set(Idx); }
    void reset(size_type Idx) { V.reset(Idx); }

```
**EN:** This section declares APIs, helper types, or inline logic related to the core X86 backend. It defines contracts that implementation files depend on.

**CN:** 本段声明与X86 后端核心相关的 API、辅助类型或内联逻辑。这些内容定义了实现文件所依赖的契约。

### Lines 251-300: Type declarations and shared abstractions / 类型声明与共享抽象
```cpp
    class iterator {
      const EdgeSet &Set;
      size_type Current;

      void advance() {
        assert(Current != -1);
        Current = Set.V.find_next(Current);
      }

    public:
      iterator(const EdgeSet &Set, size_type Begin)
          : Set{Set}, Current{Begin} {}
      iterator operator++(int) {
        iterator Tmp = *this;
        advance();
        return Tmp;
      }
      iterator &operator++() {
        advance();
        return *this;
      }
      Edge *operator*() const {
        assert(Current != -1);
        return Set.G.edges_begin() + Current;
      }
      bool operator==(const iterator &other) const {
        assert(&this->Set == &other.Set);
        return this->Current == other.Current;
      }
      bool operator!=(const iterator &other) const { return !(*this == other); }
    };

    iterator begin() const { return iterator{*this, V.find_first()}; }
    iterator end() const { return iterator{*this, -1}; }
  };

private:
  std::unique_ptr<Node[]> Nodes;
  std::unique_ptr<Edge[]> Edges;
  size_type NodesSize;
  size_type EdgesSize;
};

template <typename GraphT> class ImmutableGraphBuilder {
  using node_value_type = typename GraphT::node_value_type;
  using edge_value_type = typename GraphT::edge_value_type;
  static_assert(
      std::is_base_of<ImmutableGraph<node_value_type, edge_value_type>,
                      GraphT>::value,
      "Template argument to ImmutableGraphBuilder must derive from "
```
**EN:** This section declares APIs, helper types, or inline logic related to the core X86 backend. Key symbols include iterator. It defines contracts that implementation files depend on.

**CN:** 本段声明与X86 后端核心相关的 API、辅助类型或内联逻辑。关键符号包括 iterator。这些内容定义了实现文件所依赖的契约。

### Lines 301-350: Function implementation and target-specific behavior / 函数实现与目标特定行为
```cpp
      "ImmutableGraph<>");
  using size_type = typename GraphT::size_type;
  using NodeSet = typename GraphT::NodeSet;
  using Node = typename GraphT::Node;
  using EdgeSet = typename GraphT::EdgeSet;
  using Edge = typename GraphT::Edge;
  using BuilderEdge = std::pair<edge_value_type, size_type>;
  using EdgeList = std::vector<BuilderEdge>;
  using BuilderVertex = std::pair<node_value_type, EdgeList>;
  using VertexVec = std::vector<BuilderVertex>;

public:
  using BuilderNodeRef = size_type;

  BuilderNodeRef addVertex(const node_value_type &V) {
    auto I = AdjList.emplace(AdjList.end(), V, EdgeList{});
    return std::distance(AdjList.begin(), I);
  }

  void addEdge(const edge_value_type &E, BuilderNodeRef From,
               BuilderNodeRef To) {
    AdjList[From].second.emplace_back(E, To);
  }

  bool empty() const { return AdjList.empty(); }

  template <typename... ArgT> std::unique_ptr<GraphT> get(ArgT &&... Args) {
    size_type VertexSize = AdjList.size(), EdgeSize = 0;
    for (const auto &V : AdjList) {
      EdgeSize += V.second.size();
    }
    auto VertexArray =
        std::make_unique<Node[]>(VertexSize + 1 /* terminator node */);
    auto EdgeArray = std::make_unique<Edge[]>(EdgeSize);
    size_type VI = 0, EI = 0;
    for (; VI < VertexSize; ++VI) {
      VertexArray[VI].Value = std::move(AdjList[VI].first);
      VertexArray[VI].Edges = &EdgeArray[EI];
      auto NumEdges = static_cast<size_type>(AdjList[VI].second.size());
      for (size_type VEI = 0; VEI < NumEdges; ++VEI, ++EI) {
        auto &E = AdjList[VI].second[VEI];
        EdgeArray[EI].Value = std::move(E.first);
        EdgeArray[EI].Dest = &VertexArray[E.second];
      }
    }
    assert(VI == VertexSize && EI == EdgeSize && "ImmutableGraph malformed");
    VertexArray[VI].Edges = &EdgeArray[EdgeSize]; // terminator node
    return std::make_unique<GraphT>(std::move(VertexArray),
                                    std::move(EdgeArray), VertexSize, EdgeSize,
                                    std::forward<ArgT>(Args)...);
```
**EN:** This section declares APIs, helper types, or inline logic related to the core X86 backend. It defines contracts that implementation files depend on.

**CN:** 本段声明与X86 后端核心相关的 API、辅助类型或内联逻辑。这些内容定义了实现文件所依赖的契约。

### Lines 351-400: Function implementation and target-specific behavior / 函数实现与目标特定行为
```cpp
  }

  template <typename... ArgT>
  static std::unique_ptr<GraphT> trim(const GraphT &G, const NodeSet &TrimNodes,
                                      const EdgeSet &TrimEdges,
                                      ArgT &&... Args) {
    size_type NewVertexSize = G.nodes_size() - TrimNodes.count();
    size_type NewEdgeSize = G.edges_size() - TrimEdges.count();
    auto NewVertexArray =
        std::make_unique<Node[]>(NewVertexSize + 1 /* terminator node */);
    auto NewEdgeArray = std::make_unique<Edge[]>(NewEdgeSize);

    // Walk the nodes and determine the new index for each node.
    size_type NewNodeIndex = 0;
    std::vector<size_type> RemappedNodeIndex(G.nodes_size());
    for (const Node &N : G.nodes()) {
      if (TrimNodes.contains(N))
        continue;
      RemappedNodeIndex[G.getNodeIndex(N)] = NewNodeIndex++;
    }
    assert(NewNodeIndex == NewVertexSize &&
           "Should have assigned NewVertexSize indices");

    size_type VertexI = 0, EdgeI = 0;
    for (const Node &N : G.nodes()) {
      if (TrimNodes.contains(N))
        continue;
      NewVertexArray[VertexI].Value = N.getValue();
      NewVertexArray[VertexI].Edges = &NewEdgeArray[EdgeI];
      for (const Edge &E : N.edges()) {
        if (TrimEdges.contains(E))
          continue;
        NewEdgeArray[EdgeI].Value = E.getValue();
        size_type DestIdx = G.getNodeIndex(*E.getDest());
        size_type NewIdx = RemappedNodeIndex[DestIdx];
        assert(NewIdx < NewVertexSize);
        NewEdgeArray[EdgeI].Dest = &NewVertexArray[NewIdx];
        ++EdgeI;
      }
      ++VertexI;
    }
    assert(VertexI == NewVertexSize && EdgeI == NewEdgeSize &&
           "Gadget graph malformed");
    NewVertexArray[VertexI].Edges = &NewEdgeArray[NewEdgeSize]; // terminator
    return std::make_unique<GraphT>(std::move(NewVertexArray),
                                    std::move(NewEdgeArray), NewVertexSize,
                                    NewEdgeSize, std::forward<ArgT>(Args)...);
  }

private:
```
**EN:** This section declares APIs, helper types, or inline logic related to the core X86 backend. It defines contracts that implementation files depend on.

**CN:** 本段声明与X86 后端核心相关的 API、辅助类型或内联逻辑。这些内容定义了实现文件所依赖的契约。

### Lines 401-445: Preprocessor directives and structural setup / 预处理指令与结构初始化
```cpp
  VertexVec AdjList;
};

template <typename NodeValueT, typename EdgeValueT>
struct GraphTraits<ImmutableGraph<NodeValueT, EdgeValueT> *> {
  using GraphT = ImmutableGraph<NodeValueT, EdgeValueT>;
  using NodeRef = typename GraphT::Node const *;
  using EdgeRef = typename GraphT::Edge const &;

  static NodeRef edge_dest(EdgeRef E) { return E.getDest(); }
  using ChildIteratorType =
      mapped_iterator<typename GraphT::Edge const *, decltype(&edge_dest)>;

  static NodeRef getEntryNode(GraphT *G) { return G->nodes_begin(); }
  static ChildIteratorType child_begin(NodeRef N) {
    return {N->edges_begin(), &edge_dest};
  }
  static ChildIteratorType child_end(NodeRef N) {
    return {N->edges_end(), &edge_dest};
  }

  static NodeRef getNode(typename GraphT::Node const &N) { return NodeRef{&N}; }
  using nodes_iterator =
      mapped_iterator<typename GraphT::Node const *, decltype(&getNode)>;
  static nodes_iterator nodes_begin(GraphT *G) {
    return {G->nodes_begin(), &getNode};
  }
  static nodes_iterator nodes_end(GraphT *G) {
    return {G->nodes_end(), &getNode};
  }

  using ChildEdgeIteratorType = typename GraphT::Edge const *;

  static ChildEdgeIteratorType child_edge_begin(NodeRef N) {
    return N->edges_begin();
  }
  static ChildEdgeIteratorType child_edge_end(NodeRef N) {
    return N->edges_end();
  }
  static typename GraphT::size_type size(GraphT *G) { return G->nodes_size(); }
};

} // end namespace llvm

#endif // LLVM_LIB_TARGET_X86_IMMUTABLEGRAPH_H
```
**EN:** This section declares APIs, helper types, or inline logic related to the core X86 backend. Key symbols include GraphTraits. It defines contracts that implementation files depend on.

**CN:** 本段声明与X86 后端核心相关的 API、辅助类型或内联逻辑。关键符号包括 GraphTraits。这些内容定义了实现文件所依赖的契约。

## Key Concepts / 关键概念
- Primary topic: the immutable graph component. / 核心主题：ImmutableGraph 对应组件。
- Subsystem: the core X86 backend. / 所属子系统：X86 后端核心。
- Notable symbols: Node, Edge, NodeSet, iterator, EdgeSet, GraphTraits. / 重要符号：Node, Edge, NodeSet, iterator, EdgeSet, GraphTraits。
- The file emphasizes declarations, interfaces, and reusable helpers. / 该文件以声明、接口和可复用辅助逻辑为主。
- The implementation follows LLVM namespace conventions. / 实现遵循 LLVM 的命名空间约定。

## Dependencies / 依赖关系
- Direct includes: llvm/ADT/BitVector.h, llvm/ADT/GraphTraits.h, llvm/ADT/STLExtras.h, algorithm, iterator, utility, vector. / 直接包含：llvm/ADT/BitVector.h, llvm/ADT/GraphTraits.h, llvm/ADT/STLExtras.h, algorithm, iterator, utility, vector。
- Integrates with shared LLVM backend layers such as IR, CodeGen, MC, and X86 target support as needed. / 按需与 LLVM 的 IR、CodeGen、MC 和 X86 目标支持层集成。
- Implementation files in the same subsystem rely on these declarations. / 同一子系统中的实现文件依赖这些声明。
