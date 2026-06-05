# subgraph_utils.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `torch/csrc/jit/passes/utils/subgraph_utils.cpp`
- Repository: `pytorch`
- Purpose (EN): Implements the TorchScript JIT pass logic for subgraph utils, including graph analysis and rewrites.
- 用途 (CN): 实现与 subgraph utils 相关的 TorchScript JIT pass 逻辑，包括图分析与改写。

## Line-by-Line Analysis / 逐行分析
### Lines 1-10
```cpp
#include <torch/csrc/jit/passes/utils/subgraph_utils.h>

#include <torch/csrc/jit/passes/canonicalize.h>

#include <ATen/core/symbol.h>
#include <c10/util/irange.h>
#include <torch/csrc/jit/jit_log.h>

#include <utility>

```
- EN: Pulls in the headers needed by the subgraph utils logic. Internal dependencies: `torch/csrc/jit/passes/utils/subgraph_utils.h`, `torch/csrc/jit/passes/canonicalize.h`, `ATen/core/symbol.h`, `c10/util/irange.h`, `torch/csrc/jit/jit_log.h`; external dependencies: `utility`.
- CN: 为 subgraph utils 相关逻辑引入所需头文件。内部依赖：`torch/csrc/jit/passes/utils/subgraph_utils.h`, `torch/csrc/jit/passes/canonicalize.h`, `ATen/core/symbol.h`, `c10/util/irange.h`, `torch/csrc/jit/jit_log.h`；外部依赖：`utility`。

### Lines 11-20
```cpp
namespace torch::jit::SubgraphUtils {
namespace {

bool hasSubgraph(Node* n) {
  return n->hasAttribute(attr::Subgraph);
}

std::vector<std::optional<const Use>> gatherLastUses(
    at::ArrayRef<Value*> values) {
  return fmap(values, [&](Value* v) -> std::optional<const Use> {
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `hasSubgraph`, `hasAttribute`, `gatherLastUses`, `fmap`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`hasSubgraph`, `hasAttribute`, `gatherLastUses`, `fmap`。

### Lines 21-32
```cpp
    return firstOrLastUse(v, /*find_first*/ false);
  });
}

// When merging a node into a subgraph, we wish to preserve all of the
// aliasing properties of the node's outputs. It is difficult to track
// the node or its contained nodes through all of the ir manipulation
// involved in merging; it is pretty easy to uniquely identify the value
// based on its uses. We can identify the value by its last use in the graph.
// Values which do not have uses or which do not have a last use
// outside of the subgraph to be merged into we do not need to track.
struct ValueMapper {
```
- EN: Declares core types or data containers for this file. Prominent symbols: `firstOrLastUse`, `ValueMapper`.
- CN: 声明该文件使用的核心类型或数据容器。主要符号：`firstOrLastUse`, `ValueMapper`。

### Lines 33-44
```cpp
  // `to_merge` is the node we're merginginto a subgraph, `existing_subgraph` is
  // the subgraph node that we're merging into if it exists
  ValueMapper(
      Node* to_merge,
      AliasDb& db,
      std::optional<Node*> existing_subgraph) {
    last_uses_ = gatherLastUses(to_merge->outputs());
    if (existing_subgraph) {
      existing_last_uses_ = gatherLastUses((*existing_subgraph)->outputs());
    }
    WithInsertPoint guard(to_merge);
    auto g = to_merge->owningGraph();
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `ValueMapper`, `gatherLastUses`, `outputs`, `guard`, `owningGraph`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`ValueMapper`, `gatherLastUses`, `outputs`, `guard`, `owningGraph`。

### Lines 45-55
```cpp
    // temporary node to put the aliasing properties of the node before its
    // merged and destroyed
    placeholder_node_ = g->insertNode(g->create(prim::Uninitialized, 0));
    for (size_t i = 0; i < to_merge->outputs().size(); ++i) {
      Value* existing = to_merge->outputs().at(i);
      Value* new_value = placeholder_node_->insertOutput(i)->copyMetadata(
          to_merge->outputs().at(i));
      db.replaceWithNewValue(existing, new_value);
    }
  }

```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `insertNode`, `create`, `outputs`, `size`, `insertOutput`, `copyMetadata`, `...`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`insertNode`, `create`, `outputs`, `size`, `insertOutput`, `copyMetadata`, `...`。

### Lines 56-67
```cpp
  bool usesEqual(const Use& a, const Use& b) {
    return a.user == b.user && a.offset == b.offset;
  }

  void copyAliasing(Node* merged_node, AliasDb& db) {
    auto new_outputs = merged_node->outputs();
    for (Value* v : new_outputs) {
      auto maybe_last_use = firstOrLastUse(v, /*find_first*/ false);
      // if it doesn't have a use it shouldn't have been added as output
      TORCH_INTERNAL_ASSERT(maybe_last_use);
      const Use last_use = *maybe_last_use;

```
- EN: This block iterates over collections or graph structures; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `usesEqual`, `copyAliasing`, `outputs`, `firstOrLastUse`.
- CN: 该代码块遍历集合或图结构；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`usesEqual`, `copyAliasing`, `outputs`, `firstOrLastUse`。

### Lines 68-76
```cpp
      // existing outputs of the subgraph do not need to have alias db mappings
      // updated
      bool is_existing_value = false;
      for (size_t i = 0; i < existing_last_uses_.size() && !is_existing_value;
           ++i) {
        is_existing_value = existing_last_uses_[i].has_value() &&
            usesEqual(*existing_last_uses_[i], last_use);
      }
      if (is_existing_value) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `size`, `has_value`, `usesEqual`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`size`, `has_value`, `usesEqual`。

### Lines 77-90
```cpp
        continue;
      }

      size_t i = 0;
      while (i < last_uses_.size() && last_uses_.at(i).has_value() &&
             !usesEqual(*last_uses_.at(i), last_use)) {
        ++i;
      }
      TORCH_INTERNAL_ASSERT(i != last_uses_.size());
      db.replaceWithNewValue(placeholder_node_->outputs().at(i), v);
    }
    placeholder_node_->destroy();
  }

```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `size`, `has_value`, `usesEqual`, `replaceWithNewValue`, `outputs`, `destroy`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`size`, `has_value`, `usesEqual`, `replaceWithNewValue`, `outputs`, `destroy`。

### Lines 91-107
```cpp
  std::vector<std::optional<const Use>> last_uses_;
  std::vector<std::optional<const Use>> existing_last_uses_;
  Node* placeholder_node_;
};

Node* executeSubgraphMergeAndUpdateAliasing(
    Node* to_merge,
    std::optional<Node*> existing,
    AliasDb& db,
    const std::function<Node*(void)>& merge_fn) {
  // When we merge a node into a subgraph, the new subgraph outputs
  // have the same aliasing properties as the original node's outputs.
  // Here we create a placeholder node, transfer the aliasing properties
  // to the placeholder, execute the merge, and transfer the aliasing
  // properties to the appropriate fusion group outputs
  ValueMapper vm(to_merge, db, existing);
  Node* fusion_group = merge_fn();
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `executeSubgraphMergeAndUpdateAliasing`, `vm`, `merge_fn`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`executeSubgraphMergeAndUpdateAliasing`, `vm`, `merge_fn`。

### Lines 108-116
```cpp
  vm.copyAliasing(fusion_group, db);
  return fusion_group;
}

// Combine the nodes in two subgraph together. The nodes will end up in
// `mergeTo`, and `mergeFrom` is destroyed.
void mergeSubgraph(Node* mergeTo, Node* mergeFrom) {
  bool merge_from_is_after = mergeFrom->isAfter(mergeTo);
  Node* nodeBeforeMergeFrom = mergeFrom->prev();
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `copyAliasing`, `mergeSubgraph`, `isAfter`, `prev`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`copyAliasing`, `mergeSubgraph`, `isAfter`, `prev`。

### Lines 117-132
```cpp
  Node* nodeAfterMergeFrom = mergeFrom->next();

  unmergeSubgraph(mergeFrom);

  graph_node_list_iterator end_it;
  graph_node_list_iterator it;

  if (merge_from_is_after) {
    it = nodeBeforeMergeFrom->iterator();
    end_it = nodeAfterMergeFrom->iterator();
  } else {
    end_it = nodeBeforeMergeFrom->reverseIterator();
    it = nodeAfterMergeFrom->reverseIterator();
  }
  ++it;

```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `next`, `unmergeSubgraph`, `iterator`, `reverseIterator`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`next`, `unmergeSubgraph`, `iterator`, `reverseIterator`。

### Lines 133-141
```cpp
  while (it != end_it) {
    Node* node = *it;
    ++it;
    mergeNodeIntoSubgraph(node, mergeTo);
  }
}

struct topo_cmp_value {
  bool operator()(Value* a, Value* b) const {
```
- EN: Declares core types or data containers for this file. Prominent symbols: `mergeNodeIntoSubgraph`, `topo_cmp_value`.
- CN: 声明该文件使用的核心类型或数据容器。主要符号：`mergeNodeIntoSubgraph`, `topo_cmp_value`。

### Lines 142-150
```cpp
    if (a->node() == b->node()) {
      return a->unique() < b->unique();
    }
    return a->node()->isBefore(b->node());
  }
};

struct topo_cmp_node {
  bool operator()(Node* a, Node* b) const {
```
- EN: Declares core types or data containers for this file. Prominent symbols: `node`, `unique`, `isBefore`, `topo_cmp_node`.
- CN: 声明该文件使用的核心类型或数据容器。主要符号：`node`, `unique`, `isBefore`, `topo_cmp_node`。

### Lines 151-160
```cpp
    return a->isBefore(b);
  }
};

void collectNodesToUnfuse(Node* start, std::set<Node*, topo_cmp_node>& s) {
  if (start->kind() == prim::Return || start->kind() == prim::Param) {
    GRAPH_DEBUG("reached the param or return node", getHeader(start));
    return;
  }

```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `isBefore`, `collectNodesToUnfuse`, `kind`, `getHeader`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`isBefore`, `collectNodesToUnfuse`, `kind`, `getHeader`。

### Lines 161-169
```cpp
  if (s.count(start) != 0) {
    // already visited, no need to visit descendants
    return;
  }

  GRAPH_DEBUG("collectNodesToUnfuse: inserting node ", getHeader(start));
  s.insert(start);

  for (auto o : start->outputs()) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `count`, `getHeader`, `insert`, `outputs`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`count`, `getHeader`, `insert`, `outputs`。

### Lines 170-178
```cpp
    for (auto use : o->uses()) {
      collectNodesToUnfuse(use.user, s);
    }
  }
}

std::vector<std::set<Value*, topo_cmp_value>> buildAliasedSets(
    std::shared_ptr<Graph> subgraph) {
  auto outputs = subgraph->outputs();
```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `uses`, `collectNodesToUnfuse`, `buildAliasedSets`, `outputs`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`uses`, `collectNodesToUnfuse`, `buildAliasedSets`, `outputs`。

### Lines 179-194
```cpp
  AliasDb alias_db(std::move(subgraph));
  TORCH_INTERNAL_ASSERT(outputs.size() > 1);
  std::vector<std::set<Value*, topo_cmp_value>> res;
  for (auto o : outputs) {
    auto grouped = false;
    for (auto& s : res) {
      auto os = *s.begin();
      auto aliased = alias_db.mayContainAlias(os, o);
      GRAPH_DEBUG(
          "comparing %",
          o->debugName(),
          " with %",
          os->debugName(),
          " result ",
          aliased);
      if (aliased) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `alias_db`, `move`, `size`, `begin`, `mayContainAlias`, `debugName`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`alias_db`, `move`, `size`, `begin`, `mayContainAlias`, `debugName`。

### Lines 195-204
```cpp
        s.insert(o);
        GRAPH_DEBUG("Grouping %", o->debugName(), " with %", os->debugName());
        grouped = true;
      }
    }
    if (!grouped) {
      res.push_back({o});
    }
  }
  return res;
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `insert`, `debugName`, `push_back`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`insert`, `debugName`, `push_back`。

### Lines 205-213
```cpp
}

} // namespace

std::shared_ptr<Graph> getSubgraph(Node* n) {
  return n->g(attr::Subgraph);
}

void unmergeSubgraph(Node* subgraphNode) {
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `getSubgraph`, `g`, `unmergeSubgraph`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`getSubgraph`, `g`, `unmergeSubgraph`。

### Lines 214-225
```cpp
  // Inline the graph, replace uses of node outputs and destroy the node
  auto outerGraph = subgraphNode->owningGraph();
  WithInsertPoint guard(subgraphNode);
  const auto subgraphOutputs = insertGraph(
      *outerGraph, *getSubgraph(subgraphNode), subgraphNode->inputs());
  AT_ASSERT(subgraphOutputs.size() >= subgraphNode->outputs().size());
  for (size_t i = 0; i < subgraphNode->outputs().size(); ++i) {
    subgraphNode->outputs()[i]->replaceAllUsesWith(subgraphOutputs[i]);
  }
  subgraphNode->destroy();
}

```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects; performs optimization-oriented rewriting; protects shared state or ordering assumptions. Key symbols: `owningGraph`, `guard`, `insertGraph`, `getSubgraph`, `inputs`, `size`, `...`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象；执行面向优化的改写；保护共享状态或执行顺序假设。关键符号：`owningGraph`, `guard`, `insertGraph`, `getSubgraph`, `inputs`, `size`, `...`。

### Lines 226-236
```cpp
static void collectNestedUses(
    std::unordered_set<Value*>& closed_over_values,
    std::unordered_set<Value*>& new_values,
    std::unordered_map<Value*, Value*>& externalValuesMap,
    Node* input_node) {
  for (auto input : input_node->inputs()) {
    if (externalValuesMap.count(input) == 0 && new_values.count(input) == 0) {
      closed_over_values.insert(input);
    }
  }
  if (input_node->kind() == prim::If) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `collectNestedUses`, `inputs`, `count`, `insert`, `kind`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`collectNestedUses`, `inputs`, `count`, `insert`, `kind`。

### Lines 237-249
```cpp
    for (Block* block : input_node->blocks()) {
      for (Node* node : block->nodes()) {
        collectNestedUses(
            closed_over_values, new_values, externalValuesMap, node);
      }
      for (Value* v : block->outputs()) {
        if (externalValuesMap.count(v) == 0 && new_values.count(v) == 0) {
          closed_over_values.insert(v);
        }
      }
    }
  } else if (input_node->kind() == prim::Loop) {
    for (Value* v : input_node->inputs()) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `blocks`, `nodes`, `collectNestedUses`, `outputs`, `count`, `insert`, `...`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`blocks`, `nodes`, `collectNestedUses`, `outputs`, `count`, `insert`, `...`。

### Lines 250-258
```cpp
      if (externalValuesMap.count(v) == 0 && new_values.count(v) == 0) {
        closed_over_values.insert(v);
      }
    }
    Block* block = input_node->blocks().at(0);
    for (Value* v : block->inputs()) {
      new_values.insert(v);
    }
    for (Node* node : block->nodes()) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `count`, `insert`, `blocks`, `inputs`, `nodes`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`count`, `insert`, `blocks`, `inputs`, `nodes`。

### Lines 259-269
```cpp
      collectNestedUses(
          closed_over_values, new_values, externalValuesMap, node);
    }
  } else if (!input_node->blocks().empty()) {
    TORCH_INTERNAL_ASSERT(false, input_node, " kind not handled yet");
  }
  for (Value* output : input_node->outputs()) {
    new_values.insert(output);
  }
}

```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `collectNestedUses`, `blocks`, `empty`, `outputs`, `insert`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`collectNestedUses`, `blocks`, `empty`, `outputs`, `insert`。

### Lines 270-278
```cpp
static std::unordered_set<Value*> closedOverValues(
    Node* toMerge,
    std::unordered_map<Value*, Value*>& externalValuesMap) {
  std::unordered_set<Value*> closed_over_values;
  std::unordered_set<Value*> new_values;
  collectNestedUses(closed_over_values, new_values, externalValuesMap, toMerge);
  return closed_over_values;
}

```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `closedOverValues`, `collectNestedUses`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`closedOverValues`, `collectNestedUses`。

### Lines 279-287
```cpp
void mergeNodeIntoSubgraph(
    Node* toMerge,
    Node* subgraphNode,
    bool destroyNode) {
  AT_ASSERT(hasSubgraph(subgraphNode) && toMerge != subgraphNode);
  if (hasSubgraph(toMerge)) {
    return mergeSubgraph(subgraphNode, toMerge);
  }

```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `mergeNodeIntoSubgraph`, `hasSubgraph`, `mergeSubgraph`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`mergeNodeIntoSubgraph`, `hasSubgraph`, `mergeSubgraph`。

### Lines 288-299
```cpp
  auto subgraph = getSubgraph(subgraphNode);

  // Map from values in the surrounding graph to inputs/outputs in the subgraph
  std::unordered_map<Value*, Value*> externalValuesMap;

  AT_ASSERT(subgraphNode->inputs().size() == subgraph->inputs().size());
  size_t idx = 0;
  for (auto input : subgraphNode->inputs()) {
    externalValuesMap[input] = subgraph->inputs()[idx];
    idx++;
  }

```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `getSubgraph`, `inputs`, `size`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`getSubgraph`, `inputs`, `size`。

### Lines 300-308
```cpp
  for (size_t i = 0; i < subgraphNode->outputs().size(); ++i) {
    externalValuesMap[subgraphNode->outputs().at(i)] =
        subgraph->outputs().at(i);
  }

  // Add n's inputs to the group's input list if we don't already have them

  bool merging_node_after_subgraph = toMerge->isAfter(subgraphNode);
  Node* guard_node = merging_node_after_subgraph ? *subgraph->nodes().end()
```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `outputs`, `size`, `isAfter`, `nodes`, `end`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`outputs`, `size`, `isAfter`, `nodes`, `end`。

### Lines 309-317
```cpp
                                                 : *subgraph->nodes().begin();
  WithInsertPoint guard(guard_node);

  std::unordered_set<Value*> closedValues =
      closedOverValues(toMerge, externalValuesMap);

  // There are currently downstream usage that relies on a fixed ordering
  // of graph inputs. TODO: remove
  std::vector<Value*> orderedClosedValues;
```
- EN: This block manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `nodes`, `begin`, `guard`, `closedOverValues`.
- CN: 该代码块操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`nodes`, `begin`, `guard`, `closedOverValues`。

### Lines 318-329
```cpp
  std::unordered_set<Value*> orderedSeenValues;
  for (Value* input : toMerge->inputs()) {
    orderedClosedValues.push_back(input);
    orderedSeenValues.insert(input);
  }
  for (Value* closedValue : closedValues) {
    if (!orderedSeenValues.count(closedValue)) {
      orderedClosedValues.push_back(closedValue);
      orderedSeenValues.insert(closedValue);
    }
  }

```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `inputs`, `push_back`, `insert`, `count`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`inputs`, `push_back`, `insert`, `count`。

### Lines 330-342
```cpp
  for (auto input : orderedClosedValues) {
    if (externalValuesMap.count(input) == 0) {
      // Clone constants inside the subgraph instead of referencing them, to
      // enable more optimizations
      if (auto value = toIValue(input)) {
        auto nv = subgraph->insertConstant(*value);
        nv->copyMetadata(input);
        externalValuesMap[input] = nv;
      } else {
        // The common case: this is a regular input, so just register it with
        // the group node and inner subgraph
        subgraphNode->addInput(input);
        auto inputToGraph = subgraph->addInput();
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `count`, `toIValue`, `insertConstant`, `copyMetadata`, `addInput`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`count`, `toIValue`, `insertConstant`, `copyMetadata`, `addInput`。

### Lines 343-352
```cpp
        inputToGraph->copyMetadata(input);
        externalValuesMap[input] = inputToGraph;
      }
    }
  }

  // Merge the node into the graph
  auto mergedNode = subgraph->insertNode(subgraph->createClone(
      toMerge, [&](Value* v) { return externalValuesMap[v]; }));

```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `copyMetadata`, `insertNode`, `createClone`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`copyMetadata`, `insertNode`, `createClone`。

### Lines 353-361
```cpp
  if (!merging_node_after_subgraph) {
    // If n's outputs were inputs to `group`, remove them since we just merged
    // n in.
    //
    // i.e.,
    // x = f(w); group(x, y, z) becomes group(w, y, z).
    // x, y, z = f(w); group(x, y, z) becomes group(w).
    auto inputs = subgraphNode->inputs();
    for (size_t i = 0; i < toMerge->outputs().size(); ++i) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `inputs`, `outputs`, `size`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`inputs`, `outputs`, `size`。

### Lines 362-371
```cpp
      auto it = std::find(inputs.begin(), inputs.end(), toMerge->outputs()[i]);
      if (it != inputs.end()) {
        size_t p = it - inputs.begin();
        subgraphNode->removeInput(p);
        subgraph->inputs()[p]->replaceAllUsesWith(mergedNode->outputs()[i]);
        subgraph->eraseInput(p);
      }
    }
  }

```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `find`, `begin`, `end`, `outputs`, `removeInput`, `inputs`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`find`, `begin`, `end`, `outputs`, `removeInput`, `inputs`, `...`。

### Lines 372-382
```cpp
  // Add n's outputs to the group node and inner subgraph outputs.
  for (const auto i : c10::irange(toMerge->outputs().size())) {
    auto oldOutput = toMerge->outputs()[i];
    auto newOutput = mergedNode->outputs()[i];
    subgraph->registerOutput(newOutput);
    auto groupOutput = subgraphNode->addOutput();
    groupOutput->copyMetadata(oldOutput);
    oldOutput->replaceAllUsesWith(groupOutput);
  }
  // Remove the original node now that the merge is complete
  if (destroyNode) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `irange`, `outputs`, `size`, `registerOutput`, `addOutput`, `copyMetadata`, `...`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`irange`, `outputs`, `size`, `registerOutput`, `addOutput`, `copyMetadata`, `...`。

### Lines 383-392
```cpp
    toMerge->destroy();
  }

  // We wait till destroying `toMerge` before pruning subgraph outputs,
  // since destroying `toMerge` could cause a subgraph output to no longer
  // have any uses
  const auto hasUsesOutsideSubgraph = [&](Value* v) {
    return std::any_of(
        v->uses().cbegin(), v->uses().cend(), [&](const Use& use) {
          return use.user->isAfter(subgraphNode);
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `destroy`, `any_of`, `uses`, `cbegin`, `cend`, `isAfter`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`destroy`, `any_of`, `uses`, `cbegin`, `cend`, `isAfter`。

### Lines 393-403
```cpp
        });
  };

  for (int64_t i = subgraphNode->outputs().size() - 1; i >= 0; i--) {
    if (!hasUsesOutsideSubgraph(subgraphNode->outputs().at(i))) {
      subgraphNode->eraseOutput(i);
      subgraph->eraseOutput(i);
    }
  }
}

```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `outputs`, `size`, `hasUsesOutsideSubgraph`, `eraseOutput`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`outputs`, `size`, `hasUsesOutsideSubgraph`, `eraseOutput`。

### Lines 404-412
```cpp
Node* createSingletonSubgraph(Node* n, Symbol subgraphKind) {
  auto graph = n->owningGraph();
  auto subgraph = graph->create(subgraphKind, 0);
  subgraph->g_(attr::Subgraph, std::make_shared<Graph>(graph->current_scope()));
  subgraph->insertBefore(n);
  mergeNodeIntoSubgraph(n, subgraph);
  return subgraph;
}

```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `createSingletonSubgraph`, `owningGraph`, `create`, `g_`, `current_scope`, `insertBefore`, `...`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`createSingletonSubgraph`, `owningGraph`, `create`, `g_`, `current_scope`, `insertBefore`, `...`。

### Lines 413-422
```cpp
void mergeNodeIntoSubgraphAndUpdateAliasing(
    Node* to_merge,
    Node* subgraphNode,
    AliasDb& db) {
  executeSubgraphMergeAndUpdateAliasing(to_merge, subgraphNode, db, [&]() {
    mergeNodeIntoSubgraph(to_merge, subgraphNode);
    return subgraphNode;
  });
}

```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `mergeNodeIntoSubgraphAndUpdateAliasing`, `executeSubgraphMergeAndUpdateAliasing`, `mergeNodeIntoSubgraph`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`mergeNodeIntoSubgraphAndUpdateAliasing`, `executeSubgraphMergeAndUpdateAliasing`, `mergeNodeIntoSubgraph`。

### Lines 423-432
```cpp
Node* createSingletonSubgraphAndUpdateAliasing(
    Node* to_merge,
    Symbol subgraphKind,
    AliasDb& db) {
  return executeSubgraphMergeAndUpdateAliasing(
      to_merge, std::nullopt, db, [&]() {
        return createSingletonSubgraph(to_merge, subgraphKind);
      });
}

```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `createSingletonSubgraphAndUpdateAliasing`, `executeSubgraphMergeAndUpdateAliasing`, `createSingletonSubgraph`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`createSingletonSubgraphAndUpdateAliasing`, `executeSubgraphMergeAndUpdateAliasing`, `createSingletonSubgraph`。

### Lines 433-444
```cpp
bool unmergeOutputsAlisingInputs(Node* subgraphNode) {
  GRAPH_DEBUG("unfuseOutputsAlisingInputs on ", getHeader(subgraphNode));
  auto subgraph = subgraphNode->g(attr::Subgraph);
  AliasDb alias_db(subgraph);

  std::set<Node*, topo_cmp_node> nodes;
  for (auto o : subgraph->outputs()) {
    if (alias_db.mayContainAlias(o, subgraph->inputs())) {
      collectNodesToUnfuse(o->node(), nodes);
    }
  }

```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `unmergeOutputsAlisingInputs`, `getHeader`, `g`, `alias_db`, `outputs`, `mayContainAlias`, `...`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`unmergeOutputsAlisingInputs`, `getHeader`, `g`, `alias_db`, `outputs`, `mayContainAlias`, `...`。

### Lines 445-453
```cpp
  // unfuse in the reverse topo order
  for (auto it = nodes.rbegin(); it != nodes.rend(); it++) {
    SubgraphUtils::unmergeNode(*it, subgraphNode);
  }

  return !nodes.empty();
}

bool unmergeAliasedOutputs(Node* subgraphNode) {
```
- EN: This block iterates over collections or graph structures; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `rbegin`, `rend`, `unmergeNode`, `empty`, `unmergeAliasedOutputs`.
- CN: 该代码块遍历集合或图结构；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`rbegin`, `rend`, `unmergeNode`, `empty`, `unmergeAliasedOutputs`。

### Lines 454-463
```cpp
  GRAPH_DEBUG("unfuseAliasedOutputs on ", getHeader(subgraphNode));
  if (subgraphNode->outputs().size() < 2) {
    return false;
  }

  auto subgraph = subgraphNode->g(attr::Subgraph);
  GRAPH_DUMP("unfuseAliasedOutputs Subgraph ", subgraph);
  auto sets = buildAliasedSets(std::move(subgraph));
  GRAPH_DEBUG("buildAliasedSets sets.size() = ", sets.size());

```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `getHeader`, `outputs`, `size`, `g`, `buildAliasedSets`, `move`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`getHeader`, `outputs`, `size`, `g`, `buildAliasedSets`, `move`。

### Lines 464-477
```cpp
  std::set<Node*, topo_cmp_node> nodes;

  for (auto i : c10::irange(sets.size())) {
    if (sets[i].size() <= 1) {
      GRAPH_DEBUG(
          "Set ",
          i,
          " with leader ",
          (*(sets[i].begin()))->debugName(),
          " size = ",
          sets[i].size());
      continue;
    }

```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `irange`, `size`, `begin`, `debugName`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`irange`, `size`, `begin`, `debugName`。

### Lines 478-491
```cpp
    // we have at least two aliased outputs
    // we skip the earliest node w.r.t. the topo order
    // NB. after some nodes are unfused, the outputs of some other nodes
    // may become the outputs of the subgraph and alias the remaining ones
    // so we have to re-run this function until there are no more changes
    auto it = ++sets[i].begin();
    while (it != sets[i].end()) {
      GRAPH_DEBUG(
          "root aliased value ", (*it)->debugName(), " node ", *(*it)->node());
      collectNodesToUnfuse((*it)->node(), nodes);
      it++;
    }
  }

```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `begin`, `end`, `debugName`, `node`, `collectNodesToUnfuse`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`begin`, `end`, `debugName`, `node`, `collectNodesToUnfuse`。

### Lines 492-500
```cpp
  // unfuse in the reverse topo order
  for (auto it = nodes.rbegin(); it != nodes.rend(); it++) {
    unmergeNode(*it, subgraphNode);
  }

  return !nodes.empty();
}

void unmergeNode(Node* n, Node* subgraphNode) {
```
- EN: This block iterates over collections or graph structures; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `rbegin`, `rend`, `unmergeNode`, `empty`.
- CN: 该代码块遍历集合或图结构；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`rbegin`, `rend`, `unmergeNode`, `empty`。

### Lines 501-509
```cpp
  // collect output indices
  GRAPH_DEBUG("unfuseNode node ", getHeader(n));
  auto subgraph = n->owningGraph();

  std::set<Value*> node_outputs(n->outputs().begin(), n->outputs().end());
  std::set<size_t> output_indices;
  std::set<Value*> node_inputs(n->inputs().begin(), n->inputs().end());

  std::unordered_map<Value*, Value*> local_map;
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `getHeader`, `owningGraph`, `node_outputs`, `outputs`, `begin`, `end`, `...`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`getHeader`, `owningGraph`, `node_outputs`, `outputs`, `begin`, `end`, `...`。

### Lines 510-519
```cpp
  auto env = [&](Value* v) {
    auto it = local_map.find(v);
    if (it != local_map.end()) {
      return it->second;
    }
    TORCH_INTERNAL_ASSERT(
        false,
        "all inputs should've been mapped. Couldn't map %",
        v->debugName());
  };
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `find`, `end`, `debugName`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`find`, `end`, `debugName`。

### Lines 520-537
```cpp

  for (auto i : c10::irange(subgraph->outputs().size())) {
    if (node_outputs.count(subgraph->outputs().at(i)) != 0) {
      output_indices.insert(i);
    }

    if (node_inputs.count(subgraph->outputs().at(i)) != 0) {
      GRAPH_DEBUG(
          "output %",
          subgraph->outputs().at(i)->debugName(),
          " is already subgraph's output");
      GRAPH_DEBUG(
          "Mapping %",
          subgraph->outputs().at(i)->debugName(),
          " to %",
          subgraphNode->outputs().at(i)->debugName());
      local_map[subgraph->outputs().at(i)] = subgraphNode->outputs().at(i);
      node_inputs.erase(subgraph->outputs().at(i));
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `irange`, `outputs`, `size`, `count`, `insert`, `debugName`, `...`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`irange`, `outputs`, `size`, `count`, `insert`, `debugName`, `...`。

### Lines 538-546
```cpp
    }
  }

  WithInsertPoint wip(subgraphNode->next());

  // these node inputs need to be added to subgraph's outputs
  // put them in vmap
  for (auto ni : node_inputs) {
    if (local_map.count(ni) != 0) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `wip`, `next`, `count`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`wip`, `next`, `count`。

### Lines 547-555
```cpp
      // this could happen if `n` uses two or more outputs
      // of a constant node and we already cloned the constant
      // into the outer graph and mapped its outputs
      continue;
    }

    Value* sno = nullptr;
    if (ni->node()->kind() == prim::Constant) {
      auto copy = subgraphNode->owningGraph()->createClone(ni->node(), env);
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `node`, `kind`, `owningGraph`, `createClone`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`node`, `kind`, `owningGraph`, `createClone`。

### Lines 556-573
```cpp
      subgraphNode->owningGraph()->insertNode(copy);
      // in case we have a multi-output const, map the rest of the outputs
      // so when we get to clone `n`, `n`'s clone will use the outputs of this
      // constant clone
      for (auto i : c10::irange(n->outputs().size())) {
        GRAPH_DEBUG(
            "Mapping %",
            ni->node()->output(i)->debugName(),
            " to %",
            copy->output(i)->debugName());
        local_map[ni->node()->output(i)] = copy->output(i);
      }
    } else {
      subgraph->registerOutput(ni);
      sno = subgraphNode->addOutput();
      sno->setType(ni->type());
      GRAPH_DEBUG("Mapping %", ni->debugName(), " to %", sno->debugName());
      local_map[ni] = sno;
```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `owningGraph`, `insertNode`, `irange`, `outputs`, `size`, `node`, `...`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`owningGraph`, `insertNode`, `irange`, `outputs`, `size`, `node`, `...`。

### Lines 574-582
```cpp
    }
  }

  auto copy = subgraphNode->owningGraph()->createClone(n, env);
  GRAPH_DEBUG("copy ", *copy);

  for (auto i : c10::irange(n->outputs().size())) {
    auto oo = n->outputs()[i];
    auto no = copy->outputs()[i];
```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `owningGraph`, `createClone`, `irange`, `outputs`, `size`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`owningGraph`, `createClone`, `irange`, `outputs`, `size`。

### Lines 583-591
```cpp
    no->copyMetadata(oo);
    GRAPH_DEBUG("Mapping %", oo->debugName(), " to %", no->debugName());
    local_map[oo] = no;
  }

  subgraphNode->owningGraph()->insertNode(copy);

  for (auto it = output_indices.rbegin(); it != output_indices.rend(); it++) {
    auto replace_val = local_map[subgraph->outputs().at(*it)];
```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `copyMetadata`, `debugName`, `owningGraph`, `insertNode`, `rbegin`, `rend`, `...`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`copyMetadata`, `debugName`, `owningGraph`, `insertNode`, `rbegin`, `rend`, `...`。

### Lines 592-600
```cpp
    subgraphNode->outputs().at(*it)->replaceAllUsesWith(replace_val);
    subgraphNode->eraseOutput(*it);
    subgraph->eraseOutput(*it);
  }

  n->destroy();
}

static std::string truncateStrWithHash(const std::string& s, size_t maxlen) {
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `outputs`, `replaceAllUsesWith`, `eraseOutput`, `destroy`, `truncateStrWithHash`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`outputs`, `replaceAllUsesWith`, `eraseOutput`, `destroy`, `truncateStrWithHash`。

### Lines 601-611
```cpp
  if (s.size() <= maxlen) {
    return s;
  }
  std::string hash_str = std::to_string(c10::hash<std::string>{}(s));
  // If hash-string plus '_' can fit into maxlen, then truncate the original
  // string correspondingly so that the final string with the hash included fits
  // into maxlen. If that's not possible, at least truncate the original string
  // to maxlen (and append the hash to it).
  size_t trunc_len =
      (maxlen > hash_str.size() + 1) ? (maxlen - hash_str.size() - 1) : maxlen;
  std::stringstream truncated;
```
- EN: This block handles conditional branches; produces a result or forwards a computed value. Key symbols: `size`, `to_string`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值。关键符号：`size`, `to_string`。

### Lines 612-621
```cpp
  truncated << s.substr(0, trunc_len);
  truncated << '_' << hash_str;
  return truncated.str();
}

std::string generateNameForGraph(
    const std::shared_ptr<Graph>& graph,
    size_t maxlen,
    const std::string& prefix) {
  std::stringstream graph_name;
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `substr`, `str`, `generateNameForGraph`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`substr`, `str`, `generateNameForGraph`。

### Lines 622-631
```cpp
  graph_name << prefix;
  for (Node* node : graph->nodes()) {
    if (!node->kind().is_aten()) {
      continue;
    }
    graph_name << '_' << node->kind().toUnqualString();
  }
  return truncateStrWithHash(graph_name.str(), maxlen);
}

```
- EN: This block handles conditional branches; iterates over collections or graph structures; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `nodes`, `kind`, `is_aten`, `toUnqualString`, `truncateStrWithHash`, `str`.
- CN: 该代码块处理条件分支；遍历集合或图结构；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`nodes`, `kind`, `is_aten`, `toUnqualString`, `truncateStrWithHash`, `str`。

### Lines 632-632
```cpp
} // namespace torch::jit::SubgraphUtils
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
- Internal includes / 内部头文件: `torch/csrc/jit/passes/utils/subgraph_utils.h`, `torch/csrc/jit/passes/canonicalize.h`, `ATen/core/symbol.h`, `c10/util/irange.h`, `torch/csrc/jit/jit_log.h`
- External includes / 外部头文件: `utility`
- Namespaces / 命名空间: `torch::jit::SubgraphUtils`
- Representative symbols / 代表性符号: `hasSubgraph`, `hasAttribute`, `gatherLastUses`, `fmap`, `firstOrLastUse`, `ValueMapper`, `outputs`, `guard`, `owningGraph`, `insertNode`, `...`
