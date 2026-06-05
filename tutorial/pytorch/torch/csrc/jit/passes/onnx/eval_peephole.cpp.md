# eval_peephole.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `torch/csrc/jit/passes/onnx/eval_peephole.cpp`
- Repository: `pytorch`
- Purpose (EN): Implements the TorchScript JIT pass logic for eval peephole, including graph analysis and rewrites.
- 用途 (CN): 实现与 eval peephole 相关的 TorchScript JIT pass 逻辑，包括图分析与改写。

## Line-by-Line Analysis / 逐行分析
### Lines 1-6
```cpp
#include <torch/csrc/jit/jit_log.h>
#include <torch/csrc/jit/passes/onnx/eval_peephole.h>
#include <torch/csrc/jit/passes/onnx/helper.h>

#include <c10/util/irange.h>

```
- EN: Pulls in the headers needed by the eval peephole logic. Internal dependencies: `torch/csrc/jit/jit_log.h`, `torch/csrc/jit/passes/onnx/eval_peephole.h`, `torch/csrc/jit/passes/onnx/helper.h`, `c10/util/irange.h`; external dependencies: none.
- CN: 为 eval peephole 相关逻辑引入所需头文件。内部依赖：`torch/csrc/jit/jit_log.h`, `torch/csrc/jit/passes/onnx/eval_peephole.h`, `torch/csrc/jit/passes/onnx/helper.h`, `c10/util/irange.h`；外部依赖：无。

### Lines 7-12
```cpp
namespace torch::jit {

namespace onnx {
using namespace ::c10::onnx;
}

```
- EN: Opens or closes namespace scopes so the remaining declarations live in the expected JIT/frontend module.
- CN: 打开或关闭命名空间作用域，使后续声明归属于预期的 JIT/前端模块。

### Lines 13-17
```cpp
static std::vector<at::Tensor> getValues(
    Node* node,
    const ValueToParamPairMap& valsToParamsMap) {
  size_t numInputs = node->inputs().size();
  std::vector<at::Tensor> inputTensorValues;
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `getValues`, `inputs`, `size`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`getValues`, `inputs`, `size`。

### Lines 18-22
```cpp
  inputTensorValues.reserve(numInputs);
  for (auto val : node->inputs()) {
    if (val->node()->kind() == prim::Param) {
      auto itr = valsToParamsMap.find(val);
      if (itr == valsToParamsMap.end()) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `reserve`, `inputs`, `node`, `kind`, `find`, `end`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`reserve`, `inputs`, `node`, `kind`, `find`, `end`。

### Lines 23-32
```cpp
        continue;
      }
      inputTensorValues.push_back(itr->second.second.toTensor());
    } else if (val->node()->kind() == onnx::Constant) {
      inputTensorValues.push_back(val->node()->t(attr::value));
    } else {
      continue;
    }
  }
  return inputTensorValues;
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `push_back`, `toTensor`, `node`, `kind`, `t`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`push_back`, `toTensor`, `node`, `kind`, `t`。

### Lines 33-40
```cpp
}

// This pass fuses Conv and BatchNorm into Conv node
// Conv and BatchNorm can be fused only if inputs for BatchNorm node:
// scale, bias, mean and var are all tensors of same shape (C) and
// if the size of the first dimension (dim 0) is the same between Conv
// input weight and BatchNorm input scale.
static void fuseConvBatchNorm(Block* b, ValueToParamPairMap& valsToParamsMap) {
```
- EN: This block manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `fuseConvBatchNorm`.
- CN: 该代码块操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`fuseConvBatchNorm`。

### Lines 41-45
```cpp
  for (auto it = b->nodes().begin(), end = b->nodes().end(); it != end; ++it) {
    for (auto* child_block : it->blocks()) {
      fuseConvBatchNorm(child_block, valsToParamsMap);
    }
    if (it->kind() == onnx::Conv) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `nodes`, `begin`, `end`, `blocks`, `fuseConvBatchNorm`, `kind`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`nodes`, `begin`, `end`, `blocks`, `fuseConvBatchNorm`, `kind`。

### Lines 46-50
```cpp
      auto oldConv = *it;
      if (oldConv->outputs().at(0)->uses().size() != 1) {
        continue;
      }
      auto bnNode = oldConv->outputs().at(0)->uses()[0].user;
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `outputs`, `uses`, `size`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`outputs`, `uses`, `size`。

### Lines 51-55
```cpp
      if (bnNode->kind() != onnx::BatchNormalization) {
        continue;
      }

      if (oldConv->outputs().size() !=
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `kind`, `outputs`, `size`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`kind`, `outputs`, `size`。

### Lines 56-60
```cpp
          bnNode->outputs().size()) { // BN layer is not in eval mode
        continue;
      }

      auto epsilon = bnNode->f(attr::epsilon);
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `outputs`, `size`, `f`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`outputs`, `size`, `f`。

### Lines 61-66
```cpp
      auto convInputVals = getValues(oldConv, valsToParamsMap);
      if (convInputVals.empty() ||
          (oldConv->inputs().size() == 3 && convInputVals.size() != 2)) {
        continue;
      }

```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `getValues`, `empty`, `inputs`, `size`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`getValues`, `empty`, `inputs`, `size`。

### Lines 67-71
```cpp
      auto bnInputVals = getValues(bnNode, valsToParamsMap);
      if (bnInputVals.size() != 4) {
        continue;
      }

```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `getValues`, `size`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`getValues`, `size`。

### Lines 72-76
```cpp
      // See
      // https://github.com/onnx/onnx/blob/master/docs/Operators.md#BatchNormalization
      auto bnScale = bnInputVals[0].clone();
      auto bnB = bnInputVals[1].clone();
      auto bnMean = bnInputVals[2].clone();
```
- EN: This block implements local helper logic for eval peephole. Key symbols: `clone`.
- CN: 该代码块实现与 eval peephole 相关的局部辅助逻辑。关键符号：`clone`。

### Lines 77-81
```cpp
      auto bnVar = bnInputVals[3].clone();
      // See https://github.com/onnx/onnx/blob/master/docs/Operators.md#Conv
      auto convW = convInputVals[0].clone();
      at::Tensor convB;

```
- EN: This block implements local helper logic for eval peephole. Key symbols: `clone`.
- CN: 该代码块实现与 eval peephole 相关的局部辅助逻辑。关键符号：`clone`。

### Lines 82-91
```cpp
      if (!bnScale.is_floating_point() || !bnB.is_floating_point() ||
          !bnMean.is_floating_point() || !bnVar.is_floating_point() ||
          !convW.is_floating_point() || bnScale.dim() != 1 || bnB.dim() != 1 ||
          bnMean.dim() != 1 || bnVar.dim() != 1 ||
          !(bnScale.size(0) == bnB.size(0)) ||
          !(bnB.size(0) == bnMean.size(0)) ||
          !(bnMean.size(0) == bnVar.size(0)) || !(convW.dim() > 2) ||
          !(convW.size(0) == bnScale.size(0))) {
        continue;
      }
```
- EN: This block handles conditional branches. Key symbols: `is_floating_point`, `dim`, `size`.
- CN: 该代码块处理条件分支。关键符号：`is_floating_point`, `dim`, `size`。

### Lines 92-96
```cpp

      bnVar = bnVar.add(epsilon);
      bnVar = bnVar.sqrt();
      bnScale = bnScale.div(bnVar);

```
- EN: This block implements local helper logic for eval peephole. Key symbols: `add`, `sqrt`, `div`.
- CN: 该代码块实现与 eval peephole 相关的局部辅助逻辑。关键符号：`add`, `sqrt`, `div`。

### Lines 97-101
```cpp
      // Calculate weight
      for (const auto i : c10::irange(convW.size(0))) {
        convW[i] = convW[i].mul(bnScale[i]);
      }

```
- EN: This block iterates over collections or graph structures. Key symbols: `irange`, `size`, `mul`.
- CN: 该代码块遍历集合或图结构。关键符号：`irange`, `size`, `mul`。

### Lines 102-111
```cpp
      // Calculate bias
      if (oldConv->inputs().size() == 3) {
        convB = convInputVals[1].clone();
        convB = convB.sub(bnMean);
        convB = convB.mul(bnScale);
        convB = convB.add(bnB);
      } else {
        bnMean = bnMean.mul(bnScale);
        bnB = bnB.sub(bnMean);
        convB = bnB;
```
- EN: This block handles conditional branches. Key symbols: `inputs`, `size`, `clone`, `sub`, `mul`, `add`.
- CN: 该代码块处理条件分支。关键符号：`inputs`, `size`, `clone`, `sub`, `mul`, `add`。

### Lines 112-116
```cpp
      }

      Node* newConv = b->owningGraph()->create(onnx::Conv, 1);
      newConv->outputs().at(0)->copyMetadata(bnNode->outputs().at(0));

```
- EN: This block manipulates TorchScript IR objects. Key symbols: `owningGraph`, `create`, `outputs`, `copyMetadata`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`owningGraph`, `create`, `outputs`, `copyMetadata`。

### Lines 117-121
```cpp
      newConv->copyAttributes(*oldConv);
      newConv->insertBefore(bnNode);
      newConv->addInput(oldConv->inputs().at(0));
      newConv->copyMetadata(oldConv);

```
- EN: This block manipulates TorchScript IR objects. Key symbols: `copyAttributes`, `insertBefore`, `addInput`, `inputs`, `copyMetadata`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`copyAttributes`, `insertBefore`, `addInput`, `inputs`, `copyMetadata`。

### Lines 122-127
```cpp
      auto newConvW = b->owningGraph()->addInput();
      valsToParamsMap.insert(
          {newConvW, std::make_pair(newConvW->debugName(), convW)});
      newConvW->inferTypeFrom(convW);
      newConv->addInput(newConvW);

```
- EN: This block manipulates TorchScript IR objects. Key symbols: `owningGraph`, `addInput`, `insert`, `make_pair`, `debugName`, `inferTypeFrom`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`owningGraph`, `addInput`, `insert`, `make_pair`, `debugName`, `inferTypeFrom`。

### Lines 128-133
```cpp
      auto newConvB = b->owningGraph()->addInput();
      valsToParamsMap.insert(
          {newConvB, std::make_pair(newConvB->debugName(), convB)});
      newConvB->inferTypeFrom(convB);
      newConv->addInput(newConvB);

```
- EN: This block manipulates TorchScript IR objects. Key symbols: `owningGraph`, `addInput`, `insert`, `make_pair`, `debugName`, `inferTypeFrom`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`owningGraph`, `addInput`, `insert`, `make_pair`, `debugName`, `inferTypeFrom`。

### Lines 134-140
```cpp
      bnNode->outputs().at(0)->replaceAllUsesWith(newConv->outputs().at(0));
      bnNode->destroy();
      it.destroyCurrent();
    }
  }
}

```
- EN: This block manipulates TorchScript IR objects. Key symbols: `outputs`, `replaceAllUsesWith`, `destroy`, `destroyCurrent`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`outputs`, `replaceAllUsesWith`, `destroy`, `destroyCurrent`。

### Lines 141-146
```cpp
static void EvalPeepholeONNX(Block* b, ParamMap& paramsDict) {
  auto valsToParamsMap = buildValueToParamsMap(b, paramsDict);
  fuseConvBatchNorm(b, valsToParamsMap);
  buildParamsMapFromValueToParamsMap(valsToParamsMap, paramsDict);
}

```
- EN: This block manipulates TorchScript IR objects; performs optimization-oriented rewriting; protects shared state or ordering assumptions. Key symbols: `EvalPeepholeONNX`, `buildValueToParamsMap`, `fuseConvBatchNorm`, `buildParamsMapFromValueToParamsMap`.
- CN: 该代码块操作 TorchScript IR 对象；执行面向优化的改写；保护共享状态或执行顺序假设。关键符号：`EvalPeepholeONNX`, `buildValueToParamsMap`, `fuseConvBatchNorm`, `buildParamsMapFromValueToParamsMap`。

### Lines 147-151
```cpp
void EvalPeepholeONNX(std::shared_ptr<Graph>& g, ParamMap& paramsDict) {
  EvalPeepholeONNX(g->block(), paramsDict);
  GRAPH_DUMP("After EvalPeepholeONNX:", g);
}

```
- EN: This block manipulates TorchScript IR objects; performs optimization-oriented rewriting; protects shared state or ordering assumptions. Key symbols: `EvalPeepholeONNX`, `block`.
- CN: 该代码块操作 TorchScript IR 对象；执行面向优化的改写；保护共享状态或执行顺序假设。关键符号：`EvalPeepholeONNX`, `block`。

### Lines 152-152
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
- EN: Type and shape reasoning — the code propagates static information across the IR.
  CN: 类型与形状推理——代码会在 IR 中传播静态信息。
- EN: Export interop — the implementation helps bridge TorchScript graphs to ONNX-style representations.
  CN: 导出互操作——实现帮助 TorchScript 图与 ONNX 风格表示之间建立桥接。
- EN: Subsystem integration — the implementation collaborates with nearby JIT/frontend headers and runtime helpers.
  CN: 子系统集成——实现会与附近的 JIT/前端头文件以及运行时辅助模块协同工作。

## Dependencies / 依赖关系
- Internal includes / 内部头文件: `torch/csrc/jit/jit_log.h`, `torch/csrc/jit/passes/onnx/eval_peephole.h`, `torch/csrc/jit/passes/onnx/helper.h`, `c10/util/irange.h`
- External includes / 外部头文件: 无
- Namespaces / 命名空间: `torch::jit`, `onnx`
- Representative symbols / 代表性符号: `namespace`, `getValues`, `inputs`, `size`, `reserve`, `node`, `kind`, `find`, `end`, `push_back`, `...`
