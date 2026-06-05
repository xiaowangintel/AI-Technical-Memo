# list_model_parameters.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `torch/csrc/jit/passes/onnx/list_model_parameters.cpp`
- Repository: `pytorch`
- Purpose (EN): Implements the TorchScript JIT pass logic for list model parameters, including graph analysis and rewrites.
- 用途 (CN): 实现与 list model parameters 相关的 TorchScript JIT pass 逻辑，包括图分析与改写。

## Line-by-Line Analysis / 逐行分析
### Lines 1-5
```cpp
#include <torch/csrc/jit/frontend/error_report.h>
#include <torch/csrc/jit/jit_log.h>
#include <torch/csrc/jit/passes/onnx/list_model_parameters.h>

namespace torch::jit {
```
- EN: Opens or closes namespace scopes so the remaining declarations live in the expected JIT/frontend module.
- CN: 打开或关闭命名空间作用域，使后续声明归属于预期的 JIT/前端模块。

### Lines 6-10
```cpp

namespace onnx {
using namespace ::c10::onnx;
}

```
- EN: Opens or closes namespace scopes so the remaining declarations live in the expected JIT/frontend module.
- CN: 打开或关闭命名空间作用域，使后续声明归属于预期的 JIT/前端模块。

### Lines 11-20
```cpp
// findSubModuleAttr function chases getAttr chains backwards to locate the
// submodules. For example: module M {
//   attributes {
//     A = <SubModule at ...>
//   }
//   ...
//   %A = prim::GetAttr[name="A"](%self)
//   ...
//   %B = prim::GetAttr[name="B"](%A)
//   ...
```
- EN: Documents the intent, constraints, or compatibility notes that guide the surrounding implementation.
- CN: 说明周围实现所遵循的目标、限制条件或兼容性要求。

### Lines 21-25
```cpp
//   %weight = prim::GetAttr[name="scale"](%B)
//   ...
static std::deque<std::string> findSubModuleAttr(
    Value* input,
    std::string& name,
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `findSubModuleAttr`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`findSubModuleAttr`。

### Lines 26-30
```cpp
    Module& attrModule,
    std::shared_ptr<Graph>& graph) {
  Node* node = input->node();
  std::deque<std::string> moduleNames;

```
- EN: This block manipulates TorchScript IR objects. Key symbols: `node`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`node`。

### Lines 31-35
```cpp
  // Loop starts from inner submodule and follows the chain until reaches the
  // top module.

  while (node->outputs().at(0)->type() != graph->inputs().at(0)->type()) {
    if (node->kind() == prim::GetAttr) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `outputs`, `type`, `inputs`, `kind`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`outputs`, `type`, `inputs`, `kind`。

### Lines 36-43
```cpp
      moduleNames.push_front(node->s(attr::name));
      node = node->inputs()[0]->node();
    } else {
      return moduleNames;
    }
  }
  // Assign the inner module to attrModule.
  for (auto& moduleName : moduleNames) {
```
- EN: This block iterates over collections or graph structures; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `push_front`, `s`, `inputs`, `node`.
- CN: 该代码块遍历集合或图结构；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`push_front`, `s`, `inputs`, `node`。

### Lines 44-48
```cpp
    attrModule = attrModule.attr(moduleName).toModule();
  }
  return moduleNames;
}

```
- EN: This block produces a result or forwards a computed value. Key symbols: `attr`, `toModule`.
- CN: 该代码块返回结果或转发已计算的值。关键符号：`attr`, `toModule`。

### Lines 49-53
```cpp
static Value* addParamAsArgument(
    Function* function,
    std::string& name,
    IValue& attr) {
  auto schema = function->getSchema();
```
- EN: This block manipulates TorchScript IR objects; advances frontend parsing or lowering state. Key symbols: `addParamAsArgument`, `getSchema`.
- CN: 该代码块操作 TorchScript IR 对象；推进前端解析或降级状态。关键符号：`addParamAsArgument`, `getSchema`。

### Lines 54-63
```cpp
  auto args = schema.arguments();
  args.emplace_back(name, nullptr, std::nullopt, attr);
  auto new_schema = FunctionSchema(
      schema.name(),
      schema.overload_name(),
      args,
      schema.returns(),
      schema.is_vararg(),
      schema.is_varret());
  function->setSchema(new_schema);
```
- EN: This block advances frontend parsing or lowering state. Key symbols: `arguments`, `emplace_back`, `FunctionSchema`, `name`, `overload_name`, `returns`, `...`.
- CN: 该代码块推进前端解析或降级状态。关键符号：`arguments`, `emplace_back`, `FunctionSchema`, `name`, `overload_name`, `returns`, `...`。

### Lines 64-68
```cpp
  return toGraphFunction(*function).graph()->addInput(name)->setType(
      attr.type());
}

static std::vector<IValue> getParamAttributes(
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `toGraphFunction`, `graph`, `addInput`, `setType`, `type`, `getParamAttributes`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`toGraphFunction`, `graph`, `addInput`, `setType`, `type`, `getParamAttributes`。

### Lines 69-73
```cpp
    Block* block,
    std::shared_ptr<Graph>& graph,
    const Module& module_,
    Function* function_,
    std::unordered_map<std::string, Value*>& attrValues) {
```
- EN: This block manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: no dominant local symbol names.
- CN: 该代码块操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：无明显局部符号。

### Lines 74-78
```cpp
  auto isEval = !module_.hasattr("training") || !module_.is_training();

  Node* m = *block->nodes().begin();
  WithInsertPoint guard(m);

```
- EN: This block manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `hasattr`, `is_training`, `nodes`, `begin`, `guard`.
- CN: 该代码块操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`hasattr`, `is_training`, `nodes`, `begin`, `guard`。

### Lines 79-84
```cpp
  std::vector<IValue> parameterIValues = {};
  std::unordered_set<Node*> nodesToDestroy;
  for (auto it = block->nodes().begin(); it != block->nodes().end();) {
    Node* n = *it;
    it++; // node n can be destroyed

```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `nodes`, `begin`, `end`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`nodes`, `begin`, `end`。

### Lines 85-93
```cpp
    if (n->kind() == prim::GetAttr || n->kind() == prim::SetAttr) {
      if (n->kind() == prim::GetAttr) {
        for (auto use : n->output()->uses()) {
          if (use.user->kind() == prim::PythonOp)
            throw ErrorReport(n->sourceRange())
                << "Couldn't export Python method.";
        }
      }

```
- EN: This block handles conditional branches; iterates over collections or graph structures. Key symbols: `kind`, `output`, `uses`, `ErrorReport`, `sourceRange`.
- CN: 该代码块处理条件分支；遍历集合或图结构。关键符号：`kind`, `output`, `uses`, `ErrorReport`, `sourceRange`。

### Lines 94-98
```cpp
      auto name = n->s(attr::name);
      auto attrModule = module_;
      auto input = n->inputs()[0];

      auto moduleNames = findSubModuleAttr(input, name, attrModule, graph);
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `s`, `inputs`, `findSubModuleAttr`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`s`, `inputs`, `findSubModuleAttr`。

### Lines 99-103
```cpp
      if (!attrModule.hasattr(name))
        continue;
      auto attr = attrModule.attr(name);
      Value* paramConst = nullptr;

```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `hasattr`, `attr`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`hasattr`, `attr`。

### Lines 104-109
```cpp
      std::string fullName;
      for (auto& name : moduleNames) {
        fullName += name + '.';
      }
      fullName += name;

```
- EN: This block iterates over collections or graph structures. Key symbols: no dominant local symbol names.
- CN: 该代码块遍历集合或图结构。关键符号：无明显局部符号。

### Lines 110-115
```cpp
      auto type = attrModule.type();
      auto slot = *type->findAttributeSlot(name);

      // Add model_parameters and model_buffers as model inputs. Order is
      // preserved based on the appearance in the graph.
      if (type->is_parameter(slot) || type->is_buffer(slot) ||
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `type`, `findAttributeSlot`, `is_parameter`, `is_buffer`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`type`, `findAttributeSlot`, `is_parameter`, `is_buffer`。

### Lines 116-121
```cpp
          (attr.isObject() && !attr.toObjectRef().type()->is_module()) ||
          attr.isBool()) {
        if (attrValues.find(fullName) == attrValues.end() &&
            attr.isTensor()) { // TODO: Handle float/int
          TORCH_INTERNAL_ASSERT(attr.isTensor());
          auto tensor_ = attr.toTensor();
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `isObject`, `toObjectRef`, `type`, `is_module`, `isBool`, `find`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`isObject`, `toObjectRef`, `type`, `is_module`, `isBool`, `find`, `...`。

### Lines 122-131
```cpp
          if (isEval && tensor_.requires_grad()) {
            tensor_ = tensor_.detach();
            tensor_.set_requires_grad(false);
            attr = IValue(tensor_);
          }
          parameterIValues.emplace_back(attr.toTensor());
          paramConst = addParamAsArgument(function_, fullName, attr);
          attrValues.insert({fullName, paramConst});
        } else if (attr.isObject() && !attr.toObjectRef().type()->is_module()) {
          // Only below registered torch classes are supported.
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `requires_grad`, `detach`, `set_requires_grad`, `IValue`, `emplace_back`, `toTensor`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`requires_grad`, `detach`, `set_requires_grad`, `IValue`, `emplace_back`, `toTensor`, `...`。

### Lines 132-141
```cpp
          try {
            parameterIValues.emplace_back(
                script::Object(attr.toObject()).run_method("__getstate__"));
            paramConst = addParamAsArgument(function_, fullName, attr);
            attrValues.insert({fullName, paramConst});
          } catch (const std::exception&) {
            throw ErrorReport(n->sourceRange())
                << "Unknown type " << attr.type()->repr_str()
                << " encountered in handling model params."
                << " This class type does not extend __getstate__ method.";
```
- EN: Declares core types or data containers for this file. Prominent symbols: `emplace_back`, `Object`, `toObject`, `run_method`, `addParamAsArgument`, `insert`, `...`.
- CN: 声明该文件使用的核心类型或数据容器。主要符号：`emplace_back`, `Object`, `toObject`, `run_method`, `addParamAsArgument`, `insert`, `...`。

### Lines 142-151
```cpp
          }
        } else if (attr.isNone() || (attr.isBool() && name == "training")) {
          // This attr is constant for ONNX.
          auto attrVal = tryInsertConstant(*graph, attr);
          n->output()->replaceAllUsesWith(*attrVal);
          nodesToDestroy.emplace(n);
        }
      }
    }

```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `isNone`, `isBool`, `tryInsertConstant`, `output`, `replaceAllUsesWith`, `emplace`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`isNone`, `isBool`, `tryInsertConstant`, `output`, `replaceAllUsesWith`, `emplace`。

### Lines 152-156
```cpp
    for (Block* sub_block : n->blocks()) {
      auto nextParameterIValues =
          getParamAttributes(sub_block, graph, module_, function_, attrValues);
      parameterIValues.insert(
          std::end(parameterIValues),
```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `blocks`, `getParamAttributes`, `insert`, `end`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`blocks`, `getParamAttributes`, `insert`, `end`。

### Lines 157-161
```cpp
          std::begin(nextParameterIValues),
          std::end(nextParameterIValues));
    }
  }
  for (auto n : nodesToDestroy) {
```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `begin`, `end`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`begin`, `end`。

### Lines 162-166
```cpp
    n->destroy();
  }
  return parameterIValues;
}

```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `destroy`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`destroy`。

### Lines 167-175
```cpp
static void insertMainModuleAsConstant(const std::shared_ptr<Graph>& graph) {
  auto* constNode = graph->create(prim::CreateObject);
  constNode->output()->setType(graph->inputs().at(0)->type());
  auto it = graph->nodes().begin();
  constNode->insertBefore(*it);
  graph->inputs().at(0)->replaceAllUsesWith(constNode->output());
  graph->eraseInput(0);
}

```
- EN: This block manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `insertMainModuleAsConstant`, `create`, `output`, `setType`, `inputs`, `type`, `...`.
- CN: 该代码块操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`insertMainModuleAsConstant`, `create`, `output`, `setType`, `inputs`, `type`, `...`。

### Lines 176-180
```cpp
std::pair<Module, std::vector<IValue>> list_module_parameters(
    const Module& module) {
  Module moduleClone = module.clone(true);
  Method method = moduleClone.get_method("forward");
  auto function = &method.function();
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `list_module_parameters`, `clone`, `get_method`, `function`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`list_module_parameters`, `clone`, `get_method`, `function`。

### Lines 181-186
```cpp
  auto graph = toGraphFunction(*function).graph();
  // A map of names and values of referenced attributes, to avoid duplicates.
  std::unordered_map<std::string, Value*> attrValues = {};

  GRAPH_DEBUG("Fetch attributes for function: " + function->name());
  std::vector<IValue> parameterIValues = getParamAttributes(
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `toGraphFunction`, `graph`, `name`, `getParamAttributes`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`toGraphFunction`, `graph`, `name`, `getParamAttributes`。

### Lines 187-191
```cpp
      graph->block(), graph, moduleClone, function, attrValues);
  insertMainModuleAsConstant(graph);
  GRAPH_DEBUG("Listed parameters as inputs: ", *graph);

  return std::make_pair(moduleClone, parameterIValues);
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting; protects shared state or ordering assumptions. Key symbols: `block`, `insertMainModuleAsConstant`, `make_pair`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写；保护共享状态或执行顺序假设。关键符号：`block`, `insertMainModuleAsConstant`, `make_pair`。

### Lines 192-194
```cpp
}

} // namespace torch::jit
```
- EN: Opens or closes namespace scopes so the remaining declarations live in the expected JIT/frontend module.
- CN: 打开或关闭命名空间作用域，使后续声明归属于预期的 JIT/前端模块。


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
- Internal includes / 内部头文件: `torch/csrc/jit/frontend/error_report.h`, `torch/csrc/jit/jit_log.h`, `torch/csrc/jit/passes/onnx/list_model_parameters.h`
- External includes / 外部头文件: 无
- Namespaces / 命名空间: `torch::jit`, `onnx`
- Representative symbols / 代表性符号: `namespace`, `findSubModuleAttr`, `node`, `outputs`, `type`, `inputs`, `kind`, `push_front`, `s`, `attr`, `...`
