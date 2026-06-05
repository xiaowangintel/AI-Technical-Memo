# peephole.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `torch/csrc/jit/passes/peephole.cpp`
- Repository: `pytorch`
- Purpose (EN): Implements the TorchScript JIT pass logic for peephole, including graph analysis and rewrites.
- 用途 (CN): 实现与 peephole 相关的 TorchScript JIT pass 逻辑，包括图分析与改写。

## Line-by-Line Analysis / 逐行分析
### Lines 1-12
```cpp
#include <torch/csrc/jit/passes/peephole.h>

#include <ATen/core/jit_type.h>
#include <c10/util/irange.h>
#include <torch/csrc/jit/jit_log.h>
#include <torch/csrc/jit/passes/concat_opt.h>
#include <torch/csrc/jit/passes/dead_code_elimination.h>
#include <torch/csrc/jit/passes/peephole_alias_sensitive.h>
#include <torch/csrc/jit/passes/peephole_dict_idioms.h>
#include <torch/csrc/jit/passes/peephole_list_idioms.h>
#include <torch/csrc/jit/passes/peephole_non_tensor.h>

```
- EN: Pulls in the headers needed by the peephole logic. Internal dependencies: `torch/csrc/jit/passes/peephole.h`, `ATen/core/jit_type.h`, `c10/util/irange.h`, `torch/csrc/jit/jit_log.h`, `torch/csrc/jit/passes/concat_opt.h`, `...`; external dependencies: none.
- CN: 为 peephole 相关逻辑引入所需头文件。内部依赖：`torch/csrc/jit/passes/peephole.h`, `ATen/core/jit_type.h`, `c10/util/irange.h`, `torch/csrc/jit/jit_log.h`, `torch/csrc/jit/passes/concat_opt.h`, `...`；外部依赖：无。

### Lines 13-19
```cpp
namespace torch::jit {

// Conservatively compare two optionals. If both are undefined, assume
// they aren't equal
template <typename T>
static bool mustBeEqual(const std::optional<T>& a, const std::optional<T>& b) {
  return a == b && a.has_value();
```
- EN: Builds a reusable template/helper layer around peephole. Symbols: `mustBeEqual`, `has_value`.
- CN: 围绕 peephole 构建可复用的模板或辅助层。符号：`mustBeEqual`, `has_value`。

### Lines 20-27
```cpp
}

struct PeepholeOptimizeImpl {
  PeepholeOptimizeImpl(
      std::shared_ptr<Graph> graph,
      bool disable_shape_peepholes)
      : graph_(std::move(graph)), shape_peepholes_(!disable_shape_peepholes) {}

```
- EN: Declares core types or data containers for this file. Prominent symbols: `PeepholeOptimizeImpl`, `graph_`, `move`, `shape_peepholes_`.
- CN: 声明该文件使用的核心类型或数据容器。主要符号：`PeepholeOptimizeImpl`, `graph_`, `move`, `shape_peepholes_`。

### Lines 28-35
```cpp
  bool run() {
    bool changed = optimizeBlock(graph_->block());
    changed |= PeepholeOptimizeListIdioms(graph_);
    changed |= PeepholeOptimizeDictIdioms(graph_);
    changed |= PeepholeOptimizeAliasSensitive(graph_, shape_peepholes_);
    changed |= PeepholeOptimizeNonTensor(graph_);
    changed |= CombineConcats(graph_);
    return changed;
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting; protects shared state or ordering assumptions. Key symbols: `run`, `optimizeBlock`, `block`, `PeepholeOptimizeListIdioms`, `PeepholeOptimizeDictIdioms`, `PeepholeOptimizeAliasSensitive`, `...`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写；保护共享状态或执行顺序假设。关键符号：`run`, `optimizeBlock`, `block`, `PeepholeOptimizeListIdioms`, `PeepholeOptimizeDictIdioms`, `PeepholeOptimizeAliasSensitive`, `...`。

### Lines 36-42
```cpp
  }

  // The intent for this optimization pass is to catch all of the small, easy to
  // catch peephole optimizations you might be interested in doing.
  //
  // TODO: Decide what kind of fixed point strategy we will have
  bool optimizeBlock(Block* block) {
```
- EN: This block manipulates TorchScript IR objects; performs optimization-oriented rewriting; protects shared state or ordering assumptions. Key symbols: `optimizeBlock`.
- CN: 该代码块操作 TorchScript IR 对象；执行面向优化的改写；保护共享状态或执行顺序假设。关键符号：`optimizeBlock`。

### Lines 43-50
```cpp
    bool changed = false;
    for (auto it = block->nodes().begin(); it != block->nodes().end(); ++it) {
      auto* node = *it;

      for (Block* sub_block : node->blocks()) {
        changed |= optimizeBlock(sub_block);
      }

```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `nodes`, `begin`, `end`, `blocks`, `optimizeBlock`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`nodes`, `begin`, `end`, `blocks`, `optimizeBlock`。

### Lines 51-61
```cpp
      // XXX: remember that if you want to simplify an expression by combining
      // multiple nodes into a different one, then you need to check that they
      // all belong to the given block
      // TODO: this doesn't work with Scalar-Tensor ops! We should
      // canonicalize those
      if (node->matches(
              "aten::_grad_sum_to_size(Tensor(a) self, int[]? size) -> Tensor(a)")) {
        // Eliminate no-op _grad_sum_to_size.
        // TODO: this doesn't work with Scalar-Tensor ops! We should
        // canonicalize those
        if (node->input(1)->mustBeNone()) {
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; performs optimization-oriented rewriting; protects shared state or ordering assumptions. Key symbols: `matches`, `_grad_sum_to_size`, `Tensor`, `input`, `mustBeNone`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；执行面向优化的改写；保护共享状态或执行顺序假设。关键符号：`matches`, `_grad_sum_to_size`, `Tensor`, `input`, `mustBeNone`。

### Lines 62-69
```cpp
          GRAPH_UPDATE(
              getHeader(node),
              " (x._grad_sum_to_size(x, None) == x) is replaced with ",
              node->input(0)->debugName());
          node->output()->replaceAllUsesWith(node->input(0));
          changed = true;
        } else {
          auto uses = node->output()->uses();
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `getHeader`, `_grad_sum_to_size`, `input`, `debugName`, `output`, `replaceAllUsesWith`, `...`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`getHeader`, `_grad_sum_to_size`, `input`, `debugName`, `output`, `replaceAllUsesWith`, `...`。

### Lines 70-83
```cpp
          for (Use u : uses) {
            if (u.user->matches(
                    "aten::_grad_sum_to_size(Tensor(a) self, int[]? size) -> Tensor(a)") &&
                u.user->input(1)->type()->isSubtypeOf(*ListType::ofInts())) {
              GRAPH_UPDATE(
                  getHeader(node),
                  " (x._grad_sum_to_size(y)._grad_sum_to_size(z) == x._grad_sum_to_size(z)) is replaced with ",
                  node->inputs().at(0)->debugName());
              u.user->replaceInput(0, node->inputs().at(0));
              changed = true;
            }
          }
        }
      } else if (
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `matches`, `_grad_sum_to_size`, `Tensor`, `input`, `type`, `isSubtypeOf`, `...`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`matches`, `_grad_sum_to_size`, `Tensor`, `input`, `type`, `isSubtypeOf`, `...`。

### Lines 84-90
```cpp
          node->matches(
              "aten::expand(Tensor self, int[] size, *, bool implicit) -> Tensor",
              /*const_inputs=*/attr::size)) {
        // x.expand(x.size()) == x
        auto input_type =
            node->namedInput(attr::self)->type()->cast<TensorType>();
        if (input_type && shape_peepholes_) {
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `matches`, `expand`, `namedInput`, `type`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`matches`, `expand`, `namedInput`, `type`。

### Lines 91-104
```cpp
          auto expanded_sizes = node->get<c10::List<int64_t>>(attr::size);
          auto input_type_sizes = input_type->sizes().concrete_sizes();
          if (expanded_sizes.has_value() && input_type_sizes &&
              expanded_sizes->vec() == *input_type_sizes) {
            GRAPH_UPDATE(
                getHeader(node),
                " (x.expand(x.size()) == x) is replaced with ",
                node->namedInput(attr::self)->debugName());
            node->output()->replaceAllUsesWith(node->namedInput(attr::self));
            changed = true;
          }
        }
      } else if (node->matches("aten::t(Tensor self) -> Tensor")) {
        // x.t().t() == x
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `sizes`, `concrete_sizes`, `has_value`, `vec`, `getHeader`, `expand`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`sizes`, `concrete_sizes`, `has_value`, `vec`, `getHeader`, `expand`, `...`。

### Lines 105-118
```cpp
        Node* input_node = node->input()->node();
        if (input_node->matches("aten::t(Tensor self) -> Tensor")) {
          GRAPH_UPDATE(
              getHeader(node),
              " (x.t().t() == x) is replaced with ",
              input_node->input()->debugName());
          node->output()->replaceAllUsesWith(input_node->input());
          changed = true;
        }
      } else if (
          node->matches("aten::type_as(Tensor self, Tensor other) -> Tensor") &&
          shape_peepholes_) {
        // x.type_as(y) == x iff x.type() == y.type()
        auto self_type = node->input(0)->type()->expect<TensorType>();
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `input`, `node`, `matches`, `t`, `getHeader`, `debugName`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`input`, `node`, `matches`, `t`, `getHeader`, `debugName`, `...`。

### Lines 119-132
```cpp
        auto other_type = node->input(1)->type()->expect<TensorType>();
        if (mustBeEqual(self_type->scalarType(), other_type->scalarType()) &&
            mustBeEqual(self_type->device(), other_type->device())) {
          GRAPH_UPDATE(
              getHeader(node),
              " (x.type_as(y) == x) is replaced with ",
              node->input(0)->debugName());
          node->output()->replaceAllUsesWith(node->input(0));
          changed = true;
        }
      } else if (
          node->kind() == aten::Float || node->kind() == aten::Int ||
          node->kind() == aten::FloatImplicit ||
          node->kind() == aten::IntImplicit ||
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `input`, `type`, `mustBeEqual`, `scalarType`, `device`, `getHeader`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`input`, `type`, `mustBeEqual`, `scalarType`, `device`, `getHeader`, `...`。

### Lines 133-146
```cpp
          node->kind() == aten::ScalarImplicit) {
        Node* input_node = node->input()->node();
        if (input_node->kind() == prim::NumToTensor) {
          GRAPH_UPDATE(
              getHeader(node),
              " (x.NumToTensor() == x) is replaced with ",
              node->input()->debugName());
          node->output()->replaceAllUsesWith(input_node->input());
          changed = true;
        }
      } else if (
          node->matches("aten::size(Tensor self) -> int[]") &&
          shape_peepholes_) {
        if (auto ptt = node->input()->type()->cast<TensorType>()) {
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `kind`, `input`, `node`, `getHeader`, `NumToTensor`, `debugName`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`kind`, `input`, `node`, `getHeader`, `NumToTensor`, `debugName`, `...`。

### Lines 147-154
```cpp
          if (auto sizes = ptt->sizes().concrete_sizes()) {
            GRAPH_UPDATE(
                getHeader(node),
                " (x.size()) is replaced with ",
                node->input()->debugName());
            WithInsertPoint guard(node);
            IValue ival(sizes);
            auto const_sizes_val = node->owningGraph()->insertConstant(ival);
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; performs optimization-oriented rewriting; protects shared state or ordering assumptions. Key symbols: `sizes`, `concrete_sizes`, `getHeader`, `size`, `input`, `debugName`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；执行面向优化的改写；保护共享状态或执行顺序假设。关键符号：`sizes`, `concrete_sizes`, `getHeader`, `size`, `input`, `debugName`, `...`。

### Lines 155-163
```cpp
            node->output()->replaceAllUsesWith(const_sizes_val);
            changed = true;
          }
        }
      } else if (
          node->matches("aten::len.t(t[] a) -> int") &&
          node->input()->node()->matches("aten::size(Tensor self) -> int[]") &&
          shape_peepholes_) {
        auto ptt = node->input()->node()->input()->type()->expect<TensorType>();
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `output`, `replaceAllUsesWith`, `matches`, `t`, `input`, `node`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`output`, `replaceAllUsesWith`, `matches`, `t`, `input`, `node`, `...`。

### Lines 164-177
```cpp
        // only handle one use case for now to avoid modifying mutated lists
        // TODO: canonicalize as aten::dim ?
        if (ptt->sizes().size() && node->input()->uses().size() == 1) {
          WithInsertPoint guard(node);
          auto output = node->owningGraph()->insertConstant(
              static_cast<int64_t>(*ptt->sizes().size()));
          GRAPH_UPDATE(
              "Replacing ",
              getHeader(node),
              " with a \"dim\" constant ",
              output->debugName());
          node->output()->replaceAllUsesWith(output);
          changed = true;
        }
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; performs optimization-oriented rewriting; protects shared state or ordering assumptions. Key symbols: `sizes`, `size`, `input`, `uses`, `guard`, `owningGraph`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；执行面向优化的改写；保护共享状态或执行顺序假设。关键符号：`sizes`, `size`, `input`, `uses`, `guard`, `owningGraph`, `...`。

### Lines 178-184
```cpp
      } else if (
          node->matches("aten::size(Tensor self, int dim) -> int") &&
          shape_peepholes_) {
        if (auto ptt = node->inputs().at(0)->type()->cast<TensorType>()) {
          if (auto maybe_ndim = ptt->sizes().size()) {
            auto ndim = static_cast<int64_t>(*maybe_ndim);
            auto maybe_index = toIValue(node->inputs().at(1));
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `matches`, `size`, `inputs`, `type`, `sizes`, `toIValue`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`matches`, `size`, `inputs`, `type`, `sizes`, `toIValue`。

### Lines 185-194
```cpp
            if (!maybe_index) {
              continue;
            }
            int64_t index = maybe_index->toInt();
            int64_t norm_index = index < 0 ? ndim + index : index;
            if (norm_index >= 0 && norm_index < ndim &&
                ptt->sizes()[norm_index]) {
              WithInsertPoint guard(node);
              IValue ival(*ptt->sizes()[norm_index]);
              auto const_sizes_val = node->owningGraph()->insertConstant(ival);
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; performs optimization-oriented rewriting; protects shared state or ordering assumptions. Key symbols: `toInt`, `sizes`, `guard`, `ival`, `owningGraph`, `insertConstant`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；执行面向优化的改写；保护共享状态或执行顺序假设。关键符号：`toInt`, `sizes`, `guard`, `ival`, `owningGraph`, `insertConstant`。

### Lines 195-207
```cpp
              node->output()->replaceAllUsesWith(const_sizes_val);
              GRAPH_UPDATE(
                  getHeader(node),
                  " (x.size(dim)) is replaced with constant ",
                  const_sizes_val->debugName());
              changed = true;
            }
          }
        }
      } else if (
          node->matches("aten::is_floating_point(Tensor self) -> bool") &&
          shape_peepholes_) {
        auto ptt = node->inputs().at(0)->type()->cast<TensorType>();
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `output`, `replaceAllUsesWith`, `getHeader`, `size`, `debugName`, `matches`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`output`, `replaceAllUsesWith`, `getHeader`, `size`, `debugName`, `matches`, `...`。

### Lines 208-221
```cpp
        if (auto maybe_dtype = ptt->scalarType()) {
          c10::ScalarType dtype = *maybe_dtype;
          WithInsertPoint guard(node);
          IValue ival(at::isFloatingType(dtype));
          auto new_constant = node->owningGraph()->insertConstant(ival);
          node->output()->replaceAllUsesWith(new_constant);
          GRAPH_UPDATE(
              getHeader(node),
              " (x.is_floating_point()) is replaced with ",
              new_constant->debugName());
          changed = true;
        }
      } else if (
          node->matches("aten::is_complex(Tensor self) -> bool") &&
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; performs optimization-oriented rewriting; protects shared state or ordering assumptions. Key symbols: `scalarType`, `guard`, `ival`, `isFloatingType`, `owningGraph`, `insertConstant`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；执行面向优化的改写；保护共享状态或执行顺序假设。关键符号：`scalarType`, `guard`, `ival`, `isFloatingType`, `owningGraph`, `insertConstant`, `...`。

### Lines 222-228
```cpp
          shape_peepholes_) {
        auto ptt = node->inputs().at(0)->type()->cast<TensorType>();
        if (auto maybe_dtype = ptt->scalarType()) {
          c10::ScalarType dtype = *maybe_dtype;
          WithInsertPoint guard(node);
          IValue ival(at::isComplexType(dtype));
          auto new_constant = node->owningGraph()->insertConstant(ival);
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; performs optimization-oriented rewriting; protects shared state or ordering assumptions. Key symbols: `inputs`, `type`, `scalarType`, `guard`, `ival`, `isComplexType`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；执行面向优化的改写；保护共享状态或执行顺序假设。关键符号：`inputs`, `type`, `scalarType`, `guard`, `ival`, `isComplexType`, `...`。

### Lines 229-238
```cpp
          node->output()->replaceAllUsesWith(new_constant);
          GRAPH_UPDATE(
              getHeader(node),
              " (x.is_complex()) is replaced with ",
              new_constant->debugName());
          changed = true;
        }
      } else if (
          node->matches("prim::dtype(Tensor a) -> int") && shape_peepholes_) {
        auto ptt = node->input()->type()->expect<TensorType>();
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `output`, `replaceAllUsesWith`, `getHeader`, `is_complex`, `debugName`, `matches`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`output`, `replaceAllUsesWith`, `getHeader`, `is_complex`, `debugName`, `matches`, `...`。

### Lines 239-252
```cpp
        if (ptt->scalarType()) {
          WithInsertPoint guard(node);
          auto output = node->owningGraph()->insertConstant(
              static_cast<int64_t>(*ptt->scalarType()));
          GRAPH_UPDATE(
              "Replacing ",
              getHeader(node),
              " with a type constant ",
              output->debugName());
          node->output()->replaceAllUsesWith(output);
          changed = true;
        }
      } else if (
          node->matches("prim::device(Tensor a) -> Device") &&
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; performs optimization-oriented rewriting; protects shared state or ordering assumptions. Key symbols: `scalarType`, `guard`, `owningGraph`, `insertConstant`, `getHeader`, `debugName`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；执行面向优化的改写；保护共享状态或执行顺序假设。关键符号：`scalarType`, `guard`, `owningGraph`, `insertConstant`, `getHeader`, `debugName`, `...`。

### Lines 253-266
```cpp
          shape_peepholes_) {
        auto ptt = node->input()->type()->expect<TensorType>();
        if (ptt->device()) {
          WithInsertPoint guard(node);
          auto output = node->owningGraph()->insertConstant(*ptt->device());
          GRAPH_UPDATE(
              "Replacing ",
              getHeader(node),
              " with a device constant ",
              output->debugName());
          node->output()->replaceAllUsesWith(output);
          changed = true;
        }
      } else if (
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; performs optimization-oriented rewriting; protects shared state or ordering assumptions. Key symbols: `input`, `type`, `device`, `guard`, `owningGraph`, `insertConstant`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；执行面向优化的改写；保护共享状态或执行顺序假设。关键符号：`input`, `type`, `device`, `guard`, `owningGraph`, `insertConstant`, `...`。

### Lines 267-273
```cpp
          node->matches("aten::device(str type, int index) -> Device") &&
          shape_peepholes_) {
        auto string_type = node->inputs().at(0)->type()->expect<StringType>();
        if (string_type) {
          WithInsertPoint guard(node);
          std::string type_str = node->inputs().at(0)->node()->s(attr::value);
          auto maybe_index = toIValue(node->inputs().at(1));
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; performs optimization-oriented rewriting; protects shared state or ordering assumptions. Key symbols: `matches`, `device`, `inputs`, `type`, `guard`, `node`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；执行面向优化的改写；保护共享状态或执行顺序假设。关键符号：`matches`, `device`, `inputs`, `type`, `guard`, `node`, `...`。

### Lines 274-287
```cpp
          int64_t index = 0;
          if (maybe_index) {
            index = maybe_index->toInt();
          }
          auto device = c10::Device(type_str + ":" + std::to_string(index));
          auto output = node->owningGraph()->insertConstant(device);
          GRAPH_UPDATE(
              "Replacing ",
              getHeader(node),
              " with a device constant ",
              output->debugName());
          node->output()->replaceAllUsesWith(output);
          changed = true;
        }
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `toInt`, `Device`, `to_string`, `owningGraph`, `insertConstant`, `getHeader`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`toInt`, `Device`, `to_string`, `owningGraph`, `insertConstant`, `getHeader`, `...`。

### Lines 288-301
```cpp
      } else if (
          node->matches("aten::dim(Tensor self) -> int") && shape_peepholes_) {
        auto ptt = node->input()->type()->expect<TensorType>();
        if (auto dim = ptt->sizes().size()) {
          WithInsertPoint guard(node);
          auto output =
              node->owningGraph()->insertConstant(static_cast<int64_t>(*dim));
          GRAPH_UPDATE(
              "Replacing ",
              getHeader(node),
              " with a \"dim\" constant ",
              output->debugName());
          node->output()->replaceAllUsesWith(output);
          changed = true;
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; performs optimization-oriented rewriting; protects shared state or ordering assumptions. Key symbols: `matches`, `dim`, `input`, `type`, `sizes`, `size`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；执行面向优化的改写；保护共享状态或执行顺序假设。关键符号：`matches`, `dim`, `input`, `type`, `sizes`, `size`, `...`。

### Lines 302-309
```cpp
        }
      } else if (
          node->matches("prim::is_cuda(Tensor a) -> bool") &&
          shape_peepholes_) {
        auto ptt = node->input()->type()->expect<TensorType>();
        if (ptt->device()) {
          WithInsertPoint guard(node);
          auto output =
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; performs optimization-oriented rewriting; protects shared state or ordering assumptions. Key symbols: `matches`, `is_cuda`, `input`, `type`, `device`, `guard`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；执行面向优化的改写；保护共享状态或执行顺序假设。关键符号：`matches`, `is_cuda`, `input`, `type`, `device`, `guard`。

### Lines 310-321
```cpp
              node->owningGraph()->insertConstant((*ptt->device()).is_cuda());
          GRAPH_UPDATE(
              "Replacing ",
              getHeader(node),
              " with a is_cuda constant ",
              output->debugName());
          node->output()->replaceAllUsesWith(output);
          changed = true;
        }
      }
    }
    return changed;
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `owningGraph`, `insertConstant`, `device`, `is_cuda`, `getHeader`, `debugName`, `...`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`owningGraph`, `insertConstant`, `device`, `is_cuda`, `getHeader`, `debugName`, `...`。

### Lines 322-328
```cpp
  }

 private:
  std::shared_ptr<Graph> graph_;
  bool shape_peepholes_;
};

```
- EN: This block manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: no dominant local symbol names.
- CN: 该代码块操作 TorchScript IR 对象；执行面向优化的改写。关键符号：无明显局部符号。

### Lines 329-335
```cpp
static bool FuseAddMM(Block* block) {
  bool changed = false;
  for (Node* node : block->nodes()) {
    // XXX: remember that if you want to simplify an expression by combining
    // multiple nodes into a different one, then you need to check that they
    // all belong to the given block
    if (node->matches(
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `FuseAddMM`, `nodes`, `matches`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`FuseAddMM`, `nodes`, `matches`。

### Lines 336-348
```cpp
            "aten::add(Tensor self, Tensor other, *, Scalar alpha) -> Tensor",
            /*const_inputs=*/attr::alpha)) {
      // z + x.mm(y) == z.addmm(x, y) == x.mm(y) + z
      if (node->get<at::Scalar>(attr::alpha).value().toDouble() == 1.) {
        // Look for mm from both sides of the add
        for (const auto mm_side : c10::irange(2)) {
          // Add will accept tensors of mismatched scalar types, as long as
          // one of them is a scalar, but addmm will throw in that case, so we
          // can only perform this fusion if we're sure that it is correct,
          // and for that we need the add_mat_type. An alternative would be to
          // insert a type_as conditional on the tensor shape being a scalar,
          // but that might add overhead, and make analysis harder.
          auto add_mat_type =
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `add`, `value`, `toDouble`, `irange`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`add`, `value`, `toDouble`, `irange`。

### Lines 349-355
```cpp
              node->input(1 - mm_side)->type()->expect<TensorType>();
          // if we don't have the rank, we can't tell if the bias is a scalar
          if (!add_mat_type->sizes().size()) {
            continue;
          }

          if (node->input(mm_side)->node()->matches(
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `input`, `type`, `sizes`, `size`, `node`, `matches`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`input`, `type`, `sizes`, `size`, `node`, `matches`。

### Lines 356-364
```cpp
                  "aten::mm(Tensor self, Tensor mat2) -> Tensor")) {
            WithInsertPoint guard(node);

            auto* graph = node->owningGraph();
            auto* mm_node = node->input(mm_side)->node();
            auto* add_mat = node->input(1 - mm_side);
            auto* mat1 = mm_node->input(0);
            auto* mat2 = mm_node->input(1);

```
- EN: This block manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `mm`, `guard`, `owningGraph`, `input`, `node`.
- CN: 该代码块操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`mm`, `guard`, `owningGraph`, `input`, `node`。

### Lines 365-372
```cpp
            // Attempts to find a matrix with a defined scalar type to type as
            auto* type_as_mat = mat1;
            if (!type_as_mat->type()->expectRef<TensorType>().scalarType()) {
              type_as_mat = mat2;
            }
            auto mat_scalar_type =
                type_as_mat->type()->expectRef<TensorType>().scalarType();

```
- EN: This block handles conditional branches. Key symbols: `type`, `scalarType`.
- CN: 该代码块处理条件分支。关键符号：`type`, `scalarType`。

### Lines 373-382
```cpp
            // we can't use type_as if we don't know the target type (mm), the
            // bias needs to be coerced to
            if (!mat_scalar_type) {
              continue;
            }

            // We insert the type_as if we're sure that the added element is a
            // scalar, and we either don't know the type of the scalar, or
            // know that it's mismatched.
            if (add_mat_type->sizes().size() &&
```
- EN: This block handles conditional branches. Key symbols: `sizes`, `size`.
- CN: 该代码块处理条件分支。关键符号：`sizes`, `size`。

### Lines 383-390
```cpp
                *add_mat_type->sizes().size() == 0 &&
                !mustBeEqual(add_mat_type->scalarType(), mat_scalar_type)) {
              auto* type_as_node =
                  graph->insertNode(graph->create(aten::type_as, 1));
              type_as_node->addInput(add_mat);
              type_as_node->addInput(type_as_mat);
              add_mat = type_as_node->output();
              if (add_mat_type->isComplete()) {
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `sizes`, `size`, `mustBeEqual`, `scalarType`, `insertNode`, `create`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`sizes`, `size`, `mustBeEqual`, `scalarType`, `insertNode`, `create`, `...`。

### Lines 391-404
```cpp
                auto new_type =
                    add_mat_type->withScalarType(mat_scalar_type)->contiguous();
                add_mat->setType(new_type);
              }
            }

            auto* cOne = graph->insertConstant(1);
            auto* addmm_node = graph->insertNode(graph->create(aten::addmm, 1));
            addmm_node->addInput(add_mat);
            addmm_node->addInput(mat1);
            addmm_node->addInput(mat2);
            addmm_node->addInput(cOne);
            addmm_node->addInput(cOne);
            auto* addmm_value = addmm_node->output();
```
- EN: This block manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `withScalarType`, `contiguous`, `setType`, `insertConstant`, `insertNode`, `create`, `...`.
- CN: 该代码块操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`withScalarType`, `contiguous`, `setType`, `insertConstant`, `insertNode`, `create`, `...`。

### Lines 405-418
```cpp

            // Copy shape information from output node
            addmm_value->copyMetadata(node->output());
            GRAPH_UPDATE(
                "Fusing ",
                mm_node->input(0)->debugName(),
                ", ",
                mm_node->input(1)->debugName(),
                " and ",
                node->input(1 - mm_side)->debugName(),
                " into ",
                addmm_value->debugName());
            node->output()->replaceAllUsesWith(addmm_value);
            changed = true;
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `copyMetadata`, `output`, `input`, `debugName`, `replaceAllUsesWith`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`copyMetadata`, `output`, `input`, `debugName`, `replaceAllUsesWith`。

### Lines 419-428
```cpp
            continue;
          }
        }
      }
    }
    for (Block* b : node->blocks()) {
      changed |= FuseAddMM(b);
    }
  }
  return changed;
```
- EN: This block iterates over collections or graph structures; produces a result or forwards a computed value; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `blocks`, `FuseAddMM`.
- CN: 该代码块遍历集合或图结构；返回结果或转发已计算的值；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`blocks`, `FuseAddMM`。

### Lines 429-442
```cpp
}

// FuseAddMM is a separate pass from peephole optimize because it is currently
// used for exporting to ONNX.
// Today, fusing add + MM has no benefit within PyTorch running ATen
// ops. However, we rely on seeing the fused version of AddMM for ONNX export,
// since otherwise after ONNX translation we would see redundant Gemm ops with
// sub-optimal inputs.
// It won't be helpful for ATen until we're able to represent
//   torch.addmm(a, b, c, out=a).
// That's because addmm dispatches internally to gemm, which computes:
//   C = beta * C + alpha * A @ B
// but aten::addmm(a, b, c, 1, 1) is really:
//   D = beta * C + alpha * A @ B
```
- EN: This block performs optimization-oriented rewriting. Key symbols: no dominant local symbol names.
- CN: 该代码块执行面向优化的改写。关键符号：无明显局部符号。

### Lines 443-450
```cpp
// and because it works out of place on C, we're only trading off an
// explicit add for a copy inside the addmm function. Note that it
// doesn't even result in fewer reads, because mm won't even load C
// (because beta == 0 for it).
bool FuseAddMM(const std::shared_ptr<Graph>& graph) {
  bool changed = FuseAddMM(graph->block());
  GRAPH_DUMP("After FuseAddMM: ", graph);
  return changed;
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `FuseAddMM`, `block`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`FuseAddMM`, `block`。

### Lines 451-457
```cpp
}

bool PeepholeOptimize(
    const std::shared_ptr<Graph>& graph,
    bool addmm_fusion_enabled) {
  PeepholeOptimizeImpl peephole(graph, addmm_fusion_enabled);
  bool changed = peephole.run();
```
- EN: This block manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `PeepholeOptimize`, `peephole`, `run`.
- CN: 该代码块操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`PeepholeOptimize`, `peephole`, `run`。

### Lines 458-465
```cpp
  GRAPH_DUMP("After PeepholeOptimize: ", graph);
  // Eliminate dead code created by any peephole passes we've just done
  if (changed) {
    EliminateDeadCode(graph->block());
  }
  return changed;
}

```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting; protects shared state or ordering assumptions. Key symbols: `EliminateDeadCode`, `block`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写；保护共享状态或执行顺序假设。关键符号：`EliminateDeadCode`, `block`。

### Lines 466-466
```cpp
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
- EN: Export interop — the implementation helps bridge TorchScript graphs to ONNX-style representations.
  CN: 导出互操作——实现帮助 TorchScript 图与 ONNX 风格表示之间建立桥接。

## Dependencies / 依赖关系
- Internal includes / 内部头文件: `torch/csrc/jit/passes/peephole.h`, `ATen/core/jit_type.h`, `c10/util/irange.h`, `torch/csrc/jit/jit_log.h`, `torch/csrc/jit/passes/concat_opt.h`, `torch/csrc/jit/passes/dead_code_elimination.h`, `torch/csrc/jit/passes/peephole_alias_sensitive.h`, `torch/csrc/jit/passes/peephole_dict_idioms.h`, `torch/csrc/jit/passes/peephole_list_idioms.h`, `torch/csrc/jit/passes/peephole_non_tensor.h`
- External includes / 外部头文件: 无
- Namespaces / 命名空间: `torch::jit`
- Representative symbols / 代表性符号: `mustBeEqual`, `has_value`, `PeepholeOptimizeImpl`, `graph_`, `move`, `shape_peepholes_`, `run`, `optimizeBlock`, `block`, `PeepholeOptimizeListIdioms`, `...`
