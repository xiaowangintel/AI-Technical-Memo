# ir_printer.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/csrc/jit/tensorexpr/ir_printer.cpp`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Defines operator wrappers, schemas, or dispatch helpers that connect TorchScript nodes to runtime kernels.
- **Purpose (CN)**: 定义算子包装、schema 或分发辅助逻辑，将 TorchScript 节点连接到运行时内核。
## Line-by-Line Analysis / 逐行分析

### Lines 1-23
```cpp
#include <torch/csrc/jit/tensorexpr/ir_printer.h>

#include <torch/csrc/jit/tensorexpr/ir_simplifier.h>
#include <torch/csrc/jit/tensorexpr/reduction.h>
#include <torch/csrc/jit/tensorexpr/tensor.h>

#include <c10/util/irange.h>

#include <iostream>

namespace torch::jit::tensorexpr {

std::string IRPrinter::dtypeToCppString(const Dtype& dtype) {
  return dtype.ToCppString();
}

void IRPrinter::print(ExprHandle expr) {
  expr.node()->accept(this);
}

void IRPrinter::print(Expr& expr) {
  expr.accept(this);
}
```
- **EN**: This block assembles the compilation dependencies, pulling in local JIT headers such as torch/csrc/jit/tensorexpr/ir_printer.h, torch/csrc/jit/tensorexpr/ir_simplifier.h, torch/csrc/jit/tensorexpr/reduction.h, and 1 more; ATen/c10 facilities such as c10/util/irange.h; standard-library headers such as iostream. The namespace declarations place the code inside torch::jit::tensorexpr, matching the surrounding JIT subsystem. This chunk defines `print`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段组织编译依赖，引入了本地 JIT 头文件，如 torch/csrc/jit/tensorexpr/ir_printer.h、torch/csrc/jit/tensorexpr/ir_simplifier.h、torch/csrc/jit/tensorexpr/reduction.h 等共 4 项；ATen/c10 基础设施，如 c10/util/irange.h；标准库头文件，如 iostream。 命名空间声明把代码放入 torch::jit::tensorexpr 中，与周边 JIT 子系统保持一致。 这一段定义了 `print`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 25-45
```cpp
void IRPrinter::print(Stmt& stmt) {
  stmt.accept(this);
}
std::string IRPrinter::to_string(CompareSelectOperation op) {
  switch (op) {
    case CompareSelectOperation::kEQ:
      return "==";
    case CompareSelectOperation::kNE:
      return "!=";
    case CompareSelectOperation::kGT:
      return ">";
    case CompareSelectOperation::kGE:
      return ">=";
    case CompareSelectOperation::kLT:
      return "<";
    case CompareSelectOperation::kLE:
      return "<=";
    default:
      throw std::runtime_error("invalid compare select operator");
  }
}
```
- **EN**: This chunk defines `runtime_error`, which executes prepared logic against runtime values or IR state. Switch-based dispatch selects specialized behavior for each opcode, node kind, or schema case. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `runtime_error`，其作用是针对运行时数值或 IR 状态执行准备好的逻辑。 基于 switch 的分发为不同 opcode、节点类型或 schema 分支选择专门行为。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 47-70
```cpp
void IRPrinter::PrinterStream::initialize_imbue() {
  // Similar to https://github.com/pytorch/pytorch/issues/79583:
  // global locale can be set to something other than "C", which can add
  // extra commas in the printed numbers.
  static std::locale c_locale("C");
  // note: IRPrinter is a subclass of ostream, so imbue is a member function.
  imbue(c_locale);
}

// TODO: change whether to include the parenthesis to the parent expression,
// we need to look at the operator precedence to make the output simpler.
template <
    typename Op,
    std::enable_if_t<std::is_same_v<
        decltype(detail::bin_op_deducer(std::declval<Op>())),
        void>>* = nullptr>
static void visitBinaryOp(
    NodePtr<Op> v,
    const std::string& op_str,
    IRPrinter* printer,
    bool parens = true) {
  std::ostream& os = printer->os();
  int self_prec = getPrecedence(v->expr_type());
  int lhs_prec = getPrecedence(v->lhs()->expr_type());
```
- **EN**: This chunk defines `c_locale`, which implements a focused step in Tensor Expression analysis, lowering, or code generation.
- **CN**: 这一段定义了 `c_locale`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。

### Lines 71-90
```cpp
  int rhs_prec = getPrecedence(v->rhs()->expr_type());

  if (lhs_prec >= self_prec) {
    os << '(';
  }
  v->lhs()->accept(printer);
  if (lhs_prec >= self_prec) {
    os << ')';
  }

  os << ' ' << op_str << ' ';

  if (rhs_prec >= self_prec) {
    os << '(';
  }
  v->rhs()->accept(printer);
  if (rhs_prec >= self_prec) {
    os << ')';
  }
}
```
- **EN**: This chunk continues `c_locale` and expands its internal control flow or data movement. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段延续了 `c_locale`，进一步展开其内部控制流或数据流转。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 92-114
```cpp
void IRPrinter::visit(const AddPtr& v) {
  visitBinaryOp(v, "+", this);
}

void IRPrinter::visit(const SubPtr& v) {
  visitBinaryOp(v, "-", this);
}

void IRPrinter::visit(const MulPtr& v) {
  visitBinaryOp(v, "*", this);
}

void IRPrinter::visit(const DivPtr& v) {
  visitBinaryOp(v, "/", this);
}

void IRPrinter::visit(const AndPtr& v) {
  visitBinaryOp(v, "&", this);
}

void IRPrinter::visit(const OrPtr& v) {
  visitBinaryOp(v, "|", this);
}
```
- **EN**: This chunk defines `visit`, which visits or transforms IR nodes while preserving the intended semantics.
- **CN**: 这一段定义了 `visit`，其作用是访问或变换 IR 节点，同时保持预期语义。

### Lines 116-136
```cpp
void IRPrinter::visit(const XorPtr& v) {
  visitBinaryOp(v, "^", this);
}

void IRPrinter::visit(const LshiftPtr& v) {
  visitBinaryOp(v, "<<", this);
}

void IRPrinter::visit(const RshiftPtr& v) {
  visitBinaryOp(v, ">>", this);
}

void IRPrinter::visit(const ModPtr& v) {
  if (v->dtype().is_integral()) {
    visitBinaryOp(v, "%", this);
  } else if (v->dtype().is_floating_point()) {
    os() << "mod(" << *v->lhs() << ", " << *v->rhs() << ')';
  } else {
    throw std::runtime_error("invalid dtype: " + std::to_string(v->dtype()));
  }
}
```
- **EN**: This chunk defines `runtime_error`, which executes prepared logic against runtime values or IR state. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages.
- **CN**: 这一段定义了 `runtime_error`，其作用是针对运行时数值或 IR 状态执行准备好的逻辑。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。

### Lines 138-158
```cpp
void IRPrinter::visit(const MaxPtr& v) {
  os() << "Max(";
  v->lhs()->accept(this);
  os() << ", ";
  v->rhs()->accept(this);
  os() << ", " << (unsigned int)v->propagate_nans() << ')';
}

void IRPrinter::visit(const MinPtr& v) {
  os() << "Min(";
  v->lhs()->accept(this);
  os() << ", ";
  v->rhs()->accept(this);
  os() << ", " << (unsigned int)v->propagate_nans() << ')';
}

void IRPrinter::visit(const CompareSelectPtr& v) {
  CompareSelectOperation cmp_op = v->compare_select_op();
  int self_prec = getPrecedence(v->expr_type());
  int lhs_prec = getPrecedence(v->lhs()->expr_type());
  int rhs_prec = getPrecedence(v->rhs()->expr_type());
```
- **EN**: This chunk defines `visit`, which visits or transforms IR nodes while preserving the intended semantics.
- **CN**: 这一段定义了 `visit`，其作用是访问或变换 IR 节点，同时保持预期语义。

### Lines 160-177
```cpp
  if (lhs_prec >= self_prec) {
    os() << '(';
  }
  v->lhs()->accept(this);
  if (lhs_prec >= self_prec) {
    os() << ')';
  }

  os() << to_string(cmp_op);

  if (rhs_prec >= self_prec) {
    os() << '(';
  }
  v->rhs()->accept(this);
  if (rhs_prec >= self_prec) {
    os() << ')';
  }
  os() << " ? ";
```
- **EN**: This chunk continues `visit` and expands its internal control flow or data movement. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段延续了 `visit`，进一步展开其内部控制流或数据流转。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 179-201
```cpp
  auto withParens = [&](const ExprPtr& e) {
    auto prec = getPrecedence(e->expr_type());
    if (prec >= self_prec) {
      os() << '(';
    }
    e->accept(this);
    if (prec >= self_prec) {
      os() << ')';
    }
  };
  withParens(v->ret_val1());
  os() << " : ";
  withParens(v->ret_val2());
}

static void formatFPSuffix(std::ostream& os, double v, bool flag) {
  os << (flag && v == std::ceil(v) ? ".0" : "");
}

template <typename T>
static void formatFPSuffix(std::ostream& os, T v, bool flag) {
  os << (flag && v == std::ceil(v) ? ".f" : "f");
}
```
- **EN**: This chunk defines `formatFPSuffix`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段定义了 `formatFPSuffix`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 203-223
```cpp
template <typename T, std::enable_if_t<std::is_floating_point_v<T>>* = nullptr>
static void formatImm(std::ostream& os, T v) {
  const int precision = 16;
  const T lower_bound = static_cast<T>(-std::pow(10, precision));
  const T upper_bound = -lower_bound;
  if (std::isnan(v)) {
    os << "NAN";
  } else if (std::isinf(v)) {
    os << (v > 0 ? "POS_INFINITY" : "NEG_INFINITY");
  } else {
    os << std::setprecision(precision) << v;
    formatFPSuffix(os, v, v > lower_bound && v < upper_bound);
  }
}

static void formatIntSuffix(std::ostream& os, int64_t v) {
  os << "ll";
}

template <typename T>
static void formatIntSuffix(std::ostream& os, T v) {}
```
- **EN**: This chunk defines `formatIntSuffix`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段定义了 `formatIntSuffix`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 225-243
```cpp
template <typename T, std::enable_if_t<!std::is_floating_point_v<T>>* = nullptr>
static void formatImm(std::ostream& os, T v) {
  os << +v;
  formatIntSuffix(os, v);
}

#define IMM_PRINT_VISIT(Type, Name)              \
  void IRPrinter::visit(const Name##ImmPtr& v) { \
    formatImm(os(), v->value());                 \
  }
AT_FORALL_SCALAR_TYPES_AND3(Bool, Half, BFloat16, IMM_PRINT_VISIT)
#undef IMM_PRINT_VISIT

void IRPrinter::visit(const CastPtr& v) {
  auto dtype = v->dtype();
  os() << dtypeToCppString(dtype) << '(';
  v->src_value()->accept(this);
  os() << ')';
}
```
- **EN**: The preprocessor guard keeps declarations single-instanced when the header is included transitively. This chunk defines `visit`, which visits or transforms IR nodes while preserving the intended semantics.
- **CN**: 预处理器保护用于避免头文件在传递包含时被重复展开。 这一段定义了 `visit`，其作用是访问或变换 IR 节点，同时保持预期语义。

### Lines 245-268
```cpp
void IRPrinter::visit(const BitCastPtr& v) {
  auto dtype = v->dtype();
  os() << "BitCast<" << dtype.ToCppString() << ">(";
  v->src_value()->accept(this);
  os() << ')';
}

void IRPrinter::visit(const VarPtr& v) {
  os() << name_manager_.get_unique_name(v);
}

void IRPrinter::visit(const BufPtr& v) {
  auto dtype = v->dtype();
  os() << *v->base_handle();
  os() << "(dtype=" << dtypeToCppString(dtype);
  if (v->qscale()) {
    os() << ", qscale=";
    v->qscale()->accept(this);
  }
  if (v->qscale()) {
    os() << ", qzero=";
    v->qzero()->accept(this);
  }
  os() << ", sizes=[";
```
- **EN**: This chunk defines `visit`, which visits or transforms IR nodes while preserving the intended semantics. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段定义了 `visit`，其作用是访问或变换 IR 节点，同时保持预期语义。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 269-288
```cpp
  size_t i = 0;
  for (const ExprPtr& s : v->dims()) {
    if (i++) {
      os() << ", ";
    }
    s->accept(this);
  }
  os() << ']';
  os() << ", strides=[";
  i = 0;
  for (const ExprPtr& s : v->strides()) {
    if (i++) {
      os() << ", ";
    }
    s->accept(this);
  }
  os() << ']';

  os() << ')';
}
```
- **EN**: This chunk continues `visit` and expands its internal control flow or data movement. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段延续了 `visit`，进一步展开其内部控制流或数据流转。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 290-313
```cpp
void IRPrinter::visit(const RampPtr& v) {
  os() << "Ramp(" << *v->base() << ", " << *v->stride() << ", " << v->lanes()
       << ')';
}

void IRPrinter::visit(const LoadPtr& v) {
  // TODO: support the mask case
  if (v->indices().empty()) {
    os() << *v->base_handle();
  } else {
    os() << *v->base_handle() << '[';
    size_t i = 0;
    for (const ExprPtr& ind : v->indices()) {
      if (i++) {
        os() << ", ";
      }
      ind->accept(this);
    }
    if (v->indices().empty()) {
      os() << '0';
    }
    os() << ']';
  }
}
```
- **EN**: This chunk defines `visit`, which visits or transforms IR nodes while preserving the intended semantics. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段定义了 `visit`，其作用是访问或变换 IR 节点，同时保持预期语义。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 315-333
```cpp
void IRPrinter::visit(const BroadcastPtr& v) {
  os() << "Broadcast(" << *v->value() << ", " << v->lanes() << ')';
}

void IRPrinter::visit(const IfThenElsePtr& v) {
  os() << "IfThenElse(" << *v->condition() << ", " << *v->true_value() << ", "
       << *v->false_value() << ')';
}

void IRPrinter::visit(const IntrinsicsPtr& v) {
  os() << v->func_name() << '(';
  for (const auto i : c10::irange(v->nparams())) {
    if (i > 0) {
      os() << ", ";
    }
    os() << *v->param(i);
  }
  os() << ')';
}
```
- **EN**: This chunk defines `visit`, which visits or transforms IR nodes while preserving the intended semantics. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段定义了 `visit`，其作用是访问或变换 IR 节点，同时保持预期语义。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 335-354
```cpp
void IRPrinter::visit(const TermPtr& v) {
  os() << "Term(";
  v->scalar()->accept(this);
  for (const auto& t : v->variables()) {
    os() << ',';
    t->accept(this);
  }
  os() << ')';
}

void IRPrinter::visit(const PolynomialPtr& v) {
  bool first = true;
  os() << "Polynomial(";
  for (const auto& t : v->variables()) {
    if (!first) {
      os() << " + ";
    }
    first = false;
    t->accept(this);
  }
```
- **EN**: This chunk defines `visit`, which visits or transforms IR nodes while preserving the intended semantics. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段定义了 `visit`，其作用是访问或变换 IR 节点，同时保持预期语义。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 356-379
```cpp
  if (!first) {
    os() << " + ";
  }
  v->scalar()->accept(this);
  os() << ')';
}

void IRPrinter::visit(const RoundOffPtr& v) {
  os() << "RoundOff(";
  v->lhs()->accept(this);
  os() << ", ";
  v->rhs()->accept(this);
  os() << ')';
}

void IRPrinter::visit(const MaxTermPtr& v) {
  os() << "MaxTerm(";
  if (v->scalar()) {
    v->scalar()->accept(this);
    os() << ", ";
  }
  for (size_t i = 0; i < v->variables().size(); ++i) {
    v->variables()[i]->accept(this);
    if (i < v->variables().size() - 1) {
```
- **EN**: This chunk defines `visit`, which visits or transforms IR nodes while preserving the intended semantics. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段定义了 `visit`，其作用是访问或变换 IR 节点，同时保持预期语义。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 380-399
```cpp
      os() << ", ";
    }
  }
  os() << ')';
}

void IRPrinter::visit(const MinTermPtr& v) {
  os() << "MinTerm(";
  if (v->scalar()) {
    v->scalar()->accept(this);
    os() << ", ";
  }
  for (size_t i = 0; i < v->variables().size(); ++i) {
    v->variables()[i]->accept(this);
    if (i < v->variables().size() - 1) {
      os() << ", ";
    }
  }
  os() << ')';
}
```
- **EN**: This chunk defines `visit`, which visits or transforms IR nodes while preserving the intended semantics. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段定义了 `visit`，其作用是访问或变换 IR 节点，同时保持预期语义。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 401-421
```cpp
void IRPrinter::visit(const ReduceOpPtr& v) {
  os() << "ReduceOp(";
  os() << *v->body() << ", ";

  bool first = true;
  os() << "reduce_args={";
  for (const auto& d : v->reduce_args()) {
    if (!first) {
      os() << ", ";
    }
    os() << *d;
    first = false;
  }
  os() << "})";
}

// === Stmt visitors below ===

// Newlines and indentation are handled solely by the `Block` printer.  For
// each statement in a `Block` the printer will insert indentation before
// the statement and a newline after the statement.
```
- **EN**: This chunk defines `visit`, which visits or transforms IR nodes while preserving the intended semantics. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段定义了 `visit`，其作用是访问或变换 IR 节点，同时保持预期语义。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 423-442
```cpp
void IRPrinter::visit(const StorePtr& v) {
  // TODO: handle the mask
  if (v->indices().empty()) {
    os() << *v->base_handle() << " = " << *v->value() << ';';
    return;
  }

  os() << *v->base_handle() << '[';
  size_t i = 0;
  for (const ExprPtr& ind : v->indices()) {
    if (i++) {
      os() << ", ";
    }
    ind->accept(this);
  }
  if (v->indices().empty()) {
    os() << '0';
  }
  os() << "] = " << *v->value() << ';';
}
```
- **EN**: This chunk defines `visit`, which visits or transforms IR nodes while preserving the intended semantics. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `visit`，其作用是访问或变换 IR 节点，同时保持预期语义。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 444-463
```cpp
void IRPrinter::visit(const ForPtr& v) {
  VarPtr var = v->var();
  VarHandle vv(var);
  os() << "for (" << dtypeToCppString(var->dtype()) << ' ' << vv << " = "
       << ExprHandle(v->start()) << "; " << vv << " < " << ExprHandle(v->stop())
       << "; " << vv << "++) ";
  std::string loop_options_str = v->loop_options().ToString();
  if (!loop_options_str.empty()) {
    os() << " /* " << loop_options_str << " */";
  }
  if (v->body()) {
    os() << *v->body();
  } else {
    os() << "{}";
  }
}

void IRPrinter::visit(const BlockPtr& v) {
  os() << "{\n";
  indent_++;
```
- **EN**: This chunk defines `vv`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段定义了 `vv`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 465-486
```cpp
  for (const StmtPtr& s : *v) {
    emitIndent();
    os() << *s << '\n';
  }
  indent_--;
  emitIndent();
  os() << '}';
}

void IRPrinter::visit(const AllocatePtr& v) {
  os() << "Allocate(" << *v->buffer_var()
       << "); // dtype=" << dtypeToCppString(v->dtype());
  os() << ", dims=[";
  const std::vector<ExprPtr>& dims = v->dims();
  for (const auto i : c10::irange(dims.size())) {
    if (i != 0) {
      os() << ", ";
    }
    os() << *dims[i];
  }
  os() << ']';
}
```
- **EN**: This chunk defines `visit`, which visits or transforms IR nodes while preserving the intended semantics. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段定义了 `visit`，其作用是访问或变换 IR 节点，同时保持预期语义。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 488-508
```cpp
void IRPrinter::visit(const FreePtr& v) {
  os() << "Free(" << *v->buffer_var() << ");";
}

void IRPrinter::visit(const FreeExtPtr& v) {
  os() << "FreeExt(bufs={";
  int i = 0;
  for (const auto& buf : v->bufs()) {
    if (i++ > 0) {
      os() << ", ";
    }
    os() << *buf;
  }

  os() << "});";
}

void IRPrinter::visit(const PlacementAllocatePtr& v) {
  os() << "Alias(" << *v->buf()->base_handle() << ','
       << *v->buf_to_reuse()->base_handle() << ");";
}
```
- **EN**: This chunk defines `visit`, which visits or transforms IR nodes while preserving the intended semantics. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段定义了 `visit`，其作用是访问或变换 IR 节点，同时保持预期语义。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 510-530
```cpp
void IRPrinter::visit(const LetPtr& v) {
  os() << dtypeToCppString(v->var()->dtype()) << ' ' << *v->var();
  os() << " = " << *v->value() << ';';
}

void IRPrinter::visit(const CondPtr& v) {
  ExprPtr cond = v->condition();
  StmtPtr true_stmt = v->true_stmt();
  StmtPtr false_stmt = v->false_stmt();
  if (!true_stmt) {
    os() << "if (!" << *cond << ") ";
    os() << *false_stmt;
  } else {
    os() << "if (" << *cond << ") ";
    os() << *true_stmt;
    if (false_stmt) {
      os() << " else ";
      os() << *false_stmt;
    }
  }
}
```
- **EN**: This chunk defines `visit`, which visits or transforms IR nodes while preserving the intended semantics. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段定义了 `visit`，其作用是访问或变换 IR 节点，同时保持预期语义。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 532-552
```cpp
void IRPrinter::visit(const AtomicAddPtr& v) {
  os() << "atomicAdd(&" << *v->base_handle() << '[';
  size_t i = 0;
  for (const ExprPtr& ind : v->indices()) {
    if (i++) {
      os() << ", ";
    }
    ind->accept(this);
  }
  if (v->indices().empty()) {
    os() << '0';
  }
  os() << "], " << *v->value() << ");";
}

void IRPrinter::visit(const SyncThreadsPtr& v) {
  os() << "__syncthreads();";
}

void IRPrinter::visit(const ExternalCallPtr& v) {
  os() << *v->buf() << " = " << v->func_name() << '(';
```
- **EN**: This chunk defines `visit`, which visits or transforms IR nodes while preserving the intended semantics. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段定义了 `visit`，其作用是访问或变换 IR 节点，同时保持预期语义。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 554-572
```cpp
  os() << "buf_args={";
  int i = 0;
  for (const BufPtr& buf_arg : v->buf_args()) {
    if (i++ > 0) {
      os() << ", ";
    }
    os() << *buf_arg;
  }

  os() << "}, args={";
  i = 0;
  for (const ExprPtr& arg : v->args()) {
    if (i++ > 0) {
      os() << ", ";
    }
    os() << *arg;
  }
  os() << "})";
}
```
- **EN**: This chunk continues `visit` and expands its internal control flow or data movement. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段延续了 `visit`，进一步展开其内部控制流或数据流转。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 574-592
```cpp
void IRPrinter::visit(const ExternalCallWithAllocPtr& v) {
  int i = 0;
  for (const auto& buf_out_arg : v->buf_out_args()) {
    if (i++ > 0) {
      os() << ", ";
    }
    os() << *buf_out_arg;
  }

  os() << " := " << v->func_name() << '(';

  os() << "buf_args={";
  i = 0;
  for (const auto& buf_arg : v->buf_args()) {
    if (i++ > 0) {
      os() << ", ";
    }
    os() << *buf_arg;
  }
```
- **EN**: This chunk defines `visit`, which visits or transforms IR nodes while preserving the intended semantics. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段定义了 `visit`，其作用是访问或变换 IR 节点，同时保持预期语义。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 594-617
```cpp
  os() << "}, args={";
  i = 0;
  for (const auto& arg : v->args()) {
    if (i++ > 0) {
      os() << ", ";
    }
    os() << *arg;
  }
  os() << "})";
}

void IRPrinter::emitIndent() {
  os() << std::setw(2 * indent_) << "";
}

std::ostream& operator<<(std::ostream& stream, const ExprHandle& expr) {
  IRPrinter::PrinterStream* printer_stream =
      dynamic_cast<IRPrinter::PrinterStream*>(&stream);
  ExprHandle& mutable_expr = const_cast<ExprHandle&>(expr);
  if (printer_stream != nullptr) {
    mutable_expr.node()->accept(printer_stream->printer());
  } else {
    IRPrinter p(stream);
    p.print(mutable_expr);
```
- **EN**: This chunk defines `p`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段定义了 `p`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 618-633
```cpp
  }
  return stream;
}

std::ostream& operator<<(std::ostream& stream, const Expr& expr) {
  IRPrinter::PrinterStream* printer_stream =
      dynamic_cast<IRPrinter::PrinterStream*>(&stream);
  Expr& mutable_expr = const_cast<Expr&>(expr);
  if (printer_stream != nullptr) {
    mutable_expr.accept(printer_stream->printer());
  } else {
    IRPrinter p(stream);
    p.print(mutable_expr);
  }
  return stream;
}
```
- **EN**: This chunk defines `p`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `p`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 635-651
```cpp
std::ostream& operator<<(std::ostream& stream, const Stmt& stmt) {
  IRPrinter::PrinterStream* printer_stream =
      dynamic_cast<IRPrinter::PrinterStream*>(&stream);
  Stmt& mutable_stmt = const_cast<Stmt&>(stmt);
  if (printer_stream != nullptr) {
    mutable_stmt.accept(printer_stream->printer());
  } else {
    IRPrinter p(stream);
    p.print(mutable_stmt);
  }
  return stream;
}

std::ostream& operator<<(std::ostream& stream, const Tensor& t) {
  stream << std::to_string(t);
  return stream;
}
```
- **EN**: This chunk defines `p`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `p`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 653-674
```cpp
void print(const ExprPtr& expr) {
  if (expr) {
    IRPrinter p(std::cout);
    p.print(*expr);
  } else {
    std::cout << "(null expr)";
  }
  std::cout << '\n';
}

void print(const StmtPtr& stmt) {
  if (stmt) {
    IRPrinter p(std::cout);
    p.print(*stmt);
  } else {
    std::cout << "(null stmt)\n";
  }
}

void print(const Tensor& t) {
  std::cout << std::to_string(t);
}
```
- **EN**: This chunk defines `p`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段定义了 `p`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 676-699
```cpp
} // namespace torch::jit::tensorexpr

namespace std {
std::string to_string(const ExprPtr& expr) {
  std::ostringstream oss;
  oss << *expr;
  return oss.str();
}

std::string to_string(const StmtPtr& stmt) {
  std::ostringstream oss;
  oss << *stmt;
  return oss.str();
}

std::string to_string(const Tensor& t) {
  std::ostringstream oss;
  // TODO: move this to Buf printer
  oss << "Tensor " << t.buf()->name_hint() << '[';
  for (const auto i : c10::irange(t.buf()->ndim())) {
    if (i != 0) {
      oss << ", ";
    }
    oss << *t.buf()->dim(i);
```
- **EN**: The namespace declarations place the code inside std, matching the surrounding JIT subsystem. This chunk defines `to_string`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 命名空间声明把代码放入 std 中，与周边 JIT 子系统保持一致。 这一段定义了 `to_string`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 700-704
```cpp
  }
  oss << "]:\n" << *t.stmt() << '\n';
  return oss.str();
}
} // namespace std
```
- **EN**: This chunk continues `to_string` and expands its internal control flow or data movement. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `to_string`，进一步展开其内部控制流或数据流转。 末尾会返回累积结果，或把更新后的状态交还给调用方。

## Key Concepts / 关键概念

- **Tensor Expression IR**
  - EN: Implements Tensor Expression IR, analyses, scheduling, lowering, and backend code generation.
  - CN: 实现 Tensor Expression IR、分析、调度、降级以及后端代码生成。
- **dtypeToCppString**
  - EN: `dtypeToCppString` is a central symbol declared or implemented in this file.
  - CN: `dtypeToCppString` 是本文件声明或实现的核心符号。
- **print**
  - EN: `print` is a central symbol declared or implemented in this file.
  - CN: `print` 是本文件声明或实现的核心符号。
- **TorchScript runtime**
  - EN: Coordinates execution plans, stacks, and operator dispatch for scripted programs.
  - CN: 协调脚本程序的执行计划、栈以及算子分发。
- **Operator dispatch**
  - EN: Bridges symbolic operator names or schemas to concrete C++ implementations.
  - CN: 把符号化算子名或 schema 连接到具体的 C++ 实现。
- **IR simplification**
  - EN: Rewrites expressions and statements into simpler but equivalent forms.
  - CN: 把表达式和语句重写为更简单但等价的形式。
## Dependencies / 依赖关系

- **Internal PyTorch headers / 内部 PyTorch 头文件**: `torch/csrc/jit/tensorexpr/ir_printer.h`, `torch/csrc/jit/tensorexpr/ir_simplifier.h`, `torch/csrc/jit/tensorexpr/reduction.h`, `torch/csrc/jit/tensorexpr/tensor.h`
- **ATen/c10 foundations / ATen/c10 基础设施**: `c10/util/irange.h`
- **Standard library / 标准库**: `iostream`
- **Primary symbols in this file / 本文件核心符号**: `dtypeToCppString`, `print`, `to_string`, `runtime_error`, `initialize_imbue`, `c_locale`, `visit`, `formatFPSuffix`
