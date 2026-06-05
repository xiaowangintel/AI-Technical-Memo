# preprocess_graph.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/csrc/jit/runtime/interpreter/preprocess_graph.cpp`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements the TorchScript interpreter, bytecode execution, frame management, and graph preprocessing.
- **Purpose (CN)**: 实现 TorchScript 解释器、字节码执行、栈帧管理以及图预处理。
## Line-by-Line Analysis / 逐行分析

### Lines 1-16
```cpp
#include <torch/csrc/jit/runtime/interpreter/preprocess_graph.h>

#include <torch/csrc/jit/frontend/schema_matching.h>
#include <torch/csrc/jit/runtime/interpreter/can_emit_inline.h>

namespace torch::jit::interpreter {

namespace {

// Insert explicit prim::MethodCall nodes after prim::Enter nodes
// to actually call __enter__ on the object. All prim::Enter does
// is push the object onto the stack of currently entered objects.
// This is necessary because emitting two instructions for a
// prim::Enter nodes (one ENTER to push onto the entered objects
// stack and one CALL to call __enter__) does not work; the
// accounting that determines when to move a value out of a register
```
- **EN**: This block assembles the compilation dependencies, pulling in local JIT headers such as torch/csrc/jit/runtime/interpreter/preprocess_graph.h, torch/csrc/jit/frontend/schema_matching.h, torch/csrc/jit/runtime/interpreter/can_emit_inline.h. The namespace declarations place the code inside torch::jit::interpreter, matching the surrounding JIT subsystem.
- **CN**: 这一段组织编译依赖，引入了本地 JIT 头文件，如 torch/csrc/jit/runtime/interpreter/preprocess_graph.h、torch/csrc/jit/frontend/schema_matching.h、torch/csrc/jit/runtime/interpreter/can_emit_inline.h。 命名空间声明把代码放入 torch::jit::interpreter 中，与周边 JIT 子系统保持一致。

### Lines 17-27
```cpp
// is based on the number of uses it has in the IR.
void insertEnterMethodCalls(Graph& g) {
  std::vector<Block*> block_queue;
  std::vector<Node*> enter_nodes;
  block_queue.emplace_back(g.block());

  // Traverse the graph while drilling down into blocks belonging to
  // a node and add all encountered prim::Enter nodes to enter_nodes.
  while (!block_queue.empty()) {
    Block* block = block_queue.back();
    block_queue.pop_back();
```
- **EN**: This chunk defines `insertEnterMethodCalls`, which implements a focused step in the TorchScript execution pipeline. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically.
- **CN**: 这一段定义了 `insertEnterMethodCalls`，其作用是实现 TorchScript 执行流水线中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。

### Lines 29-39
```cpp
    for (auto node : block->nodes()) {
      if (node->kind() == prim::Enter) {
        enter_nodes.emplace_back(node);
        continue;
      }

      for (auto& node_block : node->blocks()) {
        block_queue.emplace_back(node_block);
      }
    }
  }
```
- **EN**: This chunk continues `insertEnterMethodCalls` and expands its internal control flow or data movement. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段延续了 `insertEnterMethodCalls`，进一步展开其内部控制流或数据流转。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 41-51
```cpp
  // For each prim::Enter, emit a prim::MethodCall after it that actually
  // calls __enter__ on the object.
  for (auto& enter : enter_nodes) {
    auto cls = enter->input(0)->type()->expect<ClassType>();

    MatchedSchema enter_matched_schema = matchSchema(
        cls->findMethod("__enter__")->getSchema(),
        enter->input(0)->node()->sourceRange(),
        g,
        {enter->input(0)},
        {});
```
- **EN**: This chunk continues `insertEnterMethodCalls` and expands its internal control flow or data movement. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically.
- **CN**: 这一段延续了 `insertEnterMethodCalls`，进一步展开其内部控制流或数据流转。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。

### Lines 53-68
```cpp
    Node* call = g.insertMethodCall("__enter__", enter_matched_schema)->node();
    call->moveAfter(enter);
    enter->replaceAllUsesWith(call);
  }
}

// insert Drop nodes to kill references for anything unused:
// this can happen in a few places, e.g. when a node returns
// many values but only one is used
// a, b = foo()
// return a
void dropUnused(Block* b) {
  auto createDropIfUnused = [&](ArrayRef<Value*> values) -> Node* {
    std::vector<Value*> to_drop;
    for (auto v : values) {
      if (v->uses().empty() && v->node()->kind() != prim::Constant) {
```
- **EN**: This chunk defines `dropUnused`, which implements a focused step in the TorchScript execution pipeline. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段定义了 `dropUnused`，其作用是实现 TorchScript 执行流水线中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 69-84
```cpp
        to_drop.push_back(v);
      }
    }
    if (to_drop.empty()) {
      return nullptr;
    }
    return b->owningGraph()->create(prim::Drop, to_drop, 0);
  };

  if (auto d = createDropIfUnused(b->inputs())) {
    b->prependNode(d);
  }
  for (auto n : b->nodes()) {
    if (auto d = createDropIfUnused(n->outputs())) {
      d->insertAfter(n);
    }
```
- **EN**: This chunk continues `dropUnused` and expands its internal control flow or data movement. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `dropUnused`，进一步展开其内部控制流或数据流转。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 85-100
```cpp
    for (auto block : n->blocks()) {
      dropUnused(block);
    }
  }
}

// ensure every value has a final use in the same block where it is defined.
// This already true for most nodes. The exceptions are:
// 1. A value that is unused.
// 2. A value whose last use is nested in some control flow.
// For (1) we simply add a prim::Drop node that uses the value right after
// it is defined. For (2), we insert a prim::Drop right after the control
// flow node where the last use occurs
void insertLastUses(Graph& g) {
  // struct to share common data structures
  struct InsertLastUses {
```
- **EN**: It introduces or extends InsertLastUses, which define the primary data structures or interfaces for this portion of the file. This chunk defines `insertLastUses`, which implements a focused step in the TorchScript execution pipeline. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically.
- **CN**: 它引入或扩展了 InsertLastUses，这些类型定义了本段涉及的主要数据结构或接口。 这一段定义了 `insertLastUses`，其作用是实现 TorchScript 执行流水线中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。

### Lines 101-116
```cpp
    Graph& graph;
    // have we seen this value, yet, if not, it is the last use of the value
    std::unordered_set<Value*> seen;

    // A map from an If or Loop node to the optional Drop block that
    // occurs directly after it to release any tensors that go out of scope
    // when the If/Loop exits. These are created and inserted on demand.
    std::unordered_map<Node*, Node*> drop_for_node;

    explicit InsertLastUses(Graph& g) : graph(g) {
      scanBlock(graph.block());
    }
    void scanBlock(Block* b) {
      scanNode(b->return_node());
      for (auto n : b->nodes().reverse()) {
        scanNode(n);
```
- **EN**: This chunk defines `scanBlock`, which walks input structures and records the metadata needed later. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically.
- **CN**: 这一段定义了 `scanBlock`，其作用是遍历输入结构并记录后续所需的元数据。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。

### Lines 117-132
```cpp
      }
    }
    void scanNode(Node* n) {
      for (auto b : n->blocks()) {
        scanBlock(b);
      }
      // scan backwards so if a value is used twice in the list then it is a
      // move
      for (size_t i = n->inputs().size(); i > 0; --i) {
        scanUse(n, i - 1);
      }
    }
    void scanUse(Node* n, size_t i) {
      auto v = n->inputs()[i];
      auto inserted = seen.insert(v).second;
      if (!inserted) {
```
- **EN**: This chunk defines `scanUse`, which walks input structures and records the metadata needed later. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段定义了 `scanUse`，其作用是遍历输入结构并记录后续所需的元数据。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 133-148
```cpp
        return;
      }

      // the last use of v may be in a nested block of an If or Loop statement
      // find the node 'same_depth_node' at the same depth as the definition of
      // v, and consider that node to be the last use of v. This ensures we do
      // not delete nodes in nested scopes that may be executed multiple times
      // and that nodes used on one side of an if
      // but not the other get deleted regardless of the branch
      // e.g.
      // a = 4
      // while <...>:
      //   y = a + a
      // drop(a)
      // In other words, we find the first program point for v that
      // _reverse_ dominates the definition of v, and add a drop point there.
```
- **EN**: This chunk continues `scanUse` and expands its internal control flow or data movement. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `scanUse`，进一步展开其内部控制流或数据流转。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 149-163
```cpp
      Node* same_depth_node = findOwnerInBlock(n, v->node()->owningBlock());
      AT_ASSERT(
          same_depth_node); // failure means v is not in scope for n, use lint!

      // In the case where v and n are in the same block,
      // we have a legit final use already.
      if (same_depth_node == n) {
        return;
      }

      // in the case where the use is nested in a block
      // add a Drop node after that block which will drop 'v'.
      addToDropIfNotExists(
          findOrCreateDropInstructionForNode(same_depth_node), v);
    }
```
- **EN**: This chunk continues `scanUse` and expands its internal control flow or data movement. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `scanUse`，进一步展开其内部控制流或数据流转。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 165-176
```cpp
    // finds the node in block 'block' that contains in 'n'
    // or nullptr if no such node exists, e.g.:
    // n0: a = 4
    // n1: if <cond>:
    // n2:    b = a + a
    // findOwnerInBlock(n2, n0.block()) == n1
    Node* findOwnerInBlock(Node* n, Block* block) {
      while (n != nullptr && block != n->owningBlock()) {
        n = n->owningBlock()->owningNode();
      }
      return n;
    }
```
- **EN**: This chunk defines `findOwnerInBlock`, which looks up previously defined symbols, cached plans, or registry entries. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `findOwnerInBlock`，其作用是查找已定义的符号、缓存计划或注册表条目。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 178-193
```cpp
    Node* findOrCreateDropInstructionForNode(Node* n) {
      auto it = drop_for_node.find(n);
      if (it == drop_for_node.end()) {
        auto drop_node = graph.create(prim::Drop, 0);
        drop_node->insertAfter(n);
        it = drop_for_node.emplace(n, drop_node).first;
      }
      return it->second;
    }

    void addToDropIfNotExists(Node* drop, Value* v) {
      if (v->node()->kind() == prim::Constant) {
        return;
      }
      for (auto i : drop->inputs()) {
        // we already accounted for this use
```
- **EN**: This chunk defines `addToDropIfNotExists`, which implements a focused step in the TorchScript execution pipeline. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `addToDropIfNotExists`，其作用是实现 TorchScript 执行流水线中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 194-205
```cpp
        if (i == v) {
          return;
        }
      }
      drop->addInput(v);
    }
  };

  InsertLastUses ilu(g);
}

} // namespace
```
- **EN**: This chunk defines `ilu`, which implements a focused step in the TorchScript execution pipeline. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `ilu`，其作用是实现 TorchScript 执行流水线中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 207-214
```cpp
PreprocessGraph::PreprocessGraph(Graph& g) : graph(g.copy()) {
  insertEnterMethodCalls(*graph);
  dropUnused(graph->block());
  // fill in move_flags by scanning blocks;
  insertLastUses(*graph);
  can_emit_inline = std::move(CanEmitInline(*graph).can_emit_inline_);
}
} // namespace torch::jit::interpreter
```
- **EN**: This chunk continues `ilu` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `ilu`，进一步展开其内部控制流或数据流转。

## Key Concepts / 关键概念

- **TorchScript runtime**
  - EN: Implements TorchScript runtime execution, specialization, operator dispatch, and static-runtime support.
  - CN: 实现 TorchScript 运行时执行、特化、算子分发以及静态运行时支持。
- **InsertLastUses**
  - EN: `InsertLastUses` is a central symbol declared or implemented in this file.
  - CN: `InsertLastUses` 是本文件声明或实现的核心符号。
- **insertEnterMethodCalls**
  - EN: `insertEnterMethodCalls` is a central symbol declared or implemented in this file.
  - CN: `insertEnterMethodCalls` 是本文件声明或实现的核心符号。
- **Graph IR**
  - EN: Works on graph, node, and block structures that represent TorchScript programs.
  - CN: 围绕表示 TorchScript 程序的 graph、node 与 block 结构工作。
## Dependencies / 依赖关系

- **Internal PyTorch headers / 内部 PyTorch 头文件**: `torch/csrc/jit/runtime/interpreter/preprocess_graph.h`, `torch/csrc/jit/frontend/schema_matching.h`, `torch/csrc/jit/runtime/interpreter/can_emit_inline.h`
- **Primary symbols in this file / 本文件核心符号**: `InsertLastUses`, `insertEnterMethodCalls`, `dropUnused`, `insertLastUses`, `scanBlock`, `scanNode`, `scanUse`, `findOwnerInBlock`
