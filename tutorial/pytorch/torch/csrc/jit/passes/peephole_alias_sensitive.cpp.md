# peephole_alias_sensitive.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `torch/csrc/jit/passes/peephole_alias_sensitive.cpp`
- Repository: `pytorch`
- Purpose (EN): Implements the TorchScript JIT pass logic for peephole alias sensitive, including graph analysis and rewrites.
- 用途 (CN): 实现与 peephole alias sensitive 相关的 TorchScript JIT pass 逻辑，包括图分析与改写。

## Line-by-Line Analysis / 逐行分析
### Lines 1-5
```cpp
#include <torch/csrc/jit/ir/alias_analysis.h>
#include <torch/csrc/jit/jit_log.h>
#include <torch/csrc/jit/passes/peephole_alias_sensitive.h>

namespace torch::jit {
```
- EN: Opens or closes namespace scopes so the remaining declarations live in the expected JIT/frontend module.
- CN: 打开或关闭命名空间作用域，使后续声明归属于预期的 JIT/前端模块。

### Lines 6-10
```cpp

// This pass only does optimizations which requires Alias Analysis
// It is separated out from Peephole Pass so that Peephole does not have
// maintain alias db correctness throughout the pass.
struct PeepholeOptimizeAliasSensitiveImpl {
```
- EN: Declares core types or data containers for this file. Prominent symbols: `PeepholeOptimizeAliasSensitiveImpl`.
- CN: 声明该文件使用的核心类型或数据容器。主要符号：`PeepholeOptimizeAliasSensitiveImpl`。

### Lines 11-17
```cpp
  PeepholeOptimizeAliasSensitiveImpl(
      std::shared_ptr<Graph> graph,
      bool shape_peepholes)
      : graph_(std::move(graph)),
        aliasDb_(std::make_unique<AliasDb>(graph_)),
        shape_peepholes_(shape_peepholes) {}

```
- EN: This block manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `PeepholeOptimizeAliasSensitiveImpl`, `graph_`, `move`, `aliasDb_`, `shape_peepholes_`.
- CN: 该代码块操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`PeepholeOptimizeAliasSensitiveImpl`, `graph_`, `move`, `aliasDb_`, `shape_peepholes_`。

### Lines 18-23
```cpp
  bool run() {
    return runBlock(graph_->block());
  }

 private:
  void replaceWithIValue(Value* v, const IValue& val) {
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `run`, `runBlock`, `block`, `replaceWithIValue`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`run`, `runBlock`, `block`, `replaceWithIValue`。

### Lines 24-28
```cpp
    WithInsertPoint guard(v->node());
    v->replaceAllUsesWith(v->owningGraph()->insertConstant(val));
  }

  bool isFloatingPoint(TensorType& t) {
```
- EN: This block manipulates TorchScript IR objects; performs optimization-oriented rewriting; protects shared state or ordering assumptions. Key symbols: `guard`, `node`, `replaceAllUsesWith`, `owningGraph`, `insertConstant`, `isFloatingPoint`.
- CN: 该代码块操作 TorchScript IR 对象；执行面向优化的改写；保护共享状态或执行顺序假设。关键符号：`guard`, `node`, `replaceAllUsesWith`, `owningGraph`, `insertConstant`, `isFloatingPoint`。

### Lines 29-33
```cpp
    auto input_dtype = t.scalarType();
    return (
        shape_peepholes_ && input_dtype && at::isFloatingType(*input_dtype));
  }

```
- EN: This block produces a result or forwards a computed value; performs optimization-oriented rewriting. Key symbols: `scalarType`, `isFloatingType`.
- CN: 该代码块返回结果或转发已计算的值；执行面向优化的改写。关键符号：`scalarType`, `isFloatingType`。

### Lines 34-40
```cpp
  bool runBlock(Block* block) {
    bool changed = false;
    for (Node* node : block->nodes()) {
      for (Block* b : node->blocks()) {
        changed |= runBlock(b);
      }

```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `runBlock`, `nodes`, `blocks`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`runBlock`, `nodes`, `blocks`。

### Lines 41-45
```cpp
      // dim(conv(x)) extremely common and prevents Conv->BN fusion
      if (node->kind() == aten::conv1d || node->kind() == aten::conv2d ||
          node->kind() == aten::conv3d) {
        auto dim_uses = c10::filter(node->output()->uses(), [](const Use& use) {
          return use.user->kind() == aten::dim;
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `kind`, `filter`, `output`, `uses`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`kind`, `filter`, `output`, `uses`。

### Lines 46-50
```cpp
        });
        if (dim_uses.empty()) {
          continue;
        }
        auto kind = node->kind();
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `empty`, `kind`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`empty`, `kind`。

### Lines 51-56
```cpp
        int64_t output_size =
            kind == aten::conv1d ? 3 : (kind == aten::conv2d ? 4 : 5);
        // This is to handle potential resize_ calls, however unlikely.
        // If we add more checks related to resize_ in the graph,
        // factor this out like collectResizeSet in shape_analysis.
        if (!aliasDb_->hasWriters(node->output())) {
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `hasWriters`, `output`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`hasWriters`, `output`。

### Lines 57-62
```cpp
          for (const Use& dim_use : dim_uses) {
            replaceWithIValue(dim_use.user->output(), output_size);
          }
          changed = true;
        } else {
          for (const Use& dim_use : dim_uses) {
```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `replaceWithIValue`, `output`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`replaceWithIValue`, `output`。

### Lines 63-72
```cpp
            if (aliasDb_->moveAfterTopologicallyValid(node, dim_use.user)) {
              replaceWithIValue(dim_use.user->output(), output_size);
              changed = true;
            }
          }
        }
        continue;
      } else if (
          node->matches(
              "aten::add(Tensor self, Scalar other, Scalar alpha) -> Tensor",
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `moveAfterTopologicallyValid`, `replaceWithIValue`, `output`, `matches`, `add`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`moveAfterTopologicallyValid`, `replaceWithIValue`, `output`, `matches`, `add`。

### Lines 73-80
```cpp
              /*const_inputs=*/{attr::alpha, attr::other}) ||
          node->matches(
              "aten::sub(Tensor self, Scalar other, Scalar alpha) -> Tensor",
              /*const_inputs=*/{attr::alpha, attr::other})) {
        // x + 0 == x - 0 == x
        // if either scalar input is a float, than removing this operator could
        // remove type promotion and affect semantics
        if (!isFloatingPoint(node->input(0)->type()->expectRef<TensorType>())) {
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `matches`, `sub`, `isFloatingPoint`, `input`, `type`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`matches`, `sub`, `isFloatingPoint`, `input`, `type`。

### Lines 81-87
```cpp
          auto inps = node->inputs();
          if (!inps.at(1)->type()->isSubtypeOf(IntType::get()) ||
              !inps.at(2)->type()->isSubtypeOf(IntType::get())) {
            continue;
          }
        }

```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `inputs`, `type`, `isSubtypeOf`, `get`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`inputs`, `type`, `isSubtypeOf`, `get`。

### Lines 88-97
```cpp
        if (node->get<at::Scalar>(attr::alpha)->toDouble() == 1 &&
            node->get<at::Scalar>(attr::other)->toDouble() == 0) {
          if (tryToReplaceOutputWithInput(node->input(0), node->output())) {
            GRAPH_UPDATE(
                getHeader(node),
                " (x + 0 == x - 0 == x) is replaced with ",
                node->input(0)->debugName());
            node->output()->replaceAllUsesWith(node->input(0));
            changed = true;
          }
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `toDouble`, `tryToReplaceOutputWithInput`, `input`, `output`, `getHeader`, `debugName`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`toDouble`, `tryToReplaceOutputWithInput`, `input`, `output`, `getHeader`, `debugName`, `...`。

### Lines 98-107
```cpp
        }
      } else if (
          node->matches(
              "aten::mul(Tensor self, Scalar other) -> Tensor",
              /*const_inputs=*/attr::other) ||
          node->matches(
              "aten::div(Tensor self, Scalar other) -> Tensor",
              /*const_inputs=*/attr::other)) {
        // x * 1 == x / 1 == x
        // is the node is a division or other isn't an integer, than removing
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `matches`, `mul`, `div`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`matches`, `mul`, `div`。

### Lines 108-115
```cpp
        // this operator could remove type promotion and affect semantics
        if (!isFloatingPoint(node->input(0)->type()->expectRef<TensorType>())) {
          if (node->kind() == aten::div ||
              !node->input(1)->type()->isSubtypeOf(IntType::get())) {
            continue;
          }
        }

```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `isFloatingPoint`, `input`, `type`, `kind`, `isSubtypeOf`, `get`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`isFloatingPoint`, `input`, `type`, `kind`, `isSubtypeOf`, `get`。

### Lines 116-122
```cpp
        if (node->get<at::Scalar>(attr::other)->toDouble() == 1) {
          if (tryToReplaceOutputWithInput(node->input(0), node->output())) {
            GRAPH_UPDATE(
                getHeader(node),
                " (x * 1 == x / 1 == x) is replaced with ",
                node->input(0)->debugName());

```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `toDouble`, `tryToReplaceOutputWithInput`, `input`, `output`, `getHeader`, `debugName`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`toDouble`, `tryToReplaceOutputWithInput`, `input`, `output`, `getHeader`, `debugName`。

### Lines 123-128
```cpp
            changed = true;
          }
        }
      }
    }
    return changed;
```
- EN: This block produces a result or forwards a computed value. Key symbols: no dominant local symbol names.
- CN: 该代码块返回结果或转发已计算的值。关键符号：无明显局部符号。

### Lines 129-133
```cpp
  }

  bool tryToReplaceOutputWithInput(Value* input, Value* output) {
    if (!aliasDb_->safeToChangeAliasingRelationship(input, output)) {
      return false;
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `tryToReplaceOutputWithInput`, `safeToChangeAliasingRelationship`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`tryToReplaceOutputWithInput`, `safeToChangeAliasingRelationship`。

### Lines 134-143
```cpp
    }
    // whenever we replace an output with an input, all of the aliasing
    // properties of the output are now present on the input.
    // For example, if the output aliases a graph output, the input will now
    // as well.
    // in order to avoid re-instantiating an alias db on each change, which
    // would be O(n^2), or inplace modifying it, which would involve
    // invalidating all of the memory dag caches, we just keep a set of values
    // which are "stale" (aliasing properties not up to date), and avoid doing
    // further optimizations on values which alias them
```
- EN: This block manipulates TorchScript IR objects. Key symbols: no dominant local symbol names.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：无明显局部符号。

### Lines 144-150
```cpp
    if (aliasDb_->mayAlias({input, output}, stale_alias_values_)) {
      return false;
    }
    output->replaceAllUsesWith(input);
    stale_alias_values_.insert(input);
    stale_alias_values_.insert(output);
    return true;
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `mayAlias`, `replaceAllUsesWith`, `insert`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`mayAlias`, `replaceAllUsesWith`, `insert`。

### Lines 151-155
```cpp
  }

  ValueSet stale_alias_values_;
  std::shared_ptr<Graph> graph_;
  std::unique_ptr<AliasDb> aliasDb_;
```
- EN: This block manipulates TorchScript IR objects. Key symbols: no dominant local symbol names.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：无明显局部符号。

### Lines 156-161
```cpp
  bool shape_peepholes_;
};

bool PeepholeOptimizeAliasSensitive(
    const std::shared_ptr<Graph>& graph,
    bool shape_peepholes) {
```
- EN: This block manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `PeepholeOptimizeAliasSensitive`.
- CN: 该代码块操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`PeepholeOptimizeAliasSensitive`。

### Lines 162-166
```cpp
  PeepholeOptimizeAliasSensitiveImpl opt(graph, shape_peepholes);
  return opt.run();
}

} // namespace torch::jit
```
- EN: Opens or closes namespace scopes so the remaining declarations live in the expected JIT/frontend module.
- CN: 打开或关闭命名空间作用域，使后续声明归属于预期的 JIT/前端模块。


## Key Concepts / 关键概念
- EN: IR graph manipulation — the file inspects or rewrites TorchScript graphs and nodes.
  CN: IR 图操作——该文件会检查或改写 TorchScript 的图、节点与块。
- EN: Alias and mutation reasoning — the code tracks side effects and write/read interactions.
  CN: 别名与变更分析——代码会跟踪副作用以及读写关系。
- EN: Optimization pass — the code simplifies graphs by removing redundant work or folding values.
  CN: 优化 pass——代码通过删除冗余计算或常量折叠来简化图。
- EN: Fusion support — the file prepares groups of operations for fused execution.
  CN: 融合支持——该文件会为融合执行准备算子分组。
- EN: Type and shape reasoning — the code propagates static information across the IR.
  CN: 类型与形状推理——代码会在 IR 中传播静态信息。
- EN: Subsystem integration — the implementation collaborates with nearby JIT/frontend headers and runtime helpers.
  CN: 子系统集成——实现会与附近的 JIT/前端头文件以及运行时辅助模块协同工作。

## Dependencies / 依赖关系
- Internal includes / 内部头文件: `torch/csrc/jit/ir/alias_analysis.h`, `torch/csrc/jit/jit_log.h`, `torch/csrc/jit/passes/peephole_alias_sensitive.h`
- External includes / 外部头文件: 无
- Namespaces / 命名空间: `torch::jit`
- Representative symbols / 代表性符号: `PeepholeOptimizeAliasSensitiveImpl`, `graph_`, `move`, `aliasDb_`, `shape_peepholes_`, `run`, `runBlock`, `block`, `replaceWithIValue`, `guard`, `...`
