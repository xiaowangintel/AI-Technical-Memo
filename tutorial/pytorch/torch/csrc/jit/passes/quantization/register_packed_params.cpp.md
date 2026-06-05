# register_packed_params.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `torch/csrc/jit/passes/quantization/register_packed_params.cpp`
- Repository: `pytorch`
- Purpose (EN): Implements the TorchScript JIT pass logic for register packed params, including graph analysis and rewrites.
- 用途 (CN): 实现与 register packed params 相关的 TorchScript JIT pass 逻辑，包括图分析与改写。

## Line-by-Line Analysis / 逐行分析
### Lines 1-9
```cpp
#include <stack>

#include <ATen/ATen.h>
#include <torch/csrc/jit/api/module.h>
#include <torch/csrc/jit/passes/constant_pooling.h>
#include <torch/csrc/jit/passes/constant_propagation.h>
#include <torch/csrc/jit/passes/quantization/helper.h>
#include <torch/csrc/jit/passes/quantization/register_packed_params.h>

```
- EN: Pulls in the headers needed by the register packed params logic. Internal dependencies: `ATen/ATen.h`, `torch/csrc/jit/api/module.h`, `torch/csrc/jit/passes/constant_pooling.h`, `torch/csrc/jit/passes/constant_propagation.h`, `torch/csrc/jit/passes/quantization/helper.h`, `...`; external dependencies: `stack`.
- CN: 为 register packed params 相关逻辑引入所需头文件。内部依赖：`ATen/ATen.h`, `torch/csrc/jit/api/module.h`, `torch/csrc/jit/passes/constant_pooling.h`, `torch/csrc/jit/passes/constant_propagation.h`, `torch/csrc/jit/passes/quantization/helper.h`, `...`；外部依赖：`stack`。

### Lines 10-14
```cpp
namespace torch::jit {

namespace {
bool isPrepackNode(Node* n) {
  return (
```
- EN: Opens or closes namespace scopes so the remaining declarations live in the expected JIT/frontend module.
- CN: 打开或关闭命名空间作用域，使后续声明归属于预期的 JIT/前端模块。

### Lines 15-24
```cpp
      n->kind() == Symbol::fromQualString("quantized::linear_prepack") ||
      n->kind() == Symbol::fromQualString("quantized::conv1d_prepack") ||
      n->kind() == Symbol::fromQualString("quantized::conv2d_prepack") ||
      n->kind() == Symbol::fromQualString("quantized::conv3d_prepack") ||
      n->kind() ==
          Symbol::fromQualString("quantized::conv_transpose1d_prepack") ||
      n->kind() ==
          Symbol::fromQualString("quantized::conv_transpose2d_prepack"));
}

```
- EN: This block implements local helper logic for register packed params. Key symbols: `kind`, `fromQualString`.
- CN: 该代码块实现与 register packed params 相关的局部辅助逻辑。关键符号：`kind`, `fromQualString`。

### Lines 25-29
```cpp
std::pair<Value*, std::string> findFPWeight(Node* prepack_node) {
  TORCH_CHECK(isPrepackNode(prepack_node));
  Node* n = nullptr;
  n = prepack_node->input(0)->node();
  bool is_quantize_node =
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `findFPWeight`, `isPrepackNode`, `input`, `node`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`findFPWeight`, `isPrepackNode`, `input`, `node`。

### Lines 30-37
```cpp
      (n->kind() == Symbol::fromQualString("aten::quantize_per_tensor") ||
       n->kind() == Symbol::fromQualString("aten::quantize_per_channel"));
  TORCH_CHECK(
      is_quantize_node,
      "Input to prepack node must be output of weight quantization.");
  // First input of quantize node is FP32 weight
  n = n->input(0)->node();
  bool is_getattr_node = (n->kind() == prim::GetAttr);
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `kind`, `fromQualString`, `input`, `node`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`kind`, `fromQualString`, `input`, `node`。

### Lines 38-44
```cpp
  if (is_getattr_node) {
    return {n->input(0), n->s(attr::name)};
  }
  return {nullptr, "AttributeDoesNotExist"};
}
} // namespace

```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `input`, `s`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`input`, `s`。

### Lines 45-50
```cpp
std::string joinPaths(const std::vector<std::string>& paths) {
  std::string path;
  for (const auto& p : paths) {
    path.append(p).append(".");
  }
  return path;
```
- EN: This block iterates over collections or graph structures; produces a result or forwards a computed value. Key symbols: `joinPaths`, `append`.
- CN: 该代码块遍历集合或图结构；返回结果或转发已计算的值。关键符号：`joinPaths`, `append`。

### Lines 51-59
```cpp
}
// Must run this pass after constant folding.
std::unordered_set<std::string> RegisterPrePackParams(
    Module& m,
    const std::string& method_name,
    const PrePackParamFilterFn& is_packed_param,
    const std::string& attr_prefix) {
  int64_t uid = 0; // int + method name gives unique identifier
  auto graph = m.get_method(method_name).graph();
```
- EN: This block manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `RegisterPrePackParams`, `get_method`, `graph`.
- CN: 该代码块操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`RegisterPrePackParams`, `get_method`, `graph`。

### Lines 60-64
```cpp
  std::stack<Block*> blocks_to_visit;
  blocks_to_visit.push(graph->block());
  std::string attr_name_base =
      attr_prefix + "_" + method_name + "_ondevice_ptq_packed_weight_";
  std::unordered_set<std::string> packed_param_names;
```
- EN: This block manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `push`, `block`.
- CN: 该代码块操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`push`, `block`。

### Lines 65-69
```cpp

  while (!blocks_to_visit.empty()) {
    Block* b = blocks_to_visit.top();
    blocks_to_visit.pop();
    for (Node* n : b->nodes()) {
```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `empty`, `top`, `pop`, `nodes`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`empty`, `top`, `pop`, `nodes`。

### Lines 70-74
```cpp
      if (is_packed_param(n)) {
        WithInsertPoint ins(n->next());
        Value* packed_param_value = n->output(0);
        TORCH_CHECK(n->outputs().size() == 1, "Prepack ops have single output");
        auto attr_name = attr_name_base + std::to_string(uid++);
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `is_packed_param`, `ins`, `next`, `output`, `outputs`, `size`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`is_packed_param`, `ins`, `next`, `output`, `outputs`, `size`, `...`。

### Lines 75-79
```cpp
        TORCH_CHECK(
            packed_param_value->uses().size() == 1,
            "Packed param must be used by exactly one op.");
        auto use = packed_param_value->uses()[0];
        while (m.hasattr(attr_name)) {
```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `uses`, `size`, `hasattr`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`uses`, `size`, `hasattr`。

### Lines 80-89
```cpp
          attr_name = attr_name_base + "_" + std::to_string(uid++);
        }
        // Now register attribute for this packed param but dont set it to any
        // value. No value because we dont know what the value is at this point.
        // Only when we run on-device ptq workflow, e.g. run quantize_forward
        // method, is when the linear_prepack op will be executed and at that
        // point we will have the actual value for this attribute.
        m.register_attribute(attr_name, n->output(0)->type(), IValue());
        // In order to add the output of linear_prepack, we now have to do
        // setAttr Thus when quantize_forward is actually called the attribute
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `to_string`, `register_attribute`, `output`, `type`, `IValue`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`to_string`, `register_attribute`, `output`, `type`, `IValue`。

### Lines 90-99
```cpp
        // is appropriately set.
        Node* set_attr = graph->createSetAttr(
            graph->inputs()[0], attr_name, packed_param_value);
        set_attr->insertAfter(n);
        // Now let's add GetAttr for the same attribute.
        // Why?
        // Because eventually the method being modified will be cloned into
        // quantize_forward and quantized_forward.
        // quantize_forward will only have, for example, linear_prepack and
        // SetAttr Thus when quantize_forward is run attributes on the module
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `createSetAttr`, `inputs`, `insertAfter`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`createSetAttr`, `inputs`, `insertAfter`。

### Lines 100-105
```cpp
        // are set. Then in quantized_forward we will actually get
        // packed_params, via GetAttr and supply it to, for example,
        // dynamic_linear At the end quantize_forward will not have any ops like
        // dynamic_linear and quantized_forward will not have any linear_prepack
        // or SetAttr
        Value* packed_param_attr =
```
- EN: This block manipulates TorchScript IR objects. Key symbols: no dominant local symbol names.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：无明显局部符号。

### Lines 106-115
```cpp
            graph->insertGetAttr(graph->inputs()[0], attr_name)
                ->setType(n->output(0)->type());
        // We must replace this specific usage and we cannot doe
        // replaceAllUsesWith This is because we first had to insert SetAttr
        // node. This also takes as input packed_param_value, similar to the
        // actual op. But only the use of the actual op must be replaced by
        // output of GetAttr. Input of SetAttr still must use the
        // packed_param_value
        use.user->replaceInput(use.offset, packed_param_attr);
        // Record the name of the attribute so that we can delete the SetAttr
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `insertGetAttr`, `inputs`, `setType`, `output`, `type`, `replaceInput`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`insertGetAttr`, `inputs`, `setType`, `output`, `type`, `replaceInput`。

### Lines 116-121
```cpp
        // for it
        packed_param_names.insert(std::move(attr_name));

        // Now make sure that original weight is reset such that the module
        // does not have weight attribute set anymore
        auto value_weight_names_pair = findFPWeight(n);
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `insert`, `move`, `findFPWeight`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`insert`, `move`, `findFPWeight`。

### Lines 122-126
```cpp
        Value* v = value_weight_names_pair.first;
        std::string weight_name = std::move(value_weight_names_pair.second);
        auto empty_tensor =
            at::empty({0}, at::TensorOptions().requires_grad(false));
        Node* none_node = graph->create(prim::Constant);
```
- EN: This block manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `move`, `empty`, `TensorOptions`, `requires_grad`, `create`.
- CN: 该代码块操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`move`, `empty`, `TensorOptions`, `requires_grad`, `create`。

### Lines 127-135
```cpp
        none_node->t_(attr::value, empty_tensor);
        // none_node->output()->setType(TensorType::create(at::kFloat,
        // c10::kCPU, 1, false));
        Node* set_attr_orig_weight =
            graph->createSetAttr(v, weight_name, none_node->output());
        set_attr_orig_weight->insertAfter(packed_param_attr->node());
        none_node->insertBefore(set_attr_orig_weight);
        auto* self = v->owningGraph()->inputs()[0];
        std::vector<std::string> path = getModuleAccessPath(v, self);
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `t_`, `createSetAttr`, `output`, `insertAfter`, `node`, `insertBefore`, `...`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`t_`, `createSetAttr`, `output`, `insertAfter`, `node`, `insertBefore`, `...`。

### Lines 136-143
```cpp
        packed_param_names.emplace(joinPaths(path));
      }
      for (Block* subblock : n->blocks()) {
        blocks_to_visit.push(subblock);
      }
    }
  }
  return packed_param_names;
```
- EN: This block iterates over collections or graph structures; produces a result or forwards a computed value; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `emplace`, `joinPaths`, `blocks`, `push`.
- CN: 该代码块遍历集合或图结构；返回结果或转发已计算的值；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`emplace`, `joinPaths`, `blocks`, `push`。

### Lines 144-146
```cpp
}

} // namespace torch::jit
```
- EN: Opens or closes namespace scopes so the remaining declarations live in the expected JIT/frontend module.
- CN: 打开或关闭命名空间作用域，使后续声明归属于预期的 JIT/前端模块。


## Key Concepts / 关键概念
- EN: IR graph manipulation — the file inspects or rewrites TorchScript graphs and nodes.
  CN: IR 图操作——该文件会检查或改写 TorchScript 的图、节点与块。
- EN: Optimization pass — the code simplifies graphs by removing redundant work or folding values.
  CN: 优化 pass——代码通过删除冗余计算或常量折叠来简化图。
- EN: Type and shape reasoning — the code propagates static information across the IR.
  CN: 类型与形状推理——代码会在 IR 中传播静态信息。
- EN: Subsystem integration — the implementation collaborates with nearby JIT/frontend headers and runtime helpers.
  CN: 子系统集成——实现会与附近的 JIT/前端头文件以及运行时辅助模块协同工作。

## Dependencies / 依赖关系
- Internal includes / 内部头文件: `ATen/ATen.h`, `torch/csrc/jit/api/module.h`, `torch/csrc/jit/passes/constant_pooling.h`, `torch/csrc/jit/passes/constant_propagation.h`, `torch/csrc/jit/passes/quantization/helper.h`, `torch/csrc/jit/passes/quantization/register_packed_params.h`
- External includes / 外部头文件: `stack`
- Namespaces / 命名空间: `torch::jit`
- Representative symbols / 代表性符号: `isPrepackNode`, `kind`, `fromQualString`, `findFPWeight`, `input`, `node`, `s`, `joinPaths`, `append`, `RegisterPrePackParams`, `...`
