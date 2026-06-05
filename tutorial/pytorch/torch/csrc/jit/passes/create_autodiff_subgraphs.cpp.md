# create_autodiff_subgraphs.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `torch/csrc/jit/passes/create_autodiff_subgraphs.cpp`
- Repository: `pytorch`
- Purpose (EN): Implements the TorchScript JIT pass logic for create autodiff subgraphs, including graph analysis and rewrites.
- 用途 (CN): 实现与 create autodiff subgraphs 相关的 TorchScript JIT pass 逻辑，包括图分析与改写。

## Line-by-Line Analysis / 逐行分析
### Lines 1-10
```cpp
#include <torch/csrc/jit/passes/create_autodiff_subgraphs.h>

#include <c10/util/Exception.h>
#include <torch/csrc/jit/ir/alias_analysis.h>
#include <torch/csrc/jit/ir/ir.h>
#include <torch/csrc/jit/jit_log.h>
#include <torch/csrc/jit/passes/common_subexpression_elimination.h>
#include <torch/csrc/jit/passes/utils/subgraph_utils.h>
#include <torch/csrc/jit/runtime/autodiff.h>

```
- EN: Pulls in the headers needed by the create autodiff subgraphs logic. Internal dependencies: `torch/csrc/jit/passes/create_autodiff_subgraphs.h`, `c10/util/Exception.h`, `torch/csrc/jit/ir/alias_analysis.h`, `torch/csrc/jit/ir/ir.h`, `torch/csrc/jit/jit_log.h`, `...`; external dependencies: none.
- CN: 为 create autodiff subgraphs 相关逻辑引入所需头文件。内部依赖：`torch/csrc/jit/passes/create_autodiff_subgraphs.h`, `c10/util/Exception.h`, `torch/csrc/jit/ir/alias_analysis.h`, `torch/csrc/jit/ir/ir.h`, `torch/csrc/jit/jit_log.h`, `...`；外部依赖：无。

### Lines 11-17
```cpp
namespace torch::jit {

namespace {

struct WorkBlock : public std::pair<Node*, Node*> {
  using pair::pair;

```
- EN: Opens or closes namespace scopes so the remaining declarations live in the expected JIT/frontend module.
- CN: 打开或关闭命名空间作用域，使后续声明归属于预期的 JIT/前端模块。

### Lines 18-24
```cpp
  Node* begin() {
    return this->first;
  }
  Node* end() {
    return this->second;
  }
};
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `begin`, `end`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`begin`, `end`。

### Lines 25-33
```cpp

class SubgraphSlicer {
 public:
  SubgraphSlicer(
      Block* block,
      std::shared_ptr<Graph> graph,
      size_t minSubgraphSize,
      AliasDb& aliasDb,
      std::vector<Node*>& diff_nodes)
```
- EN: Declares core types or data containers for this file. Prominent symbols: `SubgraphSlicer`.
- CN: 声明该文件使用的核心类型或数据容器。主要符号：`SubgraphSlicer`。

### Lines 34-40
```cpp
      : block_(block),
        graph_(std::move(graph)),
        minSubgraphSize_(minSubgraphSize),
        aliasDb_(aliasDb),
        diff_nodes_(diff_nodes) {}

  void run() {
```
- EN: This block manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `block_`, `graph_`, `move`, `minSubgraphSize_`, `aliasDb_`, `diff_nodes_`, `...`.
- CN: 该代码块操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`block_`, `graph_`, `move`, `minSubgraphSize_`, `aliasDb_`, `diff_nodes_`, `...`。

### Lines 41-53
```cpp
    // We maintain alias db correctness in-place while building up the autodiff
    // subgraphs, however it is difficult to preserve correctness when
    // un-inlining autodiff subgraphs. We first recursively construct all
    // subgraphs and then recursively cleanup & unmerge the small subgraphs
    buildupSubgraphs();
    GRAPH_DUMP("before unfuseAliasedOutputs", graph_);
    unfuseAliasedOutputs(block_);
    cleanupSubgraphs();
    // Run CSE globally onceto eliminate duplicates that may have occurred
    // while inlining subgraphs.
    EliminateCommonSubexpression(graph_);
  }

```
- EN: This block manipulates TorchScript IR objects; performs optimization-oriented rewriting; protects shared state or ordering assumptions. Key symbols: `buildupSubgraphs`, `unfuseAliasedOutputs`, `cleanupSubgraphs`, `EliminateCommonSubexpression`.
- CN: 该代码块操作 TorchScript IR 对象；执行面向优化的改写；保护共享状态或执行顺序假设。关键符号：`buildupSubgraphs`, `unfuseAliasedOutputs`, `cleanupSubgraphs`, `EliminateCommonSubexpression`。

### Lines 54-64
```cpp
  void cleanupSubgraphs() {
    auto curNode = *block_->nodes().rbegin();
    while (curNode != *block_->nodes().rend()) {
      // Save the previous node, since we might delete `curNode` in next block
      auto prevNode = curNode->prev();
      if (curNode->kind() == prim::DifferentiableGraph) {
        // Inlining nodes may cause some subexpression to come back in the
        // subgraphs (for example, copying constants in repeatedly will generate
        // redundant prim::Constants). Run CSE to clean them up.
        EliminateCommonSubexpression(curNode->g(attr::Subgraph));

```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects; performs optimization-oriented rewriting; protects shared state or ordering assumptions. Key symbols: `cleanupSubgraphs`, `nodes`, `rbegin`, `rend`, `prev`, `kind`, `...`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象；执行面向优化的改写；保护共享状态或执行顺序假设。关键符号：`cleanupSubgraphs`, `nodes`, `rbegin`, `rend`, `prev`, `kind`, `...`。

### Lines 65-71
```cpp
        if (!inlineIfTooSmall(curNode)) {
          diff_nodes_.push_back(curNode);
        }
      }
      curNode = prevNode;
    }

```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `inlineIfTooSmall`, `push_back`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`inlineIfTooSmall`, `push_back`。

### Lines 72-79
```cpp
    for (Node* n : block_->nodes()) {
      for (Block* b : n->blocks()) {
        SubgraphSlicer(b, graph_, minSubgraphSize_, aliasDb_, diff_nodes_)
            .cleanupSubgraphs();
      }
    }
  }

```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `nodes`, `blocks`, `SubgraphSlicer`, `cleanupSubgraphs`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`nodes`, `blocks`, `SubgraphSlicer`, `cleanupSubgraphs`。

### Lines 80-93
```cpp
  void buildupSubgraphs() {
    // We need to run the slicer multiple times in order to get all merge
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
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `buildupSubgraphs`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`buildupSubgraphs`。

### Lines 94-100
```cpp
    //   d = f(c)  <- this was node moved on the other side.

    // see [workblocks]
    auto workblocks = buildWorkBlocks();
    for (auto& workblock : workblocks) {
      bool any_changed = true;
      while (any_changed) {
```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `buildWorkBlocks`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`buildWorkBlocks`。

### Lines 101-110
```cpp
        any_changed = false;
        for (auto it = workblock.end()->reverseIterator();
             it != workblock.begin()->reverseIterator();) {
          auto [tmp_it, changed] = scanNode(*it);
          it = tmp_it;
          any_changed |= changed;
        }
      }
    }

```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `end`, `reverseIterator`, `begin`, `scanNode`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`end`, `reverseIterator`, `begin`, `scanNode`。

### Lines 111-120
```cpp
    // Construct Subgraphs Recursively
    for (Node* n : block_->nodes()) {
      for (auto subBlock : n->blocks()) {
        SubgraphSlicer(
            subBlock, graph_, minSubgraphSize_, aliasDb_, diff_nodes_)
            .buildupSubgraphs();
      }
    }
  }

```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `nodes`, `blocks`, `SubgraphSlicer`, `buildupSubgraphs`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`nodes`, `blocks`, `SubgraphSlicer`, `buildupSubgraphs`。

### Lines 121-129
```cpp
 private:
  void unfuseAliasedOutputs(Block* b) {
    bool any_changed = true;
    while (any_changed) {
      any_changed = false;
      // we walk in the reverse order, so we can skip
      // nodes that might get unfused after the current
      // prim::DifferentiableGraph
      for (auto n : b->nodes().reverse()) {
```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `unfuseAliasedOutputs`, `nodes`, `reverse`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`unfuseAliasedOutputs`, `nodes`, `reverse`。

### Lines 130-143
```cpp
        if (n->kind() == prim::DifferentiableGraph) {
          // aliased outputs in DifferentiableGraphs must be unfused
          // since autodiff doesn't know how to handle them correctly
          // N.B. Note, |= since we don't want `unfuseAliasedOutputs`
          // to short-circuit
          any_changed |= SubgraphUtils::unmergeAliasedOutputs(n);
          any_changed |= SubgraphUtils::unmergeOutputsAlisingInputs(n);
          GRAPH_DEBUG(
              "any_changed on ",
              any_changed,
              " ",
              n->g(attr::Subgraph)->toString(false));
        }
      }
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `kind`, `unmergeAliasedOutputs`, `unmergeOutputsAlisingInputs`, `g`, `toString`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`kind`, `unmergeAliasedOutputs`, `unmergeOutputsAlisingInputs`, `g`, `toString`。

### Lines 144-152
```cpp
    }

    for (Node* n : b->nodes()) {
      for (Block* ib : n->blocks()) {
        unfuseAliasedOutputs(ib);
      }
    }
  }

```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `nodes`, `blocks`, `unfuseAliasedOutputs`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`nodes`, `blocks`, `unfuseAliasedOutputs`。

### Lines 153-164
```cpp
  std::vector<WorkBlock> buildWorkBlocks() {
    // [workblocks]
    // the IR has many nodes which can never be reordered around, such as a
    // prim::Bailout. if a node N is surrounded by two nodes which cannot be
    // reordered, A and B, then a differentiable subgraph that is created from N
    // can only contain nodes from (A, B) The nodes from A to B represent one
    // work block for the subgraph slicer to work on. By creating these up
    // front, we avoid retraversing the whole graph block any time scanNode
    // returns, and we can also avoid attempting to create differentiable
    // subgraphs in work blocks that do not contain a # of differentiable nodes
    // >= minSubgraphSize_

```
- EN: This block manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `buildWorkBlocks`.
- CN: 该代码块操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`buildWorkBlocks`。

### Lines 165-171
```cpp
    Node* end_bound_node = block_->return_node();
    Node* curr = end_bound_node->prev();

    std::vector<WorkBlock> worklist;
    size_t differentiable_nodes = 0;

    while (curr != block_->param_node()) {
```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `return_node`, `prev`, `param_node`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`return_node`, `prev`, `param_node`。

### Lines 172-185
```cpp
      differentiable_nodes += shouldConsiderForMerge(curr);

      // cannot reorder around side effectful nodes
      if (curr->hasSideEffects()) {
        // not enough differentiable nodes to create a differentiable subgraph
        if (differentiable_nodes >= minSubgraphSize_) {
          worklist.emplace_back(curr, end_bound_node);
        }
        differentiable_nodes = 0;
        end_bound_node = curr;
      }
      curr = curr->prev();
    }

```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `shouldConsiderForMerge`, `hasSideEffects`, `emplace_back`, `prev`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`shouldConsiderForMerge`, `hasSideEffects`, `emplace_back`, `prev`。

### Lines 186-192
```cpp
    if (differentiable_nodes >= minSubgraphSize_) {
      worklist.emplace_back(curr, end_bound_node);
    }

    return worklist;
  }

```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `emplace_back`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`emplace_back`。

### Lines 193-199
```cpp
  // Inline this node's group subgraph into the outer graph if it's smaller
  // than the specified minimum size.
  //
  // Returns true if an inlining has occurred, false otherwise.
  bool inlineIfTooSmall(Node* n) {
    AT_ASSERT(n->kind() == prim::DifferentiableGraph);
    auto subgraph = SubgraphUtils::getSubgraph(n);
```
- EN: This block manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `inlineIfTooSmall`, `kind`, `getSubgraph`.
- CN: 该代码块操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`inlineIfTooSmall`, `kind`, `getSubgraph`。

### Lines 200-208
```cpp
    size_t i = 0;
    for (auto it = subgraph->nodes().begin(); it != subgraph->nodes().end();
         ++it) {
      i += !it->notExecutedOp();
      if (i >= minSubgraphSize_) {
        return false;
      }
    }

```
- EN: This block handles conditional branches; iterates over collections or graph structures; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `nodes`, `begin`, `end`, `notExecutedOp`.
- CN: 该代码块处理条件分支；遍历集合或图结构；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`nodes`, `begin`, `end`, `notExecutedOp`。

### Lines 209-215
```cpp
    SubgraphUtils::unmergeSubgraph(n);
    return true;
  }

  value_list sortReverseTopological(ArrayRef<Value*> inputs) {
    value_list result;
    for (auto i : inputs) {
```
- EN: This block iterates over collections or graph structures; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `unmergeSubgraph`, `sortReverseTopological`.
- CN: 该代码块遍历集合或图结构；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`unmergeSubgraph`, `sortReverseTopological`。

### Lines 216-222
```cpp
      if (i->node()->owningBlock() == block_) {
        result.push_back(i);
      }
    }
    // Sort in reverse topological order
    std::sort(result.begin(), result.end(), [&](Value* a, Value* b) {
      return a->node()->isAfter(b->node());
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `node`, `owningBlock`, `push_back`, `sort`, `begin`, `end`, `...`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`node`, `owningBlock`, `push_back`, `sort`, `begin`, `end`, `...`。

### Lines 223-236
```cpp
    });
    return result;
  }

  bool isViewOp(Node* n) {
    switch (n->kind()) {
      case aten::view:
      case aten::view_as:
      case aten::reshape:
      case aten::reshape_as:
      case aten::transpose:
      case aten::expand:
      case aten::expand_as:
        return true;
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `isViewOp`, `kind`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`isViewOp`, `kind`。

### Lines 237-243
```cpp
    }
    return false;
  }

  bool shouldConsiderForMerge(Node* node) {
    // if we're already in the process of merging
    if (node->kind() == prim::DifferentiableGraph) {
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `shouldConsiderForMerge`, `kind`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`shouldConsiderForMerge`, `kind`。

### Lines 244-252
```cpp
      return true;
    }
    if (node->kind() == prim::Constant) {
      return false;
    }

    // view ops as outputs of differentiable subgraphs can cause incorrect
    // differentiation for now, do not include them in the subgraph
    if (isViewOp(node)) {
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `kind`, `isViewOp`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`kind`, `isViewOp`。

### Lines 253-259
```cpp
      return false;
    }

    return isDifferentiable(node);
  }

  std::pair<graph_node_list::iterator, bool> scanNode(Node* consumer) {
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `isDifferentiable`, `scanNode`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`isDifferentiable`, `scanNode`。

### Lines 260-266
```cpp
    if (shouldConsiderForMerge(consumer)) {
      if (consumer->kind() != prim::DifferentiableGraph) {
        consumer = SubgraphUtils::createSingletonSubgraphAndUpdateAliasing(
            consumer, prim::DifferentiableGraph, aliasDb_);
      }
      auto inputs = sortReverseTopological(consumer->inputs());
      for (auto input : inputs) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `shouldConsiderForMerge`, `kind`, `createSingletonSubgraphAndUpdateAliasing`, `sortReverseTopological`, `inputs`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`shouldConsiderForMerge`, `kind`, `createSingletonSubgraphAndUpdateAliasing`, `sortReverseTopological`, `inputs`。

### Lines 267-274
```cpp
        if (auto group = tryMerge(consumer, input->node())) {
          // we successfully merged, so the new group's `inputs` may have
          // changed. So rescan the new group for more merging opportunities.
          return std::make_pair(group.value()->reverseIterator(), true);
        }
      }
    }

```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `tryMerge`, `node`, `make_pair`, `value`, `reverseIterator`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`tryMerge`, `node`, `make_pair`, `value`, `reverseIterator`。

### Lines 275-282
```cpp
    return std::make_pair(++consumer->reverseIterator(), false);
  }

  // Try to merge `producer` into `consumer`. If successful, this destroys
  // `producer` and returns the `consumer` group.
  std::optional<Node*> tryMerge(Node* consumer, Node* producer) {
    AT_ASSERT(consumer->kind() == prim::DifferentiableGraph);
    bool canMerge = shouldConsiderForMerge(producer) &&
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `make_pair`, `reverseIterator`, `tryMerge`, `kind`, `shouldConsiderForMerge`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`make_pair`, `reverseIterator`, `tryMerge`, `kind`, `shouldConsiderForMerge`。

### Lines 283-291
```cpp
        aliasDb_.moveBeforeTopologicallyValid(producer, consumer);

    if (!canMerge) {
      return std::nullopt;
    }

    SubgraphUtils::mergeNodeIntoSubgraphAndUpdateAliasing(
        producer, consumer, aliasDb_);
    return consumer;
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `moveBeforeTopologicallyValid`, `mergeNodeIntoSubgraphAndUpdateAliasing`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`moveBeforeTopologicallyValid`, `mergeNodeIntoSubgraphAndUpdateAliasing`。

### Lines 292-298
```cpp
  }

  Block* block_;
  std::shared_ptr<Graph> graph_;
  size_t minSubgraphSize_;
  AliasDb& aliasDb_;
  std::vector<Node*>& diff_nodes_;
```
- EN: This block manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: no dominant local symbol names.
- CN: 该代码块操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：无明显局部符号。

### Lines 299-307
```cpp
};

std::optional<bool> getProfileNodeRequiresGrad(Node* n) {
  TORCH_INTERNAL_ASSERT(n->kind() == prim::profile);
  if (!n->hasAttribute(attr::profiled_type)) {
    return std::nullopt;
  }
  auto& type = n->ty(attr::profiled_type);
  if (type->castRaw<TensorType>() == nullptr) {
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `getProfileNodeRequiresGrad`, `kind`, `hasAttribute`, `ty`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`getProfileNodeRequiresGrad`, `kind`, `hasAttribute`, `ty`。

### Lines 308-314
```cpp
    return std::nullopt;
  }
  return type->expectRef<TensorType>().requiresGrad();
}

struct ContextMapping {
  std::vector<const Node*> ctx_stack_;
```
- EN: Declares core types or data containers for this file. Prominent symbols: `requiresGrad`, `ContextMapping`.
- CN: 声明该文件使用的核心类型或数据容器。主要符号：`requiresGrad`, `ContextMapping`。

### Lines 315-326
```cpp
  std::unordered_map<const Node*, const Node*> node_to_ctx_;

  void processNode(Node* n) {
    node_to_ctx_[n] = ctx_stack_.back();

    if (n->kind() == prim::Enter) {
      ctx_stack_.push_back(n);
    } else if (n->kind() == prim::Exit) {
      ctx_stack_.pop_back();
    }
  }

```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `processNode`, `back`, `kind`, `push_back`, `pop_back`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`processNode`, `back`, `kind`, `push_back`, `pop_back`。

### Lines 327-333
```cpp
  void processBlock(Block* block) {
    for (Node* n : block->nodes()) {
      processNode(n);
      for (Block* b : n->blocks()) {
        processBlock(b);
      }
      if (n->kind() == prim::DifferentiableGraph) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `processBlock`, `nodes`, `processNode`, `blocks`, `kind`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`processBlock`, `nodes`, `processNode`, `blocks`, `kind`。

### Lines 334-344
```cpp
        const auto& subgraph = n->g(attr::Subgraph);
        processBlock(subgraph->block());
      }
    }
  }

  ContextMapping(const std::shared_ptr<Graph>& graph) {
    ctx_stack_.push_back(nullptr);
    processBlock(graph->block());
  }

```
- EN: This block manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `g`, `processBlock`, `block`, `ContextMapping`, `push_back`.
- CN: 该代码块操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`g`, `processBlock`, `block`, `ContextMapping`, `push_back`。

### Lines 345-352
```cpp
  const Node* get(const Node* n) const {
    auto it = node_to_ctx_.find(n);
    TORCH_INTERNAL_ASSERT(
        it != node_to_ctx_.end(),
        "Cannot find node in node-to-context mapping.");
    return it->second;
  }

```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `get`, `find`, `end`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`get`, `find`, `end`。

### Lines 353-359
```cpp
  bool has(const Node* n) const {
    return node_to_ctx_.find(n) != node_to_ctx_.end();
  }
};

std::optional<bool> findRequiresGradForOutput(
    Node* diff_graph,
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `has`, `find`, `end`, `findRequiresGradForOutput`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`has`, `find`, `end`, `findRequiresGradForOutput`。

### Lines 360-367
```cpp
    Value* output,
    const ContextMapping& ctx_mapping) {
  for (auto& use : output->uses()) {
    // [Only consider profiles in the same context]
    // Ignore profiled uses if the use is within a different context.
    // For example, a profile node within a no_grad() context will record the
    // wrong requires_grad information.
    if (ctx_mapping.has(use.user) &&
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `uses`, `has`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`uses`, `has`。

### Lines 368-374
```cpp
        ctx_mapping.get(use.user) != ctx_mapping.get(diff_graph)) {
      continue;
    }

    if (use.user->kind() == prim::profile) {
      auto req_grad_use = getProfileNodeRequiresGrad(use.user);
      if (req_grad_use.has_value()) {
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `get`, `kind`, `getProfileNodeRequiresGrad`, `has_value`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`get`, `kind`, `getProfileNodeRequiresGrad`, `has_value`。

### Lines 375-383
```cpp
        return req_grad_use;
      }
    }

    // maybe the profile node got absorbed into a differentiable graph
    if (use.user->kind() == prim::DifferentiableGraph) {
      const auto& dg = use.user->g(attr::Subgraph);
      // check all the uses of this graph input to look for profile nodes.
      Value* dg_value = dg->inputs()[use.offset];
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `kind`, `g`, `inputs`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`kind`, `g`, `inputs`。

### Lines 384-390
```cpp
      for (auto& dg_use : dg_value->uses()) {
        // See [Only consider profiles in the same context]
        if (ctx_mapping.has(dg_use.user) &&
            ctx_mapping.get(dg_use.user) != ctx_mapping.get(diff_graph)) {
          continue;
        }

```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `uses`, `has`, `get`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`uses`, `has`, `get`。

### Lines 391-400
```cpp
        if (dg_use.user->kind() == prim::profile) {
          auto req_grad_use = getProfileNodeRequiresGrad(dg_use.user);
          if (req_grad_use.has_value()) {
            return req_grad_use;
          }
        }
      }
    }
  }

```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `kind`, `getProfileNodeRequiresGrad`, `has_value`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`kind`, `getProfileNodeRequiresGrad`, `has_value`。

### Lines 401-409
```cpp
  return std::nullopt;
}

void AddRequiresGradToDifferentiableGraph(
    Node* diff_graph,
    const ContextMapping& ctx_mapping) {
  TORCH_INTERNAL_ASSERT(diff_graph->kind() == prim::DifferentiableGraph);
  const auto& subgraph = diff_graph->g(attr::Subgraph);
  for (auto i : c10::irange(subgraph->outputs().size())) {
```
- EN: This block iterates over collections or graph structures; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `AddRequiresGradToDifferentiableGraph`, `kind`, `g`, `irange`, `outputs`, `size`.
- CN: 该代码块遍历集合或图结构；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`AddRequiresGradToDifferentiableGraph`, `kind`, `g`, `irange`, `outputs`, `size`。

### Lines 410-419
```cpp
    Value* output = subgraph->outputs()[i];
    if (output->node()->kind() == prim::profile) {
      // already have requires_grad info from this profile node
      continue;
    }
    if (output->type()->castRaw<TensorType>() == nullptr) {
      // non-tensors don't get profiled.
      continue;
    }
    if (output->type()->expectRef<TensorType>().requiresGrad().has_value()) {
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `outputs`, `node`, `kind`, `type`, `requiresGrad`, `has_value`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`outputs`, `node`, `kind`, `type`, `requiresGrad`, `has_value`。

### Lines 420-427
```cpp
      continue;
    }

    // this node doesn't have any requires_grad info.
    // look at its uses to try to find a profile node.
    auto requires_grad = findRequiresGradForOutput(
        diff_graph, diff_graph->output(i), ctx_mapping);

```
- EN: This block manipulates TorchScript IR objects. Key symbols: `findRequiresGradForOutput`, `output`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`findRequiresGradForOutput`, `output`。

### Lines 428-436
```cpp
    output->setType(output->type()->expectRef<TensorType>().withRequiresGrad(
        requires_grad));
  }
}

void AddRequiresGradOnOutputNodes(
    Block* block,
    const ContextMapping& ctx_mapping) {
  for (Node* n : block->nodes()) {
```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `setType`, `type`, `withRequiresGrad`, `AddRequiresGradOnOutputNodes`, `nodes`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`setType`, `type`, `withRequiresGrad`, `AddRequiresGradOnOutputNodes`, `nodes`。

### Lines 437-445
```cpp
    if (n->kind() == prim::DifferentiableGraph) {
      AddRequiresGradToDifferentiableGraph(n, ctx_mapping);
    }
    for (Block* b : n->blocks()) {
      AddRequiresGradOnOutputNodes(b, ctx_mapping);
    }
  }
}

```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `kind`, `AddRequiresGradToDifferentiableGraph`, `blocks`, `AddRequiresGradOnOutputNodes`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`kind`, `AddRequiresGradToDifferentiableGraph`, `blocks`, `AddRequiresGradOnOutputNodes`。

### Lines 446-454
```cpp
// autodiff.cpp needs to know, for each output, whether or not it requires
// grad. Sometimes a profile node will be present on the output, but sometimes
// it won't be present. This might happen if there's a node with side effects
// in between the definition of the output node and the profile node; in this
// case the profile node and output node would be in different workblocks and
// couldn't be merged into the same DifferentiableGraph. (see [workblocks])
// Or it could happen if the output is profiled twice and the profile nodes get
// removed by unfusedAliasedOutputs.
void AddRequiresGradOnOutputNodes(const std::shared_ptr<Graph>& graph) {
```
- EN: This block manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `AddRequiresGradOnOutputNodes`.
- CN: 该代码块操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`AddRequiresGradOnOutputNodes`。

### Lines 455-463
```cpp
  ContextMapping ctx_mapping(graph);
  AddRequiresGradOnOutputNodes(graph->block(), ctx_mapping);
}
} // anonymous namespace

std::vector<Node*> CreateAutodiffSubgraphs(
    const std::shared_ptr<Graph>& graph,
    size_t threshold) {
  std::vector<Node*> diff_nodes;
```
- EN: This block manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `ctx_mapping`, `AddRequiresGradOnOutputNodes`, `block`, `CreateAutodiffSubgraphs`.
- CN: 该代码块操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`ctx_mapping`, `AddRequiresGradOnOutputNodes`, `block`, `CreateAutodiffSubgraphs`。

### Lines 464-470
```cpp
  AliasDb db(graph);
  GRAPH_DEBUG("Before creating autodiff subgraphs", *graph);
  SubgraphSlicer(graph->block(), graph, threshold, db, diff_nodes).run();
  GRAPH_DEBUG("After creating autodiff subgraphs", *graph);
  AddRequiresGradOnOutputNodes(graph);
  GRAPH_DEBUG("diff_nodes.size() ", diff_nodes.size());
  return diff_nodes;
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `db`, `SubgraphSlicer`, `block`, `run`, `AddRequiresGradOnOutputNodes`, `size`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`db`, `SubgraphSlicer`, `block`, `run`, `AddRequiresGradOnOutputNodes`, `size`。

### Lines 471-472
```cpp
}
} // namespace torch::jit
```
- EN: Opens or closes namespace scopes so the remaining declarations live in the expected JIT/frontend module.
- CN: 打开或关闭命名空间作用域，使后续声明归属于预期的 JIT/前端模块。


## Key Concepts / 关键概念
- EN: IR graph manipulation — the file inspects or rewrites TorchScript graphs and nodes.
  CN: IR 图操作——该文件会检查或改写 TorchScript 的图、节点与块。
- EN: Alias and mutation reasoning — the code tracks side effects and write/read interactions.
  CN: 别名与变更分析——代码会跟踪副作用以及读写关系。
- EN: Optimization pass — the code simplifies graphs by removing redundant work or folding values.
  CN: 优化 pass——代码通过删除冗余计算或常量折叠来简化图。
- EN: Inlining — the implementation merges callees or blocks into their call sites.
  CN: 内联——实现会把被调用者或代码块并入调用位置。
- EN: Fusion support — the file prepares groups of operations for fused execution.
  CN: 融合支持——该文件会为融合执行准备算子分组。
- EN: Type and shape reasoning — the code propagates static information across the IR.
  CN: 类型与形状推理——代码会在 IR 中传播静态信息。

## Dependencies / 依赖关系
- Internal includes / 内部头文件: `torch/csrc/jit/passes/create_autodiff_subgraphs.h`, `c10/util/Exception.h`, `torch/csrc/jit/ir/alias_analysis.h`, `torch/csrc/jit/ir/ir.h`, `torch/csrc/jit/jit_log.h`, `torch/csrc/jit/passes/common_subexpression_elimination.h`, `torch/csrc/jit/passes/utils/subgraph_utils.h`, `torch/csrc/jit/runtime/autodiff.h`
- External includes / 外部头文件: 无
- Namespaces / 命名空间: `torch::jit`
- Representative symbols / 代表性符号: `WorkBlock`, `pair`, `begin`, `end`, `SubgraphSlicer`, `block_`, `graph_`, `move`, `minSubgraphSize_`, `aliasDb_`, `...`
