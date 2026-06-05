# create_functional_graphs.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `torch/csrc/jit/passes/create_functional_graphs.cpp`
- Repository: `pytorch`
- Purpose (EN): Implements the TorchScript JIT pass logic for create functional graphs, including graph analysis and rewrites.
- 用途 (CN): 实现与 create functional graphs 相关的 TorchScript JIT pass 逻辑，包括图分析与改写。

## Line-by-Line Analysis / 逐行分析
### Lines 1-7
```cpp
#include <torch/csrc/jit/passes/create_functional_graphs.h>

#include <c10/util/Exception.h>
#include <torch/csrc/jit/ir/alias_analysis.h>
#include <torch/csrc/jit/passes/constant_pooling.h>
#include <torch/csrc/jit/passes/utils/subgraph_utils.h>

```
- EN: Pulls in the headers needed by the create functional graphs logic. Internal dependencies: `torch/csrc/jit/passes/create_functional_graphs.h`, `c10/util/Exception.h`, `torch/csrc/jit/ir/alias_analysis.h`, `torch/csrc/jit/passes/constant_pooling.h`, `torch/csrc/jit/passes/utils/subgraph_utils.h`; external dependencies: none.
- CN: 为 create functional graphs 相关逻辑引入所需头文件。内部依赖：`torch/csrc/jit/passes/create_functional_graphs.h`, `c10/util/Exception.h`, `torch/csrc/jit/ir/alias_analysis.h`, `torch/csrc/jit/passes/constant_pooling.h`, `torch/csrc/jit/passes/utils/subgraph_utils.h`；外部依赖：无。

### Lines 8-14
```cpp
#include <cstddef>

namespace torch::jit {

namespace {

struct FunctionalGraphSlicer {
```
- EN: Opens or closes namespace scopes so the remaining declarations live in the expected JIT/frontend module.
- CN: 打开或关闭命名空间作用域，使后续声明归属于预期的 JIT/前端模块。

### Lines 15-22
```cpp
  FunctionalGraphSlicer(std::shared_ptr<Graph> graph)
      : graph_(std::move(graph)) {}

  void run() {
    bool changed = true;
    // TODO: more sane strategy
    size_t MAX_NUM_ITERATIONS = 4;

```
- EN: This block manipulates TorchScript IR objects. Key symbols: `FunctionalGraphSlicer`, `graph_`, `move`, `run`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`FunctionalGraphSlicer`, `graph_`, `move`, `run`。

### Lines 23-33
```cpp
    // First, analyze the functional subset of the graph, and then create
    // functional graphs. The graph gets mutated when we create functional
    // subgraphs, invalidating the AliasDb, so we need to do our analysis
    // first.
    for (size_t i = 0; i < MAX_NUM_ITERATIONS && changed; ++i) {
      aliasDb_ = std::make_unique<AliasDb>(graph_);
      AnalyzeFunctionalSubset(graph_->block());
      changed = CreateFunctionalGraphsImpl(graph_->block());
    }
  }

```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `AnalyzeFunctionalSubset`, `block`, `CreateFunctionalGraphsImpl`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`AnalyzeFunctionalSubset`, `block`, `CreateFunctionalGraphsImpl`。

### Lines 34-40
```cpp
 private:
  bool isEmptyFunctionalGraph(Node* n) {
    auto g = n->g(attr::Subgraph);
    return g->inputs().empty() && g->outputs().empty();
  }

  void nonConstNodes(Block* block, size_t* num) {
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `isEmptyFunctionalGraph`, `g`, `inputs`, `empty`, `outputs`, `nonConstNodes`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`isEmptyFunctionalGraph`, `g`, `inputs`, `empty`, `outputs`, `nonConstNodes`。

### Lines 41-49
```cpp
    for (auto it = block->nodes().begin();
         it != block->nodes().end() && *num < minSubgraphSize_;
         ++it) {
      Node* n = *it;
      if (n->kind() == prim::Constant) {
        continue;
      }
      *num = *num + 1;
      for (Block* b : n->blocks()) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects; performs optimization-oriented rewriting; protects shared state or ordering assumptions. Key symbols: `nodes`, `begin`, `end`, `kind`, `blocks`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象；执行面向优化的改写；保护共享状态或执行顺序假设。关键符号：`nodes`, `begin`, `end`, `kind`, `blocks`。

### Lines 50-57
```cpp
        nonConstNodes(b, num);
      }
    }
  }

  bool inlineIfTooSmall(Node* n) {
    AT_ASSERT(n->kind() == prim::FunctionalGraph);
    auto subgraph = SubgraphUtils::getSubgraph(n);
```
- EN: This block manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `nonConstNodes`, `inlineIfTooSmall`, `kind`, `getSubgraph`.
- CN: 该代码块操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`nonConstNodes`, `inlineIfTooSmall`, `kind`, `getSubgraph`。

### Lines 58-64
```cpp
    size_t num_modes = 0;
    nonConstNodes(subgraph->block(), &num_modes);
    if (num_modes < minSubgraphSize_) {
      SubgraphUtils::unmergeSubgraph(n);
      return true;
    }
    return false;
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `nonConstNodes`, `block`, `unmergeSubgraph`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`nonConstNodes`, `block`, `unmergeSubgraph`。

### Lines 65-75
```cpp
  }

  bool CreateFunctionalGraphsImpl(Block* block) {
    /*
    Iterate the block in reverse and create FunctionalSubgraphs.
    When we encounter a node that isn't functional, we skip it. Otherwise,
    we try to merge the functional node into the current functional subgraph.
    If it can't be merged into the current functional subgraph node, then we
    start a functional subgraph group.
    */
    bool changed = false;
```
- EN: This block manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `CreateFunctionalGraphsImpl`.
- CN: 该代码块操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`CreateFunctionalGraphsImpl`。

### Lines 76-82
```cpp
    std::vector<Node*> functional_graph_nodes;

    Node* functional_subgraph_node =
        graph_->createWithSubgraph(prim::FunctionalGraph)
            ->insertBefore(block->return_node());
    auto reverse_iter = block->nodes().reverse();
    for (auto it = reverse_iter.begin(); it != reverse_iter.end();) {
```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `createWithSubgraph`, `insertBefore`, `return_node`, `nodes`, `reverse`, `begin`, `...`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`createWithSubgraph`, `insertBefore`, `return_node`, `nodes`, `reverse`, `begin`, `...`。

### Lines 83-89
```cpp
      Node* n = *it++;

      // constants get copied into the graph
      if (n->kind() == prim::Constant || n == functional_subgraph_node) {
        continue;
      }

```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `kind`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`kind`。

### Lines 90-100
```cpp
      // if `n` is functional, all of its blocks will be merged into the
      // new functional subgraph, so we only need to recurse if it is not
      // functional
      if (!functional_nodes_.count(n)) {
        for (Block* b : n->blocks()) {
          auto block_changed = CreateFunctionalGraphsImpl(b);
          changed = block_changed && changed;
        }
        continue;
      }

```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `count`, `blocks`, `CreateFunctionalGraphsImpl`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`count`, `blocks`, `CreateFunctionalGraphsImpl`。

### Lines 101-107
```cpp
      if (n->kind() == prim::FunctionalGraph &&
          isEmptyFunctionalGraph(functional_subgraph_node)) {
        functional_subgraph_node->destroy();
        functional_subgraph_node = n;
        continue;
      }

```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `kind`, `isEmptyFunctionalGraph`, `destroy`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`kind`, `isEmptyFunctionalGraph`, `destroy`。

### Lines 108-119
```cpp
      changed = true;
      if (aliasDb_->moveBeforeTopologicallyValid(n, functional_subgraph_node)) {
        SubgraphUtils::mergeNodeIntoSubgraph(n, functional_subgraph_node);
      } else {
        functional_graph_nodes.emplace_back(functional_subgraph_node);
        functional_subgraph_node =
            graph_->createWithSubgraph(prim::FunctionalGraph)->insertAfter(n);
        SubgraphUtils::mergeNodeIntoSubgraph(n, functional_subgraph_node);
      }
    }
    functional_graph_nodes.emplace_back(functional_subgraph_node);

```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `moveBeforeTopologicallyValid`, `mergeNodeIntoSubgraph`, `emplace_back`, `createWithSubgraph`, `insertAfter`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`moveBeforeTopologicallyValid`, `mergeNodeIntoSubgraph`, `emplace_back`, `createWithSubgraph`, `insertAfter`。

### Lines 120-127
```cpp
    for (Node* functional_node : functional_graph_nodes) {
      if (!inlineIfTooSmall(functional_node)) {
        ConstantPooling(functional_node->g(attr::Subgraph));
      }
    }
    return changed;
  }

```
- EN: This block handles conditional branches; iterates over collections or graph structures; produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `inlineIfTooSmall`, `ConstantPooling`, `g`.
- CN: 该代码块处理条件分支；遍历集合或图结构；返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`inlineIfTooSmall`, `ConstantPooling`, `g`。

### Lines 128-141
```cpp
  bool AnalyzeFunctionalSubset(Node* n) {
    // TODO: clarify hasSideEffects, isNondeterministic
    bool is_functional_node = true;

    // Functional Graphs are not responsible for maintaining aliasing
    // relationships. If an output of a functional graph escapes scope
    // or is mutated then we might change semantics of the program if
    // aliasing relationships are changed.
    // We don't allow any node in the functional graph to output a value
    // that escapes scope or is mutated, and we don't allow any mutating nodes
    // into the graph.
    // - allow functional graphs to have at most one value that can escape scope
    // - allow outputs which alias the wildcard set but do not "re-escape"
    for (Value* v : n->outputs()) {
```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `AnalyzeFunctionalSubset`, `outputs`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`AnalyzeFunctionalSubset`, `outputs`。

### Lines 142-149
```cpp
      bool has_writers = aliasDb_->hasWriters(v);
      bool escapes_scope = aliasDb_->escapesScope(v);
      if (has_writers) {
        mutated_values_.insert(v);
      }
      is_functional_node = is_functional_node && !escapes_scope && !has_writers;
    }

```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `hasWriters`, `escapesScope`, `insert`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`hasWriters`, `escapesScope`, `insert`。

### Lines 150-156
```cpp
    for (Block* block : n->blocks()) {
      auto functional_block = AnalyzeFunctionalSubset(block);
      is_functional_node = is_functional_node && functional_block;
    }

    is_functional_node = is_functional_node && !aliasDb_->isMutable(n);
    if (is_functional_node) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `blocks`, `AnalyzeFunctionalSubset`, `isMutable`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`blocks`, `AnalyzeFunctionalSubset`, `isMutable`。

### Lines 157-163
```cpp
      functional_nodes_.insert(n);
    }
    return is_functional_node;
  }

  void AnalyzeFunctionalSubset(at::ArrayRef<Block*> blocks) {
    for (Block* block : blocks) {
```
- EN: This block iterates over collections or graph structures; produces a result or forwards a computed value; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `insert`, `AnalyzeFunctionalSubset`.
- CN: 该代码块遍历集合或图结构；返回结果或转发已计算的值；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`insert`, `AnalyzeFunctionalSubset`。

### Lines 164-172
```cpp
      AnalyzeFunctionalSubset(block);
    }
  }

  bool AnalyzeFunctionalSubset(Block* block) {
    bool is_functional_block = true;
    // block inputs will not yet have been iterated through,
    // so we need to add them to our set of mutated & escape values.
    for (Value* v : block->inputs()) {
```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `AnalyzeFunctionalSubset`, `inputs`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`AnalyzeFunctionalSubset`, `inputs`。

### Lines 173-181
```cpp
      bool has_writers = aliasDb_->hasWriters(v);
      if (has_writers) {
        mutated_values_.insert(v);
      }
    }
    // if a block output is not functional, then the corresponding output for
    // the node that contains the block will not be functional either, so we do
    // not need to analyze the block outputs here.
    for (Node* n : block->nodes()) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `hasWriters`, `insert`, `nodes`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`hasWriters`, `insert`, `nodes`。

### Lines 182-188
```cpp
      bool functional = AnalyzeFunctionalSubset(n);
      is_functional_block = is_functional_block && functional;
    }
    return is_functional_block;
  }

  std::unordered_set<Node*> functional_nodes_;
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `AnalyzeFunctionalSubset`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`AnalyzeFunctionalSubset`。

### Lines 189-195
```cpp
  std::unordered_set<Value*> mutated_values_;
  std::shared_ptr<Graph> graph_;
  std::unique_ptr<AliasDb> aliasDb_ = nullptr;
  size_t minSubgraphSize_ = 6;
};

void InlineFunctionalGraphs(Block* block) {
```
- EN: This block manipulates TorchScript IR objects; performs optimization-oriented rewriting; protects shared state or ordering assumptions. Key symbols: `InlineFunctionalGraphs`.
- CN: 该代码块操作 TorchScript IR 对象；执行面向优化的改写；保护共享状态或执行顺序假设。关键符号：`InlineFunctionalGraphs`。

### Lines 196-202
```cpp
  for (auto it = block->nodes().begin(); it != block->nodes().end();) {
    Node* n = *it;
    it++;
    for (Block* b : n->blocks()) {
      InlineFunctionalGraphs(b);
    }
    if (n->kind() == prim::FunctionalGraph) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects; performs optimization-oriented rewriting; protects shared state or ordering assumptions. Key symbols: `nodes`, `begin`, `end`, `blocks`, `InlineFunctionalGraphs`, `kind`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象；执行面向优化的改写；保护共享状态或执行顺序假设。关键符号：`nodes`, `begin`, `end`, `blocks`, `InlineFunctionalGraphs`, `kind`。

### Lines 203-209
```cpp
      SubgraphUtils::unmergeSubgraph(n);
    }
  }
}

} // namespace

```
- EN: This block manipulates TorchScript IR objects. Key symbols: `unmergeSubgraph`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`unmergeSubgraph`。

### Lines 210-218
```cpp
void CreateFunctionalGraphs(const std::shared_ptr<Graph>& graph) {
  // Run Constant Pooling so constants get hoisted
  ConstantPooling(graph);
  FunctionalGraphSlicer func(graph);
  func.run();
  // Creation of Functional Subgraphs & Deinlining creates excess constants
  ConstantPooling(graph);
}

```
- EN: This block manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `CreateFunctionalGraphs`, `ConstantPooling`, `func`, `run`.
- CN: 该代码块操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`CreateFunctionalGraphs`, `ConstantPooling`, `func`, `run`。

### Lines 219-223
```cpp
void InlineFunctionalGraphs(const std::shared_ptr<Graph>& graph) {
  InlineFunctionalGraphs(graph->block());
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
- EN: Subsystem integration — the implementation collaborates with nearby JIT/frontend headers and runtime helpers.
  CN: 子系统集成——实现会与附近的 JIT/前端头文件以及运行时辅助模块协同工作。

## Dependencies / 依赖关系
- Internal includes / 内部头文件: `torch/csrc/jit/passes/create_functional_graphs.h`, `c10/util/Exception.h`, `torch/csrc/jit/ir/alias_analysis.h`, `torch/csrc/jit/passes/constant_pooling.h`, `torch/csrc/jit/passes/utils/subgraph_utils.h`
- External includes / 外部头文件: `cstddef`
- Namespaces / 命名空间: `torch::jit`
- Representative symbols / 代表性符号: `FunctionalGraphSlicer`, `graph_`, `move`, `run`, `AnalyzeFunctionalSubset`, `block`, `CreateFunctionalGraphsImpl`, `isEmptyFunctionalGraph`, `g`, `inputs`, `...`
