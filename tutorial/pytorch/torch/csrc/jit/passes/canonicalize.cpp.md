# canonicalize.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `torch/csrc/jit/passes/canonicalize.cpp`
- Repository: `pytorch`
- Purpose (EN): Implements the TorchScript JIT pass logic for canonicalize, including graph analysis and rewrites.
- 用途 (CN): 实现与 canonicalize 相关的 TorchScript JIT pass 逻辑，包括图分析与改写。

## Line-by-Line Analysis / 逐行分析
### Lines 1-7
```cpp
#include <torch/csrc/jit/passes/canonicalize.h>

#include <c10/util/irange.h>
#include <torch/csrc/jit/ir/ir_views.h>

namespace torch::jit {

```
- EN: Opens or closes namespace scopes so the remaining declarations live in the expected JIT/frontend module.
- CN: 打开或关闭命名空间作用域，使后续声明归属于预期的 JIT/前端模块。

### Lines 8-15
```cpp
// Canonicalize a graph, renumbering it so that all structurally equivalent
// graphs have same numbers.
// keep_unique_names: If false, canonicalizes unique names by removing them
//   and replacing them with normal value names.
//   Otherwise, ignores values with unique names.
std::shared_ptr<Graph> Canonicalize(
    const std::shared_ptr<Graph>& graph,
    bool keep_unique_names) {
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `Canonicalize`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`Canonicalize`。

### Lines 16-22
```cpp
  auto r = std::make_shared<Graph>(graph->current_scope());
  std::unordered_map<Value*, Value*> rn_env;
  auto rn_fn = [&](Value* v) { return rn_env.at(v); };
  for (auto* input : graph->inputs()) {
    auto* r_input = r->addInput();
    r_input->copyMetadata(input);
    if (!keep_unique_names)
```
- EN: This block handles conditional branches; iterates over collections or graph structures; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `current_scope`, `inputs`, `addInput`, `copyMetadata`.
- CN: 该代码块处理条件分支；遍历集合或图结构；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`current_scope`, `inputs`, `addInput`, `copyMetadata`。

### Lines 23-29
```cpp
      r_input->setDebugName("");
    rn_env[input] = r_input;
  }
  for (auto* node : graph->nodes()) {
    auto* r_node = r->createClone(node, rn_fn);
    if (!keep_unique_names) {
      for (auto* output : r_node->outputs()) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `setDebugName`, `nodes`, `createClone`, `outputs`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`setDebugName`, `nodes`, `createClone`, `outputs`。

### Lines 30-36
```cpp
        output->setDebugName("");
      }
    }
    r->appendNode(r_node);
    auto outputs = node->outputs();
    auto r_outputs = r_node->outputs();
    for (const auto i : c10::irange(outputs.size())) {
```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `setDebugName`, `appendNode`, `outputs`, `irange`, `size`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`setDebugName`, `appendNode`, `outputs`, `irange`, `size`。

### Lines 37-45
```cpp
      rn_env[outputs.at(i)] = r_outputs.at(i);
    }
    if (node->hasAttribute(attr::Subgraph)) {
      r_node->g_(
          attr::Subgraph,
          Canonicalize(node->g(attr::Subgraph), keep_unique_names));
    }
  }
  for (auto* output : graph->outputs()) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `hasAttribute`, `g_`, `Canonicalize`, `g`, `outputs`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`hasAttribute`, `g_`, `Canonicalize`, `g`, `outputs`。

### Lines 46-53
```cpp
    r->registerOutput(rn_fn(output));
  }

  return r;
}

// Which index in b's owning Node is b
static size_t blockIndex(const Block* b) {
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `registerOutput`, `rn_fn`, `blockIndex`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`registerOutput`, `rn_fn`, `blockIndex`。

### Lines 54-63
```cpp
  auto n = b->owningNode();
  AT_ASSERT(n);
  for (size_t i = 0; i < n->blocks().size(); ++i) {
    if (n->blocks()[i] == b) {
      return i;
    }
  }
  AT_ASSERT(false);
}

```
- EN: This block handles conditional branches; iterates over collections or graph structures; produces a result or forwards a computed value; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `owningNode`, `blocks`, `size`.
- CN: 该代码块处理条件分支；遍历集合或图结构；返回结果或转发已计算的值；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`owningNode`, `blocks`, `size`。

### Lines 64-75
```cpp
/*
 * This establishes a canonical ordering of nodes.
 * If n1 and n2 are in the same block, whichever node appears first
 * is before the other.
 * If n1 and n2 are contained in different blocks of an if node,
 * then whichever block is in the true block is ordered before the other.
 * If n1 contains n2, then n1 is before n2. This has the nice property that
 * whichever node appears first in a dump of the graph is before the other.
 * NB: this is not a topological index. Topologically, two nodes in
 * different blocks of an if node are not topologically < or > each other.
 */
static bool isBefore(Node* n1, Node* n2) {
```
- EN: This block manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `isBefore`.
- CN: 该代码块操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`isBefore`。

### Lines 76-82
```cpp
  // Invalid to call with the same node as both args
  AT_ASSERT(n1 != n2);

  // Set n1 and n2 to be the number of blocks from the Graph block
  size_t d_1 = n1->blocksFromGraphBlock();
  size_t d_2 = n2->blocksFromGraphBlock();

```
- EN: This block manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `blocksFromGraphBlock`.
- CN: 该代码块操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`blocksFromGraphBlock`。

### Lines 83-90
```cpp
  for (; d_1 > d_2; --d_1) {
    n1 = n1->owningBlock()->owningNode();
    // n2 contains n1
    if (n1 == n2) {
      return false;
    }
  }

```
- EN: This block handles conditional branches; iterates over collections or graph structures; produces a result or forwards a computed value; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `owningBlock`, `owningNode`.
- CN: 该代码块处理条件分支；遍历集合或图结构；返回结果或转发已计算的值；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`owningBlock`, `owningNode`。

### Lines 91-98
```cpp
  for (; d_2 > d_1; --d_2) {
    n2 = n2->owningBlock()->owningNode();
    // n1 contains n2
    if (n2 == n1) {
      return true;
    }
  }

```
- EN: This block handles conditional branches; iterates over collections or graph structures; produces a result or forwards a computed value; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `owningBlock`, `owningNode`.
- CN: 该代码块处理条件分支；遍历集合或图结构；返回结果或转发已计算的值；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`owningBlock`, `owningNode`。

### Lines 99-105
```cpp
  // Now they are the same number of blocks from the graph block,
  // recurse upwards, checking if they are on the same block
  while (true) {
    if (n1->owningBlock() == n2->owningBlock()) {
      return n1->isBefore(n2);
    }

```
- EN: This block handles conditional branches; iterates over collections or graph structures; produces a result or forwards a computed value; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `owningBlock`, `isBefore`.
- CN: 该代码块处理条件分支；遍历集合或图结构；返回结果或转发已计算的值；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`owningBlock`, `isBefore`。

### Lines 106-112
```cpp
    auto new_n1 = n1->owningBlock()->owningNode();
    auto new_n2 = n2->owningBlock()->owningNode();

    AT_ASSERT(new_n1 != nullptr);
    AT_ASSERT(new_n2 != nullptr);

    if (new_n1 == new_n2) {
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `owningBlock`, `owningNode`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`owningBlock`, `owningNode`。

### Lines 113-123
```cpp
      // take whichever node is in the earlier block
      auto index_1 = blockIndex(n1->owningBlock());
      auto index_2 = blockIndex(n2->owningBlock());
      return index_1 < index_2;
    }

    n1 = new_n1;
    n2 = new_n2;
  }
}

```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `blockIndex`, `owningBlock`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`blockIndex`, `owningBlock`。

### Lines 124-130
```cpp
static bool isBefore(const Use& a, const Use& b) {
  // If two uses are the same node, we order on offset
  if (a.user == b.user) {
    return a.offset < b.offset;
  }

  return isBefore(a.user, b.user);
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `isBefore`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`isBefore`。

### Lines 131-137
```cpp
}

static bool isAfter(const Use& a, const Use& b) {
  if (a.user == b.user && a.offset == b.offset) {
    return false;
  }
  return !isBefore(a, b);
```
- EN: This block handles conditional branches; produces a result or forwards a computed value. Key symbols: `isAfter`, `isBefore`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值。关键符号：`isAfter`, `isBefore`。

### Lines 138-144
```cpp
}

bool isBeforeOrAfter(const Use& a, const Use& b, bool checking_before) {
  return checking_before ? isBefore(a, b) : isAfter(a, b);
}

std::optional<const Use> firstOrLastUse(Value* v, bool find_first) {
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `isBeforeOrAfter`, `isBefore`, `isAfter`, `firstOrLastUse`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`isBeforeOrAfter`, `isBefore`, `isAfter`, `firstOrLastUse`。

### Lines 145-151
```cpp
  if (v->uses().empty()) {
    return std::nullopt;
  }
  Use extreme_use = v->uses()[0];
  for (size_t i = 1; i < v->uses().size(); ++i) {
    auto n_use = v->uses()[i];
    if (!isBeforeOrAfter(extreme_use, n_use, find_first)) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; produces a result or forwards a computed value. Key symbols: `uses`, `empty`, `size`, `isBeforeOrAfter`.
- CN: 该代码块处理条件分支；遍历集合或图结构；返回结果或转发已计算的值。关键符号：`uses`, `empty`, `size`, `isBeforeOrAfter`。

### Lines 152-158
```cpp
      extreme_use = n_use;
    }
  }

  return extreme_use;
}

```
- EN: This block produces a result or forwards a computed value. Key symbols: no dominant local symbol names.
- CN: 该代码块返回结果或转发已计算的值。关键符号：无明显局部符号。

### Lines 159-165
```cpp
static std::vector<std::optional<const Use>> gatherFirstUses(
    at::ArrayRef<Value*> values) {
  return fmap(values, [&](Value* v) -> std::optional<const Use> {
    return firstOrLastUse(v, true);
  });
}

```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `gatherFirstUses`, `fmap`, `firstOrLastUse`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`gatherFirstUses`, `fmap`, `firstOrLastUse`。

### Lines 166-172
```cpp
static std::vector<size_t> sort_indexes(at::ArrayRef<Value*> values) {
  // initialize original index locations
  std::vector<size_t> idx(values.size());
  std::iota(idx.begin(), idx.end(), 0);

  std::vector<std::optional<const Use>> first_uses = gatherFirstUses(values);

```
- EN: This block manipulates TorchScript IR objects. Key symbols: `sort_indexes`, `idx`, `size`, `iota`, `begin`, `end`, `...`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`sort_indexes`, `idx`, `size`, `iota`, `begin`, `end`, `...`。

### Lines 173-179
```cpp
  // Sort values based on canonical ordering of their first usage
  std::sort(idx.begin(), idx.end(), [&first_uses](size_t i1, size_t i2) {
    // if neither has any uses, use original ordering. Since the
    // only values that jitter are ones added by the compiler and are guaranteed
    // to have uses, original ordering is fine.
    if (first_uses[i1] == std::nullopt && first_uses[i2] == std::nullopt) {
      return i1 < i2;
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `sort`, `begin`, `end`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`sort`, `begin`, `end`。

### Lines 180-186
```cpp
    }
    if (first_uses[i1] == std::nullopt) {
      return false;
    } else if (first_uses[i2] == std::nullopt) {
      return true;
    }

```
- EN: This block handles conditional branches; produces a result or forwards a computed value. Key symbols: no dominant local symbol names.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值。关键符号：无明显局部符号。

### Lines 187-193
```cpp
    auto fst_v1 = *first_uses[i1];
    auto fst_v2 = *first_uses[i2];

    return isBefore(fst_v1, fst_v2);
  });

  return idx;
```
- EN: This block produces a result or forwards a computed value. Key symbols: `isBefore`.
- CN: 该代码块返回结果或转发已计算的值。关键符号：`isBefore`。

### Lines 194-200
```cpp
}

static void CanonicalizeLoopOutputs(Node* n) {
  auto new_indices = sort_indexes(n->outputs());
  LoopView(n).permuteLoopCarried(new_indices);
}

```
- EN: This block manipulates TorchScript IR objects. Key symbols: `CanonicalizeLoopOutputs`, `sort_indexes`, `outputs`, `LoopView`, `permuteLoopCarried`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`CanonicalizeLoopOutputs`, `sort_indexes`, `outputs`, `LoopView`, `permuteLoopCarried`。

### Lines 201-209
```cpp
static void CanonicalizeIfOutputs(Node* n) {
  auto new_indices = sort_indexes(n->outputs());
  IfView(n).permuteOutputs(new_indices);
}

static void CanonicalizeOutputs(Block* block) {
  // We iterate in reverse since ordering of a node's outputs is dependent on
  // the value use following it in the graph
  for (Node* n : block->nodes().reverse()) {
```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `CanonicalizeIfOutputs`, `sort_indexes`, `outputs`, `IfView`, `permuteOutputs`, `CanonicalizeOutputs`, `...`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`CanonicalizeIfOutputs`, `sort_indexes`, `outputs`, `IfView`, `permuteOutputs`, `CanonicalizeOutputs`, `...`。

### Lines 210-221
```cpp
    switch (n->kind()) {
      case prim::Loop: {
        CanonicalizeLoopOutputs(n);
      } break;
      case prim::If: {
        CanonicalizeIfOutputs(n);
      } break;
    }
    // Since an a control flow node's outputs are after
    // the values outputted within its blocks, first canonicalize
    // the nodes outputs and then recurse on its blocks
    for (Block* b : n->blocks()) {
```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `kind`, `CanonicalizeLoopOutputs`, `CanonicalizeIfOutputs`, `blocks`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`kind`, `CanonicalizeLoopOutputs`, `CanonicalizeIfOutputs`, `blocks`。

### Lines 222-230
```cpp
      CanonicalizeOutputs(b);
    }
  }
}

// Canonicalize a graph's control flow node outputs. We do this to solve jitter
// issues with outputs added to control flow nodes after the first pass of
// compilation in ir_emitter.cpp
void CanonicalizeOutputs(std::shared_ptr<Graph>& graph) {
```
- EN: This block manipulates TorchScript IR objects; advances frontend parsing or lowering state. Key symbols: `CanonicalizeOutputs`.
- CN: 该代码块操作 TorchScript IR 对象；推进前端解析或降级状态。关键符号：`CanonicalizeOutputs`。

### Lines 231-233
```cpp
  CanonicalizeOutputs(graph->block());
}
} // namespace torch::jit
```
- EN: Opens or closes namespace scopes so the remaining declarations live in the expected JIT/frontend module.
- CN: 打开或关闭命名空间作用域，使后续声明归属于预期的 JIT/前端模块。


## Key Concepts / 关键概念
- EN: IR graph manipulation — the file inspects or rewrites TorchScript graphs and nodes.
  CN: IR 图操作——该文件会检查或改写 TorchScript 的图、节点与块。
- EN: Frontend lowering — the file resolves symbols and lowers frontend constructs into IR.
  CN: 前端降级——该文件负责符号解析，并将前端结构降为 IR。
- EN: Pass orchestration — the file plugs logic into the TorchScript optimization pipeline.
  CN: Pass 编排——该文件把相关逻辑接入 TorchScript 优化流水线。
- EN: Subsystem integration — the implementation collaborates with nearby JIT/frontend headers and runtime helpers.
  CN: 子系统集成——实现会与附近的 JIT/前端头文件以及运行时辅助模块协同工作。

## Dependencies / 依赖关系
- Internal includes / 内部头文件: `torch/csrc/jit/passes/canonicalize.h`, `c10/util/irange.h`, `torch/csrc/jit/ir/ir_views.h`
- External includes / 外部头文件: 无
- Namespaces / 命名空间: `torch::jit`
- Representative symbols / 代表性符号: `Canonicalize`, `current_scope`, `inputs`, `addInput`, `copyMetadata`, `setDebugName`, `nodes`, `createClone`, `outputs`, `appendNode`, `...`
