# graph_opt.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/csrc/jit/tensorexpr/graph_opt.cpp`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Defines or manipulates expression nodes in the Tensor Expression subsystem.
- **Purpose (CN)**: 定义或操作 Tensor Expression 子系统中的表达式节点。
## Line-by-Line Analysis / 逐行分析

### Lines 1-16
```cpp
#include <torch/csrc/jit/tensorexpr/graph_opt.h>

#include <torch/csrc/jit/jit_log.h>
#include <torch/csrc/jit/passes/dead_code_elimination.h>
#include <torch/csrc/jit/runtime/symbolic_shape_registry_util.h>
#include <torch/csrc/jit/tensorexpr/kernel.h>

namespace torch::jit::tensorexpr {

// Move the given user of `aten::cat` op to its inputs.
static Node* moveCatAfterUse(
    Node* cat,
    Node* user,
    const std::shared_ptr<Graph>& subgraph) {
  // Example IR:
  //   %1 = ...
```
- **EN**: This block assembles the compilation dependencies, pulling in local JIT headers such as torch/csrc/jit/tensorexpr/graph_opt.h, torch/csrc/jit/jit_log.h, torch/csrc/jit/passes/dead_code_elimination.h, and 2 more. The namespace declarations place the code inside torch::jit::tensorexpr, matching the surrounding JIT subsystem. This chunk defines `moveCatAfterUse`, which implements a focused step in Tensor Expression analysis, lowering, or code generation.
- **CN**: 这一段组织编译依赖，引入了本地 JIT 头文件，如 torch/csrc/jit/tensorexpr/graph_opt.h、torch/csrc/jit/jit_log.h、torch/csrc/jit/passes/dead_code_elimination.h 等共 5 项。 命名空间声明把代码放入 torch::jit::tensorexpr 中，与周边 JIT 子系统保持一致。 这一段定义了 `moveCatAfterUse`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。

### Lines 17-30
```cpp
  //   %2 = ...
  //   %3 = prim::ListConstruct(%1, %2)
  //   %4 = aten::cat(%3, ...)
  //   %5 = aten::relu(%4)
  //   return (%5)
  //
  // To be transformed to:
  //   %1 = ...
  //   %2 = ...
  //   %5.1 = aten::relu(%1)
  //   %5.2 = aten::relu(%2)
  //   %3 = prim::ListConstruct(%5.1, %5.2)
  //   %4 = aten::cat(%3, ...)
  //   return (%4)
```
- **EN**: This comment block documents assumptions, constraints, or generated-code provenance that shape the implementation below.
- **CN**: 这一段注释说明了后续实现依赖的假设、约束或生成来源。

### Lines 32-42
```cpp
  TORCH_INTERNAL_ASSERT(
      cat->output()->hasUses(),
      buildErrorMessage("aten::cat output is not used."));
  TORCH_INTERNAL_ASSERT(
      cat->output()->uses().size() == 1,
      buildErrorMessage("aten::cat output is used in multiple places."));
  TORCH_INTERNAL_ASSERT(
      cat->input(0)->node()->kind() == prim::ListConstruct,
      buildErrorMessage("aten::cat inputs are not expected."));
  auto cat_list = cat->input(0)->node();
  auto cat_inputs = cat_list->inputs();
```
- **EN**: This chunk continues `moveCatAfterUse` and expands its internal control flow or data movement. Assertions and error paths make invalid states fail early instead of silently corrupting later stages.
- **CN**: 这一段延续了 `moveCatAfterUse`，进一步展开其内部控制流或数据流转。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。

### Lines 44-59
```cpp
  auto user_tensor_type = user->output()->type()->cast<c10::TensorType>();
  TORCH_INTERNAL_ASSERT(
      user_tensor_type, buildErrorMessage("Unexpected user tensor type"));
  std::unordered_map<Value*, Value*> new_cat_inputs;
  for (auto inp : cat_inputs) {
    auto new_cat_input = subgraph->createClone(
        user, [&](Value* k) { return (k == cat->output()) ? inp : k; });
    // Since we are cloning user, its result should be the same scalar type
    // as the user. But the dims should correspond to that of the input.
    auto input_tensor_type = inp->type()->cast<c10::TensorType>();
    TORCH_INTERNAL_ASSERT(
        input_tensor_type, buildErrorMessage("Unexpected input tensor type"));
    auto new_input_type =
        input_tensor_type->withScalarType(user_tensor_type->scalarType());
    new_cat_input->output()->setType(new_input_type);
    new_cat_input->insertBefore(cat_list);
```
- **EN**: This chunk continues `moveCatAfterUse` and expands its internal control flow or data movement. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `moveCatAfterUse`，进一步展开其内部控制流或数据流转。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 60-72
```cpp
    new_cat_inputs[inp] = new_cat_input->output();
  }
  auto new_cat_list = subgraph->createClone(
      cat_list, [&](Value* k) { return new_cat_inputs[k]; });
  new_cat_list->insertBefore(cat);
  auto new_cat = subgraph->createClone(cat, [&](Value* k) {
    return (k == cat_list->output()) ? new_cat_list->output() : k;
  });
  new_cat->output()->setType(user_tensor_type);
  new_cat->insertBefore(cat);

  user->output()->replaceAllUsesWith(new_cat->output());
  user->destroy();
```
- **EN**: This chunk continues `moveCatAfterUse` and expands its internal control flow or data movement. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `moveCatAfterUse`，进一步展开其内部控制流或数据流转。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 74-84
```cpp
  TORCH_INTERNAL_ASSERT(
      !cat->output()->hasUses(),
      buildErrorMessage("aten::cat output is not used."));
  cat->destroy();

  if (!cat_list->output()->hasUses()) {
    cat_list->destroy();
  }

  return new_cat;
}
```
- **EN**: This chunk continues `moveCatAfterUse` and expands its internal control flow or data movement. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `moveCatAfterUse`，进一步展开其内部控制流或数据流转。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 86-101
```cpp
static int numTensorInputs(Node* node) {
  int count = 0;
  for (auto v : node->inputs()) {
    if (v->type()->cast<c10::TensorType>()) {
      ++count;
    }
  }
  return count;
}

// Returns true if the given `cat` node promotes types.
// If the inputs to `cat` are of different types, then the implementation
// of `cat` is expected to promote type.
static bool doesCatPromoteTypes(Node* node) {
  TORCH_INTERNAL_ASSERT(
      node->kind() == aten::cat,
```
- **EN**: This chunk defines `doesCatPromoteTypes`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `doesCatPromoteTypes`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 102-117
```cpp
      buildErrorMessage("Graph node is not aten::cat."));
  TORCH_INTERNAL_ASSERT(
      node->input(0)->node()->kind() == prim::ListConstruct,
      buildErrorMessage("aten::cat inputs are not expected."));
  auto inputs = node->input(0)->node()->inputs();
  TORCH_INTERNAL_ASSERT(
      !inputs.empty(), buildErrorMessage("Empty inputs of ListConstruct"));
  auto scalar_type =
      inputs.front()->type()->cast<c10::TensorType>()->scalarType();
  for (size_t i = 1; i < inputs.size(); ++i) {
    auto inp_scalar_type =
        inputs[i]->type()->cast<c10::TensorType>()->scalarType();
    if (scalar_type != inp_scalar_type) {
      return true;
    }
  }
```
- **EN**: This chunk continues `doesCatPromoteTypes` and expands its internal control flow or data movement. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `doesCatPromoteTypes`，进一步展开其内部控制流或数据流转。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 118-133
```cpp
  return false;
}

// Move the users of the given `aten::cat` op to its inputs.
// The following constraints need to be satisfied on the cat op and its user.
//   * the cat op should have only one use.
//   * the user should be an element-wise op.
//   * the user should have only one tensor input.
//     - If the user has > 1 tensor inputs, that user op cannot be applied on
//       the inputs of cat because the other tensor inputs will not be split,
//       and hence the shape of those tensors would not match that of the
//       inputs of cat.
//       For example:
//           %1 = ...
//           %2 = ...
//           %3 = prim::ListConstruct([%1, %2])
```
- **EN**: This chunk continues `doesCatPromoteTypes` and expands its internal control flow or data movement. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `doesCatPromoteTypes`，进一步展开其内部控制流或数据流转。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 134-149
```cpp
//           %4 = aten::cat(%3, ...)
//           %5 = aten::add(%4, %0)
//       In this example, we cannot move `aten::add` to the inputs of
//       `aten::cat`, %1 and %2, because the shape of %0 will be different.
//    * the cat op does not promote types.
//      - When the cat op promote types, the type of inputs to cat after moving
//        it user needs to reflect the original type. This is currently not
//        handled. TODO
static void moveCatOpToEnd(Node* cat, const std::shared_ptr<Graph>& subgraph) {
  TORCH_INTERNAL_ASSERT(
      cat->kind() == aten::cat,
      buildErrorMessage("Graph node is not aten::cat."));
  if (cat->output()->uses().size() == 1) {
    auto use = cat->output()->uses().front();
    if (get_tensorexpr_elementwise_set().contains(use.user) &&
        numTensorInputs(use.user) == 1) {
```
- **EN**: This chunk defines `moveCatOpToEnd`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages.
- **CN**: 这一段定义了 `moveCatOpToEnd`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。

### Lines 150-160
```cpp
      if (!doesCatPromoteTypes(cat)) {
        TORCH_INTERNAL_ASSERT(
            use.user->output()->owningGraph() == subgraph.get(),
            buildErrorMessage(
                "aten::cat user graph does not math the given subgraph."));
        auto new_cat = moveCatAfterUse(cat, use.user, subgraph);
        moveCatOpToEnd(new_cat, subgraph);
      }
    }
  }
}
```
- **EN**: This chunk continues `moveCatOpToEnd` and expands its internal control flow or data movement. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages.
- **CN**: 这一段延续了 `moveCatOpToEnd`，进一步展开其内部控制流或数据流转。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。

### Lines 162-174
```cpp
// Moves the users of `aten::cat` ops to its inputs whenever possible
// in the given subgraph.
static void moveCatOpsToEnd(const std::shared_ptr<Graph>& subgraph) {
  std::vector<Node*> cat_nodes;
  for (Node* n : subgraph->nodes()) {
    if (n->kind() == aten::cat) {
      cat_nodes.push_back(n);
    }
  }
  for (auto cat : cat_nodes) {
    moveCatOpToEnd(cat, subgraph);
  }
}
```
- **EN**: This chunk defines `moveCatOpsToEnd`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段定义了 `moveCatOpsToEnd`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 176-191
```cpp
bool OptimizeCat(const std::shared_ptr<Graph>& graph) {
  if (getCatWoConditionals()) {
    moveCatOpsToEnd(graph);
    return true;
  }
  return false;
}

void annotateInputShapes(
    const std::shared_ptr<Graph>& graph,
    const std::vector<std::optional<at::Tensor>>& example_inputs) {
  TORCH_INTERNAL_ASSERT(
      graph->inputs().size() == example_inputs.size(),
      buildErrorMessage("Given inputs do not match the fuser graph inputs."));
  for (size_t idx = 0; idx < example_inputs.size(); idx++) {
    if (auto t = example_inputs[idx]) {
```
- **EN**: This chunk defines `annotateInputShapes`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `annotateInputShapes`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 192-207
```cpp
      auto concrete_tensor_type = tensorTypeInCurrentExecutionContext(*t);
      graph->inputs().at(idx)->setType(concrete_tensor_type);
    }
  }
}

std::shared_ptr<Graph> removeUnusedSelfArgument(
    const std::shared_ptr<Graph>& graph) {
  if (graph->inputs().empty()) {
    return graph;
  }
  jit::Value* self_argument = graph->inputs().at(0);
  if (!self_argument->uses().empty() || !self_argument->type()->is_module()) {
    return graph;
  }
  graph->eraseInput(0);
```
- **EN**: This chunk defines `removeUnusedSelfArgument`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `removeUnusedSelfArgument`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 208-223
```cpp
  return graph;
}

std::vector<int64_t> makeShapesSymbolic(
    std::shared_ptr<Graph>& graph,
    const std::vector<int64_t>& size_vals) {
  std::unordered_set<Value*> values;
  for (auto v : graph->inputs()) {
    values.insert(v);
  }
  for (auto v : graph->outputs()) {
    values.insert(v);
  }
  for (auto n : graph->nodes()) {
    for (auto v : n->inputs()) {
      values.insert(v);
```
- **EN**: This chunk defines `makeShapesSymbolic`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `makeShapesSymbolic`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 224-236
```cpp
    }
    for (auto v : n->outputs()) {
      values.insert(v);
    }
  }
  std::unordered_map<int64_t, int64_t> shape_to_sym_shape;
  std::vector<int64_t> new_syms;
  for (int64_t size_val : size_vals) {
    auto new_shape_symbol = at::ShapeSymbol::newSymbol().value();
    shape_to_sym_shape[size_val] = new_shape_symbol;
    new_syms.push_back(new_shape_symbol);
    graph->addInput("sym_shape")->setType(IntType::get());
  }
```
- **EN**: This chunk continues `makeShapesSymbolic` and expands its internal control flow or data movement. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically.
- **CN**: 这一段延续了 `makeShapesSymbolic`，进一步展开其内部控制流或数据流转。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。

### Lines 238-253
```cpp
  for (auto v : values) {
    if (!v->type()->cast<TensorType>()) {
      continue;
    }
    auto tt = v->type()->expect<TensorType>();
    if (!tt->symbolic_sizes().sizes()) {
      continue;
    }
    std::vector<at::ShapeSymbol> shape_vec = *tt->symbolic_sizes().sizes();

    auto new_sizes = c10::fmap(shape_vec, [&](const at::ShapeSymbol& shape) {
      auto value = shape.value();
      if (shape_to_sym_shape.count(value)) {
        return shape_to_sym_shape.at(value);
      }
      return value;
```
- **EN**: This chunk continues `makeShapesSymbolic` and expands its internal control flow or data movement. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `makeShapesSymbolic`，进一步展开其内部控制流或数据流转。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 254-269
```cpp
    });
    v->setType(tt->withSymbolicShapes(c10::SymbolicShape(new_sizes)));
  }

  return new_syms;
}

bool isGraphCompilable(const std::shared_ptr<Graph>& graph) {
  for (auto input : graph->inputs()) {
    auto const& t = input->type();
    auto const& k = t->kind();
    if (k != TypeKind::TensorType && k != TypeKind::FloatType &&
        k != TypeKind::BoolType && k != TypeKind::IntType) {
      GRAPH_DEBUG("Input %", input->debugName(), " has unsupported type ", *t);
      return false;
    }
```
- **EN**: This chunk defines `isGraphCompilable`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `isGraphCompilable`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 270-285
```cpp
  }

  for (auto n : graph->nodes()) {
    for (auto v : n->inputs()) {
      auto const& t = v->type();
      if (t->kind() == TypeKind::TensorType) {
        auto tt = t->cast<TensorType>();
        if (!tt->isComplete()) {
          GRAPH_DEBUG(
              "%",
              v->debugName(),
              " is not a complete tensor! The type is: ",
              *t);
          return false;
        }
      }
```
- **EN**: This chunk continues `isGraphCompilable` and expands its internal control flow or data movement. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `isGraphCompilable`，进一步展开其内部控制流或数据流转。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 286-298
```cpp
    }
    for (auto v : n->outputs()) {
      auto const& t = v->type();
      if (t->kind() == TypeKind::TensorType) {
        auto tt = t->cast<TensorType>();
        if (!tt->isComplete()) {
          GRAPH_DEBUG(
              "%", v->debugName(), " is not a complete! The type is: ", *t);
          return false;
        }
      }
    }
  }
```
- **EN**: This chunk continues `isGraphCompilable` and expands its internal control flow or data movement. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `isGraphCompilable`，进一步展开其内部控制流或数据流转。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 300-312
```cpp
  // TODO: check if all nodes have lowerings
  return true;
}

static void fixupTypeInfoForValue(
    Value* v,
    std::optional<at::ScalarType> scalar_type,
    std::optional<at::Device> device) {
  Node* n = v->node();
  auto const& t = v->type();
  if (t->kind() != TypeKind::TensorType) {
    return;
  }
```
- **EN**: This chunk defines `fixupTypeInfoForValue`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `fixupTypeInfoForValue`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 314-329
```cpp
  if (n->kind() == prim::Constant) {
    auto const_tensor = toIValue(v)->toTensor();
    auto concrete_tensor_type =
        tensorTypeInCurrentExecutionContext(const_tensor);
    v->setType(concrete_tensor_type);
    return;
  }

  TensorTypePtr new_tt;
  auto tt = t->cast<TensorType>();
  auto sizes = tt->sizes();
  if (!sizes.concrete_sizes()) {
    GRAPH_DEBUG("No concrete sizes for %", v->debugName());
    return;
  }
  auto strides = tt->strides();
```
- **EN**: This chunk continues `fixupTypeInfoForValue` and expands its internal control flow or data movement. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `fixupTypeInfoForValue`，进一步展开其内部控制流或数据流转。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 330-339
```cpp
  auto dtype = tt->scalarType() ? tt->scalarType() : scalar_type;
  auto concrete_sizes = *sizes.concrete_sizes();
  auto concrete_strides = strides.concrete_sizes()
      ? *strides.concrete_sizes()
      : TensorType::contiguousStridesOf(concrete_sizes);
  new_tt = TensorType::create(
      dtype, device, concrete_sizes, concrete_strides, false);

  v->setType(new_tt);
}
```
- **EN**: This chunk continues `fixupTypeInfoForValue` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `fixupTypeInfoForValue`，进一步展开其内部控制流或数据流转。

### Lines 341-356
```cpp
static std::optional<at::ScalarType> inferScalarType(Node* n) {
  std::optional<at::ScalarType> scalar_type;
  for (auto v : n->inputs()) {
    auto const& t = v->type();
    if (t->kind() == TypeKind::TensorType) {
      auto tt = t->cast<TensorType>();
      if (!scalar_type) {
        scalar_type = tt->scalarType();
      }
      if (tt->scalarType() && *tt->scalarType() != scalar_type) {
        GRAPH_DEBUG(
            "Inputs of ", n, " have different scalar types, cannot fixup!");
        return std::nullopt;
      }
    }
  }
```
- **EN**: This chunk defines `inferScalarType`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `inferScalarType`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 357-372
```cpp
  return scalar_type;
}

static std::optional<at::Device> inferDevice(Node* n) {
  std::optional<at::Device> device;
  for (auto v : n->inputs()) {
    auto const& t = v->type();
    if (t->kind() == TypeKind::TensorType) {
      auto tt = t->cast<TensorType>();
      if (!device) {
        device = tt->device();
      }
      if (tt->device() && *tt->device() != device) {
        GRAPH_DEBUG("Inputs of ", n, " have different devices, cannot fixup!");
        return std::nullopt;
      }
```
- **EN**: This chunk defines `inferDevice`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `inferDevice`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 373-388
```cpp
    }
  }
  if (!device) {
    device = at::kCPU;
  }
  return device;
}

void fixupMissingShapeInfo(const std::shared_ptr<Graph>& graph) {
  for (auto input : graph->inputs()) {
    auto const& t = input->type();
    if (t->kind() == TypeKind::TensorType) {
      auto tt = t->cast<TensorType>();
      if (!tt->scalarType()) {
        GRAPH_DEBUG("No dtype for %", input->debugName());
        return;
```
- **EN**: This chunk defines `fixupMissingShapeInfo`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `fixupMissingShapeInfo`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 389-403
```cpp
      }
      fixupTypeInfoForValue(
          input, tt->scalarType(), tt->device() ? tt->device() : at::kCPU);
    }
  }

  for (auto n : graph->nodes()) {
    std::optional<at::ScalarType> scalar_type = inferScalarType(n);
    std::optional<at::Device> device = inferDevice(n);

    for (auto v : n->outputs()) {
      fixupTypeInfoForValue(v, scalar_type, device);
    }
  }
}
```
- **EN**: This chunk continues `fixupMissingShapeInfo` and expands its internal control flow or data movement. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically.
- **CN**: 这一段延续了 `fixupMissingShapeInfo`，进一步展开其内部控制流或数据流转。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。

### Lines 405-420
```cpp
std::shared_ptr<Graph> removeGraphOutput(
    const std::shared_ptr<Graph>& graph,
    size_t idx) {
  graph->eraseOutput(idx);
  return graph;
}

std::shared_ptr<Graph> replaceListOutputWithTuple(
    const std::shared_ptr<Graph>& graph) {
  auto out = graph->outputs()[0];
  auto out_node = out->node();
  if (out_node->kind() != prim::ListConstruct) {
    return graph;
  }
  auto tuple_node = graph->createTuple(out_node->inputs());
  tuple_node->insertAfter(out_node);
```
- **EN**: This chunk defines `replaceListOutputWithTuple`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `replaceListOutputWithTuple`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 421-436
```cpp
  out->replaceAllUsesWith(tuple_node->output());
  return graph;
}

static bool trimGraphOnce(const std::shared_ptr<Graph>& graph) {
  Node* ret = graph->return_node();
  std::unordered_set<Value*> graph_inputs(
      graph->inputs().begin(), graph->inputs().end());
  std::unordered_set<Value*> outputs(
      graph->outputs().begin(), graph->outputs().end());
  bool changed = false;
  for (size_t idx = 0; idx < ret->inputs().size(); idx++) {
    auto v = ret->inputs()[idx];
    if (graph_inputs.count(v)) {
      continue;
    }
```
- **EN**: This chunk defines `outputs`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `outputs`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 437-446
```cpp
    // Delete the graph output IDX and add all inputs of the node producing that
    // value to the graph outputs
    graph->eraseOutput(idx);
    for (auto v_ins : v->node()->inputs()) {
      if (outputs.count(v_ins)) {
        continue;
      }
      if (v_ins->node()->kind() == prim::Constant) {
        continue;
      }
```
- **EN**: This chunk continues `outputs` and expands its internal control flow or data movement. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段延续了 `outputs`，进一步展开其内部控制流或数据流转。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 448-463
```cpp
      graph->registerOutput(v_ins);
    }
    changed = true;
    break;
  }
  return changed;
}

static std::shared_ptr<Graph> dequantizeResults(
    const std::shared_ptr<Graph>& graph) {
  for (auto v : graph->outputs()) {
    auto& t = v->type();
    if (t->kind() == TypeKind::TensorType) {
      auto tt = t->cast<TensorType>();
      if (!tt->scalarType() || !c10::isQIntType(*tt->scalarType())) {
        continue;
```
- **EN**: This chunk defines `dequantizeResults`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `dequantizeResults`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 464-479
```cpp
      }
      Node* deq = graph->create(aten::dequantize, {v});
      graph->appendNode(deq);
      deq->output()->setType(tt->withScalarType(c10::kFloat));
      v->replaceAllUsesAfterNodeWith(deq, deq->output());
    }
  }
  return graph;
}

std::shared_ptr<Graph> trimGraph(
    const std::shared_ptr<Graph>& graph,
    int64_t iters) {
  bool changed = true;
  int64_t iter = 0;
  while (changed && iter++ < iters) {
```
- **EN**: This chunk defines `trimGraph`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `trimGraph`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 480-491
```cpp
    changed = trimGraphOnce(graph);
    EliminateDeadCode(graph->block());
  }
  // Avoid letting quantized values to graph outputs.
  // Ideally we should allow quantized outputs as well, but currently the main
  // user of this pass - AOT NNC - does not support it.
  // TODO: remove output dequantization once NNC supports quantized outputs.
  dequantizeResults(graph);
  return graph;
}

} // namespace torch::jit::tensorexpr
```
- **EN**: This chunk continues `trimGraph` and expands its internal control flow or data movement. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `trimGraph`，进一步展开其内部控制流或数据流转。 末尾会返回累积结果，或把更新后的状态交还给调用方。

## Key Concepts / 关键概念

- **Tensor Expression IR**
  - EN: Implements Tensor Expression IR, analyses, scheduling, lowering, and backend code generation.
  - CN: 实现 Tensor Expression IR、分析、调度、降级以及后端代码生成。
- **moveCatAfterUse**
  - EN: `moveCatAfterUse` is a central symbol declared or implemented in this file.
  - CN: `moveCatAfterUse` 是本文件声明或实现的核心符号。
- **numTensorInputs**
  - EN: `numTensorInputs` is a central symbol declared or implemented in this file.
  - CN: `numTensorInputs` 是本文件声明或实现的核心符号。
- **TorchScript runtime**
  - EN: Coordinates execution plans, stacks, and operator dispatch for scripted programs.
  - CN: 协调脚本程序的执行计划、栈以及算子分发。
- **Graph IR**
  - EN: Works on graph, node, and block structures that represent TorchScript programs.
  - CN: 围绕表示 TorchScript 程序的 graph、node 与 block 结构工作。
## Dependencies / 依赖关系

- **Internal PyTorch headers / 内部 PyTorch 头文件**: `torch/csrc/jit/tensorexpr/graph_opt.h`, `torch/csrc/jit/jit_log.h`, `torch/csrc/jit/passes/dead_code_elimination.h`, `torch/csrc/jit/runtime/symbolic_shape_registry_util.h`, `torch/csrc/jit/tensorexpr/kernel.h`
- **Primary symbols in this file / 本文件核心符号**: `moveCatAfterUse`, `numTensorInputs`, `doesCatPromoteTypes`, `moveCatOpToEnd`, `moveCatOpsToEnd`, `OptimizeCat`, `annotateInputShapes`, `removeUnusedSelfArgument`
