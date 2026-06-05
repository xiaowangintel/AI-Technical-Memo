# naming.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `torch/csrc/jit/passes/onnx/naming.cpp`
- Repository: `pytorch`
- Purpose (EN): Implements the TorchScript JIT pass logic for naming, including graph analysis and rewrites.
- 用途 (CN): 实现与 naming 相关的 TorchScript JIT pass 逻辑，包括图分析与改写。

## Line-by-Line Analysis / 逐行分析
### Lines 1-7
```cpp
#include <torch/csrc/jit/passes/onnx/naming.h>
#include <torch/csrc/onnx/onnx.h>

#include <utility>

namespace torch::jit::onnx {

```
- EN: Opens or closes namespace scopes so the remaining declarations live in the expected JIT/frontend module.
- CN: 打开或关闭命名空间作用域，使后续声明归属于预期的 JIT/前端模块。

### Lines 8-14
```cpp
namespace ONNXScopeName {

using NameFunc = std::string (*)(const torch::jit::ScopePtr& scope);

const std::string name_separator = "::";

namespace {
```
- EN: Opens or closes namespace scopes so the remaining declarations live in the expected JIT/frontend module.
- CN: 打开或关闭命名空间作用域，使后续声明归属于预期的 JIT/前端模块。

### Lines 15-21
```cpp

std::string nameFromRoot(
    const torch::jit::ScopePtr& scope,
    const std::string& layer_separator,
    NameFunc name_func) {
  std::string out = (*name_func)(scope);
  if (scope->isRoot()) {
```
- EN: This block handles conditional branches. Key symbols: `nameFromRoot`, `isRoot`.
- CN: 该代码块处理条件分支。关键符号：`nameFromRoot`, `isRoot`。

### Lines 22-29
```cpp
    return out;
  }
  auto parent = scope->parent();
  while (isCompatibleScope(parent)) {
    out = std::string((*name_func)(parent)).append(layer_separator).append(out);
    parent = parent->parent();
  }
  return out;
```
- EN: This block iterates over collections or graph structures; produces a result or forwards a computed value. Key symbols: `parent`, `isCompatibleScope`, `string`, `append`.
- CN: 该代码块遍历集合或图结构；返回结果或转发已计算的值。关键符号：`parent`, `isCompatibleScope`, `string`, `append`。

### Lines 30-40
```cpp
}

std::pair<std::string, std::string> parseNameFromScope(
    const torch::jit::ScopePtr& scope) {
  std::string full_name = scope->name().toUnqualString();
  auto pos = full_name.find(name_separator);
  TORCH_CHECK(
      pos != std::string::npos,
      "Scope name (" + full_name + ") does not contain '" + name_separator +
          "'");
  return std::make_pair(full_name.substr(0, pos), full_name.substr(pos + 2));
```
- EN: This block produces a result or forwards a computed value; advances frontend parsing or lowering state. Key symbols: `parseNameFromScope`, `name`, `toUnqualString`, `find`, `make_pair`, `substr`.
- CN: 该代码块返回结果或转发已计算的值；推进前端解析或降级状态。关键符号：`parseNameFromScope`, `name`, `toUnqualString`, `find`, `make_pair`, `substr`。

### Lines 41-48
```cpp
}

} // namespace

std::string createFullScopeName(
    const std::string& class_name,
    const std::string& variable_name) {
  return std::string(class_name).append(name_separator).append(variable_name);
```
- EN: This block produces a result or forwards a computed value. Key symbols: `createFullScopeName`, `string`, `append`.
- CN: 该代码块返回结果或转发已计算的值。关键符号：`createFullScopeName`, `string`, `append`。

### Lines 49-55
```cpp
}

std::string variableName(const torch::jit::ScopePtr& scope) {
  return parseNameFromScope(scope).second;
}

std::string variableNameFromRoot(
```
- EN: This block produces a result or forwards a computed value; advances frontend parsing or lowering state. Key symbols: `variableName`, `parseNameFromScope`, `variableNameFromRoot`.
- CN: 该代码块返回结果或转发已计算的值；推进前端解析或降级状态。关键符号：`variableName`, `parseNameFromScope`, `variableNameFromRoot`。

### Lines 56-62
```cpp
    const torch::jit::ScopePtr& scope,
    const std::string& layer_separator) {
  return nameFromRoot(scope, layer_separator, &variableName);
}

std::string className(const torch::jit::ScopePtr& scope) {
  return parseNameFromScope(scope).first;
```
- EN: This block produces a result or forwards a computed value; advances frontend parsing or lowering state. Key symbols: `nameFromRoot`, `className`, `parseNameFromScope`.
- CN: 该代码块返回结果或转发已计算的值；推进前端解析或降级状态。关键符号：`nameFromRoot`, `className`, `parseNameFromScope`。

### Lines 63-70
```cpp
}

std::string classNameFromRoot(
    const torch::jit::ScopePtr& scope,
    const std::string& layer_separator) {
  return nameFromRoot(scope, layer_separator, &className);
}

```
- EN: This block produces a result or forwards a computed value. Key symbols: `classNameFromRoot`, `nameFromRoot`.
- CN: 该代码块返回结果或转发已计算的值。关键符号：`classNameFromRoot`, `nameFromRoot`。

### Lines 71-77
```cpp
bool isCompatibleScope(const torch::jit::ScopePtr& scope) {
  return !scope->isRoot() && !scope->isBlank() &&
      (std::string(scope->name().toUnqualString()).find(name_separator) !=
       std::string::npos);
}
} // namespace ONNXScopeName

```
- EN: This block produces a result or forwards a computed value. Key symbols: `isCompatibleScope`, `isRoot`, `isBlank`, `string`, `name`, `toUnqualString`, `...`.
- CN: 该代码块返回结果或转发已计算的值。关键符号：`isCompatibleScope`, `isRoot`, `isBlank`, `string`, `name`, `toUnqualString`, `...`。

### Lines 78-84
```cpp
namespace {

class NodeNameGenerator {
 public:
  NodeNameGenerator(std::shared_ptr<Graph> g) : graph_(std::move(g)) {}
  virtual ~NodeNameGenerator() = 0;
  void PopulateNodeNames();
```
- EN: Declares core types or data containers for this file. Prominent symbols: `NodeNameGenerator`, `graph_`, `move`, `~NodeNameGenerator`, `PopulateNodeNames`.
- CN: 声明该文件使用的核心类型或数据容器。主要符号：`NodeNameGenerator`, `graph_`, `move`, `~NodeNameGenerator`, `PopulateNodeNames`。

### Lines 85-91
```cpp

 protected:
  virtual void CreateNodeName(Node* n) = 0;
  void PopulateNodeNames(Block* /*b*/);
  void UpdateOutputsNames(Node* n);
  bool IsGraphOutput(const Value* v, const std::shared_ptr<Graph>& graph) const;

```
- EN: This block manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `CreateNodeName`, `PopulateNodeNames`, `UpdateOutputsNames`, `IsGraphOutput`.
- CN: 该代码块操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`CreateNodeName`, `PopulateNodeNames`, `UpdateOutputsNames`, `IsGraphOutput`。

### Lines 92-98
```cpp
 protected:
  std::string CreateUniqueName(
      std::unordered_map<std::string, size_t>& base_name_count,
      std::string base_name);

  std::unordered_map<const Node*, std::string> node_names_;
  std::unordered_map<std::string, size_t> base_node_name_counts_;
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `CreateUniqueName`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`CreateUniqueName`。

### Lines 99-105
```cpp
  std::shared_ptr<Graph> graph_;
  // NOLINTNEXTLINE(cppcoreguidelines-avoid-const-or-ref-data-members)
  const std::string layer_separator_ = "/";
};
NodeNameGenerator::~NodeNameGenerator() = default;

class ScopedNodeNameGenerator : public NodeNameGenerator {
```
- EN: Declares core types or data containers for this file. Prominent symbols: `~NodeNameGenerator`, `ScopedNodeNameGenerator`.
- CN: 声明该文件使用的核心类型或数据容器。主要符号：`~NodeNameGenerator`, `ScopedNodeNameGenerator`。

### Lines 106-112
```cpp
 public:
  ScopedNodeNameGenerator(std::shared_ptr<Graph> g)
      : NodeNameGenerator(std::move(g)) {}

 protected:
  void CreateNodeName(Node* n) override;

```
- EN: This block manipulates TorchScript IR objects. Key symbols: `ScopedNodeNameGenerator`, `NodeNameGenerator`, `move`, `CreateNodeName`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`ScopedNodeNameGenerator`, `NodeNameGenerator`, `move`, `CreateNodeName`。

### Lines 113-119
```cpp
 private:
  std::string GetFullScopeName(const ScopePtr& scope);
  std::unordered_map<ScopePtr, std::string> full_scope_names_;
  std::unordered_map<std::string, size_t> base_scope_name_counts_;
};

std::string NodeNameGenerator::CreateUniqueName(
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `GetFullScopeName`, `CreateUniqueName`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`GetFullScopeName`, `CreateUniqueName`。

### Lines 120-129
```cpp
    std::unordered_map<std::string, size_t>& base_name_count,
    std::string base_name) {
  if (base_name_count.find(base_name) == base_name_count.end()) {
    base_name_count[base_name] = 0;
  } else {
    auto count = ++base_name_count[base_name];
    base_name += "_";
    base_name += std::to_string(count);
  }
  return base_name;
```
- EN: This block handles conditional branches; produces a result or forwards a computed value. Key symbols: `find`, `end`, `to_string`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值。关键符号：`find`, `end`, `to_string`。

### Lines 130-136
```cpp
}

bool NodeNameGenerator::IsGraphOutput(
    const Value* v,
    const std::shared_ptr<Graph>& graph) const {
  for (const auto* graph_output : graph->outputs()) {
    if (v == graph_output) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `IsGraphOutput`, `outputs`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`IsGraphOutput`, `outputs`。

### Lines 137-143
```cpp
      return true;
    }
  }
  return false;
}

void NodeNameGenerator::UpdateOutputsNames(Node* n) {
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `UpdateOutputsNames`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`UpdateOutputsNames`。

### Lines 144-156
```cpp
  if (node_names_.find(n) != node_names_.end()) {
    auto node_name = node_names_[n];
    for (auto i : c10::irange(n->outputs().size())) {
      auto output = n->output(i);
      if (!IsGraphOutput(output, graph_)) {
        auto output_name = node_name;
        output_name.append("_output_").append(std::to_string(i));
        output->setDebugName(output_name);
      }
    }
  }
}

```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `find`, `end`, `irange`, `outputs`, `size`, `output`, `...`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`find`, `end`, `irange`, `outputs`, `size`, `output`, `...`。

### Lines 157-163
```cpp
void NodeNameGenerator::PopulateNodeNames() {
  PopulateNodeNames(graph_->block());
}

void NodeNameGenerator::PopulateNodeNames(Block* b) {
  for (auto* n : b->nodes()) {
    for (auto* sub_block : n->blocks()) {
```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `PopulateNodeNames`, `block`, `nodes`, `blocks`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`PopulateNodeNames`, `block`, `nodes`, `blocks`。

### Lines 164-170
```cpp
      PopulateNodeNames(sub_block);
    }
    CreateNodeName(n);
    UpdateOutputsNames(n);
  }
}

```
- EN: This block manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `PopulateNodeNames`, `CreateNodeName`, `UpdateOutputsNames`.
- CN: 该代码块操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`PopulateNodeNames`, `CreateNodeName`, `UpdateOutputsNames`。

### Lines 171-180
```cpp
void ScopedNodeNameGenerator::CreateNodeName(Node* n) {
  if (node_names_.find(n) == node_names_.end()) {
    if (!ONNXScopeName::isCompatibleScope(n->scope())) {
      return;
    }
    if (n->mustBeNone()) {
      // JIT IR does not allow attribute for None node.
      return;
    }
    auto name = GetFullScopeName(n->scope());
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `CreateNodeName`, `find`, `end`, `isCompatibleScope`, `scope`, `mustBeNone`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`CreateNodeName`, `find`, `end`, `isCompatibleScope`, `scope`, `mustBeNone`, `...`。

### Lines 181-187
```cpp
    name += layer_separator_;
    name += n->kind().toUnqualString();
    node_names_[n] = CreateUniqueName(base_node_name_counts_, name);
  }
  n->s_(Symbol::attr(::torch::onnx::kOnnxNodeNameAttribute), node_names_[n]);
}

```
- EN: This block manipulates TorchScript IR objects. Key symbols: `kind`, `toUnqualString`, `CreateUniqueName`, `s_`, `attr`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`kind`, `toUnqualString`, `CreateUniqueName`, `s_`, `attr`。

### Lines 188-195
```cpp
std::string ScopedNodeNameGenerator::GetFullScopeName(const ScopePtr& scope) {
  if (full_scope_names_.find(scope) == full_scope_names_.end()) {
    auto full_scope_name =
        ONNXScopeName::variableNameFromRoot(scope, layer_separator_);
    full_scope_names_[scope] =
        CreateUniqueName(base_scope_name_counts_, full_scope_name);
  }
  return full_scope_names_[scope];
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `GetFullScopeName`, `find`, `end`, `variableNameFromRoot`, `CreateUniqueName`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`GetFullScopeName`, `find`, `end`, `variableNameFromRoot`, `CreateUniqueName`。

### Lines 196-204
```cpp
}

} // namespace

void AssignScopedNamesForNodeAndValue(std::shared_ptr<Graph>& graph) {
  auto node_name_generator = std::make_unique<ScopedNodeNameGenerator>(graph);
  node_name_generator->PopulateNodeNames();
}

```
- EN: This block manipulates TorchScript IR objects. Key symbols: `AssignScopedNamesForNodeAndValue`, `PopulateNodeNames`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`AssignScopedNamesForNodeAndValue`, `PopulateNodeNames`。

### Lines 205-205
```cpp
} // namespace torch::jit::onnx
```
- EN: Opens or closes namespace scopes so the remaining declarations live in the expected JIT/frontend module.
- CN: 打开或关闭命名空间作用域，使后续声明归属于预期的 JIT/前端模块。


## Key Concepts / 关键概念
- EN: IR graph manipulation — the file inspects or rewrites TorchScript graphs and nodes.
  CN: IR 图操作——该文件会检查或改写 TorchScript 的图、节点与块。
- EN: Parsing pipeline — the logic translates source text into TorchScript syntax structures.
  CN: 解析流水线——相关逻辑将源码文本转换为 TorchScript 语法结构。
- EN: Export interop — the implementation helps bridge TorchScript graphs to ONNX-style representations.
  CN: 导出互操作——实现帮助 TorchScript 图与 ONNX 风格表示之间建立桥接。
- EN: Pass orchestration — the file plugs logic into the TorchScript optimization pipeline.
  CN: Pass 编排——该文件把相关逻辑接入 TorchScript 优化流水线。
- EN: Subsystem integration — the implementation collaborates with nearby JIT/frontend headers and runtime helpers.
  CN: 子系统集成——实现会与附近的 JIT/前端头文件以及运行时辅助模块协同工作。

## Dependencies / 依赖关系
- Internal includes / 内部头文件: `torch/csrc/jit/passes/onnx/naming.h`, `torch/csrc/onnx/onnx.h`
- External includes / 外部头文件: `utility`
- Namespaces / 命名空间: `torch::jit::onnx`, `ONNXScopeName`
- Representative symbols / 代表性符号: `string`, `NameFunc`, `nameFromRoot`, `isRoot`, `parent`, `isCompatibleScope`, `append`, `parseNameFromScope`, `name`, `toUnqualString`, `...`
