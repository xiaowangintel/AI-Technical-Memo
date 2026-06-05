# operator.h — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/csrc/jit/runtime/operator.h`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Defines operator wrappers, schemas, or dispatch helpers that connect TorchScript nodes to runtime kernels.
- **Purpose (CN)**: 定义算子包装、schema 或分发辅助逻辑，将 TorchScript 节点连接到运行时内核。
## Line-by-Line Analysis / 逐行分析

### Lines 1-14
```cpp
// in memory description of all ATen Ops similar to Caffe2 schema
// once C10 exists this can be removed, or stubbed out, but we need
// it now to implement correct semantic checking for script
#pragma once

#include <ATen/core/dispatch/Dispatcher.h>
#include <ATen/core/dispatch/OperatorOptions.h>
#include <ATen/core/op_registration/op_allowlist.h>
#include <ATen/core/stack.h>
#include <c10/util/Exception.h>
#include <c10/util/overloaded.h>
#include <torch/csrc/jit/frontend/function_schema_parser.h>
#include <torch/csrc/jit/runtime/operator_options.h>
#include <torch/library.h>
```
- **EN**: This block assembles the compilation dependencies, pulling in local JIT headers such as torch/csrc/jit/frontend/function_schema_parser.h, torch/csrc/jit/runtime/operator_options.h, torch/library.h; ATen/c10 facilities such as ATen/core/dispatch/Dispatcher.h, ATen/core/dispatch/OperatorOptions.h, ATen/core/op_registration/op_allowlist.h, and 3 more. The preprocessor guard keeps declarations single-instanced when the header is included transitively.
- **CN**: 这一段组织编译依赖，引入了本地 JIT 头文件，如 torch/csrc/jit/frontend/function_schema_parser.h、torch/csrc/jit/runtime/operator_options.h、torch/library.h；ATen/c10 基础设施，如 ATen/core/dispatch/Dispatcher.h、ATen/core/dispatch/OperatorOptions.h、ATen/core/op_registration/op_allowlist.h 等共 6 项。 预处理器保护用于避免头文件在传递包含时被重复展开。

### Lines 16-28
```cpp
#include <ATen/core/function_schema.h>
#include <ATen/core/symbol.h>

#include <functional>
#include <initializer_list>
#include <memory>
#include <string>
#include <unordered_map>
#include <utility>
#include <variant>
#include <vector>

namespace torch::jit {
```
- **EN**: This block assembles the compilation dependencies, pulling in ATen/c10 facilities such as ATen/core/function_schema.h, ATen/core/symbol.h; standard-library headers such as functional, initializer_list, memory, and 5 more. The namespace declarations place the code inside torch::jit, matching the surrounding JIT subsystem.
- **CN**: 这一段组织编译依赖，引入了ATen/c10 基础设施，如 ATen/core/function_schema.h、ATen/core/symbol.h；标准库头文件，如 functional、initializer_list、memory 等共 8 项。 命名空间声明把代码放入 torch::jit 中，与周边 JIT 子系统保持一致。

### Lines 30-40
```cpp
struct Node;
using ::c10::Argument;
using ::c10::FunctionSchema;
using ::c10::Symbol;

using OperationCreator = Operation (*)(const Node*);

namespace {
const std::array<at::Tag, 1> kJitOnlyOperatorTags = {
    at::Tag::pt2_compliant_tag};
}
```
- **EN**: It introduces or extends Node, which define the primary data structures or interfaces for this portion of the file.
- **CN**: 它引入或扩展了 Node，这些类型定义了本段涉及的主要数据结构或接口。

### Lines 42-56
```cpp
/*
 * Note: JIT relies on Operator instances having static lifetime, because
 * it for example stores a non-owning FunctionSchema* pointer in the Node class,
 * which points to the function schema stored in the Operator instance.
 * Also, jit::Operator is meant to store more operator related information like
 * symbolic derivatives, which also requires them to have static lifetime
 * so that changes to symbolic derivatives are remembered.
 *
 * Currently, the JIT operator library contains a jit::Operator instance
 * with a wrapper for each c10 operator. The c10 operator library registers
 * those wrappers using listeners in register_c10_ops.cpp.
 * TODO Instead of doing it this way, we should only have pure-jit ops in
 * the jit library but have the JIT operator lookup look into the c10 library
 * too.
 */
```
- **EN**: This comment block documents assumptions, constraints, or generated-code provenance that shape the implementation below.
- **CN**: 这一段注释说明了后续实现依赖的假设、约束或生成来源。

### Lines 58-73
```cpp
// An Operator is a thin wrapper around either a pure JIT operator (e.g. prim
// ops) or a c10 operator, allowing some common operations and abstracting away
// the concrete operator nature.
struct TORCH_API Operator {
 private:
  struct C10Operator final {
    c10::OperatorHandle handle_;
    Operation op_;
  };
  struct UnparsedFunctionSchema final {
    std::string schema_string_;
    mutable std::optional<c10::AliasAnalysisKind> alias_analysis_;
  };
  struct JitOnlyOperator final {
    // The only valid transition for schema_ is from right->left, i.e.
    // when the schema gets parsed.
```
- **EN**: It introduces or extends TORCH_API, C10Operator, UnparsedFunctionSchema, and 1 more, which define the primary data structures or interfaces for this portion of the file.
- **CN**: 它引入或扩展了 TORCH_API、C10Operator、UnparsedFunctionSchema 等共 4 项，这些类型定义了本段涉及的主要数据结构或接口。

### Lines 74-89
```cpp
    mutable std::variant<FunctionSchema, UnparsedFunctionSchema> schema_;

    std::variant<Operation, OperationCreator> op_;
  };

 public:
  Operator(c10::OperatorHandle opHandle, Operation operation)
      : op_(C10Operator{std::move(opHandle), std::move(operation)}) {}

  Operator(
      std::string schema,
      Operation op,
      c10::AliasAnalysisKind alias_analysis)
      : op_(JitOnlyOperator{
            UnparsedFunctionSchema{std::move(schema), alias_analysis},
            Operation(std::move(op))}) {}
```
- **EN**: This chunk continues `JitOnlyOperator` and expands its internal control flow or data movement. Registration-style entries bind symbolic schemas or library fragments to concrete runtime behavior.
- **CN**: 这一段延续了 `JitOnlyOperator`，进一步展开其内部控制流或数据流转。 这类注册条目把符号化 schema 或库片段绑定到具体的运行时行为。

### Lines 91-105
```cpp
  Operator(
      std::string name,
      std::string overload_name,
      std::vector<Argument> arguments,
      std::vector<Argument> returns,
      Operation op,
      c10::AliasAnalysisKind alias_analysis)
      : op_(JitOnlyOperator{
            FunctionSchema(varArgSchemaWithName(
                std::move(name),
                std::move(overload_name),
                std::move(arguments),
                std::move(returns),
                alias_analysis)),
            std::move(op)}) {}
```
- **EN**: This chunk continues `JitOnlyOperator` and expands its internal control flow or data movement. Registration-style entries bind symbolic schemas or library fragments to concrete runtime behavior.
- **CN**: 这一段延续了 `JitOnlyOperator`，进一步展开其内部控制流或数据流转。 这类注册条目把符号化 schema 或库片段绑定到具体的运行时行为。

### Lines 107-122
```cpp
  Operator(
      std::string schema,
      OperationCreator op_creator,
      c10::AliasAnalysisKind alias_analysis)
      : op_(JitOnlyOperator{
            UnparsedFunctionSchema{std::move(schema), alias_analysis},
            op_creator}) {}

  // Helper constructor to register `op` to run
  // run for _every_ IR Node where n.kind() == name, regardless of arguments.
  // This is accomplished by marking the schema varargs and having no required
  // arguments.
  Operator(
      Symbol name,
      OperationCreator op_creator,
      c10::AliasAnalysisKind alias_analysis)
```
- **EN**: This chunk continues `JitOnlyOperator` and expands its internal control flow or data movement. Registration-style entries bind symbolic schemas or library fragments to concrete runtime behavior.
- **CN**: 这一段延续了 `JitOnlyOperator`，进一步展开其内部控制流或数据流转。 这类注册条目把符号化 schema 或库片段绑定到具体的运行时行为。

### Lines 123-138
```cpp
      : op_(JitOnlyOperator{
            FunctionSchema(varArgSchemaWithName(name, alias_analysis)),
            op_creator}) {}

  Operation getOperation(const Node* node = nullptr) const {
    return std::visit(
        c10::overloaded(
            [](const C10Operator& op) { return op.op_; },
            [node](const JitOnlyOperator& op) {
              return std::visit(
                  c10::overloaded(
                      [](const Operation& op) { return op; },
                      [node](const OperationCreator& op_creator) {
                        return op_creator(node);
                      }),
                  op.op_);
```
- **EN**: This chunk defines `getOperation`, which implements a focused step in the TorchScript execution pipeline. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `getOperation`，其作用是实现 TorchScript 执行流水线中的一个关键步骤。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 139-154
```cpp
            }),
        op_);
  }

  Operation getOperationForDispatchKey(c10::DispatchKey dk) const {
    // TODO: some sort of caching mechanism?
    return std::visit(
        c10::overloaded(
            [dk](const C10Operator& op) {
              return Operation([op, dk](Stack& stack) {
                op.handle_.callBoxedForDispatchKey(dk, stack);
              });
            },
            [](const JitOnlyOperator& op) {
              TORCH_CHECK(
                  false,
```
- **EN**: This chunk defines `getOperationForDispatchKey`, which implements a focused step in the TorchScript execution pipeline. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `getOperationForDispatchKey`，其作用是实现 TorchScript 执行流水线中的一个关键步骤。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 155-170
```cpp
                  "calling a JIT operator for dispatch key is not supported");
              return Operation(nullptr);
            }),
        op_);
  }

  const FunctionSchema& schema() const {
    return std::visit(
        c10::overloaded(
            [](const C10Operator& op) -> const FunctionSchema& {
              return op.handle_.schema();
            },
            [](const JitOnlyOperator& op) -> const FunctionSchema& {
              // we lazily parse schema initialized from strings so that
              // we do less work during static operator registration
              if (op.schema_.index() == 1) {
```
- **EN**: This chunk defines `schema`, which implements a focused step in the TorchScript execution pipeline. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `schema`，其作用是实现 TorchScript 执行流水线中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 171-185
```cpp
                auto& unmaterializedSchema =
                    std::get<UnparsedFunctionSchema>(op.schema_);
                FunctionSchema schema =
                    parseSchema(unmaterializedSchema.schema_string_);
                if (unmaterializedSchema.alias_analysis_.has_value()) {
                  // TODO What if it gets set later?
                  schema.setAliasAnalysis(
                      *unmaterializedSchema.alias_analysis_);
                }
                op.schema_ = std::move(schema);
              }
              return std::get<FunctionSchema>(op.schema_);
            }),
        op_);
  }
```
- **EN**: This chunk continues `schema` and expands its internal control flow or data movement. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `schema`，进一步展开其内部控制流或数据流转。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 187-199
```cpp
  c10::ArrayRef<at::Tag> getTags() const {
    return std::visit(
        c10::overloaded(
            [](const C10Operator& op) { return op.handle_.getTags(); },
            [](const JitOnlyOperator& op) {
              // JitOnlyOperators don't have an c10::OperatorHandle or a way to
              // specify tags. We're grandfathering them all into
              // pt2_compliant_tag, but for anything else, please just stop
              // using JitOnlyOperator.
              return c10::ArrayRef<at::Tag>(kJitOnlyOperatorTags);
            }),
        op_);
  }
```
- **EN**: This chunk defines `getTags`, which implements a focused step in the TorchScript execution pipeline. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `getTags`，其作用是实现 TorchScript 执行流水线中的一个关键步骤。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 201-216
```cpp
  bool isC10Op() const {
    return op_.index() == 0;
  }

  c10::AliasAnalysisKind aliasAnalysisKind() const {
    const FunctionSchema& schemaRef = schema();
    c10::AliasAnalysisKind alias_analysis = schemaRef.aliasAnalysis();

    TORCH_CHECK(
        alias_analysis == AliasAnalysisKind::FROM_SCHEMA ||
            !schemaRef.hasAnyAliasInfo(),
        "In operator registration: Tried to register operator ",
        schemaRef,
        " with aliasing information in the schema but without AliasAnalysisKind::FROM_SCHEMA.");
    return alias_analysis;
  }
```
- **EN**: This chunk defines `aliasAnalysisKind`, which implements a focused step in the TorchScript execution pipeline. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `aliasAnalysisKind`，其作用是实现 TorchScript 执行流水线中的一个关键步骤。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 218-233
```cpp
  bool hasOperation() const {
    return std::visit(
        c10::overloaded(
            [](const C10Operator&) { return true; },
            [](const JitOnlyOperator& op) { return op.op_.index() == 0; }),
        op_);
  }

 private:
  static FunctionSchema varArgSchemaWithName(
      Symbol name,
      AliasAnalysisKind alias_analysis) {
    auto result = FunctionSchema(
        name,
        "",
        {},
```
- **EN**: This chunk defines `varArgSchemaWithName`, which implements a focused step in the TorchScript execution pipeline. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `varArgSchemaWithName`，其作用是实现 TorchScript 执行流水线中的一个关键步骤。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 234-249
```cpp
        {},
        /*is_vararg*/ true,
        /*is_varret*/ true);
    result.setAliasAnalysis(alias_analysis);
    return result;
  }

  static FunctionSchema varArgSchemaWithName(
      std::string name,
      std::string overload_name,
      std::vector<Argument> arguments,
      std::vector<Argument> returns,
      AliasAnalysisKind alias_analysis) {
    auto result = FunctionSchema(
        std::move(name),
        std::move(overload_name),
```
- **EN**: This chunk defines `varArgSchemaWithName`, which implements a focused step in the TorchScript execution pipeline. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `varArgSchemaWithName`，其作用是实现 TorchScript 执行流水线中的一个关键步骤。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 250-261
```cpp
        std::move(arguments),
        std::move(returns),
        /*is_vararg*/ false,
        /*is_varret*/ false);
    result.setAliasAnalysis(alias_analysis);
    return result;
  }

  std::variant<C10Operator, JitOnlyOperator> op_;
};

TORCH_API std::string canonicalSchemaString(const FunctionSchema& schema);
```
- **EN**: This chunk declares `canonicalSchemaString`, which implements a focused step in the TorchScript execution pipeline. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段声明了 `canonicalSchemaString`，其作用是实现 TorchScript 执行流水线中的一个关键步骤。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 263-275
```cpp
TORCH_API const std::vector<std::shared_ptr<Operator>> getAllOperators();
TORCH_API const std::vector<std::shared_ptr<Operator>>& getAllOperatorsFor(
    Symbol name);
// Returns operators in the order which OpOverloadPacket resolves them.
TORCH_API std::vector<std::shared_ptr<Operator>> getAllSortedOperatorsFor(
    Symbol name);

// given a operator with an overload name, find the specific operator related to
// it, may return nullptr if no operator exists.
TORCH_API std::shared_ptr<Operator> findOperatorFor(
    const c10::OperatorName& full_name);

TORCH_API std::vector<Symbol> findSimilarOperators(Symbol input_op);
```
- **EN**: This chunk declares `findSimilarOperators`, which looks up previously defined symbols, cached plans, or registry entries.
- **CN**: 这一段声明了 `findSimilarOperators`，其作用是查找已定义的符号、缓存计划或注册表条目。

### Lines 277-289
```cpp
TORCH_API void registerOperator(Operator&& op);
TORCH_API void deregisterOperator(const FunctionSchema& schema);

// XXX: this function is meant to be used with string literals only!
TORCH_API std::shared_ptr<Operator> getOperatorForLiteral(
    const char* signature);

// Ensure the thing that registers c10 ops is defined.
// Otherwise, our registry will not have c10 ops. You can run into this
// scenario if you're querying registered ops during static init.
//
// This fn is defined in register_c10_ops.cpp
TORCH_API void ensure_c10_registerer_defined();
```
- **EN**: This chunk declares `ensure_c10_registerer_defined`, which registers schemas, operators, or passes with the surrounding runtime. Registration-style entries bind symbolic schemas or library fragments to concrete runtime behavior.
- **CN**: 这一段声明了 `ensure_c10_registerer_defined`，其作用是向周边运行时注册 schema、算子或 pass。 这类注册条目把符号化 schema 或库片段绑定到具体的运行时行为。

### Lines 291-305
```cpp
// Used to assert that unschematized operators have an analysis method written
TORCH_API bool aliasAnalysisHasSpecialCaseFor(c10::Symbol sym);

// A factory function to generate an optional operator. It has two
// instantiations depending on the template bool arg value. The arg can be a
// compile-time function for the selective op registration based on schema
// string.
template <typename Func>
std::optional<Operator> OperatorGenerator(
    const char* schema_str,
    Func&& op,
    AliasAnalysisKind alias_analysis) {
  return std::optional<Operator>(Operator(
      std::string(schema_str), std::forward<Func>(op), alias_analysis));
}
```
- **EN**: This chunk defines `OperatorGenerator`, which implements a focused step in the TorchScript execution pipeline. Registration-style entries bind symbolic schemas or library fragments to concrete runtime behavior. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `OperatorGenerator`，其作用是实现 TorchScript 执行流水线中的一个关键步骤。 这类注册条目把符号化 schema 或库片段绑定到具体的运行时行为。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 307-316
```cpp
template <typename Func>
std::optional<Operator> OperatorGenerator(
    torch::detail::SelectiveStr<true> schema_str,
    Func&& op,
    AliasAnalysisKind alias_analysis) {
  return OperatorGenerator(
      static_cast<const char*>(schema_str),
      std::forward<Func>(op),
      alias_analysis);
}
```
- **EN**: This chunk defines `OperatorGenerator`, which implements a focused step in the TorchScript execution pipeline. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `OperatorGenerator`，其作用是实现 TorchScript 执行流水线中的一个关键步骤。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 318-333
```cpp
template <typename Func>
std::optional<Operator> OperatorGenerator(
    torch::detail::SelectiveStr<false> schema_str,
    Func&& op,
    AliasAnalysisKind alias_analysis) {
  return std::nullopt;
}

template <typename Func>
std::optional<Operator> OperatorGenerator(
    const std::string name,
    const std::string overload_name,
    const std::vector<c10::Argument> arguments,
    const std::vector<c10::Argument> returns,
    Func&& op,
    AliasAnalysisKind alias_analysis) {
```
- **EN**: This chunk defines `OperatorGenerator`, which implements a focused step in the TorchScript execution pipeline. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `OperatorGenerator`，其作用是实现 TorchScript 执行流水线中的一个关键步骤。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 334-343
```cpp
  return std::optional<Operator>(Operator(
      name,
      overload_name,
      arguments,
      returns,
      std::forward<Func>(op),
      alias_analysis));
}

} // namespace torch::jit
```
- **EN**: This chunk continues `OperatorGenerator` and expands its internal control flow or data movement. Registration-style entries bind symbolic schemas or library fragments to concrete runtime behavior. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `OperatorGenerator`，进一步展开其内部控制流或数据流转。 这类注册条目把符号化 schema 或库片段绑定到具体的运行时行为。 末尾会返回累积结果，或把更新后的状态交还给调用方。

## Key Concepts / 关键概念

- **TorchScript runtime**
  - EN: Implements TorchScript runtime execution, specialization, operator dispatch, and static-runtime support.
  - CN: 实现 TorchScript 运行时执行、特化、算子分发以及静态运行时支持。
- **Node**
  - EN: `Node` is a central symbol declared or implemented in this file.
  - CN: `Node` 是本文件声明或实现的核心符号。
- **TORCH_API**
  - EN: `TORCH_API` is a central symbol declared or implemented in this file.
  - CN: `TORCH_API` 是本文件声明或实现的核心符号。
- **Operator dispatch**
  - EN: Bridges symbolic operator names or schemas to concrete C++ implementations.
  - CN: 把符号化算子名或 schema 连接到具体的 C++ 实现。
- **Static analysis**
  - EN: Infers structural properties that later optimizations depend on.
  - CN: 推断后续优化所依赖的结构属性。
## Dependencies / 依赖关系

- **Internal PyTorch headers / 内部 PyTorch 头文件**: `torch/csrc/jit/frontend/function_schema_parser.h`, `torch/csrc/jit/runtime/operator_options.h`, `torch/library.h`
- **ATen/c10 foundations / ATen/c10 基础设施**: `ATen/core/dispatch/Dispatcher.h`, `ATen/core/dispatch/OperatorOptions.h`, `ATen/core/op_registration/op_allowlist.h`, `ATen/core/stack.h`, `c10/util/Exception.h`, `c10/util/overloaded.h`, `ATen/core/function_schema.h`, `ATen/core/symbol.h`
- **Standard library / 标准库**: `functional`, `initializer_list`, `memory`, `string`, `unordered_map`, `utility`, `variant`, `vector`
- **Primary symbols in this file / 本文件核心符号**: `Node`, `TORCH_API`, `C10Operator`, `UnparsedFunctionSchema`, `JitOnlyOperator`, `getOperation`, `getOperationForDispatchKey`, `schema`
- **Note / 说明**: 19 direct includes were detected; only the first few are listed above for readability. / 检测到 19 个直接包含，为便于阅读这里只列出前若干项。
