# preprocess_for_onnx.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `torch/csrc/jit/passes/onnx/preprocess_for_onnx.cpp`
- Repository: `pytorch`
- Purpose (EN): Implements the TorchScript JIT pass logic for preprocess for onnx, including graph analysis and rewrites.
- 用途 (CN): 实现与 preprocess for onnx 相关的 TorchScript JIT pass 逻辑，包括图分析与改写。

## Line-by-Line Analysis / 逐行分析
### Lines 1-7
```cpp
#include <torch/csrc/jit/passes/onnx/preprocess_for_onnx.h>

#include <ATen/ScalarOps.h>
#include <c10/util/irange.h>

#include <torch/csrc/jit/jit_log.h>

```
- EN: Pulls in the headers needed by the preprocess for onnx logic. Internal dependencies: `torch/csrc/jit/passes/onnx/preprocess_for_onnx.h`, `ATen/ScalarOps.h`, `c10/util/irange.h`, `torch/csrc/jit/jit_log.h`; external dependencies: none.
- CN: 为 preprocess for onnx 相关逻辑引入所需头文件。内部依赖：`torch/csrc/jit/passes/onnx/preprocess_for_onnx.h`, `ATen/ScalarOps.h`, `c10/util/irange.h`, `torch/csrc/jit/jit_log.h`；外部依赖：无。

### Lines 8-14
```cpp
namespace torch::jit {

namespace onnx {
using namespace ::c10::onnx;
}

namespace {
```
- EN: This block implements local helper logic for preprocess for onnx. Key symbols: `namespace`.
- CN: 该代码块实现与 preprocess for onnx 相关的局部辅助逻辑。关键符号：`namespace`。

### Lines 15-22
```cpp

std::optional<Node*> FindFusibleListUnpack(Node* n) {
  // 1. number of outputs is restricted to 1.
  // 2. output is only used by prim::ListUnpack.
  if (n->outputs().size() != 1) {
    return std::nullopt;
  }
  if (n->output()->uses().size() != 1) {
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `FindFusibleListUnpack`, `outputs`, `size`, `output`, `uses`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`FindFusibleListUnpack`, `outputs`, `size`, `output`, `uses`。

### Lines 23-29
```cpp
    return std::nullopt;
  }
  auto listUnpackNode = n->output()->uses()[0].user;
  if (listUnpackNode->kind() != prim::ListUnpack) {
    return std::nullopt;
  }
  return listUnpackNode;
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `output`, `uses`, `kind`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`output`, `uses`, `kind`。

### Lines 30-43
```cpp
}

// Fuse node + ListUnpack
// Node such as split/unbind produces tensor[] of static size,
// that is later unpacked by ListUnpack.
// This pass fuses the two nodes, and adds an additional input "_outputs" such
// that the symbolic function is aware of the number of outputs.
//
// Example IR
//  split.Tensor(Tensor(a -> *) self, int split_size, int dim=0) -> Tensor[]
//  split_with_sizes(Tensor self, int[] split_sizes, int dim=0) -> Tensor[]
//
// graph(%input : Float(5, 4, 3, strides=[12, 3, 1])):
//   %13 : int[] = prim::Constant[value=[2, 1, 2]]()
```
- EN: This block manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: no dominant local symbol names.
- CN: 该代码块操作 TorchScript IR 对象；执行面向优化的改写。关键符号：无明显局部符号。

### Lines 44-57
```cpp
//   %7 : int = prim::Constant[value=0]()
//   %8 : Tensor[] = aten::split_with_sizes(%input, %13, %7)
//   %9 : Float(2, 4, 3, strides=[12, 3, 1]), %10 : Float(1, 4, 3, strides=[12,
//   3, 1]), %11 : Float(2, 4, 3, strides=[12, 3, 1]) = prim::ListUnpack(%8)
//   return (%9, %10, %11)
//
// After fusion
// graph(%input : Float(5, 4, 3, strides=[12, 3, 1])):
//   %13 : int[] = prim::Constant[value=[2, 1, 2]]()
//   %7 : int = prim::Constant[value=0]()
//   %8 : int = prim::Constant[value=3]()  # Adding additional input of value 3
//      representing the number of outputs.
//   %14 : Float(2, 4, 3, strides=[12, 3, 1]), %15 : Float(1, 4, 3, strides=[12,
//      3, 1]), %16 : Float(2, 4, 3, strides=[12, 3, 1] =
```
- EN: Documents the intent, constraints, or compatibility notes that guide the surrounding implementation.
- CN: 说明周围实现所遵循的目标、限制条件或兼容性要求。

### Lines 58-64
```cpp
//      aten::split_with_sizes(%input, %13, %7, %8) return (%14, %15, %16)
void FuseWithListUnpack(Node* n) {
  auto found_listUnpack = FindFusibleListUnpack(n);
  if (!found_listUnpack) {
    return;
  }

```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `FuseWithListUnpack`, `FindFusibleListUnpack`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`FuseWithListUnpack`, `FindFusibleListUnpack`。

### Lines 65-75
```cpp
  auto listUnpack_node = found_listUnpack.value();

  TORCH_INTERNAL_ASSERT(n->outputs().size() == 1);
  // 1. Add internal input "_outputs" to node, so that later symbolic function
  //    conversion is aware of the number of outputs.
  // 2. Add the exact number of outputs to n, copy metadata and replace uses of
  //    listUnpack outputs.
  n->i_(
      Symbol::fromQualString("attr::_outputs"),
      static_cast<int64_t>(listUnpack_node->outputs().size()));

```
- EN: This block manipulates TorchScript IR objects. Key symbols: `value`, `outputs`, `size`, `i_`, `fromQualString`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`value`, `outputs`, `size`, `i_`, `fromQualString`。

### Lines 76-85
```cpp
  for (size_t i = 0; i < listUnpack_node->outputs().size(); ++i) {
    auto new_output = n->addOutput();
    new_output->copyMetadata(listUnpack_node->output(i));
  }
  listUnpack_node->removeAllInputs();
  // remove original output, which is input to listUnpack node.
  n->eraseOutput(0);
  listUnpack_node->replaceAllUsesWith(n);
}

```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `outputs`, `size`, `addOutput`, `copyMetadata`, `output`, `removeAllInputs`, `...`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`outputs`, `size`, `addOutput`, `copyMetadata`, `output`, `removeAllInputs`, `...`。

### Lines 86-92
```cpp
static void FuseWithListUnpack(Block* b) {
  for (auto it = b->nodes().begin(), end = b->nodes().end(); it != end; ++it) {
    for (auto* child_block : it->blocks()) {
      FuseWithListUnpack(child_block);
    }

    auto n_kind = it->kind();
```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `FuseWithListUnpack`, `nodes`, `begin`, `end`, `blocks`, `kind`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`FuseWithListUnpack`, `nodes`, `begin`, `end`, `blocks`, `kind`。

### Lines 93-106
```cpp
    switch (n_kind) {
      case aten::split:
      case aten::split_with_sizes:
      case aten::unsafe_split:
      case aten::unsafe_split_with_sizes:
      case aten::unbind:
      case aten::unsafe_chunk:
      case aten::where:
      case aten::nonzero_numpy:
        FuseWithListUnpack(*it);
        break;
      default:
        break;
    }
```
- EN: This block implements local helper logic for preprocess for onnx. Key symbols: `FuseWithListUnpack`.
- CN: 该代码块实现与 preprocess for onnx 相关的局部辅助逻辑。关键符号：`FuseWithListUnpack`。

### Lines 107-120
```cpp
  }
}

// Replace aten::add with onnx::Concat
// when inputs to the add node are two int lists
//
// before the pass:
// graph(%x.1 : Float(2, 3, 4, strides=[12, 4, 1], requires_grad=0, device=cpu),
//  %y.1 : Float(1, 2, 3, strides=[6, 3, 1], requires_grad=0, device=cpu)):
//  %2 : None = prim::Constant()
//  %3 : int[] = aten::size(%x.1)
//  %l1.1 : int[] = aten::list(%3
//  %5 : int[] = aten::size(%y.1)
//  %l2.1 : int[] = aten::list(%5)
```
- EN: This block manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: no dominant local symbol names.
- CN: 该代码块操作 TorchScript IR 对象；执行面向优化的改写。关键符号：无明显局部符号。

### Lines 121-134
```cpp
//  %7 : int[] = aten::add(%l1.1, %l2.1)
//  %8 : Tensor = aten::new_zeros(%x.1, %7, %2, %2, %2, %2)
//  return (%8)
//
// after the pass:
// graph(%x.1 : Float(2, 3, 4, strides=[12, 4, 1], requires_grad=0, device=cpu),
//  %y.1 : Float(1, 2, 3, strides=[6, 3, 1], requires_grad=0, device=cpu)):
//  %2 : None = prim::Constant()
//  %3 : int[] = aten::size(%x.1)
//  %l1.1 : int[] = aten::list(%3)
//  %5 : int[] = aten::size(%y.1)
//  %l2.1 : int[] = aten::list(%5)
//  %9 : Tensor = onnx::Concat[axis=0](%l1.1, %l2.1)
//  %8 : Tensor = aten::new_zeros(%x.1, %9, %2, %2, %2, %2)
```
- EN: Documents the intent, constraints, or compatibility notes that guide the surrounding implementation.
- CN: 说明周围实现所遵循的目标、限制条件或兼容性要求。

### Lines 135-141
```cpp
//  return (%8)
static void ReplaceAddWithConcat(Block* b) {
  for (auto it = b->nodes().begin(), end = b->nodes().end(); it != end; ++it) {
    for (auto* child_block : it->blocks()) {
      ReplaceAddWithConcat(child_block);
    }
    if (it->kind() == aten::add) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; produces a result or forwards a computed value; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `ReplaceAddWithConcat`, `nodes`, `begin`, `end`, `blocks`, `kind`.
- CN: 该代码块处理条件分支；遍历集合或图结构；返回结果或转发已计算的值；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`ReplaceAddWithConcat`, `nodes`, `begin`, `end`, `blocks`, `kind`。

### Lines 142-149
```cpp
      if (!it->input(0)->type()->cast<ListType>() ||
          !it->input(1)->type()->cast<ListType>()) {
        continue;
      }

      const auto& elem =
          it->input(0)->type()->castRaw<ListType>()->getElementType();
      if (elem->cast<IntType>()) {
```
- EN: This block handles conditional branches. Key symbols: `input`, `type`, `getElementType`.
- CN: 该代码块处理条件分支。关键符号：`input`, `type`, `getElementType`。

### Lines 150-163
```cpp
        Node* concat_node = b->owningGraph()->create(onnx::Concat, 1);
        concat_node->i_(attr::axis, 0);
        concat_node->insertBefore(*it);
        concat_node->addInput(it->input(0));
        concat_node->addInput(it->input(1));
        concat_node->outputs()[0]->setType(TensorType::fromNumberType(*elem));
        concat_node->copyMetadata(*it);
        it->replaceAllUsesWith(concat_node);
        it->removeAllInputs();
        it.destroyCurrent();
      }
    }
  }
}
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `owningGraph`, `create`, `i_`, `insertBefore`, `addInput`, `input`, `...`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`owningGraph`, `create`, `i_`, `insertBefore`, `addInput`, `input`, `...`。

### Lines 164-177
```cpp

// This pass also covers the case when the input to ListUnpack
// is int[] coming from some other op than ListConstruct (like Slice or Shape)
//
// before the pass
// graph(%x.1 : Float(2, 3, strides=[3, 1], requires_grad=0, device=cpu)):
//   %1 : None = prim::Constant()
//   %2 : int[] = aten::size(%x.1)
//   %a.1 : int, %b.1 : int = prim::ListUnpack(%2)
//   %5 : int[] = prim::ListConstruct(%a.1, %b.1)
//   %6 : Tensor = aten::new_zeros(%x.1, %5, %1, %1, %1, %1)
//
// after the pass:
// graph(%x.1 : Float(2, 3, strides=[3, 1], requires_grad=0, device=cpu)):
```
- EN: Documents the intent, constraints, or compatibility notes that guide the surrounding implementation.
- CN: 说明周围实现所遵循的目标、限制条件或兼容性要求。

### Lines 178-187
```cpp
//   %1 : None = prim::Constant()
//   %2 : int[] = aten::size(%x.1)
//   %7 : Tensor = onnx::Constant[value={0}]()
//   %8 : Tensor = onnx::Gather(%2, %7)
//   %9 : Tensor = onnx::Constant[value={1}]()
//   %10 : Tensor = onnx::Gather(%2, %9)
//   %a.1 : int, %b.1 : int = prim::ListUnpack(%2)
//   %5 : int[] = prim::ListConstruct(%8, %10)
//   %6 : Tensor = aten::new_zeros(%x.1, %5, %1, %1, %1, %1)
static void fuseListAndListUnpack(Block* b) {
```
- EN: This block manipulates TorchScript IR objects; performs optimization-oriented rewriting; protects shared state or ordering assumptions. Key symbols: `fuseListAndListUnpack`.
- CN: 该代码块操作 TorchScript IR 对象；执行面向优化的改写；保护共享状态或执行顺序假设。关键符号：`fuseListAndListUnpack`。

### Lines 188-194
```cpp
  for (auto it = b->nodes().begin(), end = b->nodes().end(); it != end; ++it) {
    for (auto* child_block : it->blocks()) {
      fuseListAndListUnpack(child_block);
    }
    if (it->kind() == prim::ListUnpack) {
      for (const auto i : c10::irange(it->outputs().size())) {
        auto output = it->outputs().at(i);
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `nodes`, `begin`, `end`, `blocks`, `fuseListAndListUnpack`, `kind`, `...`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`nodes`, `begin`, `end`, `blocks`, `fuseListAndListUnpack`, `kind`, `...`。

### Lines 195-203
```cpp
        if (it->inputs().size() == 1 &&
            it->input()->node()->kind() != prim::ListConstruct &&
            it->input()->type()->cast<ListType>() &&
            it->input()
                ->type()
                ->castRaw<ListType>()
                ->getElementType()
                ->cast<IntType>()) {
          Node* gather_indices = b->owningGraph()->create(onnx::Constant, 1);
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `inputs`, `size`, `input`, `node`, `kind`, `type`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`inputs`, `size`, `input`, `node`, `kind`, `type`, `...`。

### Lines 204-217
```cpp
          gather_indices->insertBefore(*it);
          gather_indices->t_(
              attr::value, at::scalar_to_tensor(at::Scalar(int(i))));
          Node* gather_node = b->owningGraph()->create(onnx::Gather, 1);
          gather_node->insertBefore(*it);
          gather_node->addInput(it->input());
          gather_node->addInput(gather_indices->output());
          gather_node->copyMetadata(*it);
          output->replaceAllUsesWith(gather_node->output());
        }
      }
    }
  }
}
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `insertBefore`, `t_`, `scalar_to_tensor`, `Scalar`, `int`, `owningGraph`, `...`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`insertBefore`, `t_`, `scalar_to_tensor`, `Scalar`, `int`, `owningGraph`, `...`。

### Lines 218-229
```cpp

} // namespace

void PreprocessForONNX(std::shared_ptr<Graph>& graph) {
  FuseWithListUnpack(graph->block());
  GRAPH_DUMP("After FuseWithListUnpack: ", graph);
  ReplaceAddWithConcat(graph->block());
  GRAPH_DUMP("After ReplaceAddWithConcat: ", graph);
  fuseListAndListUnpack(graph->block());
  GRAPH_DUMP("After fuseListAndListUnpack: ", graph);
}

```
- EN: This block manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `PreprocessForONNX`, `FuseWithListUnpack`, `block`, `ReplaceAddWithConcat`, `fuseListAndListUnpack`.
- CN: 该代码块操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`PreprocessForONNX`, `FuseWithListUnpack`, `block`, `ReplaceAddWithConcat`, `fuseListAndListUnpack`。

### Lines 230-230
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
- EN: Fusion support — the file prepares groups of operations for fused execution.
  CN: 融合支持——该文件会为融合执行准备算子分组。
- EN: Type and shape reasoning — the code propagates static information across the IR.
  CN: 类型与形状推理——代码会在 IR 中传播静态信息。
- EN: Export interop — the implementation helps bridge TorchScript graphs to ONNX-style representations.
  CN: 导出互操作——实现帮助 TorchScript 图与 ONNX 风格表示之间建立桥接。
- EN: Subsystem integration — the implementation collaborates with nearby JIT/frontend headers and runtime helpers.
  CN: 子系统集成——实现会与附近的 JIT/前端头文件以及运行时辅助模块协同工作。

## Dependencies / 依赖关系
- Internal includes / 内部头文件: `torch/csrc/jit/passes/onnx/preprocess_for_onnx.h`, `ATen/ScalarOps.h`, `c10/util/irange.h`, `torch/csrc/jit/jit_log.h`
- External includes / 外部头文件: 无
- Namespaces / 命名空间: `torch::jit`, `onnx`
- Representative symbols / 代表性符号: `namespace`, `FindFusibleListUnpack`, `outputs`, `size`, `output`, `uses`, `kind`, `FuseWithListUnpack`, `value`, `i_`, `...`
