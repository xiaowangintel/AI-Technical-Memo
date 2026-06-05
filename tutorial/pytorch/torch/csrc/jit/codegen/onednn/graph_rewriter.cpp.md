# graph_rewriter.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/csrc/jit/codegen/onednn/graph_rewriter.cpp`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Contains code-generation logic or generated declarations that synthesize JIT-related bindings, schemas, or helper code. This specific file centers on `graph_rewriter.cpp`. Alias and mutation tracking are important here. Generation of declarations, registrations, or helper code is part of the responsibility.
- **Purpose (CN)**: 包含代码生成逻辑或生成后的声明，用于产出 JIT 相关绑定、模式或辅助代码。 该文件具体围绕 `graph_rewriter.cpp` 展开。 别名关系与可变状态跟踪是这里的核心。 其职责包含声明、注册或辅助代码的生成。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20 / 第 1-20 行

```cpp
#include <torch/csrc/jit/codegen/onednn/graph_fuser.h>
#include <torch/csrc/jit/ir/alias_analysis.h>
#include <torch/csrc/jit/jit_log.h>

namespace torch::jit::fuser::onednn {

void GraphRewriter::cleanupSubgraphs() {
  auto curNode = *block_->nodes().rbegin();
  while (curNode != *block_->nodes().rend()) {
    // Save the previous node, since we might delete `curNode` in next block
    auto prevNode = curNode->prev();
    if (llgaHelper_.isLlgaSubgraph(curNode)) {
      // Unmerge subgraph if we don't get every nodes of a partition
      // into the subgraph due to failed alias check
      llgaHelper_.unmergeIfAnyNodeIsMissing(curNode);
    }
    curNode = prevNode;
  }
  for (Node* n : block_->nodes()) {
    for (Block* b : n->blocks()) {
```

- **EN:** It enters or references namespace scopes such as torch::jit::fuser::onednn, which positions the code inside the JIT subsystem hierarchy.
- **CN:** 这里进入或引用了 torch::jit::fuser::onednn 等命名空间作用域，用于把代码放入 JIT 子系统层级中。
- **EN:** Important callable entry points in this range include cleanupSubgraphs.
- **CN:** 这一段的重要可调用入口包括 cleanupSubgraphs。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** State, aliasing, or mutation tracking is emphasized in this block.
- **CN:** 该代码块强调了状态、别名关系或可变性跟踪。

### Lines 21-40 / 第 21-40 行

```cpp
      GraphRewriter(b, graph_, aliasDb_).cleanupSubgraphs();
    }
  }
}

void GraphRewriter::buildupSubgraphs() {
  // We need to run the rewriter multiple times in order to get all merge
  // opportunities. This is because moveBeforeTopologicalValid may reorder
  // nodes to be AFTER the current iteration point. In order to properly
  // consider those nodes for merging, we need run the pass until no changes
  // have been made.
  //
  // Example:
  //   c = f(a, b)
  //   d = f(c)
  //   e = f(d)  <- iter is here, moving upward
  // After c.moveBeforeTopologicallyValid(e), we have:
  //   c = f(a, b)
  //   e = f(d)  <- iter still here
  //   d = f(c)  <- this was node moved on the other side.
```

- **EN:** Important callable entry points in this range include GraphRewriter, buildupSubgraphs.
- **CN:** 这一段的重要可调用入口包括 GraphRewriter, buildupSubgraphs。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** State, aliasing, or mutation tracking is emphasized in this block.
- **CN:** 该代码块强调了状态、别名关系或可变性跟踪。
- **EN:** Concepts touched here: Graph IR / 图中间表示, Nodes and values / 节点与值, Optimization pass / 优化 pass, Alias analysis / 别名分析, Declared symbols / 声明的符号.
- **CN:** 这里涉及的概念包括：Graph IR / 图中间表示, Nodes and values / 节点与值, Optimization pass / 优化 pass, Alias analysis / 别名分析, Declared symbols / 声明的符号。

### Lines 41-60 / 第 41-60 行

```cpp
  // see [workblocks]
  auto workblocks = buildWorkBlocks();
  for (auto& workblock : workblocks) {
    bool any_changed = true;
    while (any_changed) {
      any_changed = false;
      auto workblock_end = workblock.end()->reverseIterator();
      auto workblock_begin = workblock.begin()->reverseIterator();
      for (auto it = workblock_end; it != workblock_begin;) {
        bool changed = false;
        std::tie(it, changed) = scanNode(*it, workblock_begin);
        any_changed |= changed;
      }
    }
  }

  // Construct Subgraphs Recursively
  for (Node* n : block_->nodes()) {
    for (auto subBlock : n->blocks()) {
      GraphRewriter(subBlock, graph_, aliasDb_).buildupSubgraphs();
```

- **EN:** Important callable entry points in this range include tie, GraphRewriter.
- **CN:** 这一段的重要可调用入口包括 tie, GraphRewriter。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** State, aliasing, or mutation tracking is emphasized in this block.
- **CN:** 该代码块强调了状态、别名关系或可变性跟踪。
- **EN:** Loop constructs iterate over IR elements, schema entries, or serialized records in this range.
- **CN:** 这一段中的循环用于遍历 IR 元素、模式条目或序列化记录。

### Lines 61-80 / 第 61-80 行

```cpp
    }
  }
}

std::vector<WorkBlock> GraphRewriter::buildWorkBlocks() {
  // [workblocks]
  // the IR has many nodes which can never be reordered around, such as a
  // prim::Bailout. if a node N is surrounded by two nodes which cannot be
  // reordered, A and B, then a fusion group that is created from N
  // can only contain nodes from (A, B) The nodes from A to B represent one
  // work block for the subgraph rewriter to work on. By creating these up
  // front, we avoid retraversing the whole graph block any time scanNode
  // returns
  Node* end_bound_node = block_->return_node();
  Node* curr = end_bound_node->prev();
  std::vector<WorkBlock> worklist;
  while (curr != block_->param_node()) {
    // cannot reorder around side effectful nodes
    if (curr->hasSideEffects()) {
      worklist.emplace_back(curr, end_bound_node);
```

- **EN:** Important callable entry points in this range include buildWorkBlocks.
- **CN:** 这一段的重要可调用入口包括 buildWorkBlocks。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。
- **EN:** Loop constructs iterate over IR elements, schema entries, or serialized records in this range.
- **CN:** 这一段中的循环用于遍历 IR 元素、模式条目或序列化记录。

### Lines 81-100 / 第 81-100 行

```cpp
      end_bound_node = curr;
    }
    curr = curr->prev();
  }
  worklist.emplace_back(curr, end_bound_node);
  return worklist;
}

std::pair<graph_node_list::iterator, bool> GraphRewriter::scanNode(
    Node* consumer,
    graph_node_list::iterator workblock_begin) {
  GRAPH_DEBUG("Scanning ", consumer->kind().toQualString());
  if (llgaHelper_.shouldConsiderForMerge(consumer)) {
    if (!llgaHelper_.isLlgaSubgraph(consumer)) {
      consumer = llgaHelper_.createSingletonSubgraph(consumer, aliasDb_);
    }
    // Iterate through the workblock to merge nodes of the
    // same partition determined by LLGA graph helper.
    // Nodes like B and C do not share a common input but belong to a
    // same partition, and thus we cannot only scan the input nodes
```

- **EN:** Important callable entry points in this range include scanNode, GRAPH_DEBUG.
- **CN:** 这一段的重要可调用入口包括 scanNode, GRAPH_DEBUG。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** State, aliasing, or mutation tracking is emphasized in this block.
- **CN:** 该代码块强调了状态、别名关系或可变性跟踪。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。

### Lines 101-120 / 第 101-120 行

```cpp
    // to find merging opportunities. Instead, we have to scan through
    // the whole workblock, which might lead to O^2 accesses in worst case
    //              A
    //      + - - / - \ - - +
    //      |    B     C    |
    //      |    |     |    |
    //      |    D     E    |
    //      + - - \ - / - - +
    //              F
    auto prev = ++consumer->reverseIterator();
    for (auto it = prev; it != workblock_begin; it++) {
      if (auto group = tryMerge(consumer, *it)) {
        // we successfully merged, so the new group's `inputs` may have
        // changed. So rescan the new group for more merging opportunities.
        return std::make_pair(group.value()->reverseIterator(), true);
      }
    }
  }
  return std::make_pair(++consumer->reverseIterator(), false);
}
```

- **EN:** Important callable entry points in this range include make_pair.
- **CN:** 这一段的重要可调用入口包括 make_pair。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。
- **EN:** Loop constructs iterate over IR elements, schema entries, or serialized records in this range.
- **CN:** 这一段中的循环用于遍历 IR 元素、模式条目或序列化记录。

### Lines 121-135 / 第 121-135 行

```cpp

// Try to merge `producer` into `consumer`. If successful, this destroys
// `producer` and returns the `consumer` group.
std::optional<Node*> GraphRewriter::tryMerge(Node* consumer, Node* producer) {
  AT_ASSERT(llgaHelper_.isLlgaSubgraph(consumer));
  bool canMerge = llgaHelper_.shouldMerge(producer, consumer) &&
      aliasDb_.moveBeforeTopologicallyValid(producer, consumer);
  if (!canMerge) {
    return std::nullopt;
  }
  llgaHelper_.mergeNodeIntoSubgraph(producer, consumer, aliasDb_);
  return consumer;
}

} // namespace torch::jit::fuser::onednn
```

- **EN:** Important callable entry points in this range include tryMerge, AT_ASSERT.
- **CN:** 这一段的重要可调用入口包括 tryMerge, AT_ASSERT。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** State, aliasing, or mutation tracking is emphasized in this block.
- **CN:** 该代码块强调了状态、别名关系或可变性跟踪。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。

## Key Concepts / 关键概念

- **Generated code pipeline** — 代码生成流水线
- **Graph IR** — 图中间表示
- **Nodes and values** — 节点与值
- **Control-flow blocks** — 控制流块
- **Operator schema** — 算子模式
- **Code generation** — 代码生成
- **Optimization pass** — 优化 pass
- **Alias analysis** — 别名分析
- **Core symbols: cleanupSubgraphs, GraphRewriter, buildupSubgraphs, tie, buildWorkBlocks, scanNode, GRAPH_DEBUG, make_pair** — 核心符号：cleanupSubgraphs、GraphRewriter、buildupSubgraphs、tie、buildWorkBlocks、scanNode、GRAPH_DEBUG、make_pair

## Dependencies / 依赖关系

- `torch/csrc/jit/codegen/onednn/graph_fuser.h`
- `torch/csrc/jit/ir/alias_analysis.h`
- `torch/csrc/jit/jit_log.h`
