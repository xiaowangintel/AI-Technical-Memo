# SubgraphRewriter.h — Code Analysis / 代码分析

## Source / 来源
- File: `torch/nativert/graph/passes/SubgraphRewriter.h`
- Repository: `pytorch/pytorch`
- Purpose (EN): Declares Native Runtime runtime interfaces, helper types, and contracts for SubgraphRewriter.
- 用途 (CN): 声明 Native Runtime 中 runtime 子模块里与 SubgraphRewriter 相关的接口、辅助类型和约定。

## Line-by-Line Analysis / 逐行分析
### Lines 1-5
```cpp
#pragma once

#include <c10/util/FbcodeMaps.h>
#include <torch/nativert/graph/Graph.h>

```
- EN: Pulls in compile-time configuration or headers required here. Internal includes: `c10/util/FbcodeMaps.h`, `torch/nativert/graph/Graph.h`; external includes: none.
- CN: 引入此处所需的编译期开关或头文件。内部依赖：`c10/util/FbcodeMaps.h`, `torch/nativert/graph/Graph.h`；外部依赖：无。

### Lines 6-15
```cpp
#include <utility>

namespace torch::nativert {

/*
 * node_map: A map from nodes in the pattern to nodes in the actual graph.
 * value_map : A map between values in the pattern to values in the actual
 * graph.
 * dummy_input_to_attribute_map: A map between the actual dummy input values to
 * constant attributes in the actual graph that should replace the dummy nodes
```
- EN: This block manipulates graph-like program structures. Key symbols: no dominant local symbols.
- CN: 该代码块操作图状程序结构。关键符号：无明显局部符号。

### Lines 16-22
```cpp
 */
struct Match {
  std::unordered_map<const Node*, Node*> node_map;
  std::unordered_map<const Value*, Value*> value_map;
  std::unordered_map<Value*, const Constant*>
      dummy_input_to_attribute_map; // For constant attrs matching graph inputs
};
```
- EN: Declares C++ types or containers needed in this file. Representative symbols: `Match`.
- CN: 声明该文件需要的 C++ 类型或容器。代表性符号：`Match`。

### Lines 23-27
```cpp

using MatchFilter = std::function<
    bool(const Match&, const c10::FastMap<std::string, const Value*>&)>;

inline std::ostream& operator<<(std::ostream& out, const Match& match) {
```
- EN: This block implements local helper logic for SubgraphRewriter. Key symbols: `MatchFilter`, `bool`.
- CN: 该代码块实现与 SubgraphRewriter 相关的局部辅助逻辑。关键符号：`MatchFilter`, `bool`。

### Lines 28-35
```cpp
  out << "\nNode mapping:\n";
  for (const auto& kv : match.node_map) {
    const Node* patternNode = kv.first;
    Node* targetNode = kv.second;
    out << "  Pattern Node: " << *patternNode
        << " -> Target Node: " << *targetNode << "\n";
  }

```
- EN: This block manipulates graph-like program structures; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块操作图状程序结构；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 36-43
```cpp
  out << "Value mapping:\n";
  for (const auto& kv : match.value_map) {
    const Value* patternValue = kv.first;
    Value* targetValue = kv.second;
    out << "  Pattern Value: " << *patternValue
        << " -> Target Value: " << *targetValue << "\n";
  }

```
- EN: This block iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 44-53
```cpp
  return out;
}

/**
 * A helper class for matching a subgraph pattern within a larger graph.
 * It attempts to match a given `pattern` graph inside a target `graph`,
 * starting from a single "root" output node in the pattern graph. The
 * matching process works backward through the graph, comparing each node
 * in the pattern to corresponding nodes in the candidate graph.
 *
```
- EN: Declares C++ types or containers needed in this file. Representative symbols: `for`.
- CN: 声明该文件需要的 C++ 类型或容器。代表性符号：`for`。

### Lines 54-63
```cpp
 * Note: This implementation currently only supports deterministic matching
 * for patterns with one output node. It also only matches nodes connecting to
 * output nodes
 *
 * Constraints for Patterns with Multiple Output Nodes:
 * To avoid an exponential increase in the search space, this implementation
 * starts searching from the first output node as an anchor as an heuristic. It
 * assumes that all other output nodes in the pattern are interconnected through
 * the graph from this anchor node, allowing the matcher to traverse from the
 * anchor to other outputs.
```
- EN: Documents intent, assumptions, or compatibility notes that guide the surrounding implementation.
- CN: 说明周围实现所遵循的目标、假设或兼容性约束。

### Lines 64-73
```cpp
 *
 * Important: The order of output nodes in the pattern matters. For example:
 *
 *   graph(%x):
 *       %a = a.aaa(input=%x)
 *       %b = b.bbb(input=%a)
 *       return (%a, %b)
 *
 * If the search starts from %a, it will not explore the portion of the graph
 * connected to %b. However, if the order is switched:
```
- EN: Documents intent, assumptions, or compatibility notes that guide the surrounding implementation.
- CN: 说明周围实现所遵循的目标、假设或兼容性约束。

### Lines 74-82
```cpp
 *
 *   graph(%x):
 *       %a = a.aaa(input=%x)
 *       %b = b.bbb(input=%a)
 *       return (%b, %a)
 *
 * The search will start from %b and successfully explore both %b and %a.
 */
class SubgraphMatcher {
```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `graph`, `aaa`, `bbb`, `SubgraphMatcher`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`graph`, `aaa`, `bbb`, `SubgraphMatcher`。

### Lines 83-89
```cpp
 public:
  explicit SubgraphMatcher(const Graph* pattern);

  /// Attempt to match the pattern at a given node in the target graph.
  /// If successful, returns a Match, otherwise std::nullopt.
  std::optional<Match> match(Node* target_node);

```
- EN: This block manipulates graph-like program structures; handles conditional control flow. Key symbols: `SubgraphMatcher`, `match`.
- CN: 该代码块操作图状程序结构；处理条件控制流。关键符号：`SubgraphMatcher`, `match`。

### Lines 90-95
```cpp
  std::vector<Match> matchAll(Graph* target_graph);

 private:
  const Graph* pattern_;
  const Node* pattern_root_;

```
- EN: This block manipulates graph-like program structures. Key symbols: `matchAll`.
- CN: 该代码块操作图状程序结构。关键符号：`matchAll`。

### Lines 96-102
```cpp
  /**
   * Finds the root output node of a Graph g to start a match from
   * Note that graphs with multiple output nodes, this will pick the first
   * output node in the order provided.
   **/
  const Node* findRootNode(const Graph* g);

```
- EN: This block manipulates graph-like program structures. Key symbols: `findRootNode`.
- CN: 该代码块操作图状程序结构。关键符号：`findRootNode`。

### Lines 103-109
```cpp
  /**
   * Tries to match nodes in the pattern_ graph with the target graph, starting
   * from pattern_node and target_node. Nodes are considered to match if they
   * have the same target type, and all input and output values to the nodes
   * match. Matching nodes are stored to `match`
   **/
  bool tryMatchNode(const Node* pattern_node, Node* target_node, Match& match);
```
- EN: This block manipulates graph-like program structures; handles conditional control flow. Key symbols: `tryMatchNode`.
- CN: 该代码块操作图状程序结构；处理条件控制流。关键符号：`tryMatchNode`。

### Lines 110-115
```cpp

  /**
   * Match inputs of pattern_node w/ target_node. Store matching values to
   *`match`
   **/
  bool tryMatchNodeInputs(
```
- EN: This block manipulates graph-like program structures. Key symbols: `tryMatchNodeInputs`.
- CN: 该代码块操作图状程序结构。关键符号：`tryMatchNodeInputs`。

### Lines 116-124
```cpp
      const Node* pattern_node,
      Node* target_node,
      Match& match);

  /**
   * Tries to match values in the pattern_ graph with the target graph, starting
   * from pval and tval. Matching values are stored to `match`.
   **/
  bool tryMatchValue(const Value* pval, Value* tval, Match& match);
```
- EN: This block manipulates graph-like program structures. Key symbols: `tryMatchValue`.
- CN: 该代码块操作图状程序结构。关键符号：`tryMatchValue`。

### Lines 125-129
```cpp

  /**
   * Returns true of val is an output of its graph, and false otherwise
   **/
  bool isOutputValue(const Value* val);
```
- EN: This block manipulates graph-like program structures. Key symbols: `isOutputValue`.
- CN: 该代码块操作图状程序结构。关键符号：`isOutputValue`。

### Lines 130-135
```cpp
};

struct RewriteRule {
  std::string pattern;
  std::string replacement;
};
```
- EN: Declares C++ types or containers needed in this file. Representative symbols: `RewriteRule`.
- CN: 声明该文件需要的 C++ 类型或容器。代表性符号：`RewriteRule`。

### Lines 136-141
```cpp

/**
 * Rewrite subgraphs in a given graph.
 * TODO: Write more detailed documentation
 **/
class SubgraphRewriter {
```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `SubgraphRewriter`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`SubgraphRewriter`。

### Lines 142-150
```cpp
 public:
  SubgraphRewriter(std::string name) : name_(std::move(name)) {}

  /**
   * Registers the rewrite pattern.
   * @param patternA The subgraph str to match.
   * @param patternB The subgraph str to replace with.
   */
  void registerRewritePattern(
```
- EN: This block manipulates graph-like program structures. Key symbols: `SubgraphRewriter`, `name_`, `move`, `registerRewritePattern`.
- CN: 该代码块操作图状程序结构。关键符号：`SubgraphRewriter`, `name_`, `move`, `registerRewritePattern`。

### Lines 151-160
```cpp
      const std::string& pattern,
      const std::string& replacement);

  /**
   * Runs the subgraph rewrite process on a graph.
   * @param graph The graph on which the rewrite is applied.
   * @param pattern The subgraph to match.
   * @param replacement The subgraph to replace with.
   * @param filters A list of filters to apply to the match. If any filter
   * predicate returns true, the match will not be considered.
```
- EN: This block manipulates graph-like program structures; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块操作图状程序结构；处理条件控制流。关键符号：无明显局部符号。

### Lines 161-167
```cpp
   */
  bool /* mutated? */ runForPattern(
      Graph* graph,
      const Graph& pattern,
      const Graph& replacement,
      const std::vector<MatchFilter>& filters);

```
- EN: This block manipulates graph-like program structures. Key symbols: `runForPattern`.
- CN: 该代码块操作图状程序结构。关键符号：`runForPattern`。

### Lines 168-172
```cpp
  bool /* mutated? */ run(
      Graph* graph,
      const MatchFilter& filter =
          [](const Match&, const c10::FastMap<std::string, const Value*>&) {
            return true;
```
- EN: This block manipulates graph-like program structures; returns results to callers or downstream stages. Key symbols: `run`.
- CN: 该代码块操作图状程序结构；向调用方或后续阶段返回结果。关键符号：`run`。

### Lines 173-177
```cpp
          }) {
    return run(graph, std::vector<MatchFilter>({filter}));
  }

  bool /* mutated? */ run(
```
- EN: This block manipulates graph-like program structures; returns results to callers or downstream stages. Key symbols: `run`.
- CN: 该代码块操作图状程序结构；向调用方或后续阶段返回结果。关键符号：`run`。

### Lines 178-187
```cpp
      Graph* graph,
      const std::vector<MatchFilter>& filters);

 private:
  std::string name_;
  std::vector<RewriteRule> patterns_; // The subgraph pattern to match
  std::unordered_set<Node*> replacedNodes_;
  std::vector<Value*> valuesToRewrite_;
  std::unordered_map<const Value*, Value*> valueRewrites_;

```
- EN: This block manipulates graph-like program structures. Key symbols: no dominant local symbols.
- CN: 该代码块操作图状程序结构。关键符号：无明显局部符号。

### Lines 188-192
```cpp
  // Helper methods
  bool overlapsWithUsedNodes(
      const Match& match,
      const std::unordered_set<Node*>& replacedNodes);
  void rewriteMatch(
```
- EN: This block manipulates graph-like program structures. Key symbols: `overlapsWithUsedNodes`, `rewriteMatch`.
- CN: 该代码块操作图状程序结构。关键符号：`overlapsWithUsedNodes`, `rewriteMatch`。

### Lines 193-197
```cpp
      Graph* graph,
      const Match& match,
      const Graph& pattern,
      const Graph& replacement);

```
- EN: This block manipulates graph-like program structures. Key symbols: no dominant local symbols.
- CN: 该代码块操作图状程序结构。关键符号：无明显局部符号。

### Lines 198-200
```cpp
  c10::FastMap<std::string, const Value*> getVmap(const Graph& pattern);
};
} // namespace torch::nativert
```
- EN: This block manipulates graph-like program structures. Key symbols: `getVmap`.
- CN: 该代码块操作图状程序结构。关键符号：`getVmap`。


## Key Concepts / 关键概念
- EN: Graph capture and transformation — the implementation works with program graphs and rewrite passes.
  CN: 图捕获与变换——实现会处理程序图及其改写 pass。
- EN: Runtime composition — the file participates in assembling native runtime data flow, execution state, or backend handoff.
  CN: 运行时组合——该文件参与组织原生运行时的数据流、执行状态或后端交接。
- EN: Header-level integration — the translation unit depends on nearby runtime and framework declarations.
  CN: 头文件级集成——该翻译单元依赖周边运行时与框架声明。

## Dependencies / 依赖关系
- Internal includes / 内部头文件: `c10/util/FbcodeMaps.h`, `torch/nativert/graph/Graph.h`
- External includes / 外部头文件: `utility`
- Namespaces / 命名空间: `torch::nativert`
- Representative symbols / 代表性符号: `Match`, `MatchFilter`, `bool`, `for`, `graph`, `aaa`, `bbb`, `SubgraphMatcher`, `match`, `matchAll`, `...`
