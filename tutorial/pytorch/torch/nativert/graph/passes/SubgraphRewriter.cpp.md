# SubgraphRewriter.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `torch/nativert/graph/passes/SubgraphRewriter.cpp`
- Repository: `pytorch/pytorch`
- Purpose (EN): Implements Native Runtime runtime behavior for SubgraphRewriter, including execution, delegation, or low-level utilities.
- 用途 (CN): 实现 Native Runtime 中 runtime 子模块里与 SubgraphRewriter 相关的行为，包括执行、delegate 协调或底层工具逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-7
```cpp
#include <variant>

#include <c10/util/Exception.h>
#include <torch/nativert/graph/Graph.h>
#include <torch/nativert/graph/passes/SubgraphRewriter.h>
namespace torch::nativert {

```
- EN: This block manipulates graph-like program structures; reports or normalizes error conditions. Key symbols: no dominant local symbols.
- CN: 该代码块操作图状程序结构；报告或规范化错误情况。关键符号：无明显局部符号。

### Lines 8-16
```cpp
const std::string kDummyTarget = "dummy";

//-------------------------
// SubgraphMatcher
//-------------------------

SubgraphMatcher::SubgraphMatcher(const Graph* pattern)
    : pattern_(pattern), pattern_root_(findRootNode(pattern_)) {}

```
- EN: This block manipulates graph-like program structures. Key symbols: `SubgraphMatcher`, `pattern_`, `pattern_root_`, `findRootNode`.
- CN: 该代码块操作图状程序结构。关键符号：`SubgraphMatcher`, `pattern_`, `pattern_root_`, `findRootNode`。

### Lines 17-23
```cpp
const Node* SubgraphMatcher::findRootNode(const Graph* g) {
  return g->outputNode()->inputs()[0].value->producer();
}

std::optional<Match> SubgraphMatcher::match(Node* target_node) {
  if (!pattern_root_) {
    return std::nullopt;
```
- EN: This block manipulates graph-like program structures; handles conditional control flow; returns results to callers or downstream stages. Key symbols: `findRootNode`, `outputNode`, `inputs`, `producer`, `match`.
- CN: 该代码块操作图状程序结构；处理条件控制流；向调用方或后续阶段返回结果。关键符号：`findRootNode`, `outputNode`, `inputs`, `producer`, `match`。

### Lines 24-35
```cpp
  }

  Match current_match;
  if (tryMatchNode(pattern_root_, target_node, current_match)) {
    for (const Value* output : pattern_->outputs()) {
      TORCH_CHECK(
          current_match.value_map.find(output) != current_match.value_map.end(),
          "Not all outputs were matched to the pattern. ",
          "Please check that the first output node suffices ",
          "to traverse all output values in the pattern.");
    }
    return current_match;
```
- EN: This block checks invariants or expected outcomes; manipulates graph-like program structures; handles conditional control flow; iterates over collections or execution units; returns results to callers or downstream stages. Key symbols: `tryMatchNode`, `outputs`, `find`, `end`.
- CN: 该代码块检查不变量或预期结果；操作图状程序结构；处理条件控制流；遍历集合或执行单元；向调用方或后续阶段返回结果。关键符号：`tryMatchNode`, `outputs`, `find`, `end`。

### Lines 36-43
```cpp
  }

  return std::nullopt;
}

std::vector<Match> SubgraphMatcher::matchAll(Graph* graph) {
  std::vector<Match> matches;

```
- EN: This block manipulates graph-like program structures; returns results to callers or downstream stages. Key symbols: `matchAll`.
- CN: 该代码块操作图状程序结构；向调用方或后续阶段返回结果。关键符号：`matchAll`。

### Lines 44-50
```cpp
  for (auto& node : graph->nodes()) {
    auto maybeMatch = match(&node);
    if (maybeMatch.has_value()) {
      matches.push_back(*maybeMatch);
    }
  }
  return matches;
```
- EN: This block manipulates graph-like program structures; handles conditional control flow; iterates over collections or execution units; returns results to callers or downstream stages. Key symbols: `nodes`, `match`, `has_value`, `push_back`.
- CN: 该代码块操作图状程序结构；处理条件控制流；遍历集合或执行单元；向调用方或后续阶段返回结果。关键符号：`nodes`, `match`, `has_value`, `push_back`。

### Lines 51-59
```cpp
}

namespace {
bool compareConstants(const Constant& a, const Constant& b) {
  return std::visit(
      [](const auto& lhs, const auto& rhs) -> bool {
        using LType = std::decay_t<decltype(lhs)>;
        using RType = std::decay_t<decltype(rhs)>;

```
- EN: This block returns results to callers or downstream stages. Key symbols: `compareConstants`, `visit`, `LType`, `RType`.
- CN: 该代码块向调用方或后续阶段返回结果。关键符号：`compareConstants`, `visit`, `LType`, `RType`。

### Lines 60-73
```cpp
        // Handle directly comparable types
        if constexpr (
            std::is_same_v<LType, RType> &&
            !std::is_same_v<LType, std::unique_ptr<Graph>>) {
          return lhs == rhs;
        }
        // Unsupported types (Graph)
        LOG(ERROR) << "Unsupported Constant types for pattern matching: "
                   << typeid(lhs).name() << " vs " << typeid(rhs).name();
        TORCH_CHECK(
            false,
            "Unsupported Constant types for pattern matching: ",
            typeid(lhs).name(),
            " vs ",
```
- EN: This block checks invariants or expected outcomes; manipulates graph-like program structures; reports or normalizes error conditions; handles conditional control flow; iterates over collections or execution units; returns results to callers or downstream stages. Key symbols: `typeid`, `name`.
- CN: 该代码块检查不变量或预期结果；操作图状程序结构；报告或规范化错误情况；处理条件控制流；遍历集合或执行单元；向调用方或后续阶段返回结果。关键符号：`typeid`, `name`。

### Lines 74-80
```cpp
            typeid(rhs).name())
      },
      a,
      b);
}

auto findMatchingAttribute(const Node* target_node, const Attribute& attr) {
```
- EN: This block manipulates graph-like program structures. Key symbols: `typeid`, `name`, `findMatchingAttribute`.
- CN: 该代码块操作图状程序结构。关键符号：`typeid`, `name`, `findMatchingAttribute`。

### Lines 81-89
```cpp
  return std::find_if(
      target_node->attributes().begin(),
      target_node->attributes().end(),
      [&](const Attribute& otherAttr) {
        return attr.name == otherAttr.name &&
            compareConstants(attr.value, otherAttr.value);
      });
}

```
- EN: This block manipulates graph-like program structures; handles conditional control flow; returns results to callers or downstream stages. Key symbols: `find_if`, `attributes`, `begin`, `end`, `compareConstants`.
- CN: 该代码块操作图状程序结构；处理条件控制流；向调用方或后续阶段返回结果。关键符号：`find_if`, `attributes`, `begin`, `end`, `compareConstants`。

### Lines 90-99
```cpp
auto findInputByName(const Node* pattern_node, const std::string& inputName) {
  return std::find_if(
      pattern_node->inputs().begin(),
      pattern_node->inputs().end(),
      [&](const NamedArgument& patternInput) {
        return inputName == patternInput.name;
      });
}
} // namespace

```
- EN: This block manipulates graph-like program structures; handles conditional control flow; returns results to callers or downstream stages. Key symbols: `findInputByName`, `find_if`, `inputs`, `begin`, `end`.
- CN: 该代码块操作图状程序结构；处理条件控制流；向调用方或后续阶段返回结果。关键符号：`findInputByName`, `find_if`, `inputs`, `begin`, `end`。

### Lines 100-109
```cpp
bool SubgraphMatcher::tryMatchNodeInputs(
    const Node* pattern_node,
    Node* target_node,
    Match& match) {
  TORCH_CHECK(
      pattern_node->numInputs() + pattern_node->attributes().size() ==
      target_node->numInputs() + target_node->attributes().size());
  TORCH_CHECK(target_node->numInputs() <= pattern_node->numInputs());
  TORCH_CHECK(pattern_node->attributes().size() <= target_node->numInputs());

```
- EN: This block checks invariants or expected outcomes; manipulates graph-like program structures. Key symbols: `tryMatchNodeInputs`, `numInputs`, `attributes`, `size`.
- CN: 该代码块检查不变量或预期结果；操作图状程序结构。关键符号：`tryMatchNodeInputs`, `numInputs`, `attributes`, `size`。

### Lines 110-116
```cpp
  // Target node inputs should match pattern node inputs
  for (const auto i : c10::irange(target_node->numInputs())) {
    // Compare input values
    // Current target node input should match a pattern node input
    const auto& inputMatch =
        findInputByName(pattern_node, target_node->inputs()[i].name);
    if (inputMatch == pattern_node->inputs().end()) {
```
- EN: This block manipulates graph-like program structures; handles conditional control flow; iterates over collections or execution units. Key symbols: `irange`, `numInputs`, `findInputByName`, `inputs`, `end`.
- CN: 该代码块操作图状程序结构；处理条件控制流；遍历集合或执行单元。关键符号：`irange`, `numInputs`, `findInputByName`, `inputs`, `end`。

### Lines 117-123
```cpp
      return false;
    }

    const Value* pval = inputMatch->value;
    Value* tval = target_node->inputs()[i].value;
    if (!tryMatchValue(pval, tval, match)) {
      return false;
```
- EN: This block manipulates graph-like program structures; handles conditional control flow; returns results to callers or downstream stages. Key symbols: `inputs`, `tryMatchValue`.
- CN: 该代码块操作图状程序结构；处理条件控制流；向调用方或后续阶段返回结果。关键符号：`inputs`, `tryMatchValue`。

### Lines 124-132
```cpp
    }
  }

  // Pattern node attributes should match target node attributes
  std::unordered_set<std::string> matched_attributes;
  for (const auto i : c10::irange(pattern_node->attributes().size())) {
    // Compare attributes
    const auto& attr = pattern_node->attributes()[i];
    auto it = findMatchingAttribute(target_node, attr);
```
- EN: This block manipulates graph-like program structures; iterates over collections or execution units. Key symbols: `irange`, `attributes`, `size`, `findMatchingAttribute`.
- CN: 该代码块操作图状程序结构；遍历集合或执行单元。关键符号：`irange`, `attributes`, `size`, `findMatchingAttribute`。

### Lines 133-141
```cpp
    if (it == target_node->attributes().end()) {
      return false; // Attribute not found or values differ
    }
    matched_attributes.insert(it->name);
  }

  // Target node attributes that do not match pattern node attributes should
  // match pattern node inputs
  for (const auto i : c10::irange(target_node->attributes().size())) {
```
- EN: This block manipulates graph-like program structures; handles conditional control flow; iterates over collections or execution units; returns results to callers or downstream stages. Key symbols: `attributes`, `end`, `insert`, `irange`, `size`.
- CN: 该代码块操作图状程序结构；处理条件控制流；遍历集合或执行单元；向调用方或后续阶段返回结果。关键符号：`attributes`, `end`, `insert`, `irange`, `size`。

### Lines 142-148
```cpp
    const auto& it = target_node->attributes()[i];
    if (matched_attributes.find(it.name) != matched_attributes.end()) {
      continue; // Skip attributes already matched
    }
    const auto& patternInput = findInputByName(pattern_node, it.name);
    if (patternInput == pattern_node->inputs().end()) {
      return false;
```
- EN: This block manipulates graph-like program structures; handles conditional control flow; returns results to callers or downstream stages. Key symbols: `attributes`, `find`, `end`, `findInputByName`, `inputs`.
- CN: 该代码块操作图状程序结构；处理条件控制流；向调用方或后续阶段返回结果。关键符号：`attributes`, `find`, `end`, `findInputByName`, `inputs`。

### Lines 149-162
```cpp
    }
    if (patternInput->value->producer()->target() != "prim.Input" ||
        patternInput->value->users().size() > 1) {
      return false; // Only a pattern graph input should match a constant attr
    }

    // Insert a dummy node to match the pattern input value
    // Record the attribute that should be used to replace the dummy node
    auto* targetGraph = target_node->owningGraph();
    Node* dummyNode = targetGraph->createNode(kDummyTarget);
    Value* dummyOutput = dummyNode->addOutput(
        targetGraph->getUniqueValueName(), Type::Kind::None);
    targetGraph->insertBefore(dummyNode, target_node);
    if (match.value_map.find(patternInput->value) != match.value_map.end()) {
```
- EN: This block manipulates graph-like program structures; handles conditional control flow; returns results to callers or downstream stages. Key symbols: `producer`, `target`, `users`, `size`, `owningGraph`, `createNode`, `...`.
- CN: 该代码块操作图状程序结构；处理条件控制流；向调用方或后续阶段返回结果。关键符号：`producer`, `target`, `users`, `size`, `owningGraph`, `createNode`, `...`。

### Lines 163-169
```cpp
      return match.value_map[patternInput->value]->producer()->target() ==
          kDummyTarget;
    }
    match.value_map[patternInput->value] = dummyOutput;
    match.dummy_input_to_attribute_map[dummyOutput] = &it.value;
  }
  return true;
```
- EN: This block returns results to callers or downstream stages. Key symbols: `producer`, `target`.
- CN: 该代码块向调用方或后续阶段返回结果。关键符号：`producer`, `target`。

### Lines 170-176
```cpp
}

bool SubgraphMatcher::tryMatchNode(
    const Node* pattern_node,
    Node* target_node,
    Match& match) {
  if (match.node_map.find(pattern_node) != match.node_map.end()) {
```
- EN: This block manipulates graph-like program structures; handles conditional control flow. Key symbols: `tryMatchNode`, `find`, `end`.
- CN: 该代码块操作图状程序结构；处理条件控制流。关键符号：`tryMatchNode`, `find`, `end`。

### Lines 177-184
```cpp
    return match.node_map[pattern_node] == target_node;
  }

  // If the pattern node is an input, it should match every node
  if (pattern_node->target() == "prim.Input") {
    return true;
  }

```
- EN: This block manipulates graph-like program structures; handles conditional control flow; returns results to callers or downstream stages. Key symbols: `target`.
- CN: 该代码块操作图状程序结构；处理条件控制流；向调用方或后续阶段返回结果。关键符号：`target`。

### Lines 185-198
```cpp
  if (pattern_node->target() != target_node->target() ||
      pattern_node->numOutputs() != target_node->numOutputs()) {
    return false;
  }

  int64_t deltaInputCount = static_cast<int64_t>(pattern_node->numInputs()) -
      static_cast<int64_t>(target_node->numInputs());
  int64_t deltaAttributesCount =
      static_cast<int64_t>(pattern_node->attributes().size()) -
      static_cast<int64_t>(target_node->attributes().size());
  // Number of inputs and attributes should match exactly
  // and the pattern should always have >= input count of the target node
  // and the pattern should always have <= attribute count of the target node
  if (deltaInputCount + deltaAttributesCount != 0 ||
```
- EN: This block manipulates graph-like program structures; handles conditional control flow; returns results to callers or downstream stages. Key symbols: `target`, `numOutputs`, `numInputs`, `attributes`, `size`.
- CN: 该代码块操作图状程序结构；处理条件控制流；向调用方或后续阶段返回结果。关键符号：`target`, `numOutputs`, `numInputs`, `attributes`, `size`。

### Lines 199-207
```cpp
      (deltaInputCount < 0 && deltaAttributesCount > 0)) {
    return false;
  }
  match.node_map[pattern_node] = target_node;

  for (const auto i : c10::irange(pattern_node->numOutputs())) {
    const Value* pval = pattern_node->outputs()[i];
    Value* tval = target_node->outputs()[i];
    if (!tryMatchValue(pval, tval, match)) {
```
- EN: This block manipulates graph-like program structures; handles conditional control flow; iterates over collections or execution units; returns results to callers or downstream stages. Key symbols: `irange`, `numOutputs`, `outputs`, `tryMatchValue`.
- CN: 该代码块操作图状程序结构；处理条件控制流；遍历集合或执行单元；向调用方或后续阶段返回结果。关键符号：`irange`, `numOutputs`, `outputs`, `tryMatchValue`。

### Lines 208-214
```cpp
      return false;
    }
  }

  return tryMatchNodeInputs(pattern_node, target_node, match);
}

```
- EN: This block manipulates graph-like program structures; returns results to callers or downstream stages. Key symbols: `tryMatchNodeInputs`.
- CN: 该代码块操作图状程序结构；向调用方或后续阶段返回结果。关键符号：`tryMatchNodeInputs`。

### Lines 215-221
```cpp
bool SubgraphMatcher::isOutputValue(const Value* val) {
  for (const auto& output : pattern_->outputs()) {
    if (val == output) {
      return true;
    }
  }
  return false;
```
- EN: This block manipulates graph-like program structures; handles conditional control flow; iterates over collections or execution units; returns results to callers or downstream stages. Key symbols: `isOutputValue`, `outputs`.
- CN: 该代码块操作图状程序结构；处理条件控制流；遍历集合或执行单元；向调用方或后续阶段返回结果。关键符号：`isOutputValue`, `outputs`。

### Lines 222-228
```cpp
}

bool SubgraphMatcher::tryMatchValue(
    const Value* pval,
    Value* tval,
    Match& match) {
  if (match.value_map.find(pval) != match.value_map.end()) {
```
- EN: This block manipulates graph-like program structures; handles conditional control flow. Key symbols: `tryMatchValue`, `find`, `end`.
- CN: 该代码块操作图状程序结构；处理条件控制流。关键符号：`tryMatchValue`, `find`, `end`。

### Lines 229-237
```cpp
    return match.value_map[pval] == tval;
  }

  const Node* pProducer = pval->producer();
  Node* tProducer = tval->producer();
  // If the value in the pattern is an input, then it could have other uses
  // outside of the subgraph. Similarly, output values can also have uses
  // outside of the matching subgraph.
  if (pval->users().size() != tval->users().size() &&
```
- EN: This block manipulates graph-like program structures; handles conditional control flow; returns results to callers or downstream stages. Key symbols: `producer`, `users`, `size`.
- CN: 该代码块操作图状程序结构；处理条件控制流；向调用方或后续阶段返回结果。关键符号：`producer`, `users`, `size`。

### Lines 238-245
```cpp
      pProducer->target() != "prim.Input" && !isOutputValue(pval)) {
    return false;
  }

  if (pval->type().kind() != tval->type().kind()) {
    return false;
  }

```
- EN: This block handles conditional control flow; returns results to callers or downstream stages. Key symbols: `target`, `isOutputValue`, `type`, `kind`.
- CN: 该代码块处理条件控制流；向调用方或后续阶段返回结果。关键符号：`target`, `isOutputValue`, `type`, `kind`。

### Lines 246-254
```cpp
  match.value_map[pval] = tval;

  return tryMatchNode(pProducer, tProducer, match);
}

//-------------------------
// SubgraphRewriter
//-------------------------

```
- EN: This block manipulates graph-like program structures; returns results to callers or downstream stages. Key symbols: `tryMatchNode`.
- CN: 该代码块操作图状程序结构；向调用方或后续阶段返回结果。关键符号：`tryMatchNode`。

### Lines 255-261
```cpp
void SubgraphRewriter::registerRewritePattern(
    const std::string& pattern,
    const std::string& replacement) {
  patterns_.emplace_back(RewriteRule{pattern, replacement});
}

bool SubgraphRewriter::run(
```
- EN: This block manipulates graph-like program structures. Key symbols: `registerRewritePattern`, `emplace_back`, `run`.
- CN: 该代码块操作图状程序结构。关键符号：`registerRewritePattern`, `emplace_back`, `run`。

### Lines 262-271
```cpp
    Graph* graph,
    const std::vector<MatchFilter>& filters) {
  bool mutated = false;
  for (const auto& [pattern, replacement] : patterns_) {
    const auto& pattern_graph = stringToGraph(pattern);
    const auto& replacement_graph = stringToGraph(replacement);
    mutated |=
        runForPattern(graph, *pattern_graph, *replacement_graph, filters);
  }
  return mutated;
```
- EN: This block manipulates graph-like program structures; iterates over collections or execution units; returns results to callers or downstream stages. Key symbols: `stringToGraph`, `runForPattern`.
- CN: 该代码块操作图状程序结构；遍历集合或执行单元；向调用方或后续阶段返回结果。关键符号：`stringToGraph`, `runForPattern`。

### Lines 272-281
```cpp
}

bool SubgraphRewriter::runForPattern(
    Graph* graph,
    const Graph& pattern,
    const Graph& replacement,
    const std::vector<MatchFilter>& filters) {
  SubgraphMatcher matcher(&pattern);
  std::vector<Match> matches = matcher.matchAll(graph);

```
- EN: This block manipulates graph-like program structures. Key symbols: `runForPattern`, `matcher`, `matchAll`.
- CN: 该代码块操作图状程序结构。关键符号：`runForPattern`, `matcher`, `matchAll`。

### Lines 282-291
```cpp
  VLOG(1) << "[GraphPasses] Found " << matches.size()
          << " matches for : " << name_;

  for (auto& m : matches) {
    if (!std::all_of(filters.begin(), filters.end(), [&](const MatchFilter& f) {
          return f(m, getVmap(pattern));
        })) {
      continue;
    }
    if (!overlapsWithUsedNodes(m, replacedNodes_)) {
```
- EN: This block manipulates graph-like program structures; handles conditional control flow; iterates over collections or execution units; returns results to callers or downstream stages. Key symbols: `size`, `all_of`, `begin`, `end`, `f`, `getVmap`, `...`.
- CN: 该代码块操作图状程序结构；处理条件控制流；遍历集合或执行单元；向调用方或后续阶段返回结果。关键符号：`size`, `all_of`, `begin`, `end`, `f`, `getVmap`, `...`。

### Lines 292-299
```cpp
      rewriteMatch(graph, m, pattern, replacement);
    }
  }

  for (auto* v : valuesToRewrite_) {
    graph->replaceAllUses(v, valueRewrites_.at(v));
  }

```
- EN: This block manipulates graph-like program structures; iterates over collections or execution units. Key symbols: `rewriteMatch`, `replaceAllUses`.
- CN: 该代码块操作图状程序结构；遍历集合或执行单元。关键符号：`rewriteMatch`, `replaceAllUses`。

### Lines 300-306
```cpp
  for (auto* n : replacedNodes_) {
    for (const auto& input : n->inputs()) {
      input.value->eraseUser(n);
    }
    n->inputs().clear();
  }

```
- EN: This block manipulates graph-like program structures; iterates over collections or execution units. Key symbols: `inputs`, `eraseUser`, `clear`.
- CN: 该代码块操作图状程序结构；遍历集合或执行单元。关键符号：`inputs`, `eraseUser`, `clear`。

### Lines 307-313
```cpp
  for (auto* n : replacedNodes_) {
    n->destroy();
  }

  bool mutated = (valuesToRewrite_.size() + valueRewrites_.size() +
                  replacedNodes_.size()) > 0;

```
- EN: This block manipulates graph-like program structures; iterates over collections or execution units. Key symbols: `destroy`, `size`.
- CN: 该代码块操作图状程序结构；遍历集合或执行单元。关键符号：`destroy`, `size`。

### Lines 314-321
```cpp
  valuesToRewrite_.clear();
  valueRewrites_.clear();
  replacedNodes_.clear();

  graph->cleanupDeadNodes();
  graph->finalize();
  graph->lint();

```
- EN: This block manipulates graph-like program structures. Key symbols: `clear`, `cleanupDeadNodes`, `finalize`, `lint`.
- CN: 该代码块操作图状程序结构。关键符号：`clear`, `cleanupDeadNodes`, `finalize`, `lint`。

### Lines 322-330
```cpp
  return mutated;
}

bool SubgraphRewriter::overlapsWithUsedNodes(
    const Match& match,
    const std::unordered_set<Node*>& usedNodes) {
  // If any node or value used by this match is already in usedNodes/usedValues,
  // then this match overlaps with a previously selected match.
  for (auto& kv : match.node_map) {
```
- EN: This block manipulates graph-like program structures; handles conditional control flow; iterates over collections or execution units; returns results to callers or downstream stages. Key symbols: `overlapsWithUsedNodes`.
- CN: 该代码块操作图状程序结构；处理条件控制流；遍历集合或执行单元；向调用方或后续阶段返回结果。关键符号：`overlapsWithUsedNodes`。

### Lines 331-338
```cpp
    Node* target_node = kv.second;
    if (usedNodes.find(target_node) != usedNodes.end()) {
      return true;
    }
  }
  return false;
}

```
- EN: This block manipulates graph-like program structures; handles conditional control flow; returns results to callers or downstream stages. Key symbols: `find`, `end`.
- CN: 该代码块操作图状程序结构；处理条件控制流；向调用方或后续阶段返回结果。关键符号：`find`, `end`。

### Lines 339-346
```cpp
void SubgraphRewriter::rewriteMatch(
    Graph* graph,
    const Match& match,
    const Graph& pattern,
    const Graph& replacement) {
  // TODO: Preserve original node metadata with python source traceback
  std::unordered_map<const Value*, Value*> valueMap;

```
- EN: This block manipulates graph-like program structures. Key symbols: `rewriteMatch`.
- CN: 该代码块操作图状程序结构。关键符号：`rewriteMatch`。

### Lines 347-358
```cpp
  // Find the point at which to insert the new subgraph
  // and get pointers to input/output values to insert at
  Node* insertionPoint = nullptr;
  std::vector<Value*> inputs, outputs;
  for (Value* v : pattern.inputs()) {
    if (match.value_map.find(v) == match.value_map.end()) {
      continue;
    }
    Value* input = match.value_map.at(v);
    // We want to insert after latest producer of any input that is not a dummy
    // node
    if (!insertionPoint ||
```
- EN: This block manipulates graph-like program structures; handles conditional control flow; iterates over collections or execution units. Key symbols: `inputs`, `find`, `end`.
- CN: 该代码块操作图状程序结构；处理条件控制流；遍历集合或执行单元。关键符号：`inputs`, `find`, `end`。

### Lines 359-366
```cpp
        (insertionPoint->isBefore(input->producer()) &&
         input->producer()->target() != kDummyTarget)) {
      insertionPoint = input->producer();
    }
    inputs.push_back(input);
  }
  TORCH_CHECK(insertionPoint, "No insertion point found");

```
- EN: This block checks invariants or expected outcomes. Key symbols: `isBefore`, `producer`, `target`, `push_back`.
- CN: 该代码块检查不变量或预期结果。关键符号：`isBefore`, `producer`, `target`, `push_back`。

### Lines 367-373
```cpp
  // Check we're not inserting after any of the outputs
  bool insertionPointValid = true;
  for (const auto* v : pattern.outputs()) {
    Value* output = match.value_map.at(v);
    outputs.push_back(match.value_map.at(v));
    for (const auto* user : output->users()) {
      if (user->isBefore(insertionPoint)) {
```
- EN: This block checks invariants or expected outcomes; handles conditional control flow; iterates over collections or execution units. Key symbols: `outputs`, `push_back`, `users`, `isBefore`.
- CN: 该代码块检查不变量或预期结果；处理条件控制流；遍历集合或执行单元。关键符号：`outputs`, `push_back`, `users`, `isBefore`。

### Lines 374-385
```cpp
        insertionPointValid = false;
        break;
      }
    }
  }
  if (!insertionPointValid) {
    return;
  }
  std::vector<Value*> newOutputs;
  {
    InsertingAfter guard(insertionPoint);

```
- EN: This block handles conditional control flow; protects shared state or ordering guarantees. Key symbols: `guard`.
- CN: 该代码块处理条件控制流；保护共享状态或执行顺序保证。关键符号：`guard`。

### Lines 386-394
```cpp
    newOutputs = graph->insertGraph(replacement, inputs, valueMap);
  }
  TORCH_CHECK(outputs.size() == newOutputs.size());

  for (auto i : c10::irange(outputs.size())) {
    valuesToRewrite_.push_back(outputs[i]);
    valueRewrites_[outputs[i]] = newOutputs[i];
  }

```
- EN: This block checks invariants or expected outcomes; manipulates graph-like program structures; iterates over collections or execution units. Key symbols: `insertGraph`, `size`, `irange`, `push_back`.
- CN: 该代码块检查不变量或预期结果；操作图状程序结构；遍历集合或执行单元。关键符号：`insertGraph`, `size`, `irange`, `push_back`。

### Lines 395-401
```cpp
  for (auto& patternNode : pattern.nodes()) {
    if (match.node_map.find(&patternNode) != match.node_map.end()) {
      Node* n = match.node_map.at(&patternNode);
      replacedNodes_.insert(n);
    }
  }

```
- EN: This block manipulates graph-like program structures; handles conditional control flow; iterates over collections or execution units. Key symbols: `nodes`, `find`, `end`, `insert`.
- CN: 该代码块操作图状程序结构；处理条件控制流；遍历集合或执行单元。关键符号：`nodes`, `find`, `end`, `insert`。

### Lines 402-410
```cpp
  // Replace dummy values with constant attributes
  for (const auto& inputToAttr : match.dummy_input_to_attribute_map) {
    auto* dummy = inputToAttr.first;
    // dummy might not be used in rewritten graph
    // e.g., casted_batch_one_hot_lengths
    if (dummy->users().empty()) {
      continue;
    }

```
- EN: This block manipulates graph-like program structures; handles conditional control flow; iterates over collections or execution units. Key symbols: `users`, `empty`.
- CN: 该代码块操作图状程序结构；处理条件控制流；遍历集合或执行单元。关键符号：`users`, `empty`。

### Lines 411-420
```cpp
    for (auto& userNode : dummy->users()) {
      auto& userInputs = userNode->inputs();
      replacedNodes_.insert(dummy->producer());
      for (auto it = userInputs.begin(); it != userInputs.end(); ++it) {
        if (it->value == dummy) {
          Attribute newAttr;
          std::visit(
              [&](auto&& val) -> void {
                using T = std::decay_t<decltype(val)>;
                if constexpr (std::is_same_v<T, std::unique_ptr<Graph>>) {
```
- EN: This block manipulates graph-like program structures; handles conditional control flow; iterates over collections or execution units. Key symbols: `users`, `inputs`, `insert`, `producer`, `begin`, `end`, `...`.
- CN: 该代码块操作图状程序结构；处理条件控制流；遍历集合或执行单元。关键符号：`users`, `inputs`, `insert`, `producer`, `begin`, `end`, `...`。

### Lines 421-434
```cpp
                  LOG(ERROR)
                      << "Graph attributes are not supported yet. Skipping attribute";
                } else {
                  newAttr.value = val;
                }
              },
              *inputToAttr.second);
          newAttr.name = it->name;
          userNode->addAttribute(std::move(newAttr));
          dummy->eraseUser(userNode);
          userInputs.erase(it);
          break;
        }
      }
```
- EN: This block manipulates graph-like program structures; reports or normalizes error conditions. Key symbols: `addAttribute`, `move`, `eraseUser`, `erase`.
- CN: 该代码块操作图状程序结构；报告或规范化错误情况。关键符号：`addAttribute`, `move`, `eraseUser`, `erase`。

### Lines 435-442
```cpp
    }
  }
}

c10::FastMap<std::string, const Value*> SubgraphRewriter::getVmap(
    const Graph& pattern) {
  c10::FastMap<std::string, const Value*> vmap;
  for (const auto& v : pattern.inputs()) {
```
- EN: This block manipulates graph-like program structures; iterates over collections or execution units. Key symbols: `getVmap`, `inputs`.
- CN: 该代码块操作图状程序结构；遍历集合或执行单元。关键符号：`getVmap`, `inputs`。

### Lines 443-450
```cpp
    vmap[std::string(v->name())] = v;
  }
  for (const auto& n : pattern.nodes()) {
    for (const Value* v : n.outputs()) {
      vmap[std::string(v->name())] = v;
    }
  }
  return vmap;
```
- EN: This block manipulates graph-like program structures; iterates over collections or execution units; returns results to callers or downstream stages. Key symbols: `string`, `name`, `nodes`, `outputs`.
- CN: 该代码块操作图状程序结构；遍历集合或执行单元；向调用方或后续阶段返回结果。关键符号：`string`, `name`, `nodes`, `outputs`。

### Lines 451-452
```cpp
}
} // namespace torch::nativert
```
- EN: This block implements local helper logic for SubgraphRewriter. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 SubgraphRewriter 相关的局部辅助逻辑。关键符号：无明显局部符号。


## Key Concepts / 关键概念
- EN: Reusable test infrastructure — the module provides shared helpers, fixtures, or data for many test suites.
  CN: 可复用测试基础设施——该模块为多个测试套件提供共享辅助函数、夹具或数据。
- EN: Graph capture and transformation — the implementation works with program graphs and rewrite passes.
  CN: 图捕获与变换——实现会处理程序图及其改写 pass。
- EN: Runtime composition — the file participates in assembling native runtime data flow, execution state, or backend handoff.
  CN: 运行时组合——该文件参与组织原生运行时的数据流、执行状态或后端交接。
- EN: Header-level integration — the translation unit depends on nearby runtime and framework declarations.
  CN: 头文件级集成——该翻译单元依赖周边运行时与框架声明。

## Dependencies / 依赖关系
- Internal includes / 内部头文件: `c10/util/Exception.h`, `torch/nativert/graph/Graph.h`, `torch/nativert/graph/passes/SubgraphRewriter.h`
- External includes / 外部头文件: `variant`
- Namespaces / 命名空间: `torch::nativert`
- Representative symbols / 代表性符号: `SubgraphMatcher`, `pattern_`, `pattern_root_`, `findRootNode`, `outputNode`, `inputs`, `producer`, `match`, `tryMatchNode`, `outputs`, `...`
