# peephole_non_tensor.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `torch/csrc/jit/passes/peephole_non_tensor.cpp`
- Repository: `pytorch`
- Purpose (EN): Implements the TorchScript JIT pass logic for peephole non tensor, including graph analysis and rewrites.
- 用途 (CN): 实现与 peephole non tensor 相关的 TorchScript JIT pass 逻辑，包括图分析与改写。

## Line-by-Line Analysis / 逐行分析
### Lines 1-7
```cpp
#include <torch/csrc/jit/passes/peephole_non_tensor.h>

#include <ATen/core/jit_type.h>
#include <c10/util/irange.h>
#include <torch/csrc/jit/ir/ir_views.h>
#include <torch/csrc/jit/jit_log.h>

```
- EN: Pulls in the headers needed by the peephole non tensor logic. Internal dependencies: `torch/csrc/jit/passes/peephole_non_tensor.h`, `ATen/core/jit_type.h`, `c10/util/irange.h`, `torch/csrc/jit/ir/ir_views.h`, `torch/csrc/jit/jit_log.h`; external dependencies: none.
- CN: 为 peephole non tensor 相关逻辑引入所需头文件。内部依赖：`torch/csrc/jit/passes/peephole_non_tensor.h`, `ATen/core/jit_type.h`, `c10/util/irange.h`, `torch/csrc/jit/ir/ir_views.h`, `torch/csrc/jit/jit_log.h`；外部依赖：无。

### Lines 8-20
```cpp
namespace torch::jit {

namespace {

/**
 * Check whether the arithmetic node is binary between integers, and return a
 * constant int value if there exists one.
 *
 * @pre node is integer arithmetic.
 * @post if there's one constant in two operands, then the second operand is
 *       constant.
 */
std::optional<int64_t> checkArithNode(Node& node) {
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `checkArithNode`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`checkArithNode`。

### Lines 21-27
```cpp
  if (node.inputs().size() != 2 || node.input(0)->type() != IntType::get() ||
      node.input(1)->type() != IntType::get()) {
    return {};
  }

  if (node.kind() == aten::mul || node.kind() == aten::add) {
    if (auto i = constant_as<int64_t>(node.input(0))) {
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `inputs`, `size`, `input`, `type`, `get`, `kind`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`inputs`, `size`, `input`, `type`, `get`, `kind`。

### Lines 28-35
```cpp
      node.permuteInputs({1, 0});
      return i;
    }
  }

  return constant_as<int64_t>(node.input(1));
}

```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `permuteInputs`, `input`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`permuteInputs`, `input`。

### Lines 36-42
```cpp
/**
 * Remove a mul/floordiv node if it is multiplication or division by 1.
 *
 * @pre node is either aten::mul, aten::floordiv or aten::div
 */
bool trySimplifyMulOrDiv(Node& node) {
  auto constant = checkArithNode(node);
```
- EN: This block manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `trySimplifyMulOrDiv`, `checkArithNode`.
- CN: 该代码块操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`trySimplifyMulOrDiv`, `checkArithNode`。

### Lines 43-50
```cpp
  if (!constant || *constant != 1) {
    return false;
  }

  node.output()->replaceAllUsesWith(node.inputs()[0]);
  return true;
}

```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `output`, `replaceAllUsesWith`, `inputs`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`output`, `replaceAllUsesWith`, `inputs`。

### Lines 51-57
```cpp
/**
 * Simplify an add/sub node with its input node, i.e. merge the constant parts
 * together.
 *
 * @pre node is either aten::add or aten::sub
 */
bool trySimplifyAddOrSub(Node& node) {
```
- EN: This block manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `trySimplifyAddOrSub`.
- CN: 该代码块操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`trySimplifyAddOrSub`。

### Lines 58-65
```cpp
  auto constant = checkArithNode(node);
  if (!constant) {
    return false;
  }

  if (constant == 0) {
    node.output()->replaceAllUsesWith(node.input(0));
    return true;
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `checkArithNode`, `output`, `replaceAllUsesWith`, `input`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`checkArithNode`, `output`, `replaceAllUsesWith`, `input`。

### Lines 66-72
```cpp
  }

  auto& dep = *node.inputs()[0]->node();
  if (dep.kind() != aten::add && dep.kind() != aten::sub) {
    return false;
  }

```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `inputs`, `node`, `kind`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`inputs`, `node`, `kind`。

### Lines 73-79
```cpp
  auto delta = checkArithNode(dep);
  if (!delta) {
    return false;
  }
  auto merged =
      dep.kind() == node.kind() ? *constant + *delta : *constant - *delta;

```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `checkArithNode`, `kind`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`checkArithNode`, `kind`。

### Lines 80-87
```cpp
  if (merged == 0) {
    node.output()->replaceAllUsesWith(dep.inputs()[0]);
  } else {
    WithInsertPoint g(&node);
    node.replaceInput(0, dep.inputs()[0]);
    node.replaceInput(1, node.owningGraph()->insertConstant(merged));
  }
  return true;
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `output`, `replaceAllUsesWith`, `inputs`, `g`, `replaceInput`, `owningGraph`, `...`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`output`, `replaceAllUsesWith`, `inputs`, `g`, `replaceInput`, `owningGraph`, `...`。

### Lines 88-95
```cpp
}

} // namespace

struct PeepholeOptimizeNonTensorImpl {
  PeepholeOptimizeNonTensorImpl(std::shared_ptr<Graph> graph)
      : graph_(std::move(graph)) {}

```
- EN: Declares core types or data containers for this file. Prominent symbols: `PeepholeOptimizeNonTensorImpl`, `graph_`, `move`.
- CN: 声明该文件使用的核心类型或数据容器。主要符号：`PeepholeOptimizeNonTensorImpl`, `graph_`, `move`。

### Lines 96-102
```cpp
  bool run() {
    return optimizeBlock(graph_->block());
  }

  bool optimizeBlock(Block* block) {
    bool changed = false;
    for (auto it = block->nodes().begin(); it != block->nodes().end(); ++it) {
```
- EN: This block iterates over collections or graph structures; produces a result or forwards a computed value; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `run`, `optimizeBlock`, `block`, `nodes`, `begin`, `end`.
- CN: 该代码块遍历集合或图结构；返回结果或转发已计算的值；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`run`, `optimizeBlock`, `block`, `nodes`, `begin`, `end`。

### Lines 103-109
```cpp
      auto* node = *it;

      for (Block* sub_block : node->blocks()) {
        changed |= optimizeBlock(sub_block);
      }

      if (node->kind() != prim::Constant) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects; performs optimization-oriented rewriting; protects shared state or ordering assumptions. Key symbols: `blocks`, `optimizeBlock`, `kind`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象；执行面向优化的改写；保护共享状态或执行顺序假设。关键符号：`blocks`, `optimizeBlock`, `kind`。

### Lines 110-123
```cpp
        WithInsertPoint guard(node);
        // Any Value whose type is None should be replaced with a Constant
        // This can occur if a module has an optional attribute, and it is
        // initialized as None.
        for (Value* output : node->outputs()) {
          if (output->type()->cast<NoneType>()) {
            output->replaceAllUsesWith(graph_->insertConstant(IValue()));
            changed = true;
          }
        }
      }
      // XXX: remember that if you want to simplify an expression by combining
      // multiple nodes into a different one, then you need to check that they
      // all belong to the given block
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects; performs optimization-oriented rewriting; protects shared state or ordering assumptions. Key symbols: `guard`, `outputs`, `type`, `replaceAllUsesWith`, `insertConstant`, `IValue`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象；执行面向优化的改写；保护共享状态或执行顺序假设。关键符号：`guard`, `outputs`, `type`, `replaceAllUsesWith`, `insertConstant`, `IValue`。

### Lines 124-130
```cpp
      // TODO: this doesn't work with Scalar-Tensor ops! We should
      // canonicalize those
      if (node->kind() == prim::If) {
        IfView n(node);
        // this handles redundant short circuits like "x and True" or "x or
        // False"
        for (const auto i : c10::irange(n.outputs().size())) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `kind`, `n`, `irange`, `outputs`, `size`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`kind`, `n`, `irange`, `outputs`, `size`。

### Lines 131-140
```cpp
          if (n.outputs().at(i)->type() != BoolType::get()) {
            continue;
          }
          bool true_val =
              constant_as<bool>(n.thenOutputs().at(i)).value_or(false);
          bool false_val =
              constant_as<bool>(n.elseOutputs().at(i)).value_or(true);
          // if an if node's output equals its condition replace output with
          // condition
          if (true_val && !false_val) {
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `outputs`, `type`, `get`, `thenOutputs`, `value_or`, `elseOutputs`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`outputs`, `type`, `get`, `thenOutputs`, `value_or`, `elseOutputs`。

### Lines 141-150
```cpp
            GRAPH_UPDATE(
                "Replacing ",
                n.outputs().at(i)->debugName(),
                " (True or False) with ",
                n.cond()->debugName());
            n.outputs().at(i)->replaceAllUsesWith(n.cond());
            changed = true;
          }
        }

```
- EN: This block manipulates TorchScript IR objects. Key symbols: `outputs`, `debugName`, `cond`, `replaceAllUsesWith`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`outputs`, `debugName`, `cond`, `replaceAllUsesWith`。

### Lines 151-157
```cpp
        // check for types that can be refined
        for (size_t i = 0; i < n.outputs().size(); ++i) {
          // common case of optional for now
          bool inputs_non_optional =
              !n.thenOutputs().at(i)->type()->cast<OptionalType>() &&
              !n.elseOutputs().at(i)->type()->cast<OptionalType>();
          auto output_optional =
```
- EN: This block iterates over collections or graph structures. Key symbols: `outputs`, `size`, `thenOutputs`, `type`, `elseOutputs`.
- CN: 该代码块遍历集合或图结构。关键符号：`outputs`, `size`, `thenOutputs`, `type`, `elseOutputs`。

### Lines 158-171
```cpp
              n.outputs().at(i)->type()->cast<OptionalType>();
          if (inputs_non_optional && output_optional) {
            if (auto unif = unifyTypes(
                    n.thenOutputs().at(i)->type(),
                    n.elseOutputs().at(i)->type())) {
              n.outputs().at(i)->setType(*unif);
              changed = true;
            }
          }
        }
      } else if (
          node->kind() == aten::__is__ || node->kind() == aten::__isnot__) {
        // if we are comparing a None value with a value that can't be None
        // replace the output with true if node is __isnot__ or false if node is
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `outputs`, `type`, `unifyTypes`, `thenOutputs`, `elseOutputs`, `setType`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`outputs`, `type`, `unifyTypes`, `thenOutputs`, `elseOutputs`, `setType`, `...`。

### Lines 172-179
```cpp
        // __is__
        AT_ASSERT(node->inputs().size() == 2);
        for (size_t check_none_index : {0, 1}) {
          bool input_must_be_none =
              node->inputs().at(check_none_index)->mustBeNone();
          bool other_must_not_be_none =
              node->inputs().at(1 - check_none_index)->mustNotBeNone();
          if (input_must_be_none && other_must_not_be_none) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `inputs`, `size`, `mustBeNone`, `mustNotBeNone`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`inputs`, `size`, `mustBeNone`, `mustNotBeNone`。

### Lines 180-193
```cpp
            WithInsertPoint guard(node);
            auto output = node->owningGraph()->insertConstant(
                node->kind() == aten::__isnot__);
            GRAPH_UPDATE(
                "Folding ", getHeader(node), " to ", output->debugName());
            node->output()->replaceAllUsesWith(output);
            changed = true;
          }
        }
      } else if (
          node->kind() == prim::unchecked_unwrap_optional ||
          node->kind() == aten::_unwrap_optional) {
        // we are unwrapping an input that can't be None, remove the unwrap
        auto input = node->input();
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; performs optimization-oriented rewriting; protects shared state or ordering assumptions. Key symbols: `guard`, `owningGraph`, `insertConstant`, `kind`, `getHeader`, `debugName`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；执行面向优化的改写；保护共享状态或执行顺序假设。关键符号：`guard`, `owningGraph`, `insertConstant`, `kind`, `getHeader`, `debugName`, `...`。

### Lines 194-207
```cpp
        if (input->mustNotBeNone()) {
          GRAPH_UPDATE(
              "Unwrapping ",
              getHeader(node),
              " as ",
              node->input(),
              " can't be optional");
          node->output()->replaceAllUsesWith(node->input());
          changed = true;
        }
      } else if (node->kind() == prim::unchecked_cast) {
        // unchecked_cast is not generated for tensor properties, so we are not
        // losing anything by calling unshapedType here
        auto input_type = unshapedType(node->input()->type());
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `mustNotBeNone`, `getHeader`, `input`, `output`, `replaceAllUsesWith`, `kind`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`mustNotBeNone`, `getHeader`, `input`, `output`, `replaceAllUsesWith`, `kind`, `...`。

### Lines 208-221
```cpp
        auto output_type = unshapedType(node->output()->type());
        if (input_type->isSubtypeOf(*output_type)) {
          GRAPH_UPDATE(
              "Removing ",
              getHeader(node),
              " as input type subtypes output type");
          node->output()->replaceAllUsesWith(node->input());
          changed = true;
        }
      } else if (
          (node->kind() == aten::Int || node->kind() == aten::ceil) &&
          node->inputs().size() == 1 &&
          node->input()->type()->cast<IntType>()) {
        GRAPH_UPDATE(
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `unshapedType`, `output`, `type`, `isSubtypeOf`, `getHeader`, `replaceAllUsesWith`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`unshapedType`, `output`, `type`, `isSubtypeOf`, `getHeader`, `replaceAllUsesWith`, `...`。

### Lines 222-230
```cpp
            "Removing ", getHeader(node), " as input is already an integer");
        node->output()->replaceAllUsesWith(node->input());
        changed = true;
      } else if (node->kind() == aten::ne || node->kind() == aten::eq) {
        if (node->inputs().size() != 2 ||
            node->inputs().at(0) != node->inputs().at(1)) {
          continue;
        }
        auto inp_type = node->inputs().at(0)->type();
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `getHeader`, `output`, `replaceAllUsesWith`, `input`, `kind`, `inputs`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`getHeader`, `output`, `replaceAllUsesWith`, `input`, `kind`, `inputs`, `...`。

### Lines 231-240
```cpp
        // only handling common immutable types here because other types like
        // Tensor or list of Tensor might throw on aten::eq
        auto immut_type = [&](const TypePtr& type) {
          auto kind = type->kind();
          static const std::vector<TypeKind> handled_immutable_types = {
              TypeKind::BoolType,
              TypeKind::IntType,
              TypeKind::FloatType,
              TypeKind::NoneType};
          return (
```
- EN: This block produces a result or forwards a computed value. Key symbols: `kind`.
- CN: 该代码块返回结果或转发已计算的值。关键符号：`kind`。

### Lines 241-247
```cpp
              std::find(
                  handled_immutable_types.begin(),
                  handled_immutable_types.end(),
                  kind) != handled_immutable_types.end());
        };
        bool non_throwing_type = false;
        if (auto li_type = inp_type->cast<ListType>()) {
```
- EN: This block handles conditional branches. Key symbols: `find`, `begin`, `end`.
- CN: 该代码块处理条件分支。关键符号：`find`, `begin`, `end`。

### Lines 248-256
```cpp
          non_throwing_type = immut_type(li_type->getElementType());
        } else if (auto di_type = inp_type->cast<DictType>()) {
          non_throwing_type =
              (immut_type(di_type->getKeyType()) &&
               immut_type(di_type->getValueType()));
        } else {
          non_throwing_type = immut_type(inp_type);
        }
        if (non_throwing_type) {
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `immut_type`, `getElementType`, `getKeyType`, `getValueType`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`immut_type`, `getElementType`, `getKeyType`, `getValueType`。

### Lines 257-270
```cpp
          WithInsertPoint guard(node);
          node->output()->replaceAllUsesWith(
              graph_->insertConstant(node->kind() == aten::eq));
          changed = true;
        }
      } else if (
          node->kind() == aten::mul || node->kind() == aten::floordiv ||
          node->kind() == aten::div) {
        changed |= trySimplifyMulOrDiv(*node);
      } else if (node->kind() == aten::add || node->kind() == aten::sub) {
        changed |= trySimplifyAddOrSub(*node);
      }
    }
    return changed;
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting; protects shared state or ordering assumptions. Key symbols: `guard`, `output`, `replaceAllUsesWith`, `insertConstant`, `kind`, `trySimplifyMulOrDiv`, `...`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写；保护共享状态或执行顺序假设。关键符号：`guard`, `output`, `replaceAllUsesWith`, `insertConstant`, `kind`, `trySimplifyMulOrDiv`, `...`。

### Lines 271-277
```cpp
  }

 private:
  std::shared_ptr<Graph> graph_;
};

bool PeepholeOptimizeNonTensor(const std::shared_ptr<Graph>& graph) {
```
- EN: This block manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `PeepholeOptimizeNonTensor`.
- CN: 该代码块操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`PeepholeOptimizeNonTensor`。

### Lines 278-284
```cpp
  PeepholeOptimizeNonTensorImpl peephole(graph);
  bool changed = peephole.run();
  GRAPH_DUMP("After PeepholeOptimize: ", graph);
  return changed;
}

} // namespace torch::jit
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `peephole`, `run`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`peephole`, `run`。


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
- Internal includes / 内部头文件: `torch/csrc/jit/passes/peephole_non_tensor.h`, `ATen/core/jit_type.h`, `c10/util/irange.h`, `torch/csrc/jit/ir/ir_views.h`, `torch/csrc/jit/jit_log.h`
- External includes / 外部头文件: 无
- Namespaces / 命名空间: `torch::jit`
- Representative symbols / 代表性符号: `checkArithNode`, `inputs`, `size`, `input`, `type`, `get`, `kind`, `permuteInputs`, `trySimplifyMulOrDiv`, `output`, `...`
