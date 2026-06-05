# constant_propagation.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `torch/csrc/jit/passes/constant_propagation.cpp`
- Repository: `pytorch`
- Purpose (EN): Implements the TorchScript JIT pass logic for constant propagation, including graph analysis and rewrites.
- 用途 (CN): 实现与 constant propagation 相关的 TorchScript JIT pass 逻辑，包括图分析与改写。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14
```cpp
#include <torch/csrc/jit/passes/constant_propagation.h>

#include <ATen/core/ivalue.h>
#include <c10/util/Exception.h>
#include <c10/util/irange.h>
#include <torch/csrc/jit/ir/alias_analysis.h>
#include <torch/csrc/jit/ir/constants.h>
#include <torch/csrc/jit/ir/ir.h>
#include <torch/csrc/jit/ir/node_hashing.h>
#include <torch/csrc/jit/jit_log.h>
#include <torch/csrc/jit/passes/dead_code_elimination.h>
#include <torch/csrc/jit/runtime/operator.h>
#include <torch/csrc/jit/runtime/vararg_functions.h>

```
- EN: Pulls in the headers needed by the constant propagation logic. Internal dependencies: `torch/csrc/jit/passes/constant_propagation.h`, `ATen/core/ivalue.h`, `c10/util/Exception.h`, `c10/util/irange.h`, `torch/csrc/jit/ir/alias_analysis.h`, `...`; external dependencies: none.
- CN: 为 constant propagation 相关逻辑引入所需头文件。内部依赖：`torch/csrc/jit/passes/constant_propagation.h`, `ATen/core/ivalue.h`, `c10/util/Exception.h`, `c10/util/irange.h`, `torch/csrc/jit/ir/alias_analysis.h`, `...`；外部依赖：无。

### Lines 15-21
```cpp
#include <utility>

namespace torch::jit {

std::optional<std::vector<IValue>> runNodeIfInputsAreConstant(
    const Node* n,
    bool ignore_custom_classes,
```
- EN: This block manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `runNodeIfInputsAreConstant`.
- CN: 该代码块操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`runNodeIfInputsAreConstant`。

### Lines 22-28
```cpp
    AliasDb* db) {
  Stack stack;
  for (auto input : n->inputs()) {
    if (auto ival = toIValue(input)) {
      stack.push_back(*ival);
    } else {
      return std::nullopt;
```
- EN: This block handles conditional branches; iterates over collections or graph structures; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `inputs`, `toIValue`, `push_back`.
- CN: 该代码块处理条件分支；遍历集合或图结构；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`inputs`, `toIValue`, `push_back`。

### Lines 29-35
```cpp
    }
  }

  switch (n->kind()) {
    case prim::ListUnpack: {
      if (stack.back().toList().size() != n->outputs().size()) {
        return std::nullopt;
```
- EN: This block handles conditional branches; produces a result or forwards a computed value. Key symbols: `kind`, `back`, `toList`, `size`, `outputs`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值。关键符号：`kind`, `back`, `toList`, `size`, `outputs`。

### Lines 36-49
```cpp
      }
      listUnpack(stack, n->outputs().size());
    } break;
    case prim::TupleConstruct: {
      auto tt = n->output()->type()->expect<TupleType>();
      if (tt->name()) {
        namedTupleConstruct(stack, std::move(tt), n->inputs().size());
      } else {
        tupleConstruct(stack, n->inputs().size());
      }
    } break;
    case prim::ListConstruct: {
      listConstruct(
          stack,
```
- EN: This block handles conditional branches. Key symbols: `listUnpack`, `outputs`, `size`, `output`, `type`, `name`, `...`.
- CN: 该代码块处理条件分支。关键符号：`listUnpack`, `outputs`, `size`, `output`, `type`, `name`, `...`。

### Lines 50-63
```cpp
          n->output()->type()->expectRef<ListType>(),
          n->inputs().size());
    } break;
    case prim::DictConstruct: {
      dictConstruct(
          stack,
          n->output()->type()->expectRef<DictType>(),
          n->inputs().size());
    } break;
    case prim::CreateObject: {
      createObject(
          stack,
          n->output()->type()->expect<ClassType>(),
          /*use_weak_ref*/ true);
```
- EN: This block implements local helper logic for constant propagation. Key symbols: `output`, `type`, `inputs`, `size`, `dictConstruct`, `createObject`.
- CN: 该代码块实现与 constant propagation 相关的局部辅助逻辑。关键符号：`output`, `type`, `inputs`, `size`, `dictConstruct`, `createObject`。

### Lines 64-74
```cpp
    } break;
    case prim::GetAttr: {
      auto attr = pop(stack).toObject()->getAttr(n->s(attr::name));
      push(stack, attr);
    } break;
    case prim::isinstance: {
      isinstance(stack, n->tys(attr::types));
    } break;
    default: {
      const auto maybe_schema = n->maybeSchema();
      if (maybe_schema && maybe_schema->is_vararg()) {
```
- EN: This block handles conditional branches; advances frontend parsing or lowering state. Key symbols: `pop`, `toObject`, `getAttr`, `s`, `push`, `isinstance`, `...`.
- CN: 该代码块处理条件分支；推进前端解析或降级状态。关键符号：`pop`, `toObject`, `getAttr`, `s`, `push`, `isinstance`, `...`。

### Lines 75-82
```cpp
        // vararg schemas require the number of inputs at the top of the stack
        // but this is broken in other places in constant prop, so disable it
        // for now
        return std::nullopt;
      }

      try {
        auto op = n->getOperation();
```
- EN: This block produces a result or forwards a computed value; advances frontend parsing or lowering state; performs optimization-oriented rewriting. Key symbols: `getOperation`.
- CN: 该代码块返回结果或转发已计算的值；推进前端解析或降级状态；执行面向优化的改写。关键符号：`getOperation`。

### Lines 83-89
```cpp
        op(stack);
      } catch (...) {
        return std::nullopt;
      }
    } break;
  }

```
- EN: This block produces a result or forwards a computed value. Key symbols: `op`.
- CN: 该代码块返回结果或转发已计算的值。关键符号：`op`。

### Lines 90-99
```cpp
  for (IValue& v : stack) {
    if (v.isTensor()) {
      const at::Tensor& t = v.toTensor();
      if (t.defined() && t.requires_grad()) {
        // requires grad tensors cannot be constants
        return std::nullopt;
      }
    }
    // Weak form of const propagation
    if (ignore_custom_classes) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `isTensor`, `toTensor`, `defined`, `requires_grad`.
- CN: 该代码块处理条件分支；遍历集合或图结构；返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`isTensor`, `toTensor`, `defined`, `requires_grad`。

### Lines 100-106
```cpp
      if (v.isCustomClass()) {
        return std::nullopt;
      }
    }
    // see [Constant Object Weak CompilationUnit Reference]
    if (v.isCustomClass()) {
      if (v.toObject()->is_weak_compilation_ref()) {
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; performs optimization-oriented rewriting. Key symbols: `isCustomClass`, `toObject`, `is_weak_compilation_ref`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；执行面向优化的改写。关键符号：`isCustomClass`, `toObject`, `is_weak_compilation_ref`。

### Lines 107-113
```cpp
        continue;
      }
      if (!db) {
        continue;
      }
      // NOLINTNEXTLINE(cppcoreguidelines-pro-type-const-cast)
      Node* n_non_const = const_cast<Node*>(n);
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: no dominant local symbol names.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：无明显局部符号。

### Lines 114-121
```cpp
      if (db->mayContainAlias(
              n_non_const->inputs(), {n_non_const->outputs()})) {
        continue;
      }
      auto obj = v.toObject();
      obj->unsafe_make_weak_compilation_ref();
    }
    if (v.isObject()) {
```
- EN: This block handles conditional branches. Key symbols: `mayContainAlias`, `inputs`, `outputs`, `toObject`, `unsafe_make_weak_compilation_ref`, `isObject`.
- CN: 该代码块处理条件分支。关键符号：`mayContainAlias`, `inputs`, `outputs`, `toObject`, `unsafe_make_weak_compilation_ref`, `isObject`。

### Lines 122-129
```cpp
      if (!v.toObject()->is_weak_compilation_ref()) {
        return std::nullopt;
      }
    }
  }
  return stack;
}

```
- EN: This block handles conditional branches; produces a result or forwards a computed value. Key symbols: `toObject`, `is_weak_compilation_ref`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值。关键符号：`toObject`, `is_weak_compilation_ref`。

### Lines 130-143
```cpp
namespace {

std::unordered_set<Symbol> skip_list = {
    prim::If,
    prim::Loop,
    prim::Closure,
    prim::Constant,
    prim::AutogradZero,
    prim::Uninitialized,
    prim::Guard,
    prim::profile,
    prim::profile_ivalue,
    prim::unchecked_unwrap_optional, // TODO remove
    prim::awaitable,
```
- EN: This block manipulates TorchScript IR objects; performs optimization-oriented rewriting; protects shared state or ordering assumptions. Key symbols: no dominant local symbol names.
- CN: 该代码块操作 TorchScript IR 对象；执行面向优化的改写；保护共享状态或执行顺序假设。关键符号：无明显局部符号。

### Lines 144-152
```cpp
    aten::dequantize,
    // TODO (zach): we should consider skipping tensor factories in the cases
    // where the constant tensor would be large but cheap to create.
};

struct ConstantPropagator {
  // Runs constant propagation with an aliasing db and checks if inputs or
  // outputs might be mutated in the graph
  static ConstantPropagator WithAliasDb(
```
- EN: Declares core types or data containers for this file. Prominent symbols: `ConstantPropagator`, `WithAliasDb`.
- CN: 声明该文件使用的核心类型或数据容器。主要符号：`ConstantPropagator`, `WithAliasDb`。

### Lines 153-160
```cpp
      std::shared_ptr<Graph> graph,
      bool ignore_custom_classes) {
    return ConstantPropagator(std::move(graph), true, ignore_custom_classes);
  }

  // Runs constant propagation only on ops that clearly do not have aliased
  // inputs or outputs without computing aliasing information
  static ConstantPropagator NoAliasDb(std::shared_ptr<Graph> graph) {
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `ConstantPropagator`, `move`, `NoAliasDb`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`ConstantPropagator`, `move`, `NoAliasDb`。

### Lines 161-168
```cpp
    return ConstantPropagator(std::move(graph), false, false);
  }

  bool run() {
    ConstantPropagation(graph_->block());
    return made_change_;
  }

```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting; protects shared state or ordering assumptions. Key symbols: `ConstantPropagator`, `move`, `run`, `ConstantPropagation`, `block`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写；保护共享状态或执行顺序假设。关键符号：`ConstantPropagator`, `move`, `run`, `ConstantPropagation`, `block`。

### Lines 169-177
```cpp
 private:
  ConstantPropagator(
      std::shared_ptr<Graph> graph,
      bool aliasing_types,
      bool ignore_custom_classes)
      : graph_(std::move(graph)),
        aliasing_types_(aliasing_types),
        ignore_custom_classes_(ignore_custom_classes) {}

```
- EN: This block manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `ConstantPropagator`, `graph_`, `move`, `aliasing_types_`, `ignore_custom_classes_`.
- CN: 该代码块操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`ConstantPropagator`, `graph_`, `move`, `aliasing_types_`, `ignore_custom_classes_`。

### Lines 178-187
```cpp
  void propagateNode(Node* n) {
    std::vector<IValue> outputs;
    if (auto outputs_opt =
            runNodeIfInputsAreConstant(n, ignore_custom_classes_)) {
      outputs = std::move(outputs_opt.value());
    } else {
      // The op failed to run, so we cannot continue constant-prop for it.
      return;
    }
    auto graph = n->owningGraph();
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `propagateNode`, `runNodeIfInputsAreConstant`, `move`, `value`, `owningGraph`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`propagateNode`, `runNodeIfInputsAreConstant`, `move`, `value`, `owningGraph`。

### Lines 188-198
```cpp
    WithInsertPoint guard(n);
    for (const auto i : c10::irange(outputs.size())) {
      auto new_output = tryInsertConstant(*graph, outputs[i]);
      if (new_output) {
        made_change_ = true;
        GRAPH_UPDATE(
            "Folding %",
            n->outputs()[i]->debugName(),
            " with ",
            getHeader((*new_output)->node()));
        if (outputs[i].isNone()) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects; performs optimization-oriented rewriting; protects shared state or ordering assumptions. Key symbols: `guard`, `irange`, `size`, `tryInsertConstant`, `outputs`, `debugName`, `...`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象；执行面向优化的改写；保护共享状态或执行顺序假设。关键符号：`guard`, `irange`, `size`, `tryInsertConstant`, `outputs`, `debugName`, `...`。

### Lines 199-207
```cpp
          (*new_output)->setType(n->outputs()[i]->type());
        }
        n->outputs()[i]->replaceAllUsesWith(*new_output);
      }
      // If we cannot insert the IValue as a constant, give up replacing the
      // node and let DCE remove it
    }
  }

```
- EN: This block manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `setType`, `outputs`, `type`, `replaceAllUsesWith`.
- CN: 该代码块操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`setType`, `outputs`, `type`, `replaceAllUsesWith`。

### Lines 208-217
```cpp
  void removeLoopNode(Node* n) {
    auto loop_input_offset = 2; // offset of loop carried deps in input list
    for (size_t i = 0; i < n->outputs().size(); ++i) {
      n->outputs().at(i)->replaceAllUsesWith(
          n->inputs().at(i + loop_input_offset));
    }
    made_change_ = true;
    n->destroy();
  }

```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `removeLoopNode`, `outputs`, `size`, `replaceAllUsesWith`, `inputs`, `destroy`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`removeLoopNode`, `outputs`, `size`, `replaceAllUsesWith`, `inputs`, `destroy`。

### Lines 218-224
```cpp
  bool loopWillNotRun(Node* node) {
    Value* trip_count = node->inputs().at(0);
    int64_t iter_len = constant_as<int64_t>(trip_count).value_or(1);

    Value* start_cond = node->inputs().at(1);
    bool cond_val = constant_as<bool>(start_cond).value_or(true);

```
- EN: This block manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `loopWillNotRun`, `inputs`, `value_or`.
- CN: 该代码块操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`loopWillNotRun`, `inputs`, `value_or`。

### Lines 225-235
```cpp
    bool loop_might_run = cond_val && iter_len > 0;
    if (!loop_might_run) {
      GRAPH_UPDATE(
          "Removing unexecuted loop: ",
          *node,
          "\ntripcount: ",
          trip_count,
          " and start_cond: ",
          getHeader(start_cond->node()));
    }
    return !loop_might_run;
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `getHeader`, `node`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`getHeader`, `node`。

### Lines 236-247
```cpp
  }

  void inlineIfBody(Block* body) {
    Node* n = body->owningNode();
    for (auto it = body->nodes().begin(); it != body->nodes().end();) {
      Node* body_node = *it;
      // advance iterator because after body_node is moved its next pointer will
      // be to n
      it++;
      body_node->moveBefore(n);
    }
    for (size_t i = 0; i < n->outputs().size(); ++i) {
```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects; performs optimization-oriented rewriting; protects shared state or ordering assumptions. Key symbols: `inlineIfBody`, `owningNode`, `nodes`, `begin`, `end`, `moveBefore`, `...`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象；执行面向优化的改写；保护共享状态或执行顺序假设。关键符号：`inlineIfBody`, `owningNode`, `nodes`, `begin`, `end`, `moveBefore`, `...`。

### Lines 248-254
```cpp
      n->outputs().at(i)->replaceAllUsesWith(body->outputs().at(i));
    }
    // NB: destroy the node here, because it might contain side effects, like
    // print
    n->destroy();
  }

```
- EN: This block manipulates TorchScript IR objects. Key symbols: `outputs`, `replaceAllUsesWith`, `destroy`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`outputs`, `replaceAllUsesWith`, `destroy`。

### Lines 255-268
```cpp
  void inlineIf(Node* n) {
    auto input_bool = constant_as<bool>(n->input());
    AT_ASSERT(input_bool);
    GRAPH_UPDATE(
        "Folding if ",
        getHeader(n->input()->node()),
        " where condition = ",
        *input_bool);
    size_t block_index = *input_bool ? 0 : 1;
    ConstantPropagation(n->blocks().at(block_index));
    inlineIfBody(n->blocks().at(block_index));
    made_change_ = true;
  }

```
- EN: This block manipulates TorchScript IR objects; performs optimization-oriented rewriting; protects shared state or ordering assumptions. Key symbols: `inlineIf`, `input`, `getHeader`, `node`, `ConstantPropagation`, `blocks`, `...`.
- CN: 该代码块操作 TorchScript IR 对象；执行面向优化的改写；保护共享状态或执行顺序假设。关键符号：`inlineIf`, `input`, `getHeader`, `node`, `ConstantPropagation`, `blocks`, `...`。

### Lines 269-275
```cpp
  void replaceAndRemoveIfOutput(Node* n, size_t i, Value* replacement) {
    n->outputs().at(i)->replaceAllUsesWith(replacement);
    n->eraseOutput(i);
    n->blocks().at(0)->eraseOutput(i);
    n->blocks().at(1)->eraseOutput(i);
  }

```
- EN: This block manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `replaceAndRemoveIfOutput`, `outputs`, `replaceAllUsesWith`, `eraseOutput`, `blocks`.
- CN: 该代码块操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`replaceAndRemoveIfOutput`, `outputs`, `replaceAllUsesWith`, `eraseOutput`, `blocks`。

### Lines 276-282
```cpp
  // remove extra outputs from the node
  void removeExtraIfOutputs(Node* n) {
    TORCH_CHECK(n->kind() == prim::If, "Only supported for If nodes");
    auto true_block = n->blocks()[0];
    auto false_block = n->blocks()[1];
    auto graph = n->owningGraph();
    auto initial_outputs = true_block->outputs().size();
```
- EN: This block manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `removeExtraIfOutputs`, `kind`, `blocks`, `owningGraph`, `outputs`, `size`.
- CN: 该代码块操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`removeExtraIfOutputs`, `kind`, `blocks`, `owningGraph`, `outputs`, `size`。

### Lines 283-289
```cpp
    WithInsertPoint guard(n);
    for (size_t i = 0; i < true_block->outputs().size();) {
      auto t_out = true_block->outputs().at(i);
      auto f_out = false_block->outputs().at(i);

      // neither block changes the output value
      if (true_block->outputs()[i] == false_block->outputs()[i]) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `guard`, `outputs`, `size`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`guard`, `outputs`, `size`。

### Lines 290-296
```cpp
        replaceAndRemoveIfOutput(n, i, true_block->outputs()[i]);
        continue;
      }

      // true block output is constant and constant matches false block output
      auto maybe_const = toIValue(t_out);
      auto eq = EqualNode();
```
- EN: This block manipulates TorchScript IR objects; performs optimization-oriented rewriting; protects shared state or ordering assumptions. Key symbols: `replaceAndRemoveIfOutput`, `outputs`, `toIValue`, `EqualNode`.
- CN: 该代码块操作 TorchScript IR 对象；执行面向优化的改写；保护共享状态或执行顺序假设。关键符号：`replaceAndRemoveIfOutput`, `outputs`, `toIValue`, `EqualNode`。

### Lines 297-307
```cpp
      if (maybe_const && eq(t_out->node(), f_out->node())) {
        auto new_const = graph->insertConstant(*maybe_const);
        replaceAndRemoveIfOutput(n, i, new_const);
        continue;
      }

      i++; // increment bc we didn't remove current index
    }
    made_change_ |= initial_outputs != true_block->outputs().size();
  }

```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; performs optimization-oriented rewriting; protects shared state or ordering assumptions. Key symbols: `eq`, `node`, `insertConstant`, `replaceAndRemoveIfOutput`, `outputs`, `size`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；执行面向优化的改写；保护共享状态或执行顺序假设。关键符号：`eq`, `node`, `insertConstant`, `replaceAndRemoveIfOutput`, `outputs`, `size`。

### Lines 308-315
```cpp
  // remove extra outputs from the node
  void removeExtraLoopOutputs(Node* node) {
    auto initial_outputs = node->outputs().size();
    auto loop_body = node->blocks().at(0);
    auto loop_input_offset = 2; // offset of loop carried deps in input list
    auto loop_body_offset =
        1; // offset to the loop carried dependencies in block inputs/outputs
    for (size_t i_1 = node->outputs().size(); i_1 > 0; --i_1) {
```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `removeExtraLoopOutputs`, `outputs`, `size`, `blocks`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`removeExtraLoopOutputs`, `outputs`, `size`, `blocks`。

### Lines 316-329
```cpp
      size_t i = i_1 - 1;
      // if the value is no longer changed remove output
      if (loop_body->inputs().at(loop_body_offset + i) ==
          loop_body->outputs().at(loop_body_offset + i)) {
        auto node_input = node->inputs().at(loop_input_offset + i);
        node->outputs().at(i)->replaceAllUsesWith(node_input);
        loop_body->inputs()
            .at(loop_body_offset + i)
            ->replaceAllUsesWith(node_input);
        node->eraseOutput(i);
        node->removeInput(loop_input_offset + i);
        loop_body->eraseInput(loop_body_offset + i);
        loop_body->eraseOutput(loop_body_offset + i);
      }
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `inputs`, `outputs`, `replaceAllUsesWith`, `eraseOutput`, `removeInput`, `eraseInput`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`inputs`, `outputs`, `replaceAllUsesWith`, `eraseOutput`, `removeInput`, `eraseInput`。

### Lines 330-336
```cpp
    }
    made_change_ |= initial_outputs != node->outputs().size();
  }

  bool noMutableValues(at::ArrayRef<Value*> values) {
    return std::none_of(values.begin(), values.end(), [](Value* v) {
      return AliasDb::isMutableType(v);
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `outputs`, `size`, `noMutableValues`, `none_of`, `begin`, `end`, `...`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`outputs`, `size`, `noMutableValues`, `none_of`, `begin`, `end`, `...`。

### Lines 337-344
```cpp
    });
  }

  AliasDb* getOrCreateAliasDb() {
    if (!aliasDb_) {
      aliasDb_ = std::make_unique<AliasDb>(graph_);
    }
    return aliasDb_.get();
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `getOrCreateAliasDb`, `get`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`getOrCreateAliasDb`, `get`。

### Lines 345-355
```cpp
  }

  bool supportedNode(Node* n) {
    bool no_mutation = false;
    if (aliasing_types_) {
      no_mutation = !getOrCreateAliasDb()->hasWriters(n);
    } else {
      no_mutation =
          noMutableValues(n->inputs()) && noMutableValues(n->outputs());
    }
    return no_mutation && !n->kind().is_onnx() &&
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `supportedNode`, `getOrCreateAliasDb`, `hasWriters`, `noMutableValues`, `inputs`, `outputs`, `...`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`supportedNode`, `getOrCreateAliasDb`, `hasWriters`, `noMutableValues`, `inputs`, `outputs`, `...`。

### Lines 356-365
```cpp
        skip_list.count(n->kind()) == 0 && !n->isNondeterministic() &&
        !n->hasSideEffects() && n->blocks().empty();
  }

  void ConstantPropagation(at::ArrayRef<Block*> blocks) {
    for (Block* block : blocks) {
      ConstantPropagation(block);
    }
  }

```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects; performs optimization-oriented rewriting; protects shared state or ordering assumptions. Key symbols: `count`, `kind`, `isNondeterministic`, `hasSideEffects`, `blocks`, `empty`, `...`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象；执行面向优化的改写；保护共享状态或执行顺序假设。关键符号：`count`, `kind`, `isNondeterministic`, `hasSideEffects`, `blocks`, `empty`, `...`。

### Lines 366-373
```cpp
  void ConstantPropagation(Node* n) {
    bool constant_inputs =
        std::all_of(n->inputs().begin(), n->inputs().end(), [&](Value* v) {
          return v->node()->kind() == prim::Constant;
        });
    if (n->kind() == prim::If) {
      // inline node if we can, otherwise check for simplified outputs
      if (constant_inputs) {
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `ConstantPropagation`, `all_of`, `inputs`, `begin`, `end`, `node`, `...`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`ConstantPropagation`, `all_of`, `inputs`, `begin`, `end`, `node`, `...`。

### Lines 374-380
```cpp
        inlineIf(n);
      } else {
        ConstantPropagation(n->blocks());
        removeExtraIfOutputs(n);
      }
    } else if (n->kind() == prim::Loop) {
      if (loopWillNotRun(n)) {
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; performs optimization-oriented rewriting; protects shared state or ordering assumptions. Key symbols: `inlineIf`, `ConstantPropagation`, `blocks`, `removeExtraIfOutputs`, `kind`, `loopWillNotRun`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；执行面向优化的改写；保护共享状态或执行顺序假设。关键符号：`inlineIf`, `ConstantPropagation`, `blocks`, `removeExtraIfOutputs`, `kind`, `loopWillNotRun`。

### Lines 381-392
```cpp
        removeLoopNode(n);
      } else {
        ConstantPropagation(n->blocks());
        removeExtraLoopOutputs(n);
      }
    } else if (constant_inputs && supportedNode(n)) {
      propagateNode(n);
    } else {
      ConstantPropagation(n->blocks());
    }
  }

```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; performs optimization-oriented rewriting; protects shared state or ordering assumptions. Key symbols: `removeLoopNode`, `ConstantPropagation`, `blocks`, `removeExtraLoopOutputs`, `supportedNode`, `propagateNode`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；执行面向优化的改写；保护共享状态或执行顺序假设。关键符号：`removeLoopNode`, `ConstantPropagation`, `blocks`, `removeExtraLoopOutputs`, `supportedNode`, `propagateNode`。

### Lines 393-400
```cpp
  void ConstantPropagation(Block* block) {
    for (auto it = block->nodes().begin(); it != block->nodes().end();) {
      Node* n = *it;
      it++; // advance iterator bc the current node may be destroyed
      ConstantPropagation(n);
    }
  }

```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects; performs optimization-oriented rewriting; protects shared state or ordering assumptions. Key symbols: `ConstantPropagation`, `nodes`, `begin`, `end`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象；执行面向优化的改写；保护共享状态或执行顺序假设。关键符号：`ConstantPropagation`, `nodes`, `begin`, `end`。

### Lines 401-407
```cpp
  std::shared_ptr<Graph> graph_;
  // lazily initialized if using aliasing_types, otherwise not initialized
  std::unique_ptr<AliasDb> aliasDb_ = nullptr;
  bool aliasing_types_;
  bool made_change_ = false;
  bool ignore_custom_classes_;
};
```
- EN: This block manipulates TorchScript IR objects. Key symbols: no dominant local symbol names.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：无明显局部符号。

### Lines 408-415
```cpp
} // anonymous namespace

bool ConstantPropagation(
    std::shared_ptr<Graph>& graph,
    bool ignore_custom_classes) {
  ConstantPropagator cp =
      ConstantPropagator::WithAliasDb(graph, ignore_custom_classes);
  bool made_change = cp.run();
```
- EN: This block manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `ConstantPropagation`, `WithAliasDb`, `run`.
- CN: 该代码块操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`ConstantPropagation`, `WithAliasDb`, `run`。

### Lines 416-422
```cpp
  if (made_change) {
    EliminateDeadCode(graph);
  }
  GRAPH_DUMP("After ConstantPropagation: ", graph);
  return made_change;
}

```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `EliminateDeadCode`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`EliminateDeadCode`。

### Lines 423-430
```cpp
bool ConstantPropagationImmutableTypes(std::shared_ptr<Graph>& graph) {
  ConstantPropagator cp = ConstantPropagator::NoAliasDb(graph);
  bool made_change = cp.run();
  if (made_change) {
    EliminateDeadCode(graph);
  }
  GRAPH_DUMP("After ConstantPropagationImmutableTypes: ", graph);
  return made_change;
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `ConstantPropagationImmutableTypes`, `NoAliasDb`, `run`, `EliminateDeadCode`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`ConstantPropagationImmutableTypes`, `NoAliasDb`, `run`, `EliminateDeadCode`。

### Lines 431-433
```cpp
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
- EN: Schema handling — the implementation parses or normalizes operator/function signatures.
  CN: 模式处理——实现会解析或规范化算子/函数签名。
- EN: Optimization pass — the code simplifies graphs by removing redundant work or folding values.
  CN: 优化 pass——代码通过删除冗余计算或常量折叠来简化图。
- EN: Inlining — the implementation merges callees or blocks into their call sites.
  CN: 内联——实现会把被调用者或代码块并入调用位置。
- EN: Type and shape reasoning — the code propagates static information across the IR.
  CN: 类型与形状推理——代码会在 IR 中传播静态信息。

## Dependencies / 依赖关系
- Internal includes / 内部头文件: `torch/csrc/jit/passes/constant_propagation.h`, `ATen/core/ivalue.h`, `c10/util/Exception.h`, `c10/util/irange.h`, `torch/csrc/jit/ir/alias_analysis.h`, `torch/csrc/jit/ir/constants.h`, `torch/csrc/jit/ir/ir.h`, `torch/csrc/jit/ir/node_hashing.h`, `torch/csrc/jit/jit_log.h`, `torch/csrc/jit/passes/dead_code_elimination.h`, `...`
- External includes / 外部头文件: `utility`
- Namespaces / 命名空间: `torch::jit`
- Representative symbols / 代表性符号: `runNodeIfInputsAreConstant`, `inputs`, `toIValue`, `push_back`, `kind`, `back`, `toList`, `size`, `outputs`, `listUnpack`, `...`
