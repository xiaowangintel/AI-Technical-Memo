# liveness.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `torch/csrc/jit/passes/liveness.cpp`
- Repository: `pytorch`
- Purpose (EN): Implements the TorchScript JIT pass logic for liveness, including graph analysis and rewrites.
- 用途 (CN): 实现与 liveness 相关的 TorchScript JIT pass 逻辑，包括图分析与改写。

## Line-by-Line Analysis / 逐行分析
### Lines 1-6
```cpp
#include <torch/csrc/jit/passes/liveness.h>

#include <torch/csrc/jit/ir/ir_views.h>
#include <iostream>
#include <memory>

```
- EN: Pulls in the headers needed by the liveness logic. Internal dependencies: `torch/csrc/jit/passes/liveness.h`, `torch/csrc/jit/ir/ir_views.h`; external dependencies: `iostream`, `memory`.
- CN: 为 liveness 相关逻辑引入所需头文件。内部依赖：`torch/csrc/jit/passes/liveness.h`, `torch/csrc/jit/ir/ir_views.h`；外部依赖：`iostream`, `memory`。

### Lines 7-11
```cpp
namespace torch::jit {

// LivenessAnalyzer computes "bailout" liveness which is equivalent to
// "{LIVE_IN} or {GEN}" or "{LIVE_OUT} - {KILL}"
struct LivenessAnalyzer {
```
- EN: Opens or closes namespace scopes so the remaining declarations live in the expected JIT/frontend module.
- CN: 打开或关闭命名空间作用域，使后续声明归属于预期的 JIT/前端模块。

### Lines 12-16
```cpp
  explicit LivenessAnalyzer(std::shared_ptr<Graph> graph)
      : graph_(std::move(graph)) {}

  std::unordered_map<Node*, std::vector<Value*>> run() {
    std::vector<Node*> counters;
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `LivenessAnalyzer`, `graph_`, `move`, `run`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`LivenessAnalyzer`, `graph_`, `move`, `run`。

### Lines 17-26
```cpp
    insertExplicitUsesOfLoopCounters(graph_->block(), counters);

    // we implement the canonical fixed-point liveness
    // the analysis is run until there are no more changes
    // to liveness sets for each node
    do {
      changed_ = false;
      processBlock(graph_->block(), SparseBitVector{});
    } while (changed_);

```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects; advances frontend parsing or lowering state; protects shared state or ordering assumptions. Key symbols: `insertExplicitUsesOfLoopCounters`, `block`, `processBlock`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象；推进前端解析或降级状态；保护共享状态或执行顺序假设。关键符号：`insertExplicitUsesOfLoopCounters`, `block`, `processBlock`。

### Lines 27-33
```cpp
    removeCounterNodes(counters);
    std::unordered_map<Node*, std::vector<Value*>> result;

    for (const auto& e : liveness_sets_) {
      result.insert({e.first, toValueVector(e.second)});
    }
    return result;
```
- EN: This block iterates over collections or graph structures; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `removeCounterNodes`, `insert`, `toValueVector`.
- CN: 该代码块遍历集合或图结构；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`removeCounterNodes`, `insert`, `toValueVector`。

### Lines 34-38
```cpp
  }

  // temporary make loop counts live for the duration of the loop
  // as they are needed by BailOuts in the loop
  void insertExplicitUsesOfLoopCounters(
```
- EN: This block implements local helper logic for liveness. Key symbols: `insertExplicitUsesOfLoopCounters`.
- CN: 该代码块实现与 liveness 相关的局部辅助逻辑。关键符号：`insertExplicitUsesOfLoopCounters`。

### Lines 39-45
```cpp
      Block* b,
      std::vector<Node*>& counters) {
    for (auto it : b->nodes()) {
      if (it->kind() == prim::Loop) {
        LoopView lv(it);
        WithInsertPoint guard(lv.bodyBlock());
        auto ctc = graph_->create(prim::Store, {lv.currentTripCount()}, 0);
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `nodes`, `kind`, `lv`, `guard`, `bodyBlock`, `create`, `...`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`nodes`, `kind`, `lv`, `guard`, `bodyBlock`, `create`, `...`。

### Lines 46-52
```cpp
        graph_->insertNode(ctc);
        counters.push_back(ctc);
        auto mtc = graph_->create(prim::Store, {lv.maxTripCount()}, 0);
        graph_->insertNode(mtc);
        counters.push_back(mtc);
      }

```
- EN: This block manipulates TorchScript IR objects. Key symbols: `insertNode`, `push_back`, `create`, `maxTripCount`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`insertNode`, `push_back`, `create`, `maxTripCount`。

### Lines 53-58
```cpp
      for (auto ib : it->blocks()) {
        insertExplicitUsesOfLoopCounters(ib, counters);
      }
    }
  }

```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `blocks`, `insertExplicitUsesOfLoopCounters`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`blocks`, `insertExplicitUsesOfLoopCounters`。

### Lines 59-64
```cpp
  void removeCounterNodes(std::vector<Node*>& counters) {
    for (auto n : counters) {
      n->destroy();
    }
  }

```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `removeCounterNodes`, `destroy`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`removeCounterNodes`, `destroy`。

### Lines 65-69
```cpp
  void dump(
      const std::unordered_map<Node*, std::vector<Value*>>& liveness_sets) {
    std::cout << "Liveness info:\n";
    for (auto e : liveness_sets) {
      if (!e.first->outputs().empty()) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `dump`, `outputs`, `empty`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`dump`, `outputs`, `empty`。

### Lines 70-74
```cpp
        std::cout << e.first->outputs()[0]->debugName();
      }

      std::cout << ' ' << e.first->kind().toQualString();
      std::cout << " = ";
```
- EN: This block implements local helper logic for liveness. Key symbols: `outputs`, `debugName`, `kind`, `toQualString`.
- CN: 该代码块实现与 liveness 相关的局部辅助逻辑。关键符号：`outputs`, `debugName`, `kind`, `toQualString`。

### Lines 75-81
```cpp
      dump(e.second);
      std::cout << '\n';
    }
    std::cout << "graph :\n";
    graph_->dump();
  }

```
- EN: This block manipulates TorchScript IR objects. Key symbols: `dump`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`dump`。

### Lines 82-86
```cpp
  void dump(const std::vector<Value*>& set) {
    bool first = true;
    std::cout << '[';
    for (auto el : set) {
      if (first) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `dump`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`dump`。

### Lines 87-91
```cpp
        first = false;
      } else {
        std::cout << ", ";
      }
      std::cout << el->debugName() << '(' << el->unique() << ')';
```
- EN: This block implements local helper logic for liveness. Key symbols: `debugName`, `unique`.
- CN: 该代码块实现与 liveness 相关的局部辅助逻辑。关键符号：`debugName`, `unique`。

### Lines 92-99
```cpp
    }
    std::cout << ']';
  }

 private:
  SparseBitVector toSparseBitVector(at::ArrayRef<Value*> values) {
    SparseBitVector sbv;
    for (auto v : values) {
```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects; advances frontend parsing or lowering state. Key symbols: `toSparseBitVector`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象；推进前端解析或降级状态。关键符号：`toSparseBitVector`。

### Lines 100-105
```cpp
      ids_to_values_[v->unique()] = v;
      sbv.set(v->unique());
    }
    return sbv;
  }

```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `unique`, `set`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`unique`, `set`。

### Lines 106-111
```cpp
  std::vector<Value*> toValueVector(const SparseBitVector& sbv) {
    std::vector<Value*> vec;
    for (auto id : sbv) {
      vec.push_back(ids_to_values_[id]);
    }
    return vec;
```
- EN: This block iterates over collections or graph structures; produces a result or forwards a computed value; manipulates TorchScript IR objects; advances frontend parsing or lowering state. Key symbols: `toValueVector`, `push_back`.
- CN: 该代码块遍历集合或图结构；返回结果或转发已计算的值；操作 TorchScript IR 对象；推进前端解析或降级状态。关键符号：`toValueVector`, `push_back`。

### Lines 112-116
```cpp
  }

  SparseBitVector processBlock(Block* b, SparseBitVector liveness) {
    // block outputs are the uses
    auto block_outputs = toSparseBitVector(b->outputs());
```
- EN: This block manipulates TorchScript IR objects; advances frontend parsing or lowering state; protects shared state or ordering assumptions. Key symbols: `processBlock`, `toSparseBitVector`, `outputs`.
- CN: 该代码块操作 TorchScript IR 对象；推进前端解析或降级状态；保护共享状态或执行顺序假设。关键符号：`processBlock`, `toSparseBitVector`, `outputs`。

### Lines 117-123
```cpp
    liveness |= block_outputs;

    SparseBitVector defs;
    for (Node* it : b->nodes().reverse()) {
      // kill outputs
      liveness -= toSparseBitVector(it->outputs());
      if (it->kind() == prim::Loop) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects; advances frontend parsing or lowering state; protects shared state or ordering assumptions. Key symbols: `nodes`, `reverse`, `toSparseBitVector`, `outputs`, `kind`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象；推进前端解析或降级状态；保护共享状态或执行顺序假设。关键符号：`nodes`, `reverse`, `toSparseBitVector`, `outputs`, `kind`。

### Lines 124-133
```cpp
        LoopView lv(it);
        // N.B. merge in changes from the loop header
        auto loop_header = *lv.bodyBlock()->nodes().begin();
        auto loop_block = liveness | liveness_sets_[loop_header];
        loop_block = processBlock(lv.bodyBlock(), loop_block);
        // loop block's inputs die outside loop's block
        loop_block -= toSparseBitVector(lv.bodyBlock()->inputs());
        liveness |= loop_block;
      } else if (it->kind() == prim::If) {
        IfView iv(it);
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; advances frontend parsing or lowering state; protects shared state or ordering assumptions. Key symbols: `lv`, `bodyBlock`, `nodes`, `begin`, `processBlock`, `toSparseBitVector`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；推进前端解析或降级状态；保护共享状态或执行顺序假设。关键符号：`lv`, `bodyBlock`, `nodes`, `begin`, `processBlock`, `toSparseBitVector`, `...`。

### Lines 134-142
```cpp
        auto true_liveness = processBlock(iv.thenBlock(), liveness);
        auto false_liveness = processBlock(iv.elseBlock(), liveness);
        liveness |= true_liveness;
        liveness |= false_liveness;
      }
      liveness |= toSparseBitVector(it->inputs());
      // `|=` returns true if new bits were set in LHS
      // after or/union with `liveness`
      auto changed = liveness_sets_[it] |= liveness;
```
- EN: This block manipulates TorchScript IR objects; advances frontend parsing or lowering state; protects shared state or ordering assumptions. Key symbols: `processBlock`, `thenBlock`, `elseBlock`, `toSparseBitVector`, `inputs`.
- CN: 该代码块操作 TorchScript IR 对象；推进前端解析或降级状态；保护共享状态或执行顺序假设。关键符号：`processBlock`, `thenBlock`, `elseBlock`, `toSparseBitVector`, `inputs`。

### Lines 143-147
```cpp
      changed_ = changed_ | changed;
    }
    return liveness;
  }

```
- EN: This block produces a result or forwards a computed value. Key symbols: no dominant local symbol names.
- CN: 该代码块返回结果或转发已计算的值。关键符号：无明显局部符号。

### Lines 148-152
```cpp
  std::shared_ptr<Graph> graph_;
  bool changed_{false};
  std::map<Node*, SparseBitVector> liveness_sets_;
  std::map<size_t, Value*> ids_to_values_;
};
```
- EN: This block manipulates TorchScript IR objects; advances frontend parsing or lowering state. Key symbols: no dominant local symbol names.
- CN: 该代码块操作 TorchScript IR 对象；推进前端解析或降级状态。关键符号：无明显局部符号。

### Lines 153-157
```cpp

std::unordered_map<Node*, std::vector<Value*>> BuildLivenessSets(
    std::shared_ptr<Graph> graph) {
  LivenessAnalyzer la(std::move(graph));
  return la.run();
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `BuildLivenessSets`, `la`, `move`, `run`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`BuildLivenessSets`, `la`, `move`, `run`。

### Lines 158-160
```cpp
}

} // namespace torch::jit
```
- EN: Opens or closes namespace scopes so the remaining declarations live in the expected JIT/frontend module.
- CN: 打开或关闭命名空间作用域，使后续声明归属于预期的 JIT/前端模块。


## Key Concepts / 关键概念
- EN: IR graph manipulation — the file inspects or rewrites TorchScript graphs and nodes.
  CN: IR 图操作——该文件会检查或改写 TorchScript 的图、节点与块。
- EN: Parsing pipeline — the logic translates source text into TorchScript syntax structures.
  CN: 解析流水线——相关逻辑将源码文本转换为 TorchScript 语法结构。
- EN: Pass orchestration — the file plugs logic into the TorchScript optimization pipeline.
  CN: Pass 编排——该文件把相关逻辑接入 TorchScript 优化流水线。
- EN: Subsystem integration — the implementation collaborates with nearby JIT/frontend headers and runtime helpers.
  CN: 子系统集成——实现会与附近的 JIT/前端头文件以及运行时辅助模块协同工作。

## Dependencies / 依赖关系
- Internal includes / 内部头文件: `torch/csrc/jit/passes/liveness.h`, `torch/csrc/jit/ir/ir_views.h`
- External includes / 外部头文件: `iostream`, `memory`
- Namespaces / 命名空间: `torch::jit`
- Representative symbols / 代表性符号: `LivenessAnalyzer`, `graph_`, `move`, `run`, `insertExplicitUsesOfLoopCounters`, `block`, `processBlock`, `removeCounterNodes`, `insert`, `toValueVector`, `...`
