# function_extraction.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `torch/csrc/jit/passes/onnx/function_extraction.cpp`
- Repository: `pytorch`
- Purpose (EN): Implements the TorchScript JIT pass logic for function extraction, including graph analysis and rewrites.
- 用途 (CN): 实现与 function extraction 相关的 TorchScript JIT pass 逻辑，包括图分析与改写。

## Line-by-Line Analysis / 逐行分析
### Lines 1-10
```cpp
#include <torch/csrc/jit/jit_log.h>
#include <torch/csrc/jit/passes/onnx/function_extraction.h>
#include <torch/csrc/jit/passes/onnx/naming.h>

namespace torch::jit::onnx {

namespace {

using scope_list = std::vector<ScopePtr>;

```
- EN: This block implements local helper logic for function extraction. Key symbols: `scope_list`.
- CN: 该代码块实现与 function extraction 相关的局部辅助逻辑。关键符号：`scope_list`。

### Lines 11-19
```cpp
// Annotated attributes retrieved from module by inspecting module annotations.
// These attributes are not used inside the subgraph of ONNX local function
// because they are not created by PyTorch JIT tracing, but they may be used by
// consumers to determine whether or not to replace the function with a
// particular fused kernel.
static std::unordered_map<ScopePtr, Node*> scope_attr_map_;
static std::shared_ptr<Graph> scope_attr_graph_ = std::make_shared<Graph>();

static bool HasSameAttribute(
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `HasSameAttribute`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`HasSameAttribute`。

### Lines 20-34
```cpp
    const Node* a,
    const Node* b,
    const c10::Symbol& attr);

struct FunctionExtractor {
 public:
  FunctionExtractor(
      std::shared_ptr<Graph>& graph,
      const std::unordered_set<std::string>& module_names,
      const std::vector<std::string>& param_names)
      : graph_(graph),
        module_names_(module_names.begin(), module_names.end()),
        param_names_(param_names.begin(), param_names.end()) {}
  NodeAttrNameMap run();

```
- EN: Declares core types or data containers for this file. Prominent symbols: `FunctionExtractor`, `graph_`, `module_names_`, `begin`, `end`, `param_names_`, `...`.
- CN: 声明该文件使用的核心类型或数据容器。主要符号：`FunctionExtractor`, `graph_`, `module_names_`, `begin`, `end`, `param_names_`, `...`。

### Lines 35-43
```cpp
 private:
  struct ScopeContext {
    std::unordered_set<ScopePtr> children_;
    ScopePtr scope_;
    node_list nlist_;
    value_list inputs_;
    value_list outputs_;
    std::unordered_map<Value*, Value*> env_to_subgraph_;

```
- EN: Declares core types or data containers for this file. Prominent symbols: `ScopeContext`.
- CN: 声明该文件使用的核心类型或数据容器。主要符号：`ScopeContext`。

### Lines 44-52
```cpp
    void PopulateInputsOutputs(
        const std::unordered_set<std::string>& param_names);
    bool IsIdenticalFunction(const ScopeContext& other_ctx) const;
  };

  using ScopeCtxPtr = ScopeContext*;
  using scope_ctx_map = std::unordered_map<ScopePtr, ScopeCtxPtr>;

  struct FunctionContext {
```
- EN: Declares core types or data containers for this file. Prominent symbols: `PopulateInputsOutputs`, `IsIdenticalFunction`, `ScopeCtxPtr`, `scope_ctx_map`, `FunctionContext`.
- CN: 声明该文件使用的核心类型或数据容器。主要符号：`PopulateInputsOutputs`, `IsIdenticalFunction`, `ScopeCtxPtr`, `scope_ctx_map`, `FunctionContext`。

### Lines 53-61
```cpp
    FunctionContext(
        ScopePtr key,
        const scope_list& scopes,
        scope_ctx_map& scope_ctxs);
    void DebugPrint() const;
    void SetAttrName(Node* ref_n, Symbol attr, const std::string& name);
    std::optional<std::string> FindAttrName(Node* ref_n, Symbol attr);
    std::optional<std::string> FindAttrName(Node* ref_const_n);

```
- EN: This block manipulates TorchScript IR objects. Key symbols: `FunctionContext`, `DebugPrint`, `SetAttrName`, `FindAttrName`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`FunctionContext`, `DebugPrint`, `SetAttrName`, `FindAttrName`。

### Lines 62-71
```cpp
    ScopePtr scope_key_;
    scope_ctx_map scope_ctxs_;
    std::unordered_map<
        Node*,
        std::unordered_map<Symbol, std::unordered_set<Node*>>>
        attribute_map_;

    // Passed later to serialization.
    NodeAttrNameMap node_attr_to_name_;
  };
```
- EN: This block manipulates TorchScript IR objects. Key symbols: no dominant local symbol names.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：无明显局部符号。

### Lines 72-80
```cpp

  using FunctionCtxPtr = FunctionContext*;
  using func_ctx_map = std::unordered_map<ScopePtr, FunctionCtxPtr>;

  static bool IsValidScope(const ScopePtr& s);
  static std::optional<ScopePtr> InferScope(Node* n);
  static bool IsAncestor(const ScopePtr& parent, ScopePtr child);
  static std::optional<ScopePtr> FindCommonAncestor(ScopePtr a, ScopePtr b);
  static std::optional<ScopePtr> FindCommonAncestor(const scope_list& scopes);
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `FunctionCtxPtr`, `func_ctx_map`, `IsValidScope`, `InferScope`, `IsAncestor`, `FindCommonAncestor`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`FunctionCtxPtr`, `func_ctx_map`, `IsValidScope`, `InferScope`, `IsAncestor`, `FindCommonAncestor`。

### Lines 81-89
```cpp
  std::shared_ptr<Graph> ConstructFuncGraph(FunctionContext& ctx);

  void ConvertScopeToFunction(
      const ScopePtr& scope_key,
      const scope_list& scope_list,
      scope_ctx_map& scope_ctxs,
      const std::shared_ptr<Graph>& graph);

  static void HandleNoScopeNodes(
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `ConstructFuncGraph`, `ConvertScopeToFunction`, `HandleNoScopeNodes`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`ConstructFuncGraph`, `ConvertScopeToFunction`, `HandleNoScopeNodes`。

### Lines 90-98
```cpp
      scope_ctx_map& /*scope_ctxs*/,
      const node_list& no_scope_nlist);
  std::tuple<scope_ctx_map, node_list> PartitionNodesByScope(Block* b);
  scope_ctx_map PartitionNodesByScope(const std::shared_ptr<Graph>& graph);
  static std::unordered_map<ScopePtr, scope_list> PartitionIdenticalScopes(
      scope_ctx_map& scope_ctxs);
  static scope_list SortScopesByMaxDepth(
      std::unordered_map<ScopePtr, scope_list>& /*identical_scope_map*/);
  Node* CreateFunctionDefNode(
```
- EN: This block manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `PartitionNodesByScope`, `PartitionIdenticalScopes`, `SortScopesByMaxDepth`, `CreateFunctionDefNode`.
- CN: 该代码块操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`PartitionNodesByScope`, `PartitionIdenticalScopes`, `SortScopesByMaxDepth`, `CreateFunctionDefNode`。

### Lines 99-109
```cpp
      FunctionContext& func_ctx,
      const std::shared_ptr<Graph>& graph,
      const std::string& domain_name,
      const std::string& func_name);
  Node* CreateFunctionNode(
      FunctionContext& func_ctx,
      ScopeContext& scope_ctx,
      const std::shared_ptr<Graph>& graph,
      const std::string& domain_name,
      const std::string& func_name);

```
- EN: This block manipulates TorchScript IR objects. Key symbols: `CreateFunctionNode`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`CreateFunctionNode`。

### Lines 110-119
```cpp
  static void DebugPrintScopeContexts(const scope_ctx_map& /*scope_ctxs*/);
  static void DebugPrintGraphWithFunction(const std::shared_ptr<Graph>& g);
  static void DebugPrintConstantDiff(const FunctionContext&);

  std::shared_ptr<Graph> graph_;
  std::unordered_set<std::string> module_names_;
  std::unordered_set<std::string> param_names_;
  // Track modules with same module name that are exported as different onnx
  // local functions.
  std::unordered_map<std::string, int> module_variant_count_;
```
- EN: This block manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `DebugPrintScopeContexts`, `DebugPrintGraphWithFunction`, `DebugPrintConstantDiff`.
- CN: 该代码块操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`DebugPrintScopeContexts`, `DebugPrintGraphWithFunction`, `DebugPrintConstantDiff`。

### Lines 120-137
```cpp
  func_ctx_map func_ctxs_;
};

FunctionExtractor::FunctionContext::FunctionContext(
    ScopePtr key,
    const scope_list& scopes,
    scope_ctx_map& scope_ctxs)
    : scope_key_(std::move(key)) {
  GRAPH_UPDATE(
      "Process function context for scope ",
      scope_key_->name().toDisplayString());
  TORCH_INTERNAL_ASSERT(!scopes.empty());
  const auto& ref_ctx = scope_ctxs[scope_key_];
  // NOTE: Function scopes must have same number and order of nodes.
  GRAPH_DEBUG(
      "Initialized function context for scope ",
      scope_key_->name().toDisplayString());

```
- EN: This block manipulates TorchScript IR objects. Key symbols: `FunctionContext`, `scope_key_`, `move`, `name`, `toDisplayString`, `empty`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`FunctionContext`, `scope_key_`, `move`, `name`, `toDisplayString`, `empty`。

### Lines 138-147
```cpp
  for (const auto& scope : scopes) {
    GRAPH_DEBUG(
        "Process function context for scope ", scope->name().toDisplayString());
    TORCH_INTERNAL_ASSERT(scope_ctxs.find(scope) != scope_ctxs.end());
    scope_ctxs_[scope] = scope_ctxs[scope];
    if (scope_key_ == scope) {
      continue;
    }
    auto& scope_ctx = scope_ctxs[scope];

```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `name`, `toDisplayString`, `find`, `end`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`name`, `toDisplayString`, `find`, `end`。

### Lines 148-156
```cpp
    const auto& ns_a = ref_ctx->nlist_;
    const auto& ns_b = scope_ctx->nlist_;
    TORCH_INTERNAL_ASSERT(ns_a.size() == ns_b.size());

    GRAPH_DEBUG("Process nodes of scope ", scope->name().toDisplayString());
    for (const auto i : c10::irange(ns_a.size())) {
      TORCH_INTERNAL_ASSERT(ns_a[i]->kind() == ns_b[i]->kind());
      auto n_a = ns_a[i];
      auto n_b = ns_b[i];
```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `size`, `name`, `toDisplayString`, `irange`, `kind`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`size`, `name`, `toDisplayString`, `irange`, `kind`。

### Lines 157-168
```cpp
      std::vector<c10::Symbol> diff_attrs;
      std::vector<c10::Symbol> same_attrs;
      auto n_a_attr_names = n_a->attributeNames();
      auto n_b_attr_names = n_b->attributeNames();
      std::sort(n_a_attr_names.begin(), n_a_attr_names.end());
      std::sort(n_b_attr_names.begin(), n_b_attr_names.end());
      std::set_difference(
          n_a_attr_names.begin(),
          n_a_attr_names.end(),
          n_b_attr_names.begin(),
          n_b_attr_names.end(),
          std::inserter(diff_attrs, diff_attrs.begin()));
```
- EN: This block implements local helper logic for function extraction. Key symbols: `attributeNames`, `sort`, `begin`, `end`, `set_difference`, `inserter`.
- CN: 该代码块实现与 function extraction 相关的局部辅助逻辑。关键符号：`attributeNames`, `sort`, `begin`, `end`, `set_difference`, `inserter`。

### Lines 169-178
```cpp
      std::set_intersection(
          n_a_attr_names.begin(),
          n_a_attr_names.end(),
          n_b_attr_names.begin(),
          n_b_attr_names.end(),
          std::inserter(same_attrs, same_attrs.begin()));
      for (auto attr_name : diff_attrs) {
        attribute_map_[n_a][attr_name].insert(n_b);
      }

```
- EN: This block iterates over collections or graph structures. Key symbols: `set_intersection`, `begin`, `end`, `inserter`, `insert`.
- CN: 该代码块遍历集合或图结构。关键符号：`set_intersection`, `begin`, `end`, `inserter`, `insert`。

### Lines 179-187
```cpp
      for (auto attr_name : same_attrs) {
        if (!HasSameAttribute(n_a, n_b, attr_name)) {
          attribute_map_[n_a][attr_name].insert(n_b);
        }
      }
    }
    GRAPH_DEBUG("Process scope complete. ", scope->name().toDisplayString());
  }

```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `HasSameAttribute`, `insert`, `name`, `toDisplayString`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`HasSameAttribute`, `insert`, `name`, `toDisplayString`。

### Lines 188-196
```cpp
  GRAPH_DEBUG(
      "Process function context complete. ",
      scope_key_->name().toDisplayString());
  DebugPrint();
}

void FunctionExtractor::FunctionContext::DebugPrint() const {
  GRAPH_DEBUG("Scope name: ", scope_key_->name().toDisplayString());

```
- EN: This block manipulates TorchScript IR objects. Key symbols: `name`, `toDisplayString`, `DebugPrint`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`name`, `toDisplayString`, `DebugPrint`。

### Lines 197-209
```cpp
  for (const auto& it : attribute_map_) {
    for (const auto& attr_it : it.second) {
      GRAPH_DEBUG(
          "Attribute value difference for attribute ",
          attr_it.first.toDisplayString());
      GRAPH_DEBUG(*it.first);
      for (auto n : attr_it.second) {
        GRAPH_DEBUG(*n);
      }
    }
  }
}

```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `toDisplayString`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`toDisplayString`。

### Lines 210-221
```cpp
void FunctionExtractor::FunctionContext::SetAttrName(
    Node* ref_n,
    Symbol attr,
    const std::string& name) {
  auto v_it =
      scope_ctxs_[scope_key_]->env_to_subgraph_.find(ref_n->outputs().at(0));
  TORCH_INTERNAL_ASSERT(
      v_it != scope_ctxs_[scope_key_]->env_to_subgraph_.end());
  auto* n_in_def = v_it->second->node();
  node_attr_to_name_[n_in_def][attr.toUnqualString()] = name;
}

```
- EN: This block manipulates TorchScript IR objects. Key symbols: `SetAttrName`, `find`, `outputs`, `end`, `node`, `toUnqualString`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`SetAttrName`, `find`, `outputs`, `end`, `node`, `toUnqualString`。

### Lines 222-231
```cpp
std::optional<std::string> FunctionExtractor::FunctionContext::FindAttrName(
    Node* ref_n,
    Symbol attr) {
  auto v_it =
      scope_ctxs_[scope_key_]->env_to_subgraph_.find(ref_n->outputs().at(0));
  if (v_it == scope_ctxs_[scope_key_]->env_to_subgraph_.end()) {
    return std::nullopt;
  }
  auto* n_in_def = v_it->second->node();
  auto n_attr_it = node_attr_to_name_.find(n_in_def);
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `FindAttrName`, `find`, `outputs`, `end`, `node`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`FindAttrName`, `find`, `outputs`, `end`, `node`。

### Lines 232-241
```cpp
  if (n_attr_it == node_attr_to_name_.end()) {
    return std::nullopt;
  }
  auto name_it = n_attr_it->second.find(attr.toUnqualString());
  if (name_it == n_attr_it->second.end()) {
    return std::nullopt;
  }
  return name_it->second;
}

```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `end`, `find`, `toUnqualString`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`end`, `find`, `toUnqualString`。

### Lines 242-251
```cpp
void FunctionExtractor::DebugPrintScopeContexts(
    const scope_ctx_map& scope_ctxs) {
  for (auto& it : scope_ctxs) {
    GRAPH_UPDATE(
        "Scope name: ",
        it.first->namesFromRoot(),
        " ",
        it.first->name().toDisplayString());
    GRAPH_UPDATE("Children scopes: ", [&]() {
      std::stringstream ss;
```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `DebugPrintScopeContexts`, `namesFromRoot`, `name`, `toDisplayString`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`DebugPrintScopeContexts`, `namesFromRoot`, `name`, `toDisplayString`。

### Lines 252-262
```cpp
      for (const auto& child_scope : it.second->children_) {
        ss << child_scope->name().toDisplayString() << ' ';
      }
      return ss.str();
    }());
    GRAPH_UPDATE("Node types: \n", [&]() {
      std::stringstream ss;
      for (auto n : it.second->nlist_) {
        ss << "  " << *n;
      }
      return ss.str();
```
- EN: This block iterates over collections or graph structures; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `name`, `toDisplayString`, `str`.
- CN: 该代码块遍历集合或图结构；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`name`, `toDisplayString`, `str`。

### Lines 263-271
```cpp
    }());
    GRAPH_UPDATE("Node count: ", it.second->nlist_.size());
  }
}

void FunctionExtractor::DebugPrintGraphWithFunction(
    const std::shared_ptr<Graph>& g) {
  GRAPH_UPDATE("Local function definitions:");
  for (auto* n : g->nodes()) {
```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `size`, `DebugPrintGraphWithFunction`, `nodes`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`size`, `DebugPrintGraphWithFunction`, `nodes`。

### Lines 272-281
```cpp
    if (n->kind() == Symbol::onnx("LocalFunctionDef")) {
      GRAPH_UPDATE(
          n->s(attr::name),
          " graph: ",
          n->g(Symbol::attr("graph"))->toString());
    }
  }
  GRAPH_UPDATE("Main graph: ", g->toString());
}

```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `kind`, `onnx`, `s`, `g`, `attr`, `toString`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`kind`, `onnx`, `s`, `g`, `attr`, `toString`。

### Lines 282-293
```cpp
bool FunctionExtractor::IsValidScope(const ScopePtr& s) {
  return !s->isRoot() && !s->isBlank();
}

bool FunctionExtractor::IsAncestor(const ScopePtr& parent, ScopePtr child) {
  if (!IsValidScope(parent) || !IsValidScope(child) ||
      parent->getDepth() >= child->getDepth()) {
    return false;
  }
  do {
    child = child->parent();
    if (parent == child) {
```
- EN: This block handles conditional branches; produces a result or forwards a computed value. Key symbols: `IsValidScope`, `isRoot`, `isBlank`, `IsAncestor`, `getDepth`, `parent`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值。关键符号：`IsValidScope`, `isRoot`, `isBlank`, `IsAncestor`, `getDepth`, `parent`。

### Lines 294-303
```cpp
      return true;
    }
  } while (IsValidScope(child));
  return false;
}

std::optional<ScopePtr> FunctionExtractor::FindCommonAncestor(
    ScopePtr a,
    ScopePtr b) {
  if (!IsValidScope(a) || !IsValidScope(b)) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; produces a result or forwards a computed value. Key symbols: `IsValidScope`, `FindCommonAncestor`.
- CN: 该代码块处理条件分支；遍历集合或图结构；返回结果或转发已计算的值。关键符号：`IsValidScope`, `FindCommonAncestor`。

### Lines 304-313
```cpp
    return std::nullopt;
  }

  auto diff =
      static_cast<int64_t>(a->getDepth()) - static_cast<int64_t>(b->getDepth());
  if (diff != 0) {
    auto deeper_scope = diff > 0 ? a : b;
    auto other_scope = diff > 0 ? b : a;
    diff = std::abs(diff);
    while (diff > 0) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; produces a result or forwards a computed value. Key symbols: `getDepth`, `abs`.
- CN: 该代码块处理条件分支；遍历集合或图结构；返回结果或转发已计算的值。关键符号：`getDepth`, `abs`。

### Lines 314-322
```cpp
      deeper_scope = deeper_scope->parent();
      diff--;
    }
    a = deeper_scope;
    b = other_scope;
  }

  while (IsValidScope(a) && IsValidScope(b)) {
    if (a == b) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures. Key symbols: `parent`, `IsValidScope`.
- CN: 该代码块处理条件分支；遍历集合或图结构。关键符号：`parent`, `IsValidScope`。

### Lines 323-332
```cpp
      return a;
    } else {
      a = a->parent();
      b = b->parent();
    }
  }

  return std::nullopt;
}

```
- EN: This block produces a result or forwards a computed value. Key symbols: `parent`.
- CN: 该代码块返回结果或转发已计算的值。关键符号：`parent`。

### Lines 333-342
```cpp
std::optional<ScopePtr> FunctionExtractor::FindCommonAncestor(
    const scope_list& scopes) {
  if (scopes.empty()) {
    return std::nullopt;
  }

  std::optional<ScopePtr> common_ancestor = scopes.at(0);
  for (const auto& scope : scopes) {
    common_ancestor = FindCommonAncestor(common_ancestor.value(), scope);
    if (!common_ancestor.has_value()) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `FindCommonAncestor`, `empty`, `value`, `has_value`.
- CN: 该代码块处理条件分支；遍历集合或图结构；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`FindCommonAncestor`, `empty`, `value`, `has_value`。

### Lines 343-360
```cpp
      return std::nullopt;
    }
  }

  return common_ancestor;
}

std::optional<ScopePtr> FunctionExtractor::InferScope(Node* n) {
  // The scope of node n is assigned based on the following rules.
  // 1. If all uses of outputs of n belongs to the same scope,
  //    assign that scope, otherwise
  // 2. If all nodes of inputs of n belongs to the same scope,
  //    assign that scope, otherwise
  // 3. Find common ancestor of the scopes of uses of outputs of n,
  //    and the scopes of nodes of inputs of n.
  scope_list input_scopes;
  scope_list output_scopes;
  for (auto input : n->inputs()) {
```
- EN: This block iterates over collections or graph structures; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `InferScope`, `inputs`.
- CN: 该代码块遍历集合或图结构；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`InferScope`, `inputs`。

### Lines 361-375
```cpp
    input_scopes.emplace_back(input->node()->scope());
  }
  for (auto output : n->outputs()) {
    for (auto use : output->uses()) {
      if (!IsValidScope(use.user->scope())) {
        auto inferred_output_scope = InferScope(use.user);
        if (inferred_output_scope.has_value() &&
            IsValidScope(inferred_output_scope.value())) {
          use.user->setScope(inferred_output_scope.value());
        }
      }
      output_scopes.emplace_back(use.user->scope());
    }
  }
  if (!output_scopes.empty() &&
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `emplace_back`, `node`, `scope`, `outputs`, `uses`, `IsValidScope`, `...`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`emplace_back`, `node`, `scope`, `outputs`, `uses`, `IsValidScope`, `...`。

### Lines 376-385
```cpp
      std::all_of(
          output_scopes.begin(),
          output_scopes.end(),
          [&output_scopes](const ScopePtr& scope) -> bool {
            return IsValidScope(scope) && scope == output_scopes.at(0);
          })) {
    return output_scopes.at(0);
  } else if (
      !input_scopes.empty() &&
      std::all_of(
```
- EN: This block handles conditional branches; produces a result or forwards a computed value. Key symbols: `all_of`, `begin`, `end`, `IsValidScope`, `empty`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值。关键符号：`all_of`, `begin`, `end`, `IsValidScope`, `empty`。

### Lines 386-394
```cpp
          input_scopes.begin(),
          input_scopes.end(),
          [&input_scopes](const ScopePtr& scope) -> bool {
            return IsValidScope(scope) && scope == input_scopes.at(0);
          })) {
    return input_scopes.at(0);
  } else {
    scope_list scopes;
    std::copy_if(
```
- EN: This block handles conditional branches; produces a result or forwards a computed value. Key symbols: `begin`, `end`, `IsValidScope`, `copy_if`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值。关键符号：`begin`, `end`, `IsValidScope`, `copy_if`。

### Lines 395-404
```cpp
        input_scopes.begin(),
        input_scopes.end(),
        std::back_inserter(scopes),
        IsValidScope);
    std::copy_if(
        output_scopes.begin(),
        output_scopes.end(),
        std::back_inserter(scopes),
        IsValidScope);
    if (!scopes.empty()) {
```
- EN: This block handles conditional branches. Key symbols: `begin`, `end`, `back_inserter`, `copy_if`, `empty`.
- CN: 该代码块处理条件分支。关键符号：`begin`, `end`, `back_inserter`, `copy_if`, `empty`。

### Lines 405-413
```cpp
      auto common_ancestor = FindCommonAncestor(scopes);
      if (common_ancestor.has_value() &&
          IsValidScope(common_ancestor.value())) {
        return common_ancestor;
      }
    }
  }

  return std::nullopt;
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `FindCommonAncestor`, `has_value`, `IsValidScope`, `value`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`FindCommonAncestor`, `has_value`, `IsValidScope`, `value`。

### Lines 414-422
```cpp
}

std::shared_ptr<Graph> FunctionExtractor::ConstructFuncGraph(
    FunctionContext& func_ctx) {
  auto& ctx = *func_ctx.scope_ctxs_[func_ctx.scope_key_];
  const auto& nlist = ctx.nlist_;
  const auto& scope = ctx.scope_;
  auto& env = ctx.env_to_subgraph_;

```
- EN: This block manipulates TorchScript IR objects. Key symbols: `ConstructFuncGraph`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`ConstructFuncGraph`。

### Lines 423-432
```cpp
  auto g = std::make_shared<Graph>();
  GRAPH_DEBUG("Constructing graph for ", scope->namesFromRoot());

  // TODO: Update input names of function to match those in Module source code
  // signature.
  // This requires mapping between function node inputs and Module inputs.
  // Due to the lack of such mapping, currently debugName is used as input
  // names.
  ctx.PopulateInputsOutputs(param_names_);
  for (auto* v : ctx.inputs_) {
```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `namesFromRoot`, `PopulateInputsOutputs`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`namesFromRoot`, `PopulateInputsOutputs`。

### Lines 433-442
```cpp
    env[v] = g->addInput()->copyMetadata(v);
    GRAPH_DEBUG(
        "Add input value ",
        env[v]->debugName(),
        " for outer scope value ",
        v->debugName(),
        " from ",
        *v->node());
  }

```
- EN: This block manipulates TorchScript IR objects. Key symbols: `addInput`, `copyMetadata`, `debugName`, `node`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`addInput`, `copyMetadata`, `debugName`, `node`。

### Lines 443-453
```cpp
  for (auto* n : nlist) {
    auto clone_n = g->createClone(n, [&](Value* v) {
      TORCH_INTERNAL_ASSERT(env.find(v) != env.end());
      return env[v];
    });
    for (const auto i : c10::irange(clone_n->outputs().size())) {
      env[n->output(i)] = clone_n->output(i);
    }
    g->insertNode(clone_n);
  }

```
- EN: This block iterates over collections or graph structures; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `createClone`, `find`, `end`, `irange`, `outputs`, `size`, `...`.
- CN: 该代码块遍历集合或图结构；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`createClone`, `find`, `end`, `irange`, `outputs`, `size`, `...`。

### Lines 454-463
```cpp
  // If values are used outside of this graph, set as graph output.
  for (auto* v : ctx.outputs_) {
    TORCH_INTERNAL_ASSERT(env.find(v) != env.end());
    g->registerOutput(env[v]);
  }

  GRAPH_DEBUG(g->toString());
  return g;
}

```
- EN: This block iterates over collections or graph structures; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `find`, `end`, `registerOutput`, `toString`.
- CN: 该代码块遍历集合或图结构；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`find`, `end`, `registerOutput`, `toString`。

### Lines 464-473
```cpp
Node* FunctionExtractor::CreateFunctionDefNode(
    FunctionContext& func_ctx,
    const std::shared_ptr<Graph>& graph,
    const std::string& domain_name,
    const std::string& func_name) {
  const auto func_def_nk = Symbol::onnx("LocalFunctionDef");
  const auto func_g_attr = Symbol::attr("graph");
  const auto func_name_attr = attr::name;
  const auto func_domain_attr = Symbol::attr("domain");

```
- EN: This block manipulates TorchScript IR objects. Key symbols: `CreateFunctionDefNode`, `onnx`, `attr`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`CreateFunctionDefNode`, `onnx`, `attr`。

### Lines 474-482
```cpp
  auto func_graph = ConstructFuncGraph(func_ctx);

  // create and insert local function definition node
  auto func_def_n = graph->create(func_def_nk, 0);
  func_def_n->g_(func_g_attr, func_graph);
  func_def_n->s_(func_name_attr, func_name);
  func_def_n->s_(func_domain_attr, domain_name);
  graph->prependNode(func_def_n);

```
- EN: This block manipulates TorchScript IR objects. Key symbols: `ConstructFuncGraph`, `create`, `g_`, `s_`, `prependNode`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`ConstructFuncGraph`, `create`, `g_`, `s_`, `prependNode`。

### Lines 483-494
```cpp
  // set constants and attributes of different values as function attributes.
  std::unordered_map<std::string, int> base_attr_name_count;
  std::vector<std::string> final_attr_names;

  auto adjust_attr_name = [&](std::string attr_name) {
    if (base_attr_name_count.find(attr_name) != base_attr_name_count.end()) {
      attr_name =
          attr_name + "." + std::to_string(base_attr_name_count[attr_name]++);
    } else {
      base_attr_name_count[attr_name] = 1;
    }
    return attr_name;
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `find`, `end`, `to_string`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`find`, `end`, `to_string`。

### Lines 495-504
```cpp
  };

  for (const auto& n_it : func_ctx.attribute_map_) {
    auto* n = n_it.first;
    for (const auto& attr_it : n_it.second) {
      const auto& attr = attr_it.first;
      // Add prefix "inferred::" to name of inferred attribute.
      // This is to differentiate from annotated attributes picked up
      // from python module annotation.
      auto attr_name = "inferred::" + std::string(n->kind().toUnqualString()) +
```
- EN: This block iterates over collections or graph structures. Key symbols: `string`, `kind`, `toUnqualString`.
- CN: 该代码块遍历集合或图结构。关键符号：`string`, `kind`, `toUnqualString`。

### Lines 505-513
```cpp
          '_' + attr.toUnqualString();
      auto final_attr_name = adjust_attr_name(attr_name);
      final_attr_names.emplace_back(final_attr_name);
      func_ctx.SetAttrName(n, attr, final_attr_name);
    }
  }

  // Set annotated attributes
  std::unordered_set<Symbol> annotated_attr_names;
```
- EN: This block implements local helper logic for function extraction. Key symbols: `toUnqualString`, `adjust_attr_name`, `emplace_back`, `SetAttrName`.
- CN: 该代码块实现与 function extraction 相关的局部辅助逻辑。关键符号：`toUnqualString`, `adjust_attr_name`, `emplace_back`, `SetAttrName`。

### Lines 514-524
```cpp
  bool first_iteration = true;
  for (const auto& it : func_ctx.scope_ctxs_) {
    auto scope = it.first;
    auto annotated_attr_node = scope_attr_map_.find(scope);
    if (annotated_attr_node != scope_attr_map_.end()) {
      auto names = annotated_attr_node->second->attributeNames();
      if (first_iteration) {
        std::copy(
            names.begin(),
            names.end(),
            std::inserter(annotated_attr_names, annotated_attr_names.end()));
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `find`, `end`, `attributeNames`, `copy`, `begin`, `inserter`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`find`, `end`, `attributeNames`, `copy`, `begin`, `inserter`。

### Lines 525-542
```cpp
        first_iteration = false;
      } else {
        auto unseen_attr_name = std::find_if(
            names.begin(),
            names.end(),
            [&annotated_attr_names](const Symbol& name) {
              return annotated_attr_names.find(name) ==
                  annotated_attr_names.end();
            });
        TORCH_CHECK(
            unseen_attr_name == names.end(),
            "Found outstanding annotated attribute ",
            *unseen_attr_name,
            " from module ",
            scope->name(),
            ". Please ensure module instances of the same class have the same set of annotated attributes.");
      }
    }
```
- EN: Declares core types or data containers for this file. Prominent symbols: `find_if`, `begin`, `end`, `find`, `name`, `have`.
- CN: 声明该文件使用的核心类型或数据容器。主要符号：`find_if`, `begin`, `end`, `find`, `name`, `have`。

### Lines 543-552
```cpp
  }
  for (auto attr_name : annotated_attr_names) {
    final_attr_names.emplace_back(attr_name.toUnqualString());
  }

  func_def_n->ss_(Symbol::attr("attributes"), final_attr_names);

  return func_def_n;
}

```
- EN: This block iterates over collections or graph structures; produces a result or forwards a computed value. Key symbols: `emplace_back`, `toUnqualString`, `ss_`, `attr`.
- CN: 该代码块遍历集合或图结构；返回结果或转发已计算的值。关键符号：`emplace_back`, `toUnqualString`, `ss_`, `attr`。

### Lines 553-564
```cpp
Node* FunctionExtractor::CreateFunctionNode(
    FunctionContext& func_ctx,
    ScopeContext& scope_ctx,
    const std::shared_ptr<Graph>& graph,
    const std::string& domain_name,
    const std::string& func_name) {
  const auto& func_scope = func_ctx.scope_key_;
  GRAPH_DEBUG(
      "Create and insert local function for scope: ",
      func_scope->namesFromRoot());
  scope_ctx.PopulateInputsOutputs(param_names_);
  auto last_n = *scope_ctx.nlist_.rbegin();
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `CreateFunctionNode`, `namesFromRoot`, `PopulateInputsOutputs`, `rbegin`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`CreateFunctionNode`, `namesFromRoot`, `PopulateInputsOutputs`, `rbegin`。

### Lines 565-576
```cpp
  auto func_n = graph->create(
      Symbol::fromQualString(domain_name + "::" + func_name),
      scope_ctx.outputs_.size());
  func_n->copyMetadata(last_n);
  for (auto* v : scope_ctx.inputs_) {
    func_n->addInput(v);
  }
  for (const auto i : c10::irange(scope_ctx.outputs_.size())) {
    func_n->output(i)->setType(scope_ctx.outputs_[i]->type());
    scope_ctx.outputs_[i]->replaceAllUsesWith(func_n->output(i));
  }

```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `create`, `fromQualString`, `size`, `copyMetadata`, `addInput`, `irange`, `...`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`create`, `fromQualString`, `size`, `copyMetadata`, `addInput`, `irange`, `...`。

### Lines 577-594
```cpp
  // set attributes of different values as function attributes.
  auto copy_attr =
      [](Node* a, Node* b, Symbol attr, const std::string& new_name) {
#define COPY_ATTR(kind)                                \
  case AttributeKind::kind: {                          \
    b->kind##_(Symbol::attr(new_name), a->kind(attr)); \
    break;                                             \
  }
        switch (a->kindOf(attr)) {
          COPY_ATTR(f)
          COPY_ATTR(fs)
          COPY_ATTR(i)
          COPY_ATTR(is)
          COPY_ATTR(s)
          COPY_ATTR(ss)
          COPY_ATTR(t)
          COPY_ATTR(ts)
#undef COPY_ATTR
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `_`, `attr`, `kind`, `kindOf`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`_`, `attr`, `kind`, `kindOf`。

### Lines 595-610
```cpp
          case AttributeKind::ival:
          case AttributeKind::g:
          case AttributeKind::gs:
          case AttributeKind::ty:
          case AttributeKind::tys:
          case AttributeKind::c:
          default:
            TORCH_INTERNAL_ASSERT(
                false,
                "Unexpected attribute type ",
                static_cast<int>(a->kindOf(attr)),
                " from node ",
                *a);
            break;
        }
      };
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `kindOf`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`kindOf`。

### Lines 611-619
```cpp

  for (const auto& it : func_ctx.attribute_map_) {
    auto* ref_n = it.first;
    for (const auto& attr_it : it.second) {
      const auto& attr = attr_it.first;
      auto attr_name = func_ctx.FindAttrName(ref_n, attr).value();
      copy_attr(ref_n, func_n, attr, attr_name);
      for (auto* n : scope_ctx.nlist_) {
        if (attr_it.second.find(n) != attr_it.second.end()) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `FindAttrName`, `value`, `copy_attr`, `find`, `end`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`FindAttrName`, `value`, `copy_attr`, `find`, `end`。

### Lines 620-628
```cpp
          copy_attr(n, func_n, attr, attr_name);
          break;
        }
      }
    }
  }

  // annotated attributes
  auto scope = scope_ctx.scope_;
```
- EN: This block implements local helper logic for function extraction. Key symbols: `copy_attr`.
- CN: 该代码块实现与 function extraction 相关的局部辅助逻辑。关键符号：`copy_attr`。

### Lines 629-638
```cpp
  auto annotated_attr_node = scope_attr_map_.find(scope);
  if (annotated_attr_node != scope_attr_map_.end()) {
    auto node = annotated_attr_node->second;
    for (auto attr : node->attributeNames()) {
      copy_attr(node, func_n, attr, attr.toUnqualString());
    }
  }

  func_n->insertAfter(last_n);
  return func_n;
```
- EN: This block handles conditional branches; iterates over collections or graph structures; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `find`, `end`, `attributeNames`, `copy_attr`, `toUnqualString`, `insertAfter`.
- CN: 该代码块处理条件分支；遍历集合或图结构；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`find`, `end`, `attributeNames`, `copy_attr`, `toUnqualString`, `insertAfter`。

### Lines 639-652
```cpp
}

void FunctionExtractor::ConvertScopeToFunction(
    const ScopePtr& scope_key,
    const scope_list& scope_list,
    scope_ctx_map& scope_ctxs,
    const std::shared_ptr<Graph>& graph) {
  // This function needs to be called always on inner most scopes.
  // 1. Generate function context, this identifies different constants and
  // attributes.
  // 2. Create function definition node, and insert to main graph.
  // 3. Create function node for each call, and replace subgraph nodes in parent
  // functions.

```
- EN: This block manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `ConvertScopeToFunction`.
- CN: 该代码块操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`ConvertScopeToFunction`。

### Lines 653-661
```cpp
  func_ctxs_.insert(std::make_pair(
      scope_key, new FunctionContext(scope_key, scope_list, scope_ctxs)));
  auto& func_ctx = *func_ctxs_[scope_key];

  const std::string module_class_name(
      ONNXScopeName::className(func_ctx.scope_key_));
  auto pos = module_class_name.rfind('.');
  TORCH_INTERNAL_ASSERT(pos != std::string::npos);

```
- EN: This block implements local helper logic for function extraction. Key symbols: `insert`, `make_pair`, `FunctionContext`, `module_class_name`, `className`, `rfind`.
- CN: 该代码块实现与 function extraction 相关的局部辅助逻辑。关键符号：`insert`, `make_pair`, `FunctionContext`, `module_class_name`, `className`, `rfind`。

### Lines 662-670
```cpp
  auto construct_unique_module_name = [&](std::string module_name) {
    auto module_name_variant = module_variant_count_.find(module_name);
    if (module_name_variant != module_variant_count_.end()) {
      module_variant_count_[module_name]++;
      module_name += ("." + std::to_string(module_name_variant->second));
    } else {
      module_variant_count_[module_name] = 0;
    }
    return module_name;
```
- EN: This block handles conditional branches; produces a result or forwards a computed value. Key symbols: `find`, `end`, `to_string`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值。关键符号：`find`, `end`, `to_string`。

### Lines 671-680
```cpp
  };

  const auto domain_name = module_class_name.substr(0, pos);
  const auto func_name =
      construct_unique_module_name(module_class_name.substr(pos + 1));

  CreateFunctionDefNode(func_ctx, graph, domain_name, func_name);

  // create and insert local function node to graph.
  for (const auto& it : func_ctx.scope_ctxs_) {
```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `substr`, `construct_unique_module_name`, `CreateFunctionDefNode`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`substr`, `construct_unique_module_name`, `CreateFunctionDefNode`。

### Lines 681-689
```cpp
    auto scope = it.first;
    auto& scope_ctx = *it.second;
    auto func_n =
        CreateFunctionNode(func_ctx, scope_ctx, graph, domain_name, func_name);

    std::unordered_set<Node*> old_nodes(
        scope_ctx.nlist_.begin(), scope_ctx.nlist_.end());

    auto last_n = *scope_ctx.nlist_.rbegin();
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `CreateFunctionNode`, `old_nodes`, `begin`, `end`, `rbegin`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`CreateFunctionNode`, `old_nodes`, `begin`, `end`, `rbegin`。

### Lines 690-698
```cpp
    // replace function body nodes in parent scopes with local function node.
    for (auto& it : scope_ctxs) {
      const auto& parent_scope = it.first;
      auto& parent_ctx = *it.second;

      if (!IsAncestor(parent_scope, scope)) {
        continue;
      }

```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `IsAncestor`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`IsAncestor`。

### Lines 699-707
```cpp
      auto& ctx_nlist = parent_ctx.nlist_;
      GRAPH_DEBUG(
          "Replace local function node in parent scope: ",
          it.first->namesFromRoot(),
          " nodes to remove: ",
          old_nodes.size(),
          " parent total nodes: ",
          ctx_nlist.size());

```
- EN: This block manipulates TorchScript IR objects. Key symbols: `namesFromRoot`, `size`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`namesFromRoot`, `size`。

### Lines 708-717
```cpp
      // insert local function node
      auto last_n_it = std::find(ctx_nlist.begin(), ctx_nlist.end(), last_n);
      ctx_nlist.insert(last_n_it, func_n);

      // remove replaced nodes from list
      std::erase_if(
          ctx_nlist, [&old_nodes](Node* n) { return old_nodes.contains(n); });

      GRAPH_DEBUG("Parent total nodes after remove: ", ctx_nlist.size());

```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `find`, `begin`, `end`, `insert`, `erase_if`, `contains`, `...`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`find`, `begin`, `end`, `insert`, `erase_if`, `contains`, `...`。

### Lines 718-726
```cpp
      // refresh inputs/outputs.
      parent_ctx.PopulateInputsOutputs(param_names_);
    }
  }

  for (const auto& it : func_ctx.scope_ctxs_) {
    auto& scope_ctx = *it.second;
    // delete replaced nodes in graph.
    for (auto it = scope_ctx.nlist_.rbegin(); it != scope_ctx.nlist_.rend();) {
```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `PopulateInputsOutputs`, `rbegin`, `rend`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`PopulateInputsOutputs`, `rbegin`, `rend`。

### Lines 727-735
```cpp
      auto* n = *it;
      it++;
      GRAPH_DEBUG("Destroying node ", *n);
      n->destroy();
    }
  }
}

bool FunctionExtractor::ScopeContext::IsIdenticalFunction(
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `destroy`, `IsIdenticalFunction`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`destroy`, `IsIdenticalFunction`。

### Lines 736-744
```cpp
    const ScopeContext& other_ctx) const {
  // Differentiate same function under different inputs.
  // When constants are passed in place of inputs, it leads to different
  // input count and node count. Likewise, due to different uses, output
  // count can be different as well.
  // For now export them as different functions.
  // Covered by `test_local_function_overloads` in
  // `test/onnx/test_utility_funs.py`.
  if (&other_ctx == this) {
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: no dominant local symbol names.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：无明显局部符号。

### Lines 745-753
```cpp
    return true;
  }
  if (ONNXScopeName::className(this->scope_) !=
      ONNXScopeName::className(other_ctx.scope_)) {
    return false;
  }
  if (this->inputs_.size() != other_ctx.inputs_.size() ||
      this->outputs_.size() != other_ctx.outputs_.size()) {
    return false;
```
- EN: This block handles conditional branches; produces a result or forwards a computed value. Key symbols: `className`, `size`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值。关键符号：`className`, `size`。

### Lines 754-762
```cpp
  }
  const auto& ns_a = this->nlist_;
  const auto& ns_b = other_ctx.nlist_;
  if (ns_a.size() != ns_b.size()) {
    return false;
  }
  for (const auto i : c10::irange(ns_a.size())) {
    if (ns_a[i]->kind() != ns_b[i]->kind()) {
      return false;
```
- EN: This block handles conditional branches; iterates over collections or graph structures; produces a result or forwards a computed value. Key symbols: `size`, `irange`, `kind`.
- CN: 该代码块处理条件分支；遍历集合或图结构；返回结果或转发已计算的值。关键符号：`size`, `irange`, `kind`。

### Lines 763-774
```cpp
    }
  }

  return true;
}

void FunctionExtractor::ScopeContext::PopulateInputsOutputs(
    const std::unordered_set<std::string>& param_names) {
  inputs_.clear();
  outputs_.clear();
  const auto& nlist = this->nlist_;
  std::unordered_set<Value*> v_set;
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `PopulateInputsOutputs`, `clear`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`PopulateInputsOutputs`, `clear`。

### Lines 775-783
```cpp
  std::unordered_set<Node*> n_set;

  value_list input_list;
  value_list initializer_list;

  // Add initializers after inputs.
  for (auto* n : nlist) {
    for (auto* v : n->inputs()) {
      if (v_set.find(v) == v_set.end()) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `inputs`, `find`, `end`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`inputs`, `find`, `end`。

### Lines 784-792
```cpp
        if (param_names.find(v->debugName()) != param_names.end()) {
          initializer_list.emplace_back(v);
        } else {
          input_list.emplace_back(v);
        }
        v_set.insert(v);
      }
    }
    for (auto* v : n->outputs()) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures. Key symbols: `find`, `debugName`, `end`, `emplace_back`, `insert`, `outputs`.
- CN: 该代码块处理条件分支；遍历集合或图结构。关键符号：`find`, `debugName`, `end`, `emplace_back`, `insert`, `outputs`。

### Lines 793-803
```cpp
      v_set.insert(v);
    }
    n_set.insert(n);
  }
  for (auto* v : input_list) {
    inputs_.emplace_back(v);
  }
  for (auto* v : initializer_list) {
    inputs_.emplace_back(v);
  }

```
- EN: This block iterates over collections or graph structures. Key symbols: `insert`, `emplace_back`.
- CN: 该代码块遍历集合或图结构。关键符号：`insert`, `emplace_back`。

### Lines 804-816
```cpp
  for (auto* n : nlist) {
    for (auto* v : n->outputs()) {
      bool used_outside = false;
      for (auto use : v->uses()) {
        used_outside |= (n_set.find(use.user) == n_set.end());
      }
      if (used_outside) {
        outputs_.emplace_back(v);
      }
    }
  }
}

```
- EN: This block handles conditional branches; iterates over collections or graph structures. Key symbols: `outputs`, `uses`, `find`, `end`, `emplace_back`.
- CN: 该代码块处理条件分支；遍历集合或图结构。关键符号：`outputs`, `uses`, `find`, `end`, `emplace_back`。

### Lines 817-829
```cpp
void FunctionExtractor::HandleNoScopeNodes(
    scope_ctx_map& scope_ctxs,
    const node_list& no_scope_nlist) {
  GRAPH_UPDATE("No scope node count: ", no_scope_nlist.size());
  for (auto n : no_scope_nlist) {
    TORCH_WARN(
        "ONNX function extraction cannot determine the scope for node: ", *n);
  }
  TORCH_INTERNAL_ASSERT(
      no_scope_nlist.empty(),
      "ONNX function extraction cannot determine the scope for the above nodes.");
}

```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `HandleNoScopeNodes`, `size`, `empty`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`HandleNoScopeNodes`, `size`, `empty`。

### Lines 830-840
```cpp
std::tuple<FunctionExtractor::scope_ctx_map, node_list> FunctionExtractor::
    PartitionNodesByScope(Block* b) {
  scope_ctx_map scope_ctxs = {};
  node_list no_scope_nlist;

  auto find_or_create_scope_ctx = [](scope_ctx_map& scope_ctxs,
                                     const ScopePtr& scope) {
    if (scope_ctxs.find(scope) == scope_ctxs.end()) {
      scope_ctxs.insert(std::make_pair(scope, new ScopeContext()));
    }
    return scope_ctxs[scope];
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `PartitionNodesByScope`, `find`, `end`, `insert`, `make_pair`, `ScopeContext`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`PartitionNodesByScope`, `find`, `end`, `insert`, `make_pair`, `ScopeContext`。

### Lines 841-849
```cpp
  };

  auto record_node_scope = [&scope_ctxs, &find_or_create_scope_ctx](Node* n) {
    const auto& scope = n->scope();
    find_or_create_scope_ctx(scope_ctxs, scope)->scope_ = scope;
    auto tmp_scope = scope;
    while (IsValidScope(tmp_scope)) {
      find_or_create_scope_ctx(scope_ctxs, tmp_scope)->nlist_.emplace_back(n);
      if (IsValidScope(tmp_scope->parent())) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `scope`, `find_or_create_scope_ctx`, `IsValidScope`, `emplace_back`, `parent`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`scope`, `find_or_create_scope_ctx`, `IsValidScope`, `emplace_back`, `parent`。

### Lines 850-858
```cpp
        find_or_create_scope_ctx(scope_ctxs, tmp_scope->parent())
            ->children_.insert(tmp_scope);
      }
      tmp_scope = tmp_scope->parent();
    }
  };

  for (auto* n : b->nodes()) {
    auto scope = n->scope();
```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `find_or_create_scope_ctx`, `parent`, `insert`, `nodes`, `scope`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`find_or_create_scope_ctx`, `parent`, `insert`, `nodes`, `scope`。

### Lines 859-872
```cpp
    if (scope && IsValidScope(scope)) {
      record_node_scope(n);
    } else {
      auto inferred_scope = InferScope(n);

      if (inferred_scope.has_value() && IsValidScope(inferred_scope.value())) {
        n->setScope(inferred_scope.value());
        record_node_scope(n);
      } else {
        GRAPH_UPDATE("Cannot infer proper scope for node: ", *n);
        no_scope_nlist.emplace_back(n);
      }
    }

```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `IsValidScope`, `record_node_scope`, `InferScope`, `has_value`, `value`, `setScope`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`IsValidScope`, `record_node_scope`, `InferScope`, `has_value`, `value`, `setScope`, `...`。

### Lines 873-881
```cpp
    for (auto* sub_b : n->blocks()) {
      auto [subblock_scope_ctxs, subblock_no_scope_nlist] =
          PartitionNodesByScope(sub_b);

      for (auto& it : subblock_scope_ctxs) {
        if (scope_ctxs.find(it.first) == scope_ctxs.end()) {
          scope_ctxs.insert(std::make_pair(it.first, it.second));
        } else {
          for (auto* s_n : it.second->nlist_) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `blocks`, `PartitionNodesByScope`, `find`, `end`, `insert`, `make_pair`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`blocks`, `PartitionNodesByScope`, `find`, `end`, `insert`, `make_pair`。

### Lines 882-896
```cpp
            scope_ctxs[it.first]->nlist_.emplace_back(s_n);
          }
          for (const auto& s_child_scope : it.second->children_) {
            scope_ctxs[it.first]->children_.insert(s_child_scope);
          }
        }
      }

      no_scope_nlist.insert(
          no_scope_nlist.end(),
          subblock_no_scope_nlist.begin(),
          subblock_no_scope_nlist.end());
    }
  }

```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `emplace_back`, `insert`, `end`, `begin`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`emplace_back`, `insert`, `end`, `begin`。

### Lines 897-909
```cpp
  for (auto& it : scope_ctxs) {
    it.second->scope_ = it.first;
    it.second->PopulateInputsOutputs(param_names_);
  }

  return std::tie(scope_ctxs, no_scope_nlist);
}

FunctionExtractor::scope_ctx_map FunctionExtractor::PartitionNodesByScope(
    const std::shared_ptr<Graph>& graph) {
  scope_ctx_map scope_ctxs;
  node_list no_scope_nlist;
  std::tie(scope_ctxs, no_scope_nlist) = PartitionNodesByScope(graph->block());
```
- EN: This block iterates over collections or graph structures; produces a result or forwards a computed value; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `PopulateInputsOutputs`, `tie`, `PartitionNodesByScope`, `block`.
- CN: 该代码块遍历集合或图结构；返回结果或转发已计算的值；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`PopulateInputsOutputs`, `tie`, `PartitionNodesByScope`, `block`。

### Lines 910-918
```cpp

  HandleNoScopeNodes(scope_ctxs, no_scope_nlist);

  return scope_ctxs;
}

std::unordered_map<ScopePtr, scope_list> FunctionExtractor::
    PartitionIdenticalScopes(FunctionExtractor::scope_ctx_map& scope_ctxs) {
  std::unordered_map<ScopePtr, scope_list> identical_scope_map;
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `HandleNoScopeNodes`, `PartitionIdenticalScopes`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`HandleNoScopeNodes`, `PartitionIdenticalScopes`。

### Lines 919-928
```cpp

  for (auto& it : scope_ctxs) {
    auto scope = it.first;
    const auto& scope_ctx = it.second;
    bool unique = true;
    for (auto& kv_it : identical_scope_map) {
      auto key_scope = kv_it.first;
      const auto& key_scope_ctx = scope_ctxs[key_scope];
      auto& key_scope_vec = kv_it.second;
      if (key_scope_ctx->IsIdenticalFunction(*scope_ctx)) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures. Key symbols: `IsIdenticalFunction`.
- CN: 该代码块处理条件分支；遍历集合或图结构。关键符号：`IsIdenticalFunction`。

### Lines 929-938
```cpp
        key_scope_vec.emplace_back(scope);
        unique = false;
        break;
      }
    }
    if (unique) {
      identical_scope_map[scope].emplace_back(scope);
    }
  }

```
- EN: This block handles conditional branches. Key symbols: `emplace_back`.
- CN: 该代码块处理条件分支。关键符号：`emplace_back`。

### Lines 939-947
```cpp
  return identical_scope_map;
}

static bool HasSameAttribute(
    const Node* a,
    const Node* b,
    const c10::Symbol& attr) {
  if (!a->hasAttribute(attr) && !b->hasAttribute(attr)) {
    return true;
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `HasSameAttribute`, `hasAttribute`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`HasSameAttribute`, `hasAttribute`。

### Lines 948-957
```cpp
  }
  if (!a->hasAttribute(attr) || !b->hasAttribute(attr)) {
    return false;
  }
  auto a_kind = a->kindOf(attr);
  auto b_kind = b->kindOf(attr);
  if (a_kind != b_kind) {
    return false;
  }

```
- EN: This block handles conditional branches; produces a result or forwards a computed value. Key symbols: `hasAttribute`, `kindOf`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值。关键符号：`hasAttribute`, `kindOf`。

### Lines 958-975
```cpp
#define COMP_ATTR(kind)              \
  case AttributeKind::kind: {        \
    const auto& a_v = a->kind(attr); \
    const auto& b_v = b->kind(attr); \
    return a_v == b_v;               \
  }

  switch (a_kind) {
    COMP_ATTR(f)
    COMP_ATTR(fs)
    COMP_ATTR(i)
    COMP_ATTR(is)
    COMP_ATTR(s)
    COMP_ATTR(ss)
#undef COMP_ATTR
    case AttributeKind::t: {
      const auto& a_v = a->t(attr);
      const auto& b_v = b->t(attr);
```
- EN: This block produces a result or forwards a computed value. Key symbols: `kind`, `t`.
- CN: 该代码块返回结果或转发已计算的值。关键符号：`kind`, `t`。

### Lines 976-987
```cpp
      return a_v.equal(b_v);
    }
    case AttributeKind::ts: {
      const auto& a_v = a->ts(attr);
      const auto& b_v = b->ts(attr);
      return std::equal(
          a_v.begin(),
          a_v.end(),
          b_v.begin(),
          b_v.end(),
          [](const at::Tensor& a_t, const at::Tensor& b_t) {
            return a_t.equal(b_t);
```
- EN: This block produces a result or forwards a computed value. Key symbols: `equal`, `ts`, `begin`, `end`.
- CN: 该代码块返回结果或转发已计算的值。关键符号：`equal`, `ts`, `begin`, `end`。

### Lines 988-1005
```cpp
          });
    }
    case AttributeKind::ival:
    case AttributeKind::g:
    case AttributeKind::gs:
    case AttributeKind::ty:
    case AttributeKind::tys:
    case AttributeKind::c:
    default:
      TORCH_INTERNAL_ASSERT(
          false,
          "Unexpected attribute type ",
          static_cast<int>(a_kind),
          " from node ",
          *a);
      break;
  }

```
- EN: This block manipulates TorchScript IR objects. Key symbols: no dominant local symbol names.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：无明显局部符号。

### Lines 1006-1015
```cpp
  return true;
}

scope_list FunctionExtractor::SortScopesByMaxDepth(
    std::unordered_map<ScopePtr, scope_list>& identical_scope_map) {
  std::unordered_map<ScopePtr, size_t> scope_max_depth;
  for (const auto& it : identical_scope_map) {
    const auto& scopes = it.second;
    size_t max_depth = 0;
    for (const auto& scope : scopes) {
```
- EN: This block iterates over collections or graph structures; produces a result or forwards a computed value. Key symbols: `SortScopesByMaxDepth`.
- CN: 该代码块遍历集合或图结构；返回结果或转发已计算的值。关键符号：`SortScopesByMaxDepth`。

### Lines 1016-1025
```cpp
      if (scope->getDepth() > max_depth) {
        max_depth = scope->getDepth();
      }
    }
    scope_max_depth[it.first] = max_depth;
  }

  scope_list sorted_scopes;
  sorted_scopes.reserve(scope_max_depth.size());
  for (const auto& it : scope_max_depth) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures. Key symbols: `getDepth`, `reserve`, `size`.
- CN: 该代码块处理条件分支；遍历集合或图结构。关键符号：`getDepth`, `reserve`, `size`。

### Lines 1026-1034
```cpp
    sorted_scopes.emplace_back(it.first);
  }
  std::sort(
      sorted_scopes.begin(),
      sorted_scopes.end(),
      [&scope_max_depth](const ScopePtr& a, const ScopePtr& b) -> bool {
        return scope_max_depth[a] >= scope_max_depth[b];
      });
  return sorted_scopes;
```
- EN: This block produces a result or forwards a computed value. Key symbols: `emplace_back`, `sort`, `begin`, `end`.
- CN: 该代码块返回结果或转发已计算的值。关键符号：`emplace_back`, `sort`, `begin`, `end`。

### Lines 1035-1043
```cpp
}

NodeAttrNameMap FunctionExtractor::run() {
  auto scope_ctxs = PartitionNodesByScope(graph_);
  DebugPrintScopeContexts(scope_ctxs);
  auto identical_scope_map = PartitionIdenticalScopes(scope_ctxs);
  // Deepest scope comes first, guaranteeing no other scope can be its child.
  auto sorted_scope_keys = SortScopesByMaxDepth(identical_scope_map);
  for (const auto& scope_key : sorted_scope_keys) {
```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `run`, `PartitionNodesByScope`, `DebugPrintScopeContexts`, `PartitionIdenticalScopes`, `SortScopesByMaxDepth`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`run`, `PartitionNodesByScope`, `DebugPrintScopeContexts`, `PartitionIdenticalScopes`, `SortScopesByMaxDepth`。

### Lines 1044-1052
```cpp
    if (module_names_.find(ONNXScopeName::className(scope_key)) !=
        module_names_.end()) {
      ConvertScopeToFunction(
          scope_key, identical_scope_map[scope_key], scope_ctxs, graph_);
    }
    GRAPH_DEBUG("Main graph afterwards: ", graph_->toString());
  }
  DebugPrintGraphWithFunction(graph_);

```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `find`, `className`, `end`, `ConvertScopeToFunction`, `toString`, `DebugPrintGraphWithFunction`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`find`, `className`, `end`, `ConvertScopeToFunction`, `toString`, `DebugPrintGraphWithFunction`。

### Lines 1053-1062
```cpp
  // Construct return mappings
  NodeAttrNameMap node_attr_to_name;

  for (const auto& it : func_ctxs_) {
    auto func_ref_map = it.second->node_attr_to_name_;
    node_attr_to_name.insert(func_ref_map.begin(), func_ref_map.end());
  }

  // Clear
  for (auto& it : scope_ctxs) {
```
- EN: This block iterates over collections or graph structures; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `insert`, `begin`, `end`.
- CN: 该代码块遍历集合或图结构；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`insert`, `begin`, `end`。

### Lines 1063-1071
```cpp
    delete it.second;
  }
  scope_ctxs.clear();
  for (auto& it : func_ctxs_) {
    delete it.second;
  }
  func_ctxs_.clear();

  return node_attr_to_name;
```
- EN: This block iterates over collections or graph structures; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `clear`.
- CN: 该代码块遍历集合或图结构；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`clear`。

### Lines 1072-1080
```cpp
}

// Retrieves the node representing the most recent
// ScopePtr. This function should only be invoked from module forward hook. At
// this point, module forward call is completed, and the most recent ScopePtr
// is popped from TracingState.
// This function inspects the node, and its subblock, to find
// the node associated with the most recent ScopePtr.
Node* NodeOfMostRecentScope(Node* forward_node) {
```
- EN: This block manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `NodeOfMostRecentScope`.
- CN: 该代码块操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`NodeOfMostRecentScope`。

### Lines 1081-1089
```cpp
  TORCH_INTERNAL_ASSERT(
      forward_node->kind() == prim::TracedModuleForward,
      "forward_node got kind: ",
      forward_node->kind().toDisplayString());
  auto* block = forward_node->blocks()[0];
  for (auto* node : block->nodes().reverse()) {
    if (node->kind() == prim::TracedModuleForward) {
      Node* target_node = NodeOfMostRecentScope(node);
      if (scope_attr_map_.find(node->scope()) == scope_attr_map_.end()) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `kind`, `toDisplayString`, `blocks`, `nodes`, `reverse`, `NodeOfMostRecentScope`, `...`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`kind`, `toDisplayString`, `blocks`, `nodes`, `reverse`, `NodeOfMostRecentScope`, `...`。

### Lines 1090-1098
```cpp
        return target_node;
      }
    }
  }
  return forward_node;
}

} // namespace

```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: no dominant local symbol names.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：无明显局部符号。

### Lines 1099-1112
```cpp
// FunctionExtractor runs in the following steps. Updates are made inplace to
// the graph argument.
//    1. Partition nodes into groups based on their scope information.
//    Each scope represents an individual nn.Module call. A ScopeContext object
//    is created for each group.
//    2. Compare and find groups with the same subgraph pattern from step 1.
//    3. Scopes are nested. Starting from the deepest scope, extract the
//    subgraph pattern, and define as local function node. Replace subgraph
//    pattern with a single node of the new local function node type. A
//    FunctionContext object is created for each function.
//    4. Construct NodeAttrNameMap tracking mapping from attribute name of
//    IR Node inside function subgraph, to function attribute name.
NodeAttrNameMap ONNXFunctionExtraction(
    std::shared_ptr<Graph>& graph,
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `ONNXFunctionExtraction`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`ONNXFunctionExtraction`。

### Lines 1113-1121
```cpp
    const std::unordered_set<std::string>& module_names,
    const std::vector<std::string>& param_names) {
  GRAPH_UPDATE(
      "Export these module forward calls as functions: ",
      std::vector<std::string>{module_names.begin(), module_names.end()});
  FunctionExtractor fe(graph, module_names, param_names);
  return fe.run();
}

```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `begin`, `end`, `fe`, `run`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`begin`, `end`, `fe`, `run`。

### Lines 1122-1138
```cpp
void ONNXClearScopeRecords() {
  scope_attr_map_.clear();
  scope_attr_graph_ = std::make_shared<Graph>();
}

void ONNXTrackScopeAttributes(
    std::shared_ptr<Graph>& graph,
    std::map<std::string, IValue>& attributes) {
  // Skip the "real" last node which is `return_node`.
  auto* last_node = graph->nodes().back()->prev();
  auto* scope_node = NodeOfMostRecentScope(last_node);
  auto* attr_node = scope_attr_graph_->create(prim::TracedModuleForward);
  attr_node->setScope(scope_node->scope());
  TORCH_INTERNAL_ASSERT(
      scope_attr_map_.find(scope_node->scope()) == scope_attr_map_.end());
  scope_attr_map_[scope_node->scope()] = attr_node;

```
- EN: This block manipulates TorchScript IR objects. Key symbols: `ONNXClearScopeRecords`, `clear`, `ONNXTrackScopeAttributes`, `nodes`, `back`, `prev`, `...`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`ONNXClearScopeRecords`, `clear`, `ONNXTrackScopeAttributes`, `nodes`, `back`, `prev`, `...`。

### Lines 1139-1155
```cpp
  for (const auto& it : attributes) {
    auto k = Symbol::attr(it.first);
    auto v = it.second;
    if (v.isTensor()) {
      attr_node->t_(k, v.toTensor());
    } else if (v.isInt()) {
      attr_node->i_(k, v.toInt());
    } else if (v.isDouble()) {
      attr_node->f_(k, v.toDouble());
    } else if (v.isBool()) {
      attr_node->i_(k, v.toBool());
    } else if (v.isString()) {
      attr_node->s_(k, v.toStringRef());
    } else if (v.isIntList()) {
      attr_node->is_(k, v.toIntList().vec());
    } else if (v.isBoolList()) {
      auto bool_list = v.toBoolList();
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `attr`, `isTensor`, `t_`, `toTensor`, `isInt`, `i_`, `...`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`attr`, `isTensor`, `t_`, `toTensor`, `isInt`, `i_`, `...`。

### Lines 1156-1164
```cpp
      attr_node->is_(
          k, std::vector<int64_t>(bool_list.begin(), bool_list.end()));
    } else if (v.isDoubleList()) {
      attr_node->fs_(k, v.toDoubleList().vec());
    }
  }
}

} // namespace torch::jit::onnx
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `is_`, `begin`, `end`, `isDoubleList`, `fs_`, `toDoubleList`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`is_`, `begin`, `end`, `isDoubleList`, `fs_`, `toDoubleList`, `...`。


## Key Concepts / 关键概念
- EN: IR graph manipulation — the file inspects or rewrites TorchScript graphs and nodes.
  CN: IR 图操作——该文件会检查或改写 TorchScript 的图、节点与块。
- EN: Schema handling — the implementation parses or normalizes operator/function signatures.
  CN: 模式处理——实现会解析或规范化算子/函数签名。
- EN: Optimization pass — the code simplifies graphs by removing redundant work or folding values.
  CN: 优化 pass——代码通过删除冗余计算或常量折叠来简化图。
- EN: Fusion support — the file prepares groups of operations for fused execution.
  CN: 融合支持——该文件会为融合执行准备算子分组。
- EN: Type and shape reasoning — the code propagates static information across the IR.
  CN: 类型与形状推理——代码会在 IR 中传播静态信息。
- EN: Export interop — the implementation helps bridge TorchScript graphs to ONNX-style representations.
  CN: 导出互操作——实现帮助 TorchScript 图与 ONNX 风格表示之间建立桥接。

## Dependencies / 依赖关系
- Internal includes / 内部头文件: `torch/csrc/jit/jit_log.h`, `torch/csrc/jit/passes/onnx/function_extraction.h`, `torch/csrc/jit/passes/onnx/naming.h`
- External includes / 外部头文件: 无
- Namespaces / 命名空间: `torch::jit::onnx`
- Representative symbols / 代表性符号: `scope_list`, `HasSameAttribute`, `FunctionExtractor`, `graph_`, `module_names_`, `begin`, `end`, `param_names_`, `run`, `ScopeContext`, `...`
