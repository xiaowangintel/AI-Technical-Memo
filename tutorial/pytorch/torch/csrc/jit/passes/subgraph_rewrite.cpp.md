# subgraph_rewrite.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `torch/csrc/jit/passes/subgraph_rewrite.cpp`
- Repository: `pytorch`
- Purpose (EN): Implements the TorchScript JIT pass logic for subgraph rewrite, including graph analysis and rewrites.
- 用途 (CN): 实现与 subgraph rewrite 相关的 TorchScript JIT pass 逻辑，包括图分析与改写。

## Line-by-Line Analysis / 逐行分析
### Lines 1-7
```cpp
#include <torch/csrc/jit/passes/subgraph_rewrite.h>

#include <torch/csrc/jit/ir/irparser.h>
#include <torch/csrc/jit/ir/subgraph_matcher.h>

#include <c10/util/irange.h>

```
- EN: Pulls in the headers needed by the subgraph rewrite logic. Internal dependencies: `torch/csrc/jit/passes/subgraph_rewrite.h`, `torch/csrc/jit/ir/irparser.h`, `torch/csrc/jit/ir/subgraph_matcher.h`, `c10/util/irange.h`; external dependencies: none.
- CN: 为 subgraph rewrite 相关逻辑引入所需头文件。内部依赖：`torch/csrc/jit/passes/subgraph_rewrite.h`, `torch/csrc/jit/ir/irparser.h`, `torch/csrc/jit/ir/subgraph_matcher.h`, `c10/util/irange.h`；外部依赖：无。

### Lines 8-16
```cpp
#include <utility>

namespace torch::jit {

namespace {
void update_source_range_and_cs_ptr(
    const std::set<const Node*>& input_nodes,
    const Match& m,
    std::unordered_map<Node*, Node*>& pattern_node_map) {
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `update_source_range_and_cs_ptr`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`update_source_range_and_cs_ptr`。

### Lines 17-23
```cpp
  // pattern_node_map, maps nodes of the replacement graph
  // to the nodes of the pattern graph.
  // Now we iterate over each node of the replacement graph
  // and find the corresponding pattern node in the match.
  // The matched's node's source range and callstack is then
  // used to update replacement node's source range and callstack
  for (auto& it : pattern_node_map) {
```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: no dominant local symbol names.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象。关键符号：无明显局部符号。

### Lines 24-36
```cpp
    Node* replacement_node = it.first;
    Node* pattern_node = it.second;
    if (!input_nodes.count(pattern_node)) {
      Node* orig_node = m.nodes_map.at(pattern_node);
      replacement_node->setSourceRange(orig_node->sourceRange());
      if (orig_node->callstack()) {
        replacement_node->setCallStack(orig_node->callstack().value());
      }
    }
  }
}
} // namespace

```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `count`, `setSourceRange`, `sourceRange`, `callstack`, `setCallStack`, `value`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`count`, `setSourceRange`, `sourceRange`, `callstack`, `setCallStack`, `value`。

### Lines 37-44
```cpp
void SubgraphRewriter::RegisterDefaultPatterns() {
  // TODO: Add actual patterns (like Conv-Relu).
  RegisterRewritePattern(
      R"IR(
graph(%x, %w, %b):
  %c = aten::conv(%x, %w, %b)
  %r = aten::relu(%c)
  return (%r))IR",
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `RegisterDefaultPatterns`, `RegisterRewritePattern`, `graph`, `conv`, `relu`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`RegisterDefaultPatterns`, `RegisterRewritePattern`, `graph`, `conv`, `relu`。

### Lines 45-51
```cpp
      R"IR(
graph(%x, %w, %b):
  %r = aten::convrelu(%x, %w, %b)
  return (%r))IR",
      {{"r", "c"}});
}

```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `graph`, `convrelu`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`graph`, `convrelu`。

### Lines 52-61
```cpp
void SubgraphRewriter::RegisterRewritePattern(
    const std::string& pattern,
    const std::string& replacement,
    const std::vector<std::pair<std::string, std::string>>& value_name_pairs) {
  std::unordered_map<std::string, std::string> value_name_map(
      value_name_pairs.begin(), value_name_pairs.end());
  RewritePatternDescr d = {pattern, replacement, std::move(value_name_map)};
  patterns_.push_back(std::move(d));
}

```
- EN: This block manipulates TorchScript IR objects. Key symbols: `RegisterRewritePattern`, `value_name_map`, `begin`, `end`, `move`, `push_back`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`RegisterRewritePattern`, `value_name_map`, `begin`, `end`, `move`, `push_back`。

### Lines 62-68
```cpp
Module SubgraphRewriter::runOnModule(const Module& module) {
  nodes_to_delete_.clear();
  for (const auto& m : module.get_methods()) {
    auto g = toGraphFunction(m.function()).graph();
    runOnGraph(g);
  }
  return module;
```
- EN: This block iterates over collections or graph structures; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `runOnModule`, `clear`, `get_methods`, `toGraphFunction`, `function`, `graph`, `...`.
- CN: 该代码块遍历集合或图结构；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`runOnModule`, `clear`, `get_methods`, `toGraphFunction`, `function`, `graph`, `...`。

### Lines 69-78
```cpp
}

void SubgraphRewriter::runOnGraph(
    std::shared_ptr<Graph>& graph,
    const std::vector<MatchFilter>& filters) {
  for (const RewritePatternDescr& pattern : patterns_) {
    rewriteSinglePatternOnGraph(graph, pattern, filters);
  }
}

```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `runOnGraph`, `rewriteSinglePatternOnGraph`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`runOnGraph`, `rewriteSinglePatternOnGraph`。

### Lines 79-85
```cpp
void SubgraphRewriter::rewriteSinglePatternOnGraph(
    std::shared_ptr<Graph>& graph,
    const RewritePatternDescr& pattern,
    const std::vector<MatchFilter>& filters) {
  std::unordered_map<Value*, Value*> rewrite_map;
  std::vector<Value*> values_to_rewrite;

```
- EN: This block manipulates TorchScript IR objects. Key symbols: `rewriteSinglePatternOnGraph`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`rewriteSinglePatternOnGraph`。

### Lines 86-93
```cpp
  Graph pattern_graph;
  std::unordered_map<std::string, Value*> vmap;
  parseIR(pattern.pattern, &pattern_graph, vmap);

  Graph replacement_graph;
  std::unordered_map<std::string, Value*> vmap_replacement;
  parseIR(pattern.replacement, &replacement_graph, vmap_replacement);

```
- EN: This block manipulates TorchScript IR objects; advances frontend parsing or lowering state. Key symbols: `parseIR`.
- CN: 该代码块操作 TorchScript IR 对象；推进前端解析或降级状态。关键符号：`parseIR`。

### Lines 94-100
```cpp
  // First construct map of Node*-to-Node*
  // This maps Nodes in replacement graph to nodes in pattern graph
  // given the value_name_map, which maps value names from replacement
  // pattern to value name in pattern
  std::unordered_map<Node*, Node*> pattern_node_map;
  std::set<const Node*> pattern_input_nodes;
  for (auto& it : vmap_replacement) {
```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: no dominant local symbol names.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象。关键符号：无明显局部符号。

### Lines 101-109
```cpp
    const auto& replacement_value_name = it.first;
    Node* replacement_value_node = it.second->node();
    if (pattern.value_name_map.count(replacement_value_name)) {
      const auto& pattern_value_name =
          pattern.value_name_map.at(replacement_value_name);
      TORCH_CHECK(
          vmap.count(pattern_value_name),
          "Value must be found in the replacement graph.");
      Node* pattern_value_node = vmap.at(pattern_value_name)->node();
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `node`, `count`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`node`, `count`。

### Lines 110-116
```cpp
      pattern_node_map.emplace(replacement_value_node, pattern_value_node);
    }
  }

  const auto& matches = findPatternMatches(pattern_graph, *graph);
  for (const Match& match : matches) {
    if (!std::all_of(filters.begin(), filters.end(), [&](const MatchFilter& f) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `emplace`, `findPatternMatches`, `all_of`, `begin`, `end`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`emplace`, `findPatternMatches`, `all_of`, `begin`, `end`。

### Lines 117-124
```cpp
          return f(match, vmap);
        })) {
      continue;
    }
    // Matches might overlap with each other, in that case some of the nodes in
    // the current match might have already been used in another folded pattern.
    // We need to skip such matches.
    if (overlapsWithPreviousMatches(&match)) {
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `f`, `overlapsWithPreviousMatches`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`f`, `overlapsWithPreviousMatches`。

### Lines 125-131
```cpp
      continue;
    }

    // Figure out what values we need to use as inputs and outputs for the
    // replacement subgraph and where the replacement subgraph needs to be
    // inserted.
    Node* ins_point = nullptr;
```
- EN: This block manipulates TorchScript IR objects. Key symbols: no dominant local symbol names.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：无明显局部符号。

### Lines 132-141
```cpp
    std::vector<Value*> inputs, outputs;
    for (Value* v : pattern_graph.inputs()) {
      Value* input = match.values_map.at(v);
      if (!ins_point || ins_point->isBefore(input->node())) {
        ins_point = input->node();
      }
      inputs.push_back(input);
    }
    AT_ASSERT(ins_point);

```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `inputs`, `isBefore`, `node`, `push_back`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`inputs`, `isBefore`, `node`, `push_back`。

### Lines 142-148
```cpp
    // Check that the insertion point we've chosen precedes all the uses of the
    // outputs - otherwise the replacement is incorrect and we have to skip it.
    bool ins_point_before_uses = true;
    for (Value* v : pattern_graph.outputs()) {
      Value* output = match.values_map.at(v);
      outputs.push_back(match.values_map.at(v));

```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `outputs`, `push_back`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`outputs`, `push_back`。

### Lines 149-156
```cpp
      for (const Use& u : output->uses()) {
        if (u.user->isBefore(ins_point)) {
          ins_point_before_uses = false;
          break;
        }
      }
    }

```
- EN: This block handles conditional branches; iterates over collections or graph structures. Key symbols: `uses`, `isBefore`.
- CN: 该代码块处理条件分支；遍历集合或图结构。关键符号：`uses`, `isBefore`。

### Lines 157-170
```cpp
    if (!ins_point_before_uses) {
      continue;
    }

    // Before rewriting the graph, update source range and callstack
    // info of the replacement pattern graph so that the rewritten graph
    // has the updated info
    update_source_range_and_cs_ptr(
        pattern_input_nodes, match, pattern_node_map);
    // Insert a clone of replacement subgraph.
    // `inputs` vector holds values that we would use as incoming values to the
    // new subgraph, and we will get `new_outputs` vector containing values
    // produced by this new subgraph - we will then rewrite old outputs with the
    // new ones.
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `update_source_range_and_cs_ptr`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`update_source_range_and_cs_ptr`。

### Lines 171-177
```cpp
    WithInsertPoint insert_point(ins_point->next());
    std::vector<Value*> new_outputs =
        insertGraph(*graph, replacement_graph, inputs);

    // Record all planned rewritings
    AT_ASSERT(outputs.size() == new_outputs.size());
    for (const auto idx : c10::irange(outputs.size())) {
```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `insert_point`, `next`, `insertGraph`, `size`, `irange`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`insert_point`, `next`, `insertGraph`, `size`, `irange`。

### Lines 178-184
```cpp
      values_to_rewrite.push_back(outputs[idx]);
      rewrite_map[outputs[idx]] =
          new_outputs[idx]->setType(outputs[idx]->type());
    }
    // Record all planned deletions
    for (Node* pattern_n : pattern_graph.nodes()) {
      if (match.nodes_map.count(pattern_n)) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `push_back`, `setType`, `type`, `nodes`, `count`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`push_back`, `setType`, `type`, `nodes`, `count`。

### Lines 185-192
```cpp
        Node* n = match.nodes_map.at(pattern_n);
        nodes_to_delete_.insert(n);
      }
    }
  }

  // Perform planned rewritings
  for (auto v : values_to_rewrite) {
```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `insert`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`insert`。

### Lines 193-200
```cpp
    v->replaceAllUsesWith(rewrite_map.at(v));
  }

  // Perform planned deletions
  for (auto n : nodes_to_delete_) {
    n->removeAllInputs();
  }
  for (auto n : nodes_to_delete_) {
```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `replaceAllUsesWith`, `removeAllInputs`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`replaceAllUsesWith`, `removeAllInputs`。

### Lines 201-207
```cpp
    n->destroy();
  }
  nodes_to_delete_.clear();
}

bool SubgraphRewriter::overlapsWithPreviousMatches(const Match* match) {
  for (auto n : match->nodes_map) {
```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `destroy`, `clear`, `overlapsWithPreviousMatches`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`destroy`, `clear`, `overlapsWithPreviousMatches`。

### Lines 208-214
```cpp
    if (nodes_to_delete_.count(n.second)) {
      return true;
    }
  }
  return false;
}

```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `count`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`count`。

### Lines 215-221
```cpp
Module PatternBasedRewrite(const Module& module) {
  // TODO: Deep-copy the module
  SubgraphRewriter subgraph_rewriter;
  subgraph_rewriter.RegisterDefaultPatterns();
  return subgraph_rewriter.runOnModule(module);
}

```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `PatternBasedRewrite`, `RegisterDefaultPatterns`, `runOnModule`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`PatternBasedRewrite`, `RegisterDefaultPatterns`, `runOnModule`。

### Lines 222-222
```cpp
} // namespace torch::jit
```
- EN: Opens or closes namespace scopes so the remaining declarations live in the expected JIT/frontend module.
- CN: 打开或关闭命名空间作用域，使后续声明归属于预期的 JIT/前端模块。


## Key Concepts / 关键概念
- EN: IR graph manipulation — the file inspects or rewrites TorchScript graphs and nodes.
  CN: IR 图操作——该文件会检查或改写 TorchScript 的图、节点与块。
- EN: Parsing pipeline — the logic translates source text into TorchScript syntax structures.
  CN: 解析流水线——相关逻辑将源码文本转换为 TorchScript 语法结构。
- EN: Optimization pass — the code simplifies graphs by removing redundant work or folding values.
  CN: 优化 pass——代码通过删除冗余计算或常量折叠来简化图。
- EN: Fusion support — the file prepares groups of operations for fused execution.
  CN: 融合支持——该文件会为融合执行准备算子分组。
- EN: Type and shape reasoning — the code propagates static information across the IR.
  CN: 类型与形状推理——代码会在 IR 中传播静态信息。
- EN: Subsystem integration — the implementation collaborates with nearby JIT/frontend headers and runtime helpers.
  CN: 子系统集成——实现会与附近的 JIT/前端头文件以及运行时辅助模块协同工作。

## Dependencies / 依赖关系
- Internal includes / 内部头文件: `torch/csrc/jit/passes/subgraph_rewrite.h`, `torch/csrc/jit/ir/irparser.h`, `torch/csrc/jit/ir/subgraph_matcher.h`, `c10/util/irange.h`
- External includes / 外部头文件: `utility`
- Namespaces / 命名空间: `torch::jit`
- Representative symbols / 代表性符号: `update_source_range_and_cs_ptr`, `count`, `setSourceRange`, `sourceRange`, `callstack`, `setCallStack`, `value`, `RegisterDefaultPatterns`, `RegisterRewritePattern`, `graph`, `...`
