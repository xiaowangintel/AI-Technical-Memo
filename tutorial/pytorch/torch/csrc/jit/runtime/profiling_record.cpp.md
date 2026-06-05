# profiling_record.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/csrc/jit/runtime/profiling_record.cpp`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements TorchScript automatic-differentiation helpers, graph partitioning, and gradient-related transforms.
- **Purpose (CN)**: 实现 TorchScript 自动求导辅助逻辑、图划分以及与梯度相关的变换。
## Line-by-Line Analysis / 逐行分析

### Lines 1-14
```cpp
#include <torch/csrc/jit/runtime/profiling_record.h>

#include <ATen/core/symbol.h>
#include <c10/util/irange.h>
#include <torch/csrc/jit/jit_log.h>
#include <torch/csrc/jit/passes/clear_profiling.h>
#include <torch/csrc/jit/passes/tensorexpr_fuser.h>
#include <torch/csrc/jit/runtime/autodiff.h>
#include <torch/csrc/jit/runtime/graph_executor.h>
#include <torch/csrc/jit/runtime/interpreter.h>

namespace torch::jit {

namespace {
```
- **EN**: This block assembles the compilation dependencies, pulling in local JIT headers such as torch/csrc/jit/runtime/profiling_record.h, torch/csrc/jit/jit_log.h, torch/csrc/jit/passes/clear_profiling.h, and 4 more; ATen/c10 facilities such as ATen/core/symbol.h, c10/util/irange.h. The namespace declarations place the code inside torch::jit, matching the surrounding JIT subsystem.
- **CN**: 这一段组织编译依赖，引入了本地 JIT 头文件，如 torch/csrc/jit/runtime/profiling_record.h、torch/csrc/jit/jit_log.h、torch/csrc/jit/passes/clear_profiling.h 等共 7 项；ATen/c10 基础设施，如 ATen/core/symbol.h、c10/util/irange.h。 命名空间声明把代码放入 torch::jit 中，与周边 JIT 子系统保持一致。

### Lines 16-26
```cpp
class ProfileRegistry {
 public:
  static ProfileRegistry* getRegistry() {
    static ProfileRegistry profile_registry_;
    return &profile_registry_;
  }

  void registerProfileNode(const std::function<bool(const Node*)>& func) {
    std::lock_guard<std::mutex> guard(mutex_);
    registry_funcs_.push_back(func);
  }
```
- **EN**: It introduces or extends ProfileRegistry, which define the primary data structures or interfaces for this portion of the file. This chunk defines `guard`, which implements a focused step in the TorchScript execution pipeline. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 它引入或扩展了 ProfileRegistry，这些类型定义了本段涉及的主要数据结构或接口。 这一段定义了 `guard`，其作用是实现 TorchScript 执行流水线中的一个关键步骤。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 28-41
```cpp
  bool shouldProfileNode(const Node* node) {
    std::lock_guard<std::mutex> guard(mutex_);
    // to guard differentiable graphs, we want profiling information
    // (in particular requires_grad) for nodes handled by autodiff
    if (isDifferentiable(node)) {
      return true;
    }
    for (const auto& func : registry_funcs_) {
      if (func(node)) {
        return true;
      }
    }
    return false;
  }
```
- **EN**: This chunk defines `guard`, which implements a focused step in the TorchScript execution pipeline. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `guard`，其作用是实现 TorchScript 执行流水线中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 43-52
```cpp
 private:
  std::vector<std::function<bool(const Node*)>> registry_funcs_;
  std::mutex mutex_;
};

} // namespace

void RegisterProfilingNode(const std::function<bool(const Node*)>& func) {
  ProfileRegistry::getRegistry()->registerProfileNode(func);
}
```
- **EN**: This chunk defines `RegisterProfilingNode`, which registers schemas, operators, or passes with the surrounding runtime.
- **CN**: 这一段定义了 `RegisterProfilingNode`，其作用是向周边运行时注册 schema、算子或 pass。

### Lines 54-67
```cpp
bool ShapeSymbolTable::bindSymbolicShapes(
    at::IntArrayRef new_sizes,
    const c10::SymbolicShape& sym_shapes) {
  if (!sym_shapes.rank().has_value()) {
    return true;
  }
  if (*sym_shapes.rank() != new_sizes.size()) {
    return false;
  }
  for (const auto i : c10::irange(new_sizes.size())) {
    auto symbol = (*sym_shapes.sizes())[i];
    if (!symbol.is_static()) {
      continue;
    }
```
- **EN**: This chunk defines `bindSymbolicShapes`, which implements a focused step in the TorchScript execution pipeline. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `bindSymbolicShapes`，其作用是实现 TorchScript 执行流水线中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 69-82
```cpp
    if (!isBound(symbol)) {
      assign(symbol, new_sizes[i]);
      continue;
    }

    if (getValue(symbol) != new_sizes[i]) {
      return false;
    }
  }
  return true;
}

ProfilingRecord::ProfilingRecord(std::shared_ptr<Graph> g)
    : profiled_graph_(std::move(g)), profiling_count_(getNumProfiledRuns()) {}
```
- **EN**: This chunk continues `bindSymbolicShapes` and expands its internal control flow or data movement. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `bindSymbolicShapes`，进一步展开其内部控制流或数据流转。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 84-93
```cpp
ProfileOp* ProfilingRecord::createProfileNode(
    const std::function<void(Stack&)>& fp,
    at::ArrayRef<Value*> inputs) {
  auto pn = new ProfileOp(profiled_graph_.get(), fp);

  for (auto in : inputs) {
    pn->addInput(in);
  }
  return pn;
}
```
- **EN**: This chunk defines `createProfileNode`, which constructs derived state from the current inputs and invariants. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `createProfileNode`，其作用是根据当前输入和约束构建派生状态。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 95-110
```cpp
ProfileIValueOp* ProfilingRecord::createProfileIValueNode(Value* in_val) {
  auto pn = new ProfileIValueOp(this->profiled_graph_.get(), nullptr);
  pn->addInput(in_val);
  auto pno = pn->addOutput();
  pno->setType(in_val->type());
  return pn;
}

ProfileIValueOp* ProfilingRecord::createProfileIValueNode(
    ArrayRef<Value*> inputs) {
  auto pn = new ProfileIValueOp(this->profiled_graph_.get(), nullptr);
  for (auto inp : inputs) {
    pn->addInput(inp);
    auto pno = pn->addOutput();
    pno->setType(inp->type());
  }
```
- **EN**: This chunk defines `createProfileIValueNode`, which constructs derived state from the current inputs and invariants. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `createProfileIValueNode`，其作用是根据当前输入和约束构建派生状态。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 111-122
```cpp
  return pn;
}

namespace {
bool isOptionalTensorType(const TypePtr& type) {
  if (type->kind() != c10::TypeKind::OptionalType) {
    return false;
  }
  const auto& kind = type->expectRef<OptionalType>().getElementType()->kind();
  return kind == c10::TypeKind::TensorType;
}
} // namespace
```
- **EN**: This chunk defines `isOptionalTensorType`, which implements a focused step in the TorchScript execution pipeline. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `isOptionalTensorType`，其作用是实现 TorchScript 执行流水线中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 124-139
```cpp
// Inserts profiling nodes.
//
// The prim::profile node profiles Tensor and Optional[Tensor].
//
// It stores two fields:
// 1. attr::seen_none, an integer, which is initially 0 and is set to 1 if the
// profiled value is ever `None`
// 2. attr::profiled_type, which is the most specific Tensor type that matches
// all the non-null inputs observed during profiling.
void ProfilingRecord::insertShapeProfile(
    Node* n,
    size_t offset,
    const TypePtr& input_type) {
  Value* i = n->input(offset);
  auto pn = createProfileNode(nullptr, {i});
  auto pno = pn->addOutput();
```
- **EN**: This chunk defines `insertShapeProfile`, which implements a focused step in the TorchScript execution pipeline.
- **CN**: 这一段定义了 `insertShapeProfile`，其作用是实现 TorchScript 执行流水线中的一个关键步骤。

### Lines 140-155
```cpp
  pn->ty_(attr::profiled_type, TensorType::get());
  pn->i_(attr::seen_none, 0);
  if (isOptionalTensorType(input_type)) {
    pno->setType(OptionalType::create(TensorType::get()));
  } else if (input_type->kind() == c10::TypeKind::TensorType) {
    pno->setType(TensorType::get());
  } else {
    TORCH_INTERNAL_ASSERT(
        false,
        "Trying to profile an unsupported type (neither Tensor or Optional[Tensor]): ",
        input_type->str());
  }
  std::function<void(Stack&)> shape_profiler = [this, pn, pno](Stack& stack) {
    int64_t frame_id = 0;
    pop(stack, frame_id);
    IValue v;
```
- **EN**: This chunk continues `insertShapeProfile` and expands its internal control flow or data movement. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages.
- **CN**: 这一段延续了 `insertShapeProfile`，进一步展开其内部控制流或数据流转。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。

### Lines 156-171
```cpp
    pop(stack, v);

    TensorTypePtr new_tensor_type = nullptr;
    if (v.isTensor()) {
      auto& t = v.toTensor();
      new_tensor_type = tensorTypeInCurrentExecutionContext(t);
    }

    if (v.isTensor() || v.isNone()) {
      std::lock_guard<std::mutex> lock(this->mutex_);
      if (profiling_count_ > 0) {
        GRAPH_DEBUG(
            "In run ",
            frame_id,
            " annotating %",
            pno->debugName(),
```
- **EN**: This chunk defines `lock`, which implements a focused step in the TorchScript execution pipeline. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段定义了 `lock`，其作用是实现 TorchScript 执行流水线中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 172-187
```cpp
            " with ",
            *new_tensor_type);

        if (new_tensor_type != nullptr) {
          if (pn->hasSeenTensor()) {
            const auto& existing_tensor_type =
                pn->ty(attr::profiled_type)->expectRef<TensorType>();
            GRAPH_DEBUG(
                "Existing type for %",
                pno->debugName(),
                ": ",
                existing_tensor_type);
            auto merged_type = new_tensor_type->merge(existing_tensor_type);
            GRAPH_DEBUG(
                "Merged type for %", pno->debugName(), ": ", *merged_type);
            pn->ty_(attr::profiled_type, std::move(merged_type));
```
- **EN**: This chunk continues `lock` and expands its internal control flow or data movement. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段延续了 `lock`，进一步展开其内部控制流或数据流转。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 188-200
```cpp
          } else {
            pn->setHasSeenTensor(true);
            pn->ty_(attr::profiled_type, std::move(new_tensor_type));
          }
        }
        if (v.isNone()) {
          pn->i_(attr::seen_none, 1);
        }
      }
    }
    // passing t through
    push(stack, v);
  };
```
- **EN**: This chunk continues `lock` and expands its internal control flow or data movement. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段延续了 `lock`，进一步展开其内部控制流或数据流转。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 202-217
```cpp
  pn->setCallback(shape_profiler);
  pn->insertBefore(n);
  n->replaceInput(offset, pn->output());
}

static bool needsProfiledInputs(Node* n) {
  if (tensorexpr::isSupported(n)) {
    return true;
  }

  switch (n->kind()) {
    // specialize_autogradzero
    case prim::AutogradAdd:
    case prim::AutogradAnyNonZero:
    case prim::AutogradAllNonZero:
    case prim::AutogradAllZero:
```
- **EN**: This chunk defines `needsProfiledInputs`, which implements a focused step in the TorchScript execution pipeline. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Switch-based dispatch selects specialized behavior for each opcode, node kind, or schema case. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `needsProfiledInputs`，其作用是实现 TorchScript 执行流水线中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 基于 switch 的分发为不同 opcode、节点类型或 schema 分支选择专门行为。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 218-233
```cpp
    case prim::AutogradZero:
    // peephole
    case aten::dim:
    case aten::size:
    case aten::expand:
    case prim::dtype:
    case prim::device:
    case prim::is_cuda:
    case aten::is_floating_point:
    case aten::type_as:
    // TODO: hack to make `test_lstm_gates_permutations_cuda`
    // pass.
    case aten::t:
    case aten::mm:
      return true;
    default:
```
- **EN**: This chunk continues `needsProfiledInputs` and expands its internal control flow or data movement. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `needsProfiledInputs`，进一步展开其内部控制流或数据流转。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 234-249
```cpp
      return ProfileRegistry::getRegistry()->shouldProfileNode(n);
  }
}

static bool needsProfiledOutput(Node* n) {
  if (tensorexpr::isSupported(n)) {
    return true;
  }

  switch (n->kind()) {
    case prim::AutogradAdd:
    case prim::AutogradZero:
      return true;
    default:
      return ProfileRegistry::getRegistry()->shouldProfileNode(n);
  }
```
- **EN**: This chunk defines `needsProfiledOutput`, which implements a focused step in the TorchScript execution pipeline. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Switch-based dispatch selects specialized behavior for each opcode, node kind, or schema case. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `needsProfiledOutput`，其作用是实现 TorchScript 执行流水线中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 基于 switch 的分发为不同 opcode、节点类型或 schema 分支选择专门行为。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 250-263
```cpp
}

void ProfilingRecord::removeProfileCounter(Block* b) {
  for (auto it = b->nodes().rbegin(); it != b->nodes().rend();) {
    auto n = *it;
    if (n->kind() == prim::profile && n->inputs().empty()) {
      it.destroyCurrent();
      // there is only one counter node
      return;
    } else {
      it++;
    }
  }
}
```
- **EN**: This chunk defines `removeProfileCounter`, which implements a focused step in the TorchScript execution pipeline. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `removeProfileCounter`，其作用是实现 TorchScript 执行流水线中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 265-276
```cpp
void ProfilingRecord::instrumentBlock(Block* block) {
  for (auto it = block->nodes().begin(); it != block->nodes().end(); ++it) {
    auto n = *it;
    for (const auto offset : c10::irange(n->inputs().size())) {
      auto i = n->input(offset);
      if ((needsProfiledInputs(n) || needsProfiledOutput(i->node()))) {
        if (i->type()->kind() == c10::TypeKind::TensorType ||
            isOptionalTensorType(i->type())) {
          insertShapeProfile(n, offset, i->type());
        }
      }
    }
```
- **EN**: This chunk defines `instrumentBlock`, which implements a focused step in the TorchScript execution pipeline. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段定义了 `instrumentBlock`，其作用是实现 TorchScript 执行流水线中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 278-293
```cpp
    for (auto b : n->blocks()) {
      instrumentBlock(b);
    }
  }

  // inserting profile nodes on block outputs
  // allows us to eliminate more guards as
  // the use of a guard is now in the same
  // block as opposed to being separated from
  // the definition by block boundaries
  for (size_t offset = 0; offset < block->return_node()->inputs().size();
       offset++) {
    auto i = block->return_node()->input(offset);
    if (i->type()->isSubtypeOf(*TensorType::get()) ||
        isOptionalTensorType(i->type())) {
      insertShapeProfile(block->return_node(), offset, i->type());
```
- **EN**: This chunk continues `instrumentBlock` and expands its internal control flow or data movement. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段延续了 `instrumentBlock`，进一步展开其内部控制流或数据流转。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 294-309
```cpp
    }
  }
}

void ProfilingRecord::removeProfilingNodes(Block* b) {
  for (auto it = b->nodes().begin(); it != b->nodes().end(); it++) {
    if (it->kind() == prim::profile || it->kind() == prim::profile_ivalue) {
      it->output()->replaceAllUsesWith(it->input());
      it.destroyCurrent();
    } else {
      for (Block* ib : it->blocks()) {
        removeProfilingNodes(ib);
      }
    }
  }
}
```
- **EN**: This chunk defines `removeProfilingNodes`, which implements a focused step in the TorchScript execution pipeline. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段定义了 `removeProfilingNodes`，其作用是实现 TorchScript 执行流水线中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 311-324
```cpp
bool ProfilingRecord::ready() const {
  std::lock_guard<std::mutex> lock(this->mutex_);
  return profiling_count_ == 0;
}

std::unique_ptr<ProfilingRecord> ProfilingRecord::instrumentGraph(
    const std::shared_ptr<Graph>& graph) {
  auto new_g = graph->copy();

  auto pr = std::unique_ptr<ProfilingRecord>(new ProfilingRecord(new_g));
  auto raw_pr = pr.get();
  unprofileGraphInputs(new_g);
  unprofileBlock(new_g->block());
  pr->instrumentBlock(new_g->block());
```
- **EN**: This chunk defines `instrumentGraph`, which implements a focused step in the TorchScript execution pipeline. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `instrumentGraph`，其作用是实现 TorchScript 执行流水线中的一个关键步骤。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 326-335
```cpp
  std::function<void(Stack&)> counter = [raw_pr](Stack& stack) {
    int64_t frame_id = 0;
    pop(stack, frame_id);

    std::lock_guard<std::mutex> lock(raw_pr->mutex_);

    if (raw_pr->profiling_count_ > 0) {
      raw_pr->profiling_count_--;
    }
  };
```
- **EN**: This chunk defines `lock`, which implements a focused step in the TorchScript execution pipeline. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段定义了 `lock`，其作用是实现 TorchScript 执行流水线中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 337-343
```cpp
  auto pop = pr->createProfileNode(counter, {});
  new_g->appendNode(pop);
  GRAPH_DUMP("Instrumented Graph: ", new_g);
  return pr;
}

} // namespace torch::jit
```
- **EN**: This chunk continues `lock` and expands its internal control flow or data movement. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `lock`，进一步展开其内部控制流或数据流转。 末尾会返回累积结果，或把更新后的状态交还给调用方。

## Key Concepts / 关键概念

- **TorchScript runtime**
  - EN: Implements TorchScript runtime execution, specialization, operator dispatch, and static-runtime support.
  - CN: 实现 TorchScript 运行时执行、特化、算子分发以及静态运行时支持。
- **ProfileRegistry**
  - EN: `ProfileRegistry` is a central symbol declared or implemented in this file.
  - CN: `ProfileRegistry` 是本文件声明或实现的核心符号。
- **getRegistry**
  - EN: `getRegistry` is a central symbol declared or implemented in this file.
  - CN: `getRegistry` 是本文件声明或实现的核心符号。
- **Graph IR**
  - EN: Works on graph, node, and block structures that represent TorchScript programs.
  - CN: 围绕表示 TorchScript 程序的 graph、node 与 block 结构工作。
- **IValue data model**
  - EN: Moves values through generic boxed runtime containers used by TorchScript.
  - CN: 通过 TorchScript 使用的通用装箱运行时容器传递数值。
- **Runtime profiling**
  - EN: Captures observed types or shapes so later passes can specialize execution.
  - CN: 捕获观测到的类型或形状，以便后续 pass 进行特化。
## Dependencies / 依赖关系

- **Internal PyTorch headers / 内部 PyTorch 头文件**: `torch/csrc/jit/runtime/profiling_record.h`, `torch/csrc/jit/jit_log.h`, `torch/csrc/jit/passes/clear_profiling.h`, `torch/csrc/jit/passes/tensorexpr_fuser.h`, `torch/csrc/jit/runtime/autodiff.h`, `torch/csrc/jit/runtime/graph_executor.h`, `torch/csrc/jit/runtime/interpreter.h`
- **ATen/c10 foundations / ATen/c10 基础设施**: `ATen/core/symbol.h`, `c10/util/irange.h`
- **Primary symbols in this file / 本文件核心符号**: `ProfileRegistry`, `getRegistry`, `registerProfileNode`, `guard`, `shouldProfileNode`, `RegisterProfilingNode`, `bindSymbolicShapes`, `createProfileNode`
