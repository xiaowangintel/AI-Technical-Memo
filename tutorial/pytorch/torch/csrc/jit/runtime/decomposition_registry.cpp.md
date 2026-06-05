# decomposition_registry.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/csrc/jit/runtime/decomposition_registry.cpp`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Defines graph-executor interfaces and plan-selection logic for optimized TorchScript execution.
- **Purpose (CN)**: 定义图执行器接口以及用于优化 TorchScript 执行的计划选择逻辑。
## Line-by-Line Analysis / 逐行分析

### Lines 1-15
```cpp
#include <torch/csrc/jit/jit_log.h>
#include <torch/csrc/jit/passes/constant_propagation.h>
#include <torch/csrc/jit/passes/peephole.h>
#include <torch/csrc/jit/runtime/decomposition_registry.h>
#include <torch/csrc/jit/runtime/decomposition_registry_util.h>
#include <torch/csrc/jit/runtime/operator.h>
#include <torch/csrc/jit/serialization/import_source.h>

#include <c10/util/Exception.h>
#include <torch/csrc/autograd/jit_decomp_interface.h>
#include <torch/csrc/jit/ir/ir.h>
#include <torch/csrc/jit/passes/inliner.h>
#include <torch/csrc/jit/runtime/graph_executor.h>
#include <memory>
#include <unordered_map>
```
- **EN**: This block assembles the compilation dependencies, pulling in local JIT headers such as torch/csrc/jit/jit_log.h, torch/csrc/jit/passes/constant_propagation.h, torch/csrc/jit/passes/peephole.h, and 8 more; ATen/c10 facilities such as c10/util/Exception.h; standard-library headers such as memory, unordered_map.
- **CN**: 这一段组织编译依赖，引入了本地 JIT 头文件，如 torch/csrc/jit/jit_log.h、torch/csrc/jit/passes/constant_propagation.h、torch/csrc/jit/passes/peephole.h 等共 11 项；ATen/c10 基础设施，如 c10/util/Exception.h；标准库头文件，如 memory、unordered_map。

### Lines 17-30
```cpp
namespace torch::jit {
namespace {
std::mutex lock;

// CompilationUnit that holds all these Functions and keeps them alive.
auto compilation_unit = std::make_shared<CompilationUnit>();
std::unordered_map<const FunctionSchema*, std::shared_ptr<Graph>>
    schema_to_decomposition;

// Holds User-Registered Functions and keeps them alive
std::unordered_map<const FunctionSchema*, std::unique_ptr<Function>>
    user_registered_funcs;

std::unordered_map<const FunctionSchema*, Function*> schema_to_function;
```
- **EN**: The namespace declarations place the code inside torch::jit, matching the surrounding JIT subsystem.
- **CN**: 命名空间声明把代码放入 torch::jit 中，与周边 JIT 子系统保持一致。

### Lines 32-46
```cpp
void loadModule(const CompilationUnit& module) {
  const auto& mappings = GetDecompositionMapping().getAllKeysAndValues();
  for (const auto& pair : mappings) {
    const FunctionSchema* schema = &pair.first->schema();
    const std::string& decomposition_function_name = pair.second;

    Function& decomposition_function =
        module.get_function(decomposition_function_name);
    std::shared_ptr<Graph> graph =
        toGraphFunction(decomposition_function).graph();

    schema_to_function[schema] = &decomposition_function;
    schema_to_decomposition[schema] = graph;
  }
}
```
- **EN**: This chunk defines `loadModule`, which implements a focused step in the TorchScript execution pipeline. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically.
- **CN**: 这一段定义了 `loadModule`，其作用是实现 TorchScript 执行流水线中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。

### Lines 48-63
```cpp
void loadDecompositionFunctions() {
  std::lock_guard<std::mutex> guard(lock);
  if (!schema_to_decomposition.empty()) {
    return;
  }

  auto src = std::make_shared<Source>(GetSerializedDecompositions());
  std::stringstream ss;
  std::vector<at::IValue> constantTable;
  auto resolver = std::make_shared<SourceImporterImpl>(
      compilation_unit,
      &constantTable,
      [&](const std::string& name) -> std::shared_ptr<Source> { return src; },
      1);
  compilation_unit->define(
      std::nullopt, GetSerializedDecompositions(), resolver, nullptr);
```
- **EN**: This chunk defines `guard`, which implements a focused step in the TorchScript execution pipeline. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `guard`，其作用是实现 TorchScript 执行流水线中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 64-79
```cpp
  loadModule(*compilation_unit);
}

} // anonymous namespace

static void DecomposeOp(Node* n) {
  auto schema = n->maybeSchema();
  if (!schema) {
    return;
  }
  auto decomposition = GetDecomposition(n->schema());
  if (!decomposition) {
    return;
  }
  WithInsertPoint guard(n);
  auto outputs = insertGraph(*n->owningGraph(), **decomposition, n->inputs());
```
- **EN**: This chunk defines `guard`, which implements a focused step in the TorchScript execution pipeline. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `guard`，其作用是实现 TorchScript 执行流水线中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 80-95
```cpp
  TORCH_INTERNAL_ASSERT(outputs.size() == n->outputs().size());
  for (size_t i : c10::irange(outputs.size())) {
    n->outputs().at(i)->replaceAllUsesWith(outputs[i]);
  }
  n->destroy();
}

static void RunDecompositions(Block* block) {
  for (auto it = block->nodes().begin(); it != block->nodes().end();) {
    Node* n = *it;
    it++; // advance iterator bc the current node may be destroyed
    for (Block* b : n->blocks()) {
      RunDecompositions(b);
    }
    DecomposeOp(n);
  }
```
- **EN**: This chunk defines `RunDecompositions`, which executes prepared logic against runtime values or IR state. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Assertions and error paths make invalid states fail early instead of silently corrupting later stages.
- **CN**: 这一段定义了 `RunDecompositions`，其作用是针对运行时数值或 IR 状态执行准备好的逻辑。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。

### Lines 96-111
```cpp
}

void RunDecompositions(std::shared_ptr<Graph> g) {
  RunDecompositions(g->block());
  for ([[maybe_unused]] const auto _ : c10::irange(2)) {
    PeepholeOptimize(g, /*disable_shape_peephole*/ true);
    ConstantPropagation(g);
  }
}

std::optional<std::shared_ptr<Graph>> GetDecomposition(
    const FunctionSchema& schema) {
  loadDecompositionFunctions();
  GRAPH_DEBUG("Trying to find schema: ", schema);
  auto cache_it = schema_to_decomposition.find(&schema);
  if (cache_it != schema_to_decomposition.end()) {
```
- **EN**: This chunk defines `GetDecomposition`, which implements a focused step in the TorchScript execution pipeline. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段定义了 `GetDecomposition`，其作用是实现 TorchScript 执行流水线中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 112-127
```cpp
    return cache_it->second;
  }
  GRAPH_DEBUG("Could not find schema: ", schema);

  return std::nullopt;
}

std::optional<GraphFunction*> GetDecompositionFunction(
    const FunctionSchema& schema) {
  loadDecompositionFunctions();
  auto cache_it = schema_to_function.find(&schema);
  GRAPH_DEBUG("Trying to find schema: ", schema);
  if (cache_it == schema_to_function.end()) {
    GRAPH_DEBUG("Could not find schema: ", schema);
    return std::nullopt;
  }
```
- **EN**: This chunk defines `GetDecompositionFunction`, which implements a focused step in the TorchScript execution pipeline. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `GetDecompositionFunction`，其作用是实现 TorchScript 执行流水线中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 128-143
```cpp
  auto& func = toGraphFunction(*cache_it->second);
  // Simple Executor:
  // To allow decomposition to run on tensor subclasses such as batched tensors,
  // we set decomposition execution to use the simple executor so that
  // optimizations that do not compose with arbitrary subclasses (such as
  // fusion) do not run
  func._set_initial_executor_execution_mode(ExecutorExecutionMode::SIMPLE);
  return &func;
}

// Decomposition registers a Graph so that we can initialize a GraphFunction
// that will run with Simple Executor
void RegisterDecomposition(
    const FunctionSchema& schema,
    std::shared_ptr<Graph> g) {
  loadDecompositionFunctions();
```
- **EN**: This chunk defines `RegisterDecomposition`, which registers schemas, operators, or passes with the surrounding runtime. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `RegisterDecomposition`，其作用是向周边运行时注册 schema、算子或 pass。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 144-157
```cpp
  std::lock_guard<std::mutex> guard(lock);
  Inline(*g);
  for (const auto i : c10::irange(2)) {
    (void)i; // Suppress unused variable warning
    PeepholeOptimize(g);
    ConstantPropagationImmutableTypes(g);
  }

  auto new_func = std::make_unique<GraphFunction>(
      schema.name(), g, nullptr, ExecutorExecutionMode::SIMPLE);
  user_registered_funcs.emplace(&schema, std::move(new_func));
  schema_to_function[&schema] = user_registered_funcs[&schema].get();
  schema_to_decomposition[&schema] = g;
}
```
- **EN**: This chunk defines `guard`, which implements a focused step in the TorchScript execution pipeline. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically.
- **CN**: 这一段定义了 `guard`，其作用是实现 TorchScript 执行流水线中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。

### Lines 159-168
```cpp
// see NOTE: [Jit Decomposition Interface]
struct JitDecomp final : torch::autograd::impl::JitDecompInterface {
  bool has_jit_decomposition(const c10::FunctionSchema& schema) const override;
  void run_jit_decomposition(
      const c10::OperatorHandle& op,
      torch::jit::Stack* stack) const override;
};

static JitDecomp jitDecomp;
static torch::autograd::impl::JitDecompRegisterer registerJitDecomp(&jitDecomp);
```
- **EN**: It introduces or extends JitDecomp, which define the primary data structures or interfaces for this portion of the file. This chunk defines `registerJitDecomp`, which registers schemas, operators, or passes with the surrounding runtime.
- **CN**: 它引入或扩展了 JitDecomp，这些类型定义了本段涉及的主要数据结构或接口。 这一段定义了 `registerJitDecomp`，其作用是向周边运行时注册 schema、算子或 pass。

### Lines 170-185
```cpp
void JitDecomp::run_jit_decomposition(
    const c10::OperatorHandle& op,
    torch::jit::Stack* stack) const {
  ::torch::jit::run_jit_decomposition(op, stack);
}

bool JitDecomp::has_jit_decomposition(const FunctionSchema& schema) const {
  return ::torch::jit::has_jit_decomposition(schema);
}

void run_jit_decomposition(
    const c10::OperatorHandle& op,
    torch::jit::Stack* stack) {
  const auto& schema = op.schema();
  // TODO: templatize based on op and keep static trace_exec
  auto* trace_exec = torch::jit::GetDecompositionExecutor(schema);
```
- **EN**: This chunk defines `has_jit_decomposition`, which implements a focused step in the TorchScript execution pipeline. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `has_jit_decomposition`，其作用是实现 TorchScript 执行流水线中的一个关键步骤。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 186-198
```cpp
  trace_exec->run((*stack));
  if (stack->back().isTuple()) {
    at::IValue tup = std::move(stack->back());
    stack->pop_back();
    for (const auto& elem : tup.toTuple()->elements()) {
      stack->push_back(elem);
    }
  }
}

bool has_jit_decomposition(const FunctionSchema& schema) {
  return GetDecompositionFunction(schema).has_value();
}
```
- **EN**: This chunk defines `has_jit_decomposition`, which implements a focused step in the TorchScript execution pipeline. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `has_jit_decomposition`，其作用是实现 TorchScript 执行流水线中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 200-211
```cpp
Function* GetDecompositionExecutor(const FunctionSchema& schema) {
  auto maybe_func = GetDecompositionFunction(schema);
  TORCH_INTERNAL_ASSERT(maybe_func);
  return *maybe_func;
}

Function* GetDecompositionExecutor(const char* schema_literal) {
  auto& schema = getOperatorForLiteral(schema_literal)->schema();
  return GetDecompositionExecutor(schema);
}

} // namespace torch::jit
```
- **EN**: This chunk defines `GetDecompositionExecutor`, which implements a focused step in the TorchScript execution pipeline. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `GetDecompositionExecutor`，其作用是实现 TorchScript 执行流水线中的一个关键步骤。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

## Key Concepts / 关键概念

- **TorchScript runtime**
  - EN: Implements TorchScript runtime execution, specialization, operator dispatch, and static-runtime support.
  - CN: 实现 TorchScript 运行时执行、特化、算子分发以及静态运行时支持。
- **JitDecomp**
  - EN: `JitDecomp` is a central symbol declared or implemented in this file.
  - CN: `JitDecomp` 是本文件声明或实现的核心符号。
- **loadModule**
  - EN: `loadModule` is a central symbol declared or implemented in this file.
  - CN: `loadModule` 是本文件声明或实现的核心符号。
- **Graph IR**
  - EN: Works on graph, node, and block structures that represent TorchScript programs.
  - CN: 围绕表示 TorchScript 程序的 graph、node 与 block 结构工作。
- **IValue data model**
  - EN: Moves values through generic boxed runtime containers used by TorchScript.
  - CN: 通过 TorchScript 使用的通用装箱运行时容器传递数值。
- **Operator dispatch**
  - EN: Bridges symbolic operator names or schemas to concrete C++ implementations.
  - CN: 把符号化算子名或 schema 连接到具体的 C++ 实现。
## Dependencies / 依赖关系

- **Internal PyTorch headers / 内部 PyTorch 头文件**: `torch/csrc/jit/jit_log.h`, `torch/csrc/jit/passes/constant_propagation.h`, `torch/csrc/jit/passes/peephole.h`, `torch/csrc/jit/runtime/decomposition_registry.h`, `torch/csrc/jit/runtime/decomposition_registry_util.h`, `torch/csrc/jit/runtime/operator.h`, `torch/csrc/jit/serialization/import_source.h`, `torch/csrc/autograd/jit_decomp_interface.h`, `torch/csrc/jit/ir/ir.h`, `torch/csrc/jit/passes/inliner.h`, `torch/csrc/jit/runtime/graph_executor.h`
- **ATen/c10 foundations / ATen/c10 基础设施**: `c10/util/Exception.h`
- **Standard library / 标准库**: `memory`, `unordered_map`
- **Primary symbols in this file / 本文件核心符号**: `JitDecomp`, `loadModule`, `loadDecompositionFunctions`, `guard`, `DecomposeOp`, `RunDecompositions`, `GetDecomposition`, `GetDecompositionFunction`
- **Note / 说明**: 14 direct includes were detected; only the first few are listed above for readability. / 检测到 14 个直接包含，为便于阅读这里只列出前若干项。
