# cast_all_constant_to_floating.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `torch/csrc/jit/passes/onnx/cast_all_constant_to_floating.cpp`
- Repository: `pytorch`
- Purpose (EN): Implements the TorchScript JIT pass logic for cast all constant to floating, including graph analysis and rewrites.
- 用途 (CN): 实现与 cast all constant to floating 相关的 TorchScript JIT pass 逻辑，包括图分析与改写。

## Line-by-Line Analysis / 逐行分析
### Lines 1-3
```cpp
#include <torch/csrc/jit/passes/onnx/cast_all_constant_to_floating.h>
#include <torch/csrc/jit/passes/onnx/helper.h>

```
- EN: Pulls in the headers needed by the cast all constant to floating logic. Internal dependencies: `torch/csrc/jit/passes/onnx/cast_all_constant_to_floating.h`, `torch/csrc/jit/passes/onnx/helper.h`; external dependencies: none.
- CN: 为 cast all constant to floating 相关逻辑引入所需头文件。内部依赖：`torch/csrc/jit/passes/onnx/cast_all_constant_to_floating.h`, `torch/csrc/jit/passes/onnx/helper.h`；外部依赖：无。

### Lines 4-8
```cpp
namespace torch::jit {
namespace onnx {
using namespace ::c10::onnx;
}

```
- EN: Opens or closes namespace scopes so the remaining declarations live in the expected JIT/frontend module.
- CN: 打开或关闭命名空间作用域，使后续声明归属于预期的 JIT/前端模块。

### Lines 9-14
```cpp
// For ONNX opset < 9, constant operator supports only three data types:
// float16, float, and double. Constants of other data types are exported as
// float or double and then cast back to their original data type with a cast
// node. The above transformation is done in this pass. The motivation behind
// having it as a post process pass opposed to handling in symbolic, is that
// many constant operators would have already been removed in the export before
```
- EN: Documents the intent, constraints, or compatibility notes that guide the surrounding implementation.
- CN: 说明周围实现所遵循的目标、限制条件或兼容性要求。

### Lines 15-17
```cpp
// this step. On the other hand if cast is inserted in symbolic, subsequent node
// conversion will break if it depends on certain inputs being constant.
static void CastAllConstantToFloating(Block* block) {
```
- EN: This block manipulates TorchScript IR objects; performs optimization-oriented rewriting; protects shared state or ordering assumptions. Key symbols: `CastAllConstantToFloating`.
- CN: 该代码块操作 TorchScript IR 对象；执行面向优化的改写；保护共享状态或执行顺序假设。关键符号：`CastAllConstantToFloating`。

### Lines 18-20
```cpp
  auto graph = block->owningGraph();
  auto it = block->nodes().begin();
  while (it != block->nodes().end()) {
```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `owningGraph`, `nodes`, `begin`, `end`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`owningGraph`, `nodes`, `begin`, `end`。

### Lines 21-23
```cpp
    auto node = *it;
    ++it;
    for (auto block : node->blocks()) {
```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `blocks`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`blocks`。

### Lines 24-26
```cpp
      CastAllConstantToFloating(block);
    }

```
- EN: This block manipulates TorchScript IR objects; performs optimization-oriented rewriting; protects shared state or ordering assumptions. Key symbols: `CastAllConstantToFloating`.
- CN: 该代码块操作 TorchScript IR 对象；执行面向优化的改写；保护共享状态或执行顺序假设。关键符号：`CastAllConstantToFloating`。

### Lines 27-30
```cpp
    if (node->kind() == onnx::Constant) {
      auto val = node->t(attr::value);
      at::ScalarType dtype = val.scalar_type();
      auto val_type = TensorType::create(val);
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `kind`, `t`, `scalar_type`, `create`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`kind`, `t`, `scalar_type`, `create`。

### Lines 31-36
```cpp
      if (dtype != at::ScalarType::Double && dtype != at::ScalarType::Float &&
          dtype != at::ScalarType::Half) {
        int to_type = 0;
        switch (val.scalar_type()) {
          case at::ScalarType::Byte:
          case at::ScalarType::Char:
```
- EN: This block handles conditional branches. Key symbols: `scalar_type`.
- CN: 该代码块处理条件分支。关键符号：`scalar_type`。

### Lines 37-42
```cpp
          case at::ScalarType::Int:
          case at::ScalarType::Short:
          case at::ScalarType::Bool:
            to_type = ATenTypeToOnnxType(val.scalar_type());
            val = val.to(at::ScalarType::Float);
            break;
```
- EN: This block implements local helper logic for cast all constant to floating. Key symbols: `ATenTypeToOnnxType`, `scalar_type`, `to`.
- CN: 该代码块实现与 cast all constant to floating 相关的局部辅助逻辑。关键符号：`ATenTypeToOnnxType`, `scalar_type`, `to`。

### Lines 43-48
```cpp

          case at::ScalarType::Long:
            to_type = ATenTypeToOnnxType(val.scalar_type());
            val = val.to(at::ScalarType::Double);
            break;

```
- EN: This block implements local helper logic for cast all constant to floating. Key symbols: `ATenTypeToOnnxType`, `scalar_type`, `to`.
- CN: 该代码块实现与 cast all constant to floating 相关的局部辅助逻辑。关键符号：`ATenTypeToOnnxType`, `scalar_type`, `to`。

### Lines 49-54
```cpp
          default:
            throw std::runtime_error("Unsupported types: complex, string");
        }
        // create a cast node
        node->removeAttribute(attr::value);
        node->t_(attr::value, val);
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `runtime_error`, `removeAttribute`, `t_`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`runtime_error`, `removeAttribute`, `t_`。

### Lines 55-60
```cpp
        Node* cast_node = graph->create(onnx::Cast, 1);
        cast_node->i_(attr::to, to_type);
        cast_node->output()->setType(val_type);
        cast_node->insertAfter(node);
        // get input from cast node
        node->outputs().at(0)->replaceAllUsesWith(cast_node->outputs().at(0));
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `create`, `i_`, `output`, `setType`, `insertAfter`, `outputs`, `...`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`create`, `i_`, `output`, `setType`, `insertAfter`, `outputs`, `...`。

### Lines 61-66
```cpp
        // add input from constant to cast node
        cast_node->addInput(node->outputs().at(0));
        cast_node->copyMetadata(node);
      }
    }
  }
```
- EN: This block manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `addInput`, `outputs`, `copyMetadata`.
- CN: 该代码块操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`addInput`, `outputs`, `copyMetadata`。

### Lines 67-69
```cpp
}

void CastAllConstantToFloating(const std::shared_ptr<Graph>& graph) {
```
- EN: This block manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `CastAllConstantToFloating`.
- CN: 该代码块操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`CastAllConstantToFloating`。

### Lines 70-72
```cpp
  CastAllConstantToFloating(graph->block());
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
- EN: Export interop — the implementation helps bridge TorchScript graphs to ONNX-style representations.
  CN: 导出互操作——实现帮助 TorchScript 图与 ONNX 风格表示之间建立桥接。
- EN: Subsystem integration — the implementation collaborates with nearby JIT/frontend headers and runtime helpers.
  CN: 子系统集成——实现会与附近的 JIT/前端头文件以及运行时辅助模块协同工作。

## Dependencies / 依赖关系
- Internal includes / 内部头文件: `torch/csrc/jit/passes/onnx/cast_all_constant_to_floating.h`, `torch/csrc/jit/passes/onnx/helper.h`
- External includes / 外部头文件: 无
- Namespaces / 命名空间: `torch::jit`, `onnx`
- Representative symbols / 代表性符号: `namespace`, `CastAllConstantToFloating`, `owningGraph`, `nodes`, `begin`, `end`, `blocks`, `kind`, `t`, `scalar_type`, `...`
