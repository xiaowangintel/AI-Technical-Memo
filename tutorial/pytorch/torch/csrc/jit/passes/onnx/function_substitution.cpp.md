# function_substitution.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `torch/csrc/jit/passes/onnx/function_substitution.cpp`
- Repository: `pytorch`
- Purpose (EN): Implements the TorchScript JIT pass logic for function substitution, including graph analysis and rewrites.
- 用途 (CN): 实现与 function substitution 相关的 TorchScript JIT pass 逻辑，包括图分析与改写。

## Line-by-Line Analysis / 逐行分析
### Lines 1-6
```cpp
#include <torch/csrc/jit/passes/onnx/function_substitution.h>

#include <torch/csrc/jit/jit_log.h>
#include <torch/csrc/jit/passes/onnx/helper.h>
#include <torch/csrc/jit/passes/onnx/naming.h>

```
- EN: Pulls in the headers needed by the function substitution logic. Internal dependencies: `torch/csrc/jit/passes/onnx/function_substitution.h`, `torch/csrc/jit/jit_log.h`, `torch/csrc/jit/passes/onnx/helper.h`, `torch/csrc/jit/passes/onnx/naming.h`; external dependencies: none.
- CN: 为 function substitution 相关逻辑引入所需头文件。内部依赖：`torch/csrc/jit/passes/onnx/function_substitution.h`, `torch/csrc/jit/jit_log.h`, `torch/csrc/jit/passes/onnx/helper.h`, `torch/csrc/jit/passes/onnx/naming.h`；外部依赖：无。

### Lines 7-12
```cpp
namespace torch::jit {

namespace {

const std::string kTopModuleVariableName;

```
- EN: Opens or closes namespace scopes so the remaining declarations live in the expected JIT/frontend module.
- CN: 打开或关闭命名空间作用域，使后续声明归属于预期的 JIT/前端模块。

### Lines 13-18
```cpp
std::string TidyClassNameFromTorchScript(
    const std::optional<c10::QualifiedName>& class_name) {
  if (!class_name) {
    return "UNKNOWN_CLASS";
  }
  std::string out;
```
- EN: This block handles conditional branches; produces a result or forwards a computed value. Key symbols: `TidyClassNameFromTorchScript`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值。关键符号：`TidyClassNameFromTorchScript`。

### Lines 19-23
```cpp
  for (const auto& atom : class_name->atoms()) {
    bool is_internal_torch_atom = (atom == "__torch__");
    bool is_mangle_atom = (atom.find("__torch_mangle") != std::string::npos);
    if (!is_internal_torch_atom && !is_mangle_atom) {
      if (!out.empty()) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures. Key symbols: `atoms`, `find`, `empty`.
- CN: 该代码块处理条件分支；遍历集合或图结构。关键符号：`atoms`, `find`, `empty`。

### Lines 24-29
```cpp
        out += ".";
      }
      out += atom;
    }
  }
  return out;
```
- EN: This block produces a result or forwards a computed value. Key symbols: no dominant local symbol names.
- CN: 该代码块返回结果或转发已计算的值。关键符号：无明显局部符号。

### Lines 30-36
```cpp
}

std::string GetCallNodeVariableName(const Node* call_node) {
  TORCH_INTERNAL_ASSERT(
      call_node->kind() == prim::CallFunction ||
      call_node->kind() == prim::CallMethod);
  auto module_node = call_node->input(0)->node();
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `GetCallNodeVariableName`, `kind`, `input`, `node`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`GetCallNodeVariableName`, `kind`, `input`, `node`。

### Lines 37-41
```cpp

  if (!module_node->hasAttribute(attr::name)) {
    return "";
  }
  std::string module_name = module_node->s(attr::name);
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `hasAttribute`, `s`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`hasAttribute`, `s`。

### Lines 42-47
```cpp
  if (module_node->inputs().empty()) {
    return module_name;
  }
  // If module is from container, attr::name in module node only carries
  // index info. Need to check parent node (container) for variable name.
  auto parent_module_value = module_node->input(0);
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `inputs`, `empty`, `input`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`inputs`, `empty`, `input`。

### Lines 48-53
```cpp
  while (parent_module_value) {
    auto parent_module_type = parent_module_value->type()->cast<ClassType>();
    if (parent_module_type &&
        parent_module_type->name() ==
            "__torch__.torch.nn.modules.container.ModuleList") {
      auto parent_module_node = parent_module_value->node();
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `type`, `name`, `node`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`type`, `name`, `node`。

### Lines 54-62
```cpp
      module_name = parent_module_node->s(attr::name) + "." + module_name;
      parent_module_value = !parent_module_node->inputs().empty()
          ? parent_module_node->input(0)
          : nullptr;
    } else {
      break;
    }
  }

```
- EN: This block manipulates TorchScript IR objects. Key symbols: `s`, `inputs`, `empty`, `input`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`s`, `inputs`, `empty`, `input`。

### Lines 63-69
```cpp
  return module_name;
}

ScopePtr ForwardCallScope(Graph& graph, Node* call_node) {
  TORCH_INTERNAL_ASSERT(call_node->kind() == prim::CallMethod);
  const std::string& method_name = call_node->s(attr::name);
  if (method_name == "forward") {
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `ForwardCallScope`, `kind`, `s`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`ForwardCallScope`, `kind`, `s`。

### Lines 70-75
```cpp
    const auto type = call_node->input(0)->type()->expect<c10::NamedType>();
    const std::string class_name = TidyClassNameFromTorchScript(type->name());
    const std::string variable_name = GetCallNodeVariableName(call_node);
    const std::string scope_name =
        onnx::ONNXScopeName::createFullScopeName(class_name, variable_name);
    return graph.current_scope()->push(Symbol::scope(scope_name));
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `input`, `type`, `TidyClassNameFromTorchScript`, `name`, `GetCallNodeVariableName`, `createFullScopeName`, `...`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`input`, `type`, `TidyClassNameFromTorchScript`, `name`, `GetCallNodeVariableName`, `createFullScopeName`, `...`。

### Lines 76-80
```cpp
  }
  return graph.current_scope();
}

void functionCallSubstitution(Block* block) {
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `current_scope`, `functionCallSubstitution`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`current_scope`, `functionCallSubstitution`。

### Lines 81-88
```cpp
  auto graph = block->owningGraph();
  for (auto it = block->nodes().begin(), end = block->nodes().end();
       it != end;) {
    Node* cur = *it++;
    switch (cur->kind()) {
      case prim::CallFunction: {
        TORCH_INTERNAL_ASSERT(cur->input(0)->node()->kind() == prim::Constant);
        auto function_constant = cur->input(0)->node();
```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects; performs optimization-oriented rewriting; protects shared state or ordering assumptions. Key symbols: `owningGraph`, `nodes`, `begin`, `end`, `kind`, `input`, `...`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象；执行面向优化的改写；保护共享状态或执行顺序假设。关键符号：`owningGraph`, `nodes`, `begin`, `end`, `kind`, `input`, `...`。

### Lines 89-97
```cpp
        auto fun_type =
            function_constant->output()->type()->expect<FunctionType>();

        if ((fun_type->function()->qualname().qualifiedName().find(
                 "torch.nn.functional") != std::string::npos) &&
            (fun_type->function()->qualname().qualifiedName().find(
                 "interpolate") != std::string::npos)) {
          // Remove input[0] and the node that feeds into it
          auto input_node_0 = cur->input(0)->node();
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `output`, `type`, `function`, `qualname`, `qualifiedName`, `find`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`output`, `type`, `function`, `qualname`, `qualifiedName`, `find`, `...`。

### Lines 98-102
```cpp
          cur->removeInput(0);
          if (!input_node_0->hasUses()) {
            input_node_0->destroy();
          }
          Node* interpolate_node = block->owningGraph()->create(
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `removeInput`, `hasUses`, `destroy`, `owningGraph`, `create`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`removeInput`, `hasUses`, `destroy`, `owningGraph`, `create`。

### Lines 103-112
```cpp
              Symbol::fromQualString("aten::__interpolate"),
              {cur->inputs()},
              cur->outputs().size());
          interpolate_node->output()->copyMetadata(cur->output());
          interpolate_node->insertAfter(cur);
          interpolate_node->copyMetadata(cur);
          cur->replaceAllUsesWith(interpolate_node);
          cur->removeAllInputs();
          cur->destroy();
          GRAPH_UPDATE(
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `fromQualString`, `inputs`, `outputs`, `size`, `output`, `copyMetadata`, `...`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`fromQualString`, `inputs`, `outputs`, `size`, `output`, `copyMetadata`, `...`。

### Lines 113-121
```cpp
              "ONNX function call substitution function: '",
              fun_type->function()->name(),
              "' to aten::__interpolate");
          GRAPH_UPDATE(
              "Function in ONNX function call substitution body: ",
              toGraphFunction(*fun_type->function()).optimized_graph());
        } else {
          // Remove input[0] and the node that feeds into it
          auto input_node_0 = cur->input(0)->node();
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `function`, `name`, `toGraphFunction`, `optimized_graph`, `input`, `node`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`function`, `name`, `toGraphFunction`, `optimized_graph`, `input`, `node`。

### Lines 122-131
```cpp
          cur->removeInput(0);
          if (!input_node_0->hasUses()) {
            input_node_0->destroy();
          }
          auto& graphFunction = toGraphFunction(*fun_type->function());
          functionCallSubstitution(graphFunction.graph()->block());
          inlineCallTo(cur, &graphFunction, false);
        }
      } break;
      case prim::CallMethod: {
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; performs optimization-oriented rewriting; protects shared state or ordering assumptions. Key symbols: `removeInput`, `hasUses`, `destroy`, `toGraphFunction`, `function`, `functionCallSubstitution`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；执行面向优化的改写；保护共享状态或执行顺序假设。关键符号：`removeInput`, `hasUses`, `destroy`, `toGraphFunction`, `function`, `functionCallSubstitution`, `...`。

### Lines 132-140
```cpp
        const std::string& name = cur->s(attr::name);
        if (auto class_type = cur->input(0)->type()->cast<ClassType>()) {
          Function& function = class_type->getMethod(name);
          ScopePtr call_scope = ForwardCallScope(*graph, cur);
          WithCurrentScope scope_guard(*graph, call_scope);
          GRAPH_DEBUG(
              "Setting scope guard for forward call: ",
              graph->current_scope()->namesFromRoot());
          if (auto graphFunction = tryToGraphFunction(function)) {
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `s`, `input`, `type`, `getMethod`, `ForwardCallScope`, `scope_guard`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`s`, `input`, `type`, `getMethod`, `ForwardCallScope`, `scope_guard`, `...`。

### Lines 141-150
```cpp
            GRAPH_DEBUG(
                "Inner graph for method call ",
                name,
                ": ",
                *graphFunction->graph());
            WithCurrentScope inner_graph_scope_guard(
                *graphFunction->graph(), call_scope);
            functionCallSubstitution(graphFunction->graph()->block());
            inlineCallTo(cur, graphFunction, false);
          }
```
- EN: This block manipulates TorchScript IR objects; performs optimization-oriented rewriting; protects shared state or ordering assumptions. Key symbols: `graph`, `inner_graph_scope_guard`, `functionCallSubstitution`, `block`, `inlineCallTo`.
- CN: 该代码块操作 TorchScript IR 对象；执行面向优化的改写；保护共享状态或执行顺序假设。关键符号：`graph`, `inner_graph_scope_guard`, `functionCallSubstitution`, `block`, `inlineCallTo`。

### Lines 151-157
```cpp
        }
      } break;
      default: {
        if (!graph->current_scope()->isBlank()) {
          cur->setScope(graph->current_scope());
        }
        for (auto b : cur->blocks()) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `current_scope`, `isBlank`, `setScope`, `blocks`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`current_scope`, `isBlank`, `setScope`, `blocks`。

### Lines 158-167
```cpp
          functionCallSubstitution(b);
        }
      } break;
    }
    GRAPH_DEBUG(
        "Graph current scope after node process: ",
        graph->current_scope()->namesFromRoot());
  }
}

```
- EN: This block manipulates TorchScript IR objects. Key symbols: `functionCallSubstitution`, `current_scope`, `namesFromRoot`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`functionCallSubstitution`, `current_scope`, `namesFromRoot`。

### Lines 168-172
```cpp
ScopePtr ONNXGraphTopLevelScope(Graph& graph) {
  if (graph.inputs().empty()) {
    return graph.current_scope();
  }
  if (auto top_module_type = graph.inputs().at(0)->type()->cast<ClassType>()) {
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `ONNXGraphTopLevelScope`, `inputs`, `empty`, `current_scope`, `type`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`ONNXGraphTopLevelScope`, `inputs`, `empty`, `current_scope`, `type`。

### Lines 173-178
```cpp
    auto scope_name = ::torch::jit::onnx::ONNXScopeName::createFullScopeName(
        TidyClassNameFromTorchScript(top_module_type->name()),
        kTopModuleVariableName);
    return graph.current_scope()->push(Symbol::scope(scope_name));
  }
  return graph.current_scope();
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `createFullScopeName`, `TidyClassNameFromTorchScript`, `name`, `current_scope`, `push`, `scope`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`createFullScopeName`, `TidyClassNameFromTorchScript`, `name`, `current_scope`, `push`, `scope`。

### Lines 179-188
```cpp
}

} // namespace

// This pass is to be used for ONNX conversion only. The ONNX converter depends
// on a number of deprecated aten operators. These operators are removed from IR
// and replaced by the compiled python function code. However, in-order to
// maintain the behavior for ONNX conversion, we replace these function calls
// with the aten symbolic which can still be used by the ONNX converter.
void ONNXFunctionCallSubstitution(Graph& graph) {
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `ONNXFunctionCallSubstitution`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`ONNXFunctionCallSubstitution`。

### Lines 189-194
```cpp
  GRAPH_DUMP("Before function call substitution calls: ", &graph);
  WithCurrentScope top_level_scope_guard(graph, ONNXGraphTopLevelScope(graph));
  functionCallSubstitution(graph.block());
  GRAPH_DUMP("After function call substitution calls: ", &graph);
}

```
- EN: This block manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `top_level_scope_guard`, `ONNXGraphTopLevelScope`, `functionCallSubstitution`, `block`.
- CN: 该代码块操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`top_level_scope_guard`, `ONNXGraphTopLevelScope`, `functionCallSubstitution`, `block`。

### Lines 195-195
```cpp
} // namespace torch::jit
```
- EN: Opens or closes namespace scopes so the remaining declarations live in the expected JIT/frontend module.
- CN: 打开或关闭命名空间作用域，使后续声明归属于预期的 JIT/前端模块。


## Key Concepts / 关键概念
- EN: IR graph manipulation — the file inspects or rewrites TorchScript graphs and nodes.
  CN: IR 图操作——该文件会检查或改写 TorchScript 的图、节点与块。
- EN: Optimization pass — the code simplifies graphs by removing redundant work or folding values.
  CN: 优化 pass——代码通过删除冗余计算或常量折叠来简化图。
- EN: Inlining — the implementation merges callees or blocks into their call sites.
  CN: 内联——实现会把被调用者或代码块并入调用位置。
- EN: Fusion support — the file prepares groups of operations for fused execution.
  CN: 融合支持——该文件会为融合执行准备算子分组。
- EN: Type and shape reasoning — the code propagates static information across the IR.
  CN: 类型与形状推理——代码会在 IR 中传播静态信息。
- EN: Export interop — the implementation helps bridge TorchScript graphs to ONNX-style representations.
  CN: 导出互操作——实现帮助 TorchScript 图与 ONNX 风格表示之间建立桥接。

## Dependencies / 依赖关系
- Internal includes / 内部头文件: `torch/csrc/jit/passes/onnx/function_substitution.h`, `torch/csrc/jit/jit_log.h`, `torch/csrc/jit/passes/onnx/helper.h`, `torch/csrc/jit/passes/onnx/naming.h`
- External includes / 外部头文件: 无
- Namespaces / 命名空间: `torch::jit`
- Representative symbols / 代表性符号: `TidyClassNameFromTorchScript`, `atoms`, `find`, `empty`, `GetCallNodeVariableName`, `kind`, `input`, `node`, `hasAttribute`, `s`, `...`
