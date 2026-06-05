# jit_trace.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/csrc/jit/runtime/jit_trace.cpp`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Defines graph-executor interfaces and plan-selection logic for optimized TorchScript execution.
- **Purpose (CN)**: 定义图执行器接口以及用于优化 TorchScript 执行的计划选择逻辑。
## Line-by-Line Analysis / 逐行分析

### Lines 2-16
```cpp
#include <ATen/core/ivalue.h>
#include <ATen/core/symbol.h>
#include <torch/csrc/jit/ir/ir_views.h>
#include <torch/csrc/jit/jit_log.h>
#include <torch/csrc/jit/passes/dead_code_elimination.h>
#include <torch/csrc/jit/passes/inliner.h>
#include <torch/csrc/jit/runtime/graph_executor.h>
#include <torch/csrc/jit/runtime/interpreter.h>
#include <torch/csrc/jit/runtime/jit_trace.h>
#include <torch/csrc/jit/runtime/profiling_record.h>
#include <unordered_map>

namespace torch::jit {

namespace {
```
- **EN**: This block assembles the compilation dependencies, pulling in local JIT headers such as torch/csrc/jit/ir/ir_views.h, torch/csrc/jit/jit_log.h, torch/csrc/jit/passes/dead_code_elimination.h, and 5 more; ATen/c10 facilities such as ATen/core/ivalue.h, ATen/core/symbol.h; standard-library headers such as unordered_map. The namespace declarations place the code inside torch::jit, matching the surrounding JIT subsystem.
- **CN**: 这一段组织编译依赖，引入了本地 JIT 头文件，如 torch/csrc/jit/ir/ir_views.h、torch/csrc/jit/jit_log.h、torch/csrc/jit/passes/dead_code_elimination.h 等共 8 项；ATen/c10 基础设施，如 ATen/core/ivalue.h、ATen/core/symbol.h；标准库头文件，如 unordered_map。 命名空间声明把代码放入 torch::jit 中，与周边 JIT 子系统保持一致。

### Lines 18-28
```cpp
// A helper structure to maintain the mappings
// between values from a scripted graph and
// a traced graph
struct TracingData {
  std::unordered_map<Value*, Value*> old_to_new_;
  std::shared_ptr<Graph> traced_graph_ = nullptr;

  TracingData() {
    traced_graph_ = std::make_shared<Graph>();
  }
};
```
- **EN**: It introduces or extends TracingData, which define the primary data structures or interfaces for this portion of the file.
- **CN**: 它引入或扩展了 TracingData，这些类型定义了本段涉及的主要数据结构或接口。

### Lines 30-45
```cpp
// create a node in the traced graph that corresponds to `node`
// in the scripted graph. Similar to how `cloneNode` works
Node* traceNode(Node* node, TracingData& td, Stack& stack) {
  GRAPH_DEBUG("Tracing node ", getHeader(node));
  auto* block = td.traced_graph_->block();
  auto env = [&td](Value* v) { return td.old_to_new_.at(v); };

  auto new_node = block->appendNode(td.traced_graph_->createClone(node, env));
  for (size_t i = 0; i < node->outputs().size(); ++i) {
    auto oo = node->outputs()[i];
    auto no = new_node->outputs()[i];
    no->copyMetadata(oo);
    td.old_to_new_[oo] = no;
    GRAPH_DEBUG(
        "Mapping ",
        oo->debugName(),
```
- **EN**: This chunk defines `traceNode`, which implements a focused step in the TorchScript execution pipeline. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `traceNode`，其作用是实现 TorchScript 执行流水线中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 46-57
```cpp
        " to ",
        no->debugName()); // old to new outputs
  }
  return new_node;
}

void eraseAllOutputs(Node* opt_pn) {
  for (auto i = static_cast<int64_t>(opt_pn->outputs().size()) - 1; i >= 0;
       i--) {
    opt_pn->eraseOutput(i);
  }
}
```
- **EN**: This chunk defines `eraseAllOutputs`, which implements a focused step in the TorchScript execution pipeline. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `eraseAllOutputs`，其作用是实现 TorchScript 执行流水线中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 59-74
```cpp
void insertTracingNodes(
    Block* /*block*/,
    ProfilingRecord* /*pr*/,
    TracingData& /*td*/);

// The subtlety in `createPropNodeForIfBlock` is that we need to create
// a "propagate" node that will propagate the mapping between the outputs
// of a then/else block and the outputs in the traced graph onto the outputs
// of the if node in the scripted node. Note, if nodes will disappear in the
// the traced graph but they are still used in the scripted graph.
void createPropNodeForIfBlock(
    Block* b,
    Node* n,
    ProfilingRecord* pr,
    TracingData& td) {
  std::vector<Value*> empty_values{};
```
- **EN**: This chunk defines `createPropNodeForIfBlock`, which constructs derived state from the current inputs and invariants.
- **CN**: 这一段定义了 `createPropNodeForIfBlock`，其作用是根据当前输入和约束构建派生状态。

### Lines 75-85
```cpp
  auto opt_pn = pr->createProfileIValueNode(empty_values);
  eraseAllOutputs(opt_pn);
  insertTracingNodes(b, pr, td);
  b->appendNode(opt_pn);
  std::function<void(Stack&)> optional_profiler =
      [pr, n, b, &td](Stack& stack) {
        std::lock_guard<std::mutex> lock(pr->mutex_);

        // frame_id is unused
        int64_t frame_id = 0;
        pop(stack, frame_id);
```
- **EN**: This chunk defines `lock`, which implements a focused step in the TorchScript execution pipeline.
- **CN**: 这一段定义了 `lock`，其作用是实现 TorchScript 执行流水线中的一个关键步骤。

### Lines 87-97
```cpp
        for (size_t i = 0; i < b->outputs().size(); i++) {
          // propagate a then-block or else-output to an if-output
          auto nbo = td.old_to_new_.at(b->outputs()[i]);
          td.old_to_new_[n->outputs()[i]] = nbo;
          GRAPH_DEBUG(
              "Map ",
              td.old_to_new_[n->outputs()[i]]->debugName(),
              " to ",
              nbo->debugName());
        }
      };
```
- **EN**: This chunk continues `lock` and expands its internal control flow or data movement. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically.
- **CN**: 这一段延续了 `lock`，进一步展开其内部控制流或数据流转。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。

### Lines 99-114
```cpp
  // uncomment for debugging
  // opt_pn->i_(Symbol::attr("propagate"), 1);
  opt_pn->setCallback(optional_profiler);
}

// loop counter is implicit in the loop body outputs, we need to make
// it explicit so it can used in 2+ iterations
void traceLoopCounter(Node* n, ProfilingRecord* pr, TracingData& td) {
  LoopView lv(n);
  auto opt_pn = pr->createProfileIValueNode(lv.currentTripCount());
  eraseAllOutputs(opt_pn);
  lv.bodyBlock()->prependNode(opt_pn);
  std::function<void(Stack&)> optional_profiler = [pr, n, &td](Stack& stack) {
    std::lock_guard<std::mutex> lock(pr->mutex_);
    // frame_id is unused
    int64_t frame_id = 0;
```
- **EN**: This chunk defines `lock`, which implements a focused step in the TorchScript execution pipeline.
- **CN**: 这一段定义了 `lock`，其作用是实现 TorchScript 执行流水线中的一个关键步骤。

### Lines 115-127
```cpp
    pop(stack, frame_id);
    int64_t loop_counter = 0;
    pop(stack, loop_counter);
    WithInsertPoint wip(td.traced_graph_->block());
    auto lc = td.traced_graph_->insertConstant(loop_counter);
    LoopView lv(n);
    td.old_to_new_[lv.currentTripCount()] = lc;
  };

  // uncomment for debugging
  // opt_pn->i_(Symbol::attr("loop_counter"), 1);
  opt_pn->setCallback(optional_profiler);
}
```
- **EN**: This chunk declares `lv`, which implements a focused step in the TorchScript execution pipeline.
- **CN**: 这一段声明了 `lv`，其作用是实现 TorchScript 执行流水线中的一个关键步骤。

### Lines 129-144
```cpp
// Similar to how we propagate the mappings for If nodes, we need to propagate
// the mappings from the loop body to the beginning of the block in case we
// run another iteration and to the outputs of the Loop node, for any logic
// downstream that uses the output values of the loop node
static void traceLoop(Node* n, ProfilingRecord* pr, TracingData& td) {
  std::vector<Value*> empty_values{};

  // this is a propagation node for block inputs (phi values)
  // these come from either `prim::Loop` inputs or loop body outputs
  {
    auto opt_pn = pr->createProfileIValueNode(empty_values);
    eraseAllOutputs(opt_pn);
    opt_pn->insertBefore(n);
    LoopView lv(n);
    std::function<void(Stack&)> optional_profiler = [pr, n, &td](Stack& stack) {
      std::lock_guard<std::mutex> lock(pr->mutex_);
```
- **EN**: This chunk defines `lock`, which implements a focused step in the TorchScript execution pipeline.
- **CN**: 这一段定义了 `lock`，其作用是实现 TorchScript 执行流水线中的一个关键步骤。

### Lines 146-161
```cpp
      // frame_id is unused
      int64_t frame_id = 0;
      pop(stack, frame_id);

      LoopView lv(n);
      TORCH_INTERNAL_ASSERT(
          lv.bodyCarriedInputs().size() == lv.carriedInputs().size());
      for (size_t i = 0; i < lv.bodyCarriedInputs().size(); i++) {
        auto bno = td.old_to_new_.at(lv.carriedInputs()[i]);
        td.old_to_new_[lv.bodyCarriedInputs()[i]] = bno;
        GRAPH_DEBUG(
            "Map ",
            td.old_to_new_[lv.bodyCarriedInputs()[i]]->debugName(),
            " to ",
            bno->debugName());
      }
```
- **EN**: This chunk defines `lv`, which implements a focused step in the TorchScript execution pipeline. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Assertions and error paths make invalid states fail early instead of silently corrupting later stages.
- **CN**: 这一段定义了 `lv`，其作用是实现 TorchScript 执行流水线中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。

### Lines 162-172
```cpp
    };

    // uncomment for debugging
    // opt_pn->i_(Symbol::attr("loop_entry"), 1);
    opt_pn->setCallback(optional_profiler);
  }

  {
    insertTracingNodes(LoopView(n).bodyBlock(), pr, td);
    traceLoopCounter(n, pr, td);
  }
```
- **EN**: This chunk continues `lv` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `lv`，进一步展开其内部控制流或数据流转。

### Lines 174-187
```cpp
  // this is a propagation node for loop outputs
  {
    auto opt_pn = pr->createProfileIValueNode(empty_values);
    eraseAllOutputs(opt_pn);
    LoopView(n).bodyBlock()->appendNode(opt_pn);

    // opt_pn->i_(Symbol::attr("loop_propagate"), 1);

    std::function<void(Stack&)> optional_profiler = [pr, n, &td](Stack& stack) {
      std::lock_guard<std::mutex> lock(pr->mutex_);

      // frame_id is unused
      int64_t frame_id = 0;
      pop(stack, frame_id);
```
- **EN**: This chunk defines `lock`, which implements a focused step in the TorchScript execution pipeline.
- **CN**: 这一段定义了 `lock`，其作用是实现 TorchScript 执行流水线中的一个关键步骤。

### Lines 189-202
```cpp
      LoopView lv(n);

      TORCH_INTERNAL_ASSERT(
          lv.bodyCarriedOutputs().size() == lv.carriedOutputs().size());
      for (size_t i = 0; i < lv.bodyCarriedOutputs().size(); i++) {
        auto bno = td.old_to_new_.at(lv.bodyCarriedOutputs()[i]);
        td.old_to_new_[lv.carriedOutputs()[i]] = bno;
        GRAPH_DEBUG(
            "Map ",
            td.old_to_new_[lv.bodyCarriedOutputs()[i]]->debugName(),
            " to ",
            bno->debugName());
      }
    };
```
- **EN**: This chunk defines `lv`, which implements a focused step in the TorchScript execution pipeline. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Assertions and error paths make invalid states fail early instead of silently corrupting later stages.
- **CN**: 这一段定义了 `lv`，其作用是实现 TorchScript 执行流水线中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。

### Lines 204-215
```cpp
    // uncomment for debugging
    // opt_pn->i_(Symbol::attr("loop_exit"), 1);
    opt_pn->setCallback(optional_profiler);
  }
}

// walks all the nodes in a block and adds profiled nodes to each node
// see the comment for `optional_profiler` below
void insertTracingNodes(Block* block, ProfilingRecord* pr, TracingData& td) {
  for (auto it = block->nodes().begin(); it != block->nodes().end();) {
    auto n = *it;
    it++;
```
- **EN**: This chunk defines `insertTracingNodes`, which implements a focused step in the TorchScript execution pipeline. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically.
- **CN**: 这一段定义了 `insertTracingNodes`，其作用是实现 TorchScript 执行流水线中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。

### Lines 217-228
```cpp
    GRAPH_DEBUG("Inserting trace for ", getHeader(n));
    if (n->kind() == prim::If) {
      IfView ifv(n);
      createPropNodeForIfBlock(ifv.thenBlock(), n, pr, td);
      createPropNodeForIfBlock(ifv.elseBlock(), n, pr, td);
      continue;
    }

    if (n->kind() == prim::Loop) {
      traceLoop(n, pr, td);
      continue;
    }
```
- **EN**: This chunk defines `ifv`, which implements a focused step in the TorchScript execution pipeline. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段定义了 `ifv`，其作用是实现 TorchScript 执行流水线中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 230-244
```cpp
    TORCH_INTERNAL_ASSERT(n->blocks().empty());
    auto opt_pn = pr->createProfileIValueNode(n->outputs());
    eraseAllOutputs(opt_pn);
    opt_pn->insertAfter(n);

    // we only use the `opt_pn->node()` to trigger the handler
    // we still capture the actual scripted node `n` we want to trace
    // we look at its inputs, map them to the inputs in the traced graph
    // and create a new node with `traceNode`
    std::function<void(Stack&)> optional_profiler = [pr, n, &td](Stack& stack) {
      std::lock_guard<std::mutex> lock(pr->mutex_);

      // frame_id is unused
      int64_t frame_id = 0;
      pop(stack, frame_id);
```
- **EN**: This chunk defines `lock`, which implements a focused step in the TorchScript execution pipeline. Assertions and error paths make invalid states fail early instead of silently corrupting later stages.
- **CN**: 这一段定义了 `lock`，其作用是实现 TorchScript 执行流水线中的一个关键步骤。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。

### Lines 246-259
```cpp
      GRAPH_DEBUG("Tracing ", getHeader(n));
      auto tracer = traceNode(n, td, stack);
      auto outputs_size = n->outputs().size();
      auto iivs = pop(stack, outputs_size);
      for (size_t j = 0; j < outputs_size; j++) {
        auto& iiv = iivs[j];
        if (iiv.isTensor()) {
          auto t = iiv.toTensor();
          auto type = t.defined() ? tensorTypeInCurrentExecutionContext(t)
                                  : TensorType::get();
          tracer->outputs().at(j)->setType(type);
        }
      }
    };
```
- **EN**: This chunk continues `lock` and expands its internal control flow or data movement. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段延续了 `lock`，进一步展开其内部控制流或数据流转。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 261-276
```cpp
    opt_pn->setCallback(optional_profiler);
  }
}
} // namespace

// To trace graph we create a profile node for every one
// in a scripted graph. When a profiled node handler runs
// we insert a new traced node in a trace graph
// If the profiled node handler is called in a loop
// we will have multiple nodes.
// We also maintain the mapping between the outputs of traced
// nodes and the outputs of the node in the scripted graph.
// There are a few subtleties with tracing Ifs and Loops
// discussed above
std::shared_ptr<Graph> TraceGraph(
    const std::shared_ptr<Graph>& graph,
```
- **EN**: This chunk continues `lock` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `lock`，进一步展开其内部控制流或数据流转。

### Lines 277-289
```cpp
    Stack& stack) {
  TracingData td;
  GRAPH_DUMP("Before Inline:", graph);
  Inline(*graph);
  EliminateDeadCode(graph);
  GRAPH_DUMP("After Inline:", graph);
  auto pr = ProfilingRecord::instrumentGraph(graph);
  for (auto inp : pr->profiled_graph_->inputs()) {
    auto ni = td.traced_graph_->addInput();
    ni->copyMetadata(inp);
    ni->setType(ni->type());
    td.old_to_new_[inp] = ni;
  }
```
- **EN**: This chunk continues `lock` and expands its internal control flow or data movement. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically.
- **CN**: 这一段延续了 `lock`，进一步展开其内部控制流或数据流转。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。

### Lines 291-306
```cpp
  // Set type of the graph inputs using the inputs from the stack.
  // This needs to be done before running the interpreter because the stack
  // will only have the outputs after the run.
  for (auto i : c10::irange(stack.size())) {
    if (stack[i].isTensor()) {
      td.traced_graph_->inputs().at(i)->setType(
          tensorTypeInCurrentExecutionContext(stack[i].toTensor()));
    }
  }

  ProfilingRecord::removeProfileCounter(pr->profiled_graph_->block());
  ProfilingRecord::removeProfilingNodes(pr->profiled_graph_->block());
  insertTracingNodes(pr->profiled_graph_->block(), pr.get(), td);
  GRAPH_DUMP("Profiling Graph:", pr->profiled_graph_);
  Code cd(pr->profiled_graph_, "");
  InterpreterState is{cd};
```
- **EN**: This chunk defines `cd`, which implements a focused step in the TorchScript execution pipeline. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段定义了 `cd`，其作用是实现 TorchScript 执行流水线中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 307-315
```cpp
  is.run(stack);
  for (auto out : pr->profiled_graph_->outputs()) {
    td.traced_graph_->block()->registerOutput(td.old_to_new_.at(out));
  }

  GRAPH_DUMP("Traced graph:", td.traced_graph_);
  return td.traced_graph_;
}
} // namespace torch::jit
```
- **EN**: This chunk continues `cd` and expands its internal control flow or data movement. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `cd`，进一步展开其内部控制流或数据流转。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 末尾会返回累积结果，或把更新后的状态交还给调用方。

## Key Concepts / 关键概念

- **TorchScript runtime**
  - EN: Implements TorchScript runtime execution, specialization, operator dispatch, and static-runtime support.
  - CN: 实现 TorchScript 运行时执行、特化、算子分发以及静态运行时支持。
- **TracingData**
  - EN: `TracingData` is a central symbol declared or implemented in this file.
  - CN: `TracingData` 是本文件声明或实现的核心符号。
- **traceNode**
  - EN: `traceNode` is a central symbol declared or implemented in this file.
  - CN: `traceNode` 是本文件声明或实现的核心符号。
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

- **Internal PyTorch headers / 内部 PyTorch 头文件**: `torch/csrc/jit/ir/ir_views.h`, `torch/csrc/jit/jit_log.h`, `torch/csrc/jit/passes/dead_code_elimination.h`, `torch/csrc/jit/passes/inliner.h`, `torch/csrc/jit/runtime/graph_executor.h`, `torch/csrc/jit/runtime/interpreter.h`, `torch/csrc/jit/runtime/jit_trace.h`, `torch/csrc/jit/runtime/profiling_record.h`
- **ATen/c10 foundations / ATen/c10 基础设施**: `ATen/core/ivalue.h`, `ATen/core/symbol.h`
- **Standard library / 标准库**: `unordered_map`
- **Primary symbols in this file / 本文件核心符号**: `TracingData`, `traceNode`, `eraseAllOutputs`, `insertTracingNodes`, `createPropNodeForIfBlock`, `lock`, `traceLoopCounter`, `lv`
