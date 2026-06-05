# prepare_binary.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/csrc/jit/codegen/onednn/prepare_binary.cpp`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Contains code-generation logic or generated declarations that synthesize JIT-related bindings, schemas, or helper code. This specific file centers on `prepare_binary.cpp`. Generation of declarations, registrations, or helper code is part of the responsibility.
- **Purpose (CN)**: 包含代码生成逻辑或生成后的声明，用于产出 JIT 相关绑定、模式或辅助代码。 该文件具体围绕 `prepare_binary.cpp` 展开。 其职责包含声明、注册或辅助代码的生成。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20 / 第 1-20 行

```cpp
#include <aten/src/ATen/core/jit_type.h>
#include <torch/csrc/jit/codegen/onednn/prepare_binary.h>
#include <torch/csrc/jit/passes/dead_code_elimination.h>
#include <torch/csrc/jit/passes/shape_analysis.h>

namespace torch::jit::fuser::onednn {

static bool compareConstValue(Value* v, double d) {
  auto ival = toIValue(v);
  return ival.has_value() &&
      ((ival->isInt() && static_cast<int>(ival->toInt()) == d) ||
       (ival->isDouble() && ival->toDouble() == d));
}

static void handleBinaryOpInputs(Node* node) {
  // We do not handle binary ops with two scalar inputs,
  // and we assume scalar is always at the second place.
  if (node->input(0)->type()->isSubtypeOf(TensorType::get())) {
    auto dtypeOfFirstInput =
        node->input(0)->type()->cast<TensorType>()->scalarType().value();
```

- **EN:** It enters or references namespace scopes such as torch::jit::fuser::onednn, which positions the code inside the JIT subsystem hierarchy.
- **CN:** 这里进入或引用了 torch::jit::fuser::onednn 等命名空间作用域，用于把代码放入 JIT 子系统层级中。
- **EN:** Important callable entry points in this range include compareConstValue, handleBinaryOpInputs.
- **CN:** 这一段的重要可调用入口包括 compareConstValue, handleBinaryOpInputs。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。

### Lines 21-40 / 第 21-40 行

```cpp
    if (node->input(1)->type()->isSubtypeOf(FloatType::get()) ||
        node->input(1)->type()->isSubtypeOf(IntType::get())) {
      // If a scalar is added to be a tensor, we would assume that the
      // scalar is of the same dtype as the tensor, as oneDNN graph
      // currently requires inputs of binary ops to have the same dtype.
      // We create a 1D tensor from the scalar input & "promote" its
      // dtype to that of the first input. Doing so helps us satisfy PyTorch's
      // type promotion rules.
      // Although we convert the scalar to a tensor, we still need to promote
      // types, as if the second input were still a scalar.
      // The following sample code-snippet illustrates that converting a scalar
      // input to a 1-D tensor may result in a different output dtype than would
      // otherwise have been the case.
      // clang-format off
      //   >>> (1. + torch.rand([2]).half()).dtype
      //       torch.float16
      //   >>> (torch.tensor(1.).unsqueeze(0) + (torch.rand([2]).half())).dtype
      //       torch.float32
      // clang-format on
      auto promotedDtype = dtypeOfFirstInput;
```

- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Comments in this range document the intent of the API or the invariants expected by later code.
- **CN:** 这一段中的注释说明了 API 意图，或描述了后续代码依赖的不变式。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。
- **EN:** Concepts touched here: Graph IR / 图中间表示, Nodes and values / 节点与值, Type system / 类型系统, Branching logic / 分支逻辑.
- **CN:** 这里涉及的概念包括：Graph IR / 图中间表示, Nodes and values / 节点与值, Type system / 类型系统, Branching logic / 分支逻辑。

### Lines 41-60 / 第 41-60 行

```cpp
      auto scalar = node->input(1);
      WithInsertPoint guard(node);
      auto g = node->owningGraph();
      // 42 : Scalar  -->  tensor(42.0) : Float([])
      auto t = g->insert(aten::as_tensor, {scalar}, {{"dtype", promotedDtype}});
      // add dim & stride info to IR
      std::optional<size_t> t_dim = 1;
      auto target_type = TensorTypePtr(
          TensorType::create(promotedDtype, at::kCPU, t_dim, false));
      target_type = target_type->withSizes({1});
      t->setType(target_type);

      // tensor(42.0) : Float([])  -->  tensor([42.0]) : Float([1])
      auto unsqueezed = g->insert(aten::unsqueeze, {t, 0});
      unsqueezed->setType(target_type);
      node->replaceInput(1, unsqueezed);

      // dtype might have changed, so needs to be updated in IR as well
      node->output()->setType(
          node->output()->type()->expect<TensorType>()->withScalarType(
```

- **EN:** Important callable entry points in this range include guard, create.
- **CN:** 这一段的重要可调用入口包括 guard, create。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Concepts touched here: Graph IR / 图中间表示, Nodes and values / 节点与值, Type system / 类型系统, Shape/resource guard / 形状或资源保护.
- **CN:** 这里涉及的概念包括：Graph IR / 图中间表示, Nodes and values / 节点与值, Type system / 类型系统, Shape/resource guard / 形状或资源保护。

### Lines 61-80 / 第 61-80 行

```cpp
              promotedDtype));
    } else if (node->input(1)->type()->isSubtypeOf(TensorType::get())) {
      // Here, both inputs are tensors, and we just wanna make sure that they
      // are the same dtype, as oneDNN Graph requires both inputs to have the
      // same dtype. We'll follow PyTorch's type-promotion rules here.
      auto second_input_typeptr = node->input(1)->type()->expect<TensorType>();
      std::optional<at::ScalarType> second_input_type =
          second_input_typeptr->scalarType();
      if (second_input_type != std::nullopt) {
        // dtype of the second tensor might not be available in the IR
        auto dtypeOfSecondInput = second_input_type.value();
        if (dtypeOfFirstInput != dtypeOfSecondInput) {
          // Type promotion is required
          auto promotedDtype =
              c10::promoteTypes(dtypeOfFirstInput, dtypeOfSecondInput);
          WithInsertPoint guard(node);
          auto g = node->owningGraph();
          if (promotedDtype == dtypeOfFirstInput) {
            auto to_node_output = g->insert(
                aten::to, {node->input(1)}, {{"dtype", promotedDtype}});
```

- **EN:** Important callable entry points in this range include promoteTypes, guard.
- **CN:** 这一段的重要可调用入口包括 promoteTypes, guard。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。
- **EN:** Concepts touched here: Graph IR / 图中间表示, Nodes and values / 节点与值, Type system / 类型系统, Shape/resource guard / 形状或资源保护, Branching logic / 分支逻辑.
- **CN:** 这里涉及的概念包括：Graph IR / 图中间表示, Nodes and values / 节点与值, Type system / 类型系统, Shape/resource guard / 形状或资源保护, Branching logic / 分支逻辑。

### Lines 81-100 / 第 81-100 行

```cpp
            to_node_output->setType(
                node->input(1)->type()->expect<TensorType>()->withScalarType(
                    promotedDtype));
            node->replaceInput(1, to_node_output);
          } else {
            auto to_node_output = g->insert(
                aten::to, {node->input(0)}, {{"dtype", promotedDtype}});
            to_node_output->setType(
                node->input(0)->type()->expect<TensorType>()->withScalarType(
                    promotedDtype));
            node->replaceInput(0, to_node_output);
          }
          // dtype might have changed, so needs to be updated in IR as well
          node->output()->setType(
              node->output()->type()->expect<TensorType>()->withScalarType(
                  promotedDtype));
        } else {
          // both dtypes are same
          // IR info of dtypes is missing sometimes in JIT IR,
          // and we shouldn't treat those tensors as FP32 tensors by default.
```

- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Comments in this range document the intent of the API or the invariants expected by later code.
- **CN:** 这一段中的注释说明了 API 意图，或描述了后续代码依赖的不变式。
- **EN:** Concepts touched here: Nodes and values / 节点与值, Type system / 类型系统.
- **CN:** 这里涉及的概念包括：Nodes and values / 节点与值, Type system / 类型系统。

### Lines 101-120 / 第 101-120 行

```cpp
          node->output()->setType(
              node->output()->type()->expect<TensorType>()->withScalarType(
                  dtypeOfFirstInput));
        }
      } // end inner if block
    } // end outer if block
  }
}

static void ConvertScalarToTensor(Block* block) {
  for (auto node : block->nodes()) {
    for (auto sub : node->blocks()) {
      ConvertScalarToTensor(sub);
    }

    if (node->kind() == aten::add || node->kind() == aten::mul ||
        node->kind() == aten::div) {
      handleBinaryOpInputs(node);
    }
  }
```

- **EN:** Important callable entry points in this range include ConvertScalarToTensor, handleBinaryOpInputs.
- **CN:** 这一段的重要可调用入口包括 ConvertScalarToTensor, handleBinaryOpInputs。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。
- **EN:** Loop constructs iterate over IR elements, schema entries, or serialized records in this range.
- **CN:** 这一段中的循环用于遍历 IR 元素、模式条目或序列化记录。

### Lines 121-140 / 第 121-140 行

```cpp
}

static void mayDecomposeAdd(Node* node) {
  if (node->inputs().size() < 3) {
    return; // corner-case in BERT-mrpc that's not in line with
            // native_functions.yaml
  }
  if (toIValue(node->namedInput("alpha")).has_value()) {
    auto alphaEqualsOne = compareConstValue(node->namedInput("alpha"), 1.0);
    if (!alphaEqualsOne) {
      WithInsertPoint guard(node);
      auto g = node->owningGraph();
      auto mul = g->insert(
          aten::mul, {node->namedInput("other"), node->namedInput("alpha")});
      if (node->namedInput("other")->type()->isSubtypeOf(TensorType::get())) {
        auto mulTensorTypePtr = node->namedInput("other")->type();
        mul->setType(mulTensorTypePtr);
      }
      node->replaceInput(1, mul);
      auto one = g->insertConstant(1.0);
```

- **EN:** Important callable entry points in this range include mayDecomposeAdd, guard.
- **CN:** 这一段的重要可调用入口包括 mayDecomposeAdd, guard。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。
- **EN:** Concepts touched here: Graph IR / 图中间表示, Nodes and values / 节点与值, Type system / 类型系统, Operator schema / 算子模式, Shape/resource guard / 形状或资源保护, Branching logic / 分支逻辑.
- **CN:** 这里涉及的概念包括：Graph IR / 图中间表示, Nodes and values / 节点与值, Type system / 类型系统, Operator schema / 算子模式, Shape/resource guard / 形状或资源保护, Branching logic / 分支逻辑。

### Lines 141-160 / 第 141-160 行

```cpp
      node->replaceInput(2, one);
    }
  }
}

static void DecomposeFusedAdd(Block* block) {
  for (auto node : block->nodes()) {
    for (auto sub : node->blocks()) {
      DecomposeFusedAdd(sub);
    }

    if (node->kind() == aten::add) {
      mayDecomposeAdd(node);
    }
  }
}

static void EliminateIdentityMulAdd(Block* block) {
  for (auto node : block->nodes()) {
    for (auto sub : node->blocks()) {
```

- **EN:** Important callable entry points in this range include DecomposeFusedAdd, mayDecomposeAdd, EliminateIdentityMulAdd.
- **CN:** 这一段的重要可调用入口包括 DecomposeFusedAdd, mayDecomposeAdd, EliminateIdentityMulAdd。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。
- **EN:** Loop constructs iterate over IR elements, schema entries, or serialized records in this range.
- **CN:** 这一段中的循环用于遍历 IR 元素、模式条目或序列化记录。

### Lines 161-179 / 第 161-179 行

```cpp
      EliminateIdentityMulAdd(sub);
    }

    if ((node->kind() == aten::add && compareConstValue(node->input(1), 0.0)) ||
        (node->kind() == aten::mul && compareConstValue(node->input(1), 1.0))) {
      node->output()->replaceAllUsesWith(node->namedInput("self"));
    }
  }
}

void PrepareBinaryForLLGA(const std::shared_ptr<Graph>& graph) {
  DecomposeFusedAdd(graph->block());
  EliminateIdentityMulAdd(graph->block());
  EliminateDeadCode(graph);
  // ConvertScalarToTensor must be placed after EliminateIdentityMulAdd
  ConvertScalarToTensor(graph->block());
}

} // namespace torch::jit::fuser::onednn
```

- **EN:** Important callable entry points in this range include EliminateIdentityMulAdd, PrepareBinaryForLLGA, DecomposeFusedAdd, EliminateDeadCode, ConvertScalarToTensor.
- **CN:** 这一段的重要可调用入口包括 EliminateIdentityMulAdd, PrepareBinaryForLLGA, DecomposeFusedAdd, EliminateDeadCode, ConvertScalarToTensor。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。
- **EN:** Concepts touched here: Graph IR / 图中间表示, Nodes and values / 节点与值, Control-flow blocks / 控制流块, Declared symbols / 声明的符号, Namespace scoping / 命名空间作用域, Branching logic / 分支逻辑.
- **CN:** 这里涉及的概念包括：Graph IR / 图中间表示, Nodes and values / 节点与值, Control-flow blocks / 控制流块, Declared symbols / 声明的符号, Namespace scoping / 命名空间作用域, Branching logic / 分支逻辑。

## Key Concepts / 关键概念

- **Generated code pipeline** — 代码生成流水线
- **Graph IR** — 图中间表示
- **Nodes and values** — 节点与值
- **Control-flow blocks** — 控制流块
- **Type system** — 类型系统
- **Operator schema** — 算子模式
- **Code generation** — 代码生成
- **Optimization pass** — 优化 pass
- **Shape/resource guard** — 形状或资源保护
- **Core symbols: compareConstValue, handleBinaryOpInputs, guard, create, promoteTypes, ConvertScalarToTensor, mayDecomposeAdd, DecomposeFusedAdd** — 核心符号：compareConstValue、handleBinaryOpInputs、guard、create、promoteTypes、ConvertScalarToTensor、mayDecomposeAdd、DecomposeFusedAdd

## Dependencies / 依赖关系

- `torch/csrc/jit/codegen/onednn/prepare_binary.h`
- `torch/csrc/jit/passes/dead_code_elimination.h`
- `torch/csrc/jit/passes/shape_analysis.h`
