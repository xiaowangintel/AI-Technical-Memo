# reduction.h — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/csrc/jit/tensorexpr/reduction.h`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Defines operator wrappers, schemas, or dispatch helpers that connect TorchScript nodes to runtime kernels.
- **Purpose (CN)**: 定义算子包装、schema 或分发辅助逻辑，将 TorchScript 节点连接到运行时内核。
## Line-by-Line Analysis / 逐行分析

### Lines 1-13
```cpp
#pragma once

#include <torch/csrc/jit/tensorexpr/expr.h>
#include <torch/csrc/jit/tensorexpr/ir.h>
#include <torch/csrc/jit/tensorexpr/ir_printer.h>
#include <torch/csrc/jit/tensorexpr/stmt.h>
#include <torch/csrc/jit/tensorexpr/types.h>

#include <functional>
#include <utility>
#include <vector>

namespace torch::jit::tensorexpr {
```
- **EN**: This block assembles the compilation dependencies, pulling in local JIT headers such as torch/csrc/jit/tensorexpr/expr.h, torch/csrc/jit/tensorexpr/ir.h, torch/csrc/jit/tensorexpr/ir_printer.h, and 2 more; standard-library headers such as functional, utility, vector. The preprocessor guard keeps declarations single-instanced when the header is included transitively. The namespace declarations place the code inside torch::jit::tensorexpr, matching the surrounding JIT subsystem.
- **CN**: 这一段组织编译依赖，引入了本地 JIT 头文件，如 torch/csrc/jit/tensorexpr/expr.h、torch/csrc/jit/tensorexpr/ir.h、torch/csrc/jit/tensorexpr/ir_printer.h 等共 5 项；标准库头文件，如 functional、utility、vector。 预处理器保护用于避免头文件在传递包含时被重复展开。 命名空间声明把代码放入 torch::jit::tensorexpr 中，与周边 JIT 子系统保持一致。

### Lines 15-26
```cpp
using ParameterList = const std::vector<VarHandle>;
using ReduceInteraction = std::function<ExprHandle(ExprHandle, ExprHandle)>;

// A Reducer is a user interface describing a particular reduction
// operation. It has three components: An initialization value, a way of
// interacting each value with the accumulation, and a method for obtaining the
// current value to be reduced. It is materialized into a ReduceOp when loop
// variables are known.
class TORCH_API Reducer {
 public:
  Reducer(ExprHandle init, ReduceInteraction& interaction)
      : init_(init.node()), interaction_(interaction) {}
```
- **EN**: It introduces or extends TORCH_API, which define the primary data structures or interfaces for this portion of the file.
- **CN**: 它引入或扩展了 TORCH_API，这些类型定义了本段涉及的主要数据结构或接口。

### Lines 28-40
```cpp
  template <typename RI>
  Reducer(ExprHandle init, RI interaction)
      : init_(init.node()), interaction_(std::move(interaction)) {}

  ExprPtr initializer() const {
    return init_;
  }

  ExprHandle operator()(
      const BufHandle& result_buf,
      ExprHandle body,
      const std::vector<ExprHandle>& output,
      const std::vector<VarHandle>& inner) const;
```
- **EN**: This chunk defines `initializer`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Registration-style entries bind symbolic schemas or library fragments to concrete runtime behavior. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `initializer`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 这类注册条目把符号化 schema 或库片段绑定到具体的运行时行为。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 42-53
```cpp
  ReduceOpPtr operator()(
      const BufPtr& result_buf,
      ExprPtr body,
      const std::vector<ExprPtr>& output,
      const std::vector<VarPtr>& inner) const;

  ExprHandle operator()(
      const BufHandle& result_buf,
      BufHandle acc_buf,
      const ExprHandle& body,
      const std::vector<ExprHandle>& output,
      const std::vector<VarHandle>& inner) const;
```
- **EN**: This chunk continues `initializer` and expands its internal control flow or data movement. Registration-style entries bind symbolic schemas or library fragments to concrete runtime behavior.
- **CN**: 这一段延续了 `initializer`，进一步展开其内部控制流或数据流转。 这类注册条目把符号化 schema 或库片段绑定到具体的运行时行为。

### Lines 55-67
```cpp
  // Polymorphic handling of Body functions with a variety of parameters.
  static ExprHandle getReduceBody(
      const std::function<ExprHandle(ParameterList&)>& func,
      const std::vector<VarHandle>& vars) {
    return func(vars);
  }

  static ExprHandle getReduceBody(
      const std::function<ExprHandle(const VarHandle&)>& func,
      const std::vector<VarHandle>& vars) {
    if (vars.size() != 1) {
      throw malformed_input("mismatch between reduce body and arg size (1)");
    }
```
- **EN**: This chunk defines `malformed_input`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `malformed_input`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 69-79
```cpp
    return func(vars[0]);
  }

  static ExprHandle getReduceBody(
      const std::function<ExprHandle(const VarHandle&, const VarHandle&)>& func,
      const std::vector<VarHandle>& vars) {
    if (vars.size() != 2) {
      throw malformed_input("mismatch between reduce body and arg size (2)");
    }
    return func(vars[0], vars[1]);
  }
```
- **EN**: This chunk defines `malformed_input`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `malformed_input`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 81-90
```cpp
  static ExprHandle getReduceBody(
      const std::function<
          ExprHandle(const VarHandle&, const VarHandle&, const VarHandle&)>&
          func,
      const std::vector<VarHandle>& vars) {
    if (vars.size() != 3) {
      throw malformed_input("mismatch between reduce body and arg size (3)");
    }
    return func(vars[0], vars[1], vars[2]);
  }
```
- **EN**: This chunk defines `malformed_input`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `malformed_input`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 92-103
```cpp
  static ExprHandle getReduceBody(
      const std::function<ExprHandle(
          const VarHandle&,
          const VarHandle&,
          const VarHandle&,
          const VarHandle&)>& func,
      const std::vector<VarHandle>& vars) {
    if (vars.size() != 4) {
      throw malformed_input("mismatch between reduce body and arg size (4)");
    }
    return func(vars[0], vars[1], vars[2], vars[3]);
  }
```
- **EN**: This chunk defines `malformed_input`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `malformed_input`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 105-120
```cpp
  // Completes the reduction operator by applying the interaction function to
  // the accumulation and the body expression.
  static ExprPtr complete(
      const BufPtr& accumulator,
      const ReduceInteraction& interaction,
      ExprHandle body,
      const std::vector<ExprPtr>& output_args,
      const std::vector<VarPtr>& reduce_args) {
    ExprHandle accum =
        ExprHandle(alloc<Load>(body.dtype(), accumulator, output_args));
    auto e = interaction(std::move(accum), std::move(body));
    return e.node();
  }
  static ExprHandle complete(
      const BufHandle& accumulator,
      const ReduceInteraction& interaction,
```
- **EN**: This chunk defines `complete`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `complete`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 121-132
```cpp
      ExprHandle body,
      const std::vector<ExprHandle>& output_args,
      const std::vector<VarHandle>& reduce_args) {
    ExprHandle accum = Load::make(body.dtype(), accumulator, output_args);
    auto e = interaction(std::move(accum), std::move(body));
    return e;
  }

 private:
  ExprPtr init_;
  ReduceInteraction interaction_;
};
```
- **EN**: This chunk continues `complete` and expands its internal control flow or data movement. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `complete`，进一步展开其内部控制流或数据流转。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 134-149
```cpp
// An expression representing a Reduction operation (e.g. Sum, Max) broken into
// it's component parts: initialization, accumulation var, acquisition of value
// to be reduced and interaction.
//
// This is intended to be expanded in the loopnest and not make it to codegen.
class TORCH_API ReduceOp : public ExprNode<ReduceOp> {
 public:
  ReduceOp(
      const ExprPtr& body,
      std::vector<VarPtr> reduce_args,
      Reducer reducer)
      : ExprNodeBase(body->dtype()),
        body_(body),
        reduce_args_(std::move(reduce_args)),
        reducer_(std::move(reducer)) {
    result_buf_ = nullptr;
```
- **EN**: It introduces or extends TORCH_API, which define the primary data structures or interfaces for this portion of the file.
- **CN**: 它引入或扩展了 TORCH_API，这些类型定义了本段涉及的主要数据结构或接口。

### Lines 150-165
```cpp
    acc_buf_ = nullptr;
    ri_operand_ = nullptr;
  }

  ReduceOp(
      const ExprPtr& body,
      std::vector<VarPtr> reduce_args,
      BufPtr result_buf,
      BufPtr acc_buf,
      ExprPtr ri_operand,
      Reducer reducer)
      : ExprNodeBase(body->dtype()),
        body_(body),
        reduce_args_(std::move(reduce_args)),
        result_buf_(std::move(result_buf)),
        acc_buf_(std::move(acc_buf)),
```
- **EN**: This chunk continues `TORCH_API` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `TORCH_API`，进一步展开其内部控制流或数据流转。

### Lines 166-180
```cpp
        ri_operand_(std::move(ri_operand)),
        reducer_(std::move(reducer)) {}

  static ExprHandle make(
      ExprHandle body,
      const std::vector<VarHandle>& reduce_args,
      const Reducer& reducer);

  static ExprHandle make(
      ExprHandle body,
      const std::vector<VarHandle>& reduce_args,
      BufHandle result_buf,
      BufHandle acc_buf,
      ExprHandle ri_operand,
      const Reducer& reducer);
```
- **EN**: This chunk defines `make`, which implements a focused step in Tensor Expression analysis, lowering, or code generation.
- **CN**: 这一段定义了 `make`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。

### Lines 182-195
```cpp
  // return the body expression which obtains the value to be reduced.
  ExprPtr body() const {
    return body_;
  }

  // Returns the original Reducer factory that can create ReduceOps.
  const Reducer& reducer() const {
    return reducer_;
  }

  // returns variables associated with the axes of reduction.
  const std::vector<VarPtr>& reduce_args() const {
    return reduce_args_;
  }
```
- **EN**: This chunk defines `reduce_args`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `reduce_args`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 197-209
```cpp
  void setAccBuf(BufHandle acc_buf) {
    acc_buf_ = acc_buf.node();
  }
  BufPtr getAccBuf() {
    return acc_buf_;
  }

  void setResultBuf(BufHandle buf) {
    result_buf_ = buf.node();
  }
  BufPtr getResultBuf() {
    return result_buf_;
  }
```
- **EN**: This chunk defines `getResultBuf`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `getResultBuf`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 211-225
```cpp
  void setRiOperand(ExprHandle ri_operand) {
    ri_operand_ = ri_operand.node();
  }
  ExprPtr getRiOperand() {
    return ri_operand_;
  }

 private:
  // body_ = reducer_->interaction_(result_buf_, ri_operand_)
  ExprPtr body_;
  std::vector<VarPtr> reduce_args_;

  BufPtr result_buf_;
  BufPtr acc_buf_;
  ExprPtr ri_operand_;
```
- **EN**: This chunk defines `getRiOperand`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `getRiOperand`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 227-236
```cpp
  const Reducer reducer_;
};

class Sum : public Reducer {
 public:
  Sum()
      : Reducer(ExprHandle(0), [](const ExprHandle& a, const ExprHandle& b) {
          return a + b;
        }) {}
};
```
- **EN**: It introduces or extends Sum, which define the primary data structures or interfaces for this portion of the file. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 它引入或扩展了 Sum，这些类型定义了本段涉及的主要数据结构或接口。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 238-249
```cpp
inline ExprHandle maximumVal(ScalarType type) {
  switch (type) {
#define MAX_BY_TYPE_CASE(Type, Name) \
  case ScalarType::Name:             \
    return ExprHandle(std::numeric_limits<Type>::max());
    AT_FORALL_SCALAR_TYPES_AND3(Bool, Half, BFloat16, MAX_BY_TYPE_CASE)
#undef MAX_BY_TYPE_CASE
    default:
      throw unsupported_dtype();
  }
  return ExprHandle();
}
```
- **EN**: The preprocessor guard keeps declarations single-instanced when the header is included transitively. This chunk defines `maximumVal`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Switch-based dispatch selects specialized behavior for each opcode, node kind, or schema case. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 预处理器保护用于避免头文件在传递包含时被重复展开。 这一段定义了 `maximumVal`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 基于 switch 的分发为不同 opcode、节点类型或 schema 分支选择专门行为。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 251-261
```cpp
inline ExprHandle minimumVal(ScalarType type) {
  switch (type) {
#define MAX_BY_TYPE_CASE(Type, Name) \
  case ScalarType::Name:             \
    return ExprHandle(std::numeric_limits<Type>::min());
    AT_FORALL_SCALAR_TYPES_AND3(Bool, Half, BFloat16, MAX_BY_TYPE_CASE)
#undef MAX_BY_TYPE_CASE
    default:
      throw unsupported_dtype();
  }
}
```
- **EN**: The preprocessor guard keeps declarations single-instanced when the header is included transitively. This chunk defines `minimumVal`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Switch-based dispatch selects specialized behavior for each opcode, node kind, or schema case. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 预处理器保护用于避免头文件在传递包含时被重复展开。 这一段定义了 `minimumVal`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 基于 switch 的分发为不同 opcode、节点类型或 schema 分支选择专门行为。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 263-278
```cpp
class Maximum : public Reducer {
 public:
  // TODO possible to remove this arg by deferring the init value until we
  // know the dtype of the body.
  Maximum(Dtype dtype)
      : Reducer(
            minimumVal(dtype.scalar_type()),
            [](const ExprHandle& a, const ExprHandle& b) {
              return Max::make(a, b, true);
            }) {}
  Maximum(ExprHandle initializer)
      : Reducer(
            std::move(initializer),
            [](const ExprHandle& a, const ExprHandle& b) {
              return Max::make(a, b, true);
            }) {}
```
- **EN**: It introduces or extends Maximum, which define the primary data structures or interfaces for this portion of the file. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 它引入或扩展了 Maximum，这些类型定义了本段涉及的主要数据结构或接口。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 279-293
```cpp
};

class Minimum : public Reducer {
 public:
  Minimum(Dtype dtype)
      : Reducer(
            maximumVal(dtype.scalar_type()),
            [](const ExprHandle& a, const ExprHandle& b) {
              return Min::make(a, b, true);
            }) {}
  Minimum(const ExprHandle& initializer)
      : Reducer(initializer, [](const ExprHandle& a, const ExprHandle& b) {
          return Min::make(a, b, true);
        }) {}
};
```
- **EN**: It introduces or extends Minimum, which define the primary data structures or interfaces for this portion of the file. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 它引入或扩展了 Minimum，这些类型定义了本段涉及的主要数据结构或接口。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 295-306
```cpp
class ReductionExpander : public IRMutator {
 public:
  StmtPtr expand(const StmtPtr& s) {
    return s->accept_mutator(this);
  }

  ExprPtr mutate(const ReduceOpPtr& v) override {
    return v->body();
  }
};

} // namespace torch::jit::tensorexpr
```
- **EN**: It introduces or extends ReductionExpander, which define the primary data structures or interfaces for this portion of the file. This chunk defines `mutate`, which visits or transforms IR nodes while preserving the intended semantics. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 它引入或扩展了 ReductionExpander，这些类型定义了本段涉及的主要数据结构或接口。 这一段定义了 `mutate`，其作用是访问或变换 IR 节点，同时保持预期语义。 末尾会返回累积结果，或把更新后的状态交还给调用方。

## Key Concepts / 关键概念

- **Tensor Expression IR**
  - EN: Implements Tensor Expression IR, analyses, scheduling, lowering, and backend code generation.
  - CN: 实现 Tensor Expression IR、分析、调度、降级以及后端代码生成。
- **TORCH_API**
  - EN: `TORCH_API` is a central symbol declared or implemented in this file.
  - CN: `TORCH_API` 是本文件声明或实现的核心符号。
- **Sum**
  - EN: `Sum` is a central symbol declared or implemented in this file.
  - CN: `Sum` 是本文件声明或实现的核心符号。
- **Operator dispatch**
  - EN: Bridges symbolic operator names or schemas to concrete C++ implementations.
  - CN: 把符号化算子名或 schema 连接到具体的 C++ 实现。
- **Loop nest scheduling**
  - EN: Reorders, splits, and fuses loops to improve locality and code generation quality.
  - CN: 通过循环重排、切分与融合来提升局部性和代码生成质量。
- **Backend code generation**
  - EN: Translates IR into backend-specific source, kernels, or machine-level artifacts.
  - CN: 把 IR 翻译为后端特定的源码、内核或机器级产物。
## Dependencies / 依赖关系

- **Internal PyTorch headers / 内部 PyTorch 头文件**: `torch/csrc/jit/tensorexpr/expr.h`, `torch/csrc/jit/tensorexpr/ir.h`, `torch/csrc/jit/tensorexpr/ir_printer.h`, `torch/csrc/jit/tensorexpr/stmt.h`, `torch/csrc/jit/tensorexpr/types.h`
- **Standard library / 标准库**: `functional`, `utility`, `vector`
- **Primary symbols in this file / 本文件核心符号**: `TORCH_API`, `Sum`, `initializer`, `getReduceBody`, `malformed_input`, `complete`, `make`, `body`
