# ir_cloner.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/csrc/jit/tensorexpr/ir_cloner.cpp`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Simplifies Tensor Expression IR by rewriting algebraic expressions and control-flow patterns.
- **Purpose (CN)**: 通过重写代数表达式和控制流模式来简化 Tensor Expression IR。
## Line-by-Line Analysis / 逐行分析

### Lines 1-16
```cpp
#include <torch/csrc/jit/tensorexpr/ir_cloner.h>

#include <torch/csrc/jit/tensorexpr/ir.h>
#include <torch/csrc/jit/tensorexpr/ir_simplifier.h>
#include <torch/csrc/jit/tensorexpr/reduction.h>

namespace torch::jit::tensorexpr {

template <
    typename Op,
    std::enable_if_t<std::is_same_v<
        decltype(detail::bin_op_deducer(std::declval<Op>())),
        void>>* = nullptr>
static ExprPtr mutate_binary_op(
    NodePtr<Op> v,
    IRCloner* cloner,
```
- **EN**: This block assembles the compilation dependencies, pulling in local JIT headers such as torch/csrc/jit/tensorexpr/ir_cloner.h, torch/csrc/jit/tensorexpr/ir.h, torch/csrc/jit/tensorexpr/ir_simplifier.h, and 1 more. The namespace declarations place the code inside torch::jit::tensorexpr, matching the surrounding JIT subsystem.
- **CN**: 这一段组织编译依赖，引入了本地 JIT 头文件，如 torch/csrc/jit/tensorexpr/ir_cloner.h、torch/csrc/jit/tensorexpr/ir.h、torch/csrc/jit/tensorexpr/ir_simplifier.h 等共 4 项。 命名空间声明把代码放入 torch::jit::tensorexpr 中，与周边 JIT 子系统保持一致。

### Lines 17-32
```cpp
    bool option = false) {
  ExprPtr lhs_new = v->lhs()->accept_mutator(cloner);
  ExprPtr rhs_new = v->rhs()->accept_mutator(cloner);
  IRNodeType expr_type = v->expr_type();
  switch (expr_type) {
    case IRNodeType::kAdd:
      return alloc<Add>(lhs_new, rhs_new);
    case IRNodeType::kSub:
      return alloc<Sub>(lhs_new, rhs_new);
    case IRNodeType::kMul:
      return alloc<Mul>(lhs_new, rhs_new);
    case IRNodeType::kDiv:
      return alloc<Div>(lhs_new, rhs_new);
    case IRNodeType::kMod:
      return alloc<Mod>(lhs_new, rhs_new);
    case IRNodeType::kMax:
```
- **EN**: Switch-based dispatch selects specialized behavior for each opcode, node kind, or schema case. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 基于 switch 的分发为不同 opcode、节点类型或 schema 分支选择专门行为。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 33-48
```cpp
      return alloc<Max>(lhs_new, rhs_new, option);
    case IRNodeType::kMin:
      return alloc<Min>(lhs_new, rhs_new, option);
    case IRNodeType::kAnd:
      return alloc<And>(lhs_new, rhs_new);
    case IRNodeType::kOr:
      return alloc<Or>(lhs_new, rhs_new);
    case IRNodeType::kXor:
      return alloc<Xor>(lhs_new, rhs_new);
    case IRNodeType::kLshift:
      return alloc<Lshift>(lhs_new, rhs_new);
    case IRNodeType::kRshift:
      return alloc<Rshift>(lhs_new, rhs_new);
    default:
      throw unimplemented_lowering(v);
  }
```
- **EN**: This chunk declares `unimplemented_lowering`, which lowers higher-level IR into a backend-specific executable form. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段声明了 `unimplemented_lowering`，其作用是把高层 IR 降级为后端特定的可执行形式。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 49-61
```cpp
}

ExprPtr IRCloner::mutate(const AddPtr& v) {
  return mutate_binary_op(v, this);
}

ExprPtr IRCloner::mutate(const SubPtr& v) {
  return mutate_binary_op(v, this);
}

ExprPtr IRCloner::mutate(const MulPtr& v) {
  return mutate_binary_op(v, this);
}
```
- **EN**: This chunk defines `mutate`, which visits or transforms IR nodes while preserving the intended semantics. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `mutate`，其作用是访问或变换 IR 节点，同时保持预期语义。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 63-77
```cpp
ExprPtr IRCloner::mutate(const DivPtr& v) {
  return mutate_binary_op(v, this);
}

ExprPtr IRCloner::mutate(const ModPtr& v) {
  return mutate_binary_op(v, this);
}

ExprPtr IRCloner::mutate(const AndPtr& v) {
  return mutate_binary_op(v, this);
}

ExprPtr IRCloner::mutate(const OrPtr& v) {
  return mutate_binary_op(v, this);
}
```
- **EN**: This chunk defines `mutate`, which visits or transforms IR nodes while preserving the intended semantics. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `mutate`，其作用是访问或变换 IR 节点，同时保持预期语义。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 79-93
```cpp
ExprPtr IRCloner::mutate(const XorPtr& v) {
  return mutate_binary_op(v, this);
}

ExprPtr IRCloner::mutate(const LshiftPtr& v) {
  return mutate_binary_op(v, this);
}

ExprPtr IRCloner::mutate(const RshiftPtr& v) {
  return mutate_binary_op(v, this);
}

ExprPtr IRCloner::mutate(const MaxPtr& v) {
  return mutate_binary_op(v, this, v->propagate_nans());
}
```
- **EN**: This chunk defines `mutate`, which visits or transforms IR nodes while preserving the intended semantics. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `mutate`，其作用是访问或变换 IR 节点，同时保持预期语义。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 95-110
```cpp
ExprPtr IRCloner::mutate(const MinPtr& v) {
  return mutate_binary_op(v, this, v->propagate_nans());
}

ExprPtr IRCloner::mutate(const CompareSelectPtr& v) {
  ExprPtr lhs_new = v->lhs()->accept_mutator(this);
  ExprPtr rhs_new = v->rhs()->accept_mutator(this);
  ExprPtr retval1_new = v->ret_val1()->accept_mutator(this);
  ExprPtr retval2_new = v->ret_val2()->accept_mutator(this);
  return alloc<CompareSelect>(
      lhs_new,
      rhs_new,
      retval1_new,
      retval2_new,
      v->compare_select_op(),
      v->bias());
```
- **EN**: This chunk defines `mutate`, which visits or transforms IR nodes while preserving the intended semantics. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `mutate`，其作用是访问或变换 IR 节点，同时保持预期语义。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 111-123
```cpp
}

#define IMM_MUTATE_DEFINE(_1, Name)                 \
  ExprPtr IRCloner::mutate(const Name##ImmPtr& v) { \
    return v;                                       \
  }
AT_FORALL_SCALAR_TYPES_AND3(Bool, Half, BFloat16, IMM_MUTATE_DEFINE)
#undef IMM_MUTATE_DEFINE

ExprPtr IRCloner::mutate(const CastPtr& v) {
  ExprPtr src_value_new = v->src_value()->accept_mutator(this);
  return alloc<Cast>(v->dtype(), src_value_new);
}
```
- **EN**: The preprocessor guard keeps declarations single-instanced when the header is included transitively. This chunk defines `mutate`, which visits or transforms IR nodes while preserving the intended semantics. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 预处理器保护用于避免头文件在传递包含时被重复展开。 这一段定义了 `mutate`，其作用是访问或变换 IR 节点，同时保持预期语义。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 125-134
```cpp
ExprPtr IRCloner::mutate(const BitCastPtr& v) {
  ExprPtr src_value_new = v->src_value()->accept_mutator(this);
  return alloc<BitCast>(v->dtype(), src_value_new);
}

ExprPtr IRCloner::mutate(const RampPtr& v) {
  ExprPtr base_new = v->base()->accept_mutator(this);
  ExprPtr stride_new = v->stride()->accept_mutator(this);
  return alloc<Ramp>(base_new, stride_new, v->lanes());
}
```
- **EN**: This chunk defines `mutate`, which visits or transforms IR nodes while preserving the intended semantics. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `mutate`，其作用是访问或变换 IR 节点，同时保持预期语义。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 136-150
```cpp
ExprPtr IRCloner::mutate(const LoadPtr& v) {
  std::vector<ExprPtr> indices_new;
  indices_new.reserve(v->indices().size());
  for (const ExprPtr& ind : v->indices()) {
    indices_new.push_back(ind->accept_mutator(this));
  }
  BufPtr buf_new = to<Buf>(v->buf()->accept_mutator(this));
  return alloc<Load>(v->dtype(), buf_new, indices_new);
}

// We do not clone Vars since the original IR and cloned IR are expected to
// share the underlying variables.
ExprPtr IRCloner::mutate(const VarPtr& v) {
  return v;
}
```
- **EN**: This chunk defines `mutate`, which visits or transforms IR nodes while preserving the intended semantics. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `mutate`，其作用是访问或变换 IR 节点，同时保持预期语义。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 152-165
```cpp
// We do not clone Bufs since the original IR and cloned IR are expected to
// share the underlying Bufs. In spite of Bufs having expressions as dims and
// initializers, this is the expected usage of clone at this point.
//
// TODO: Revisit this if Bufs need to be cloned as well.
ExprPtr IRCloner::mutate(const BufPtr& v) {
  return v;
}

ExprPtr IRCloner::mutate(const BroadcastPtr& v) {
  auto lanes = v->lanes();
  ExprPtr value_new = v->value()->accept_mutator(this);
  return alloc<Broadcast>(value_new, lanes);
}
```
- **EN**: This chunk defines `mutate`, which visits or transforms IR nodes while preserving the intended semantics. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `mutate`，其作用是访问或变换 IR 节点，同时保持预期语义。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 167-182
```cpp
ExprPtr IRCloner::mutate(const IfThenElsePtr& v) {
  ExprPtr condition_new = v->condition()->accept_mutator(this);
  ExprPtr true_value_new = v->true_value()->accept_mutator(this);
  ExprPtr false_value_new = v->false_value()->accept_mutator(this);

  return alloc<IfThenElse>(condition_new, true_value_new, false_value_new);
}

ExprPtr IRCloner::mutate(const IntrinsicsPtr& v) {
  std::vector<ExprPtr> params_new;
  params_new.reserve(v->nparams());
  for (const auto& param : v->params()) {
    params_new.push_back(param->accept_mutator(this));
  }
  return alloc<Intrinsics>(v->op_type(), v->dtype(), params_new);
}
```
- **EN**: This chunk defines `mutate`, which visits or transforms IR nodes while preserving the intended semantics. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `mutate`，其作用是访问或变换 IR 节点，同时保持预期语义。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 184-196
```cpp
ExprPtr IRCloner::mutate(const TermPtr& v) {
  ExprPtr scalar_new = v->scalar()->accept_mutator(this);

  std::vector<ExprPtr> variables_new;
  variables_new.reserve(v->variables().size());
  for (const auto& t : v->variables()) {
    variables_new.push_back(t->accept_mutator(this));
  }
  return alloc<Term>(v->hasher(), scalar_new, variables_new);
}

ExprPtr IRCloner::mutate(const PolynomialPtr& v) {
  ExprPtr scalar_new = v->scalar()->accept_mutator(this);
```
- **EN**: This chunk defines `mutate`, which visits or transforms IR nodes while preserving the intended semantics. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `mutate`，其作用是访问或变换 IR 节点，同时保持预期语义。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 198-209
```cpp
  std::vector<TermPtr> variables_new;
  variables_new.reserve(v->variables().size());
  for (const auto& t : v->variables()) {
    variables_new.push_back(static_to<Term>(t->accept_mutator(this)));
  }
  return alloc<Polynomial>(v->hasher(), scalar_new, variables_new);
}

ExprPtr IRCloner::mutate(const RoundOffPtr& v) {
  return alloc<RoundOff>(
      v->lhs()->accept_mutator(this), v->rhs()->accept_mutator(this));
}
```
- **EN**: This chunk defines `mutate`, which visits or transforms IR nodes while preserving the intended semantics. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `mutate`，其作用是访问或变换 IR 节点，同时保持预期语义。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 211-222
```cpp
ExprPtr IRCloner::mutate(const MaxTermPtr& v) {
  ExprPtr scalar_new =
      v->scalar() ? v->scalar()->accept_mutator(this) : nullptr;

  std::vector<ExprPtr> variables_new;
  variables_new.reserve(v->variables().size());
  for (const auto& t : v->variables()) {
    variables_new.push_back(t->accept_mutator(this));
  }
  return alloc<MaxTerm>(
      v->hasher(), scalar_new, v->propagate_nans(), variables_new);
}
```
- **EN**: This chunk defines `mutate`, which visits or transforms IR nodes while preserving the intended semantics. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `mutate`，其作用是访问或变换 IR 节点，同时保持预期语义。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 224-238
```cpp
ExprPtr IRCloner::mutate(const MinTermPtr& v) {
  ExprPtr scalar_new =
      v->scalar() ? v->scalar()->accept_mutator(this) : nullptr;

  std::vector<ExprPtr> variables_new;
  variables_new.reserve(v->variables().size());
  for (const auto& t : v->variables()) {
    variables_new.push_back(t->accept_mutator(this));
  }
  return alloc<MinTerm>(
      v->hasher(), scalar_new, v->propagate_nans(), variables_new);
}

ExprPtr IRCloner::mutate(const ReduceOpPtr& v) {
  ExprPtr body_new = v->body()->accept_mutator(this);
```
- **EN**: This chunk defines `mutate`, which visits or transforms IR nodes while preserving the intended semantics. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `mutate`，其作用是访问或变换 IR 节点，同时保持预期语义。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 240-252
```cpp
  std::vector<VarPtr> reduce_args_new;
  reduce_args_new.reserve(v->reduce_args().size());
  for (const auto& r : v->reduce_args()) {
    reduce_args_new.push_back(static_to<Var>(r->accept_mutator(this)));
  }

  return alloc<ReduceOp>(body_new, reduce_args_new, v->reducer());
}

StmtPtr IRCloner::mutate(const ForPtr& v) {
  auto start_new = v->start()->accept_mutator(this);
  auto stop_new = v->stop()->accept_mutator(this);
  auto body_new = v->body()->accept_mutator(this);
```
- **EN**: This chunk defines `mutate`, which visits or transforms IR nodes while preserving the intended semantics. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `mutate`，其作用是访问或变换 IR 节点，同时保持预期语义。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 254-264
```cpp
  return alloc<For>(v->var(), start_new, stop_new, body_new, v->loop_options());
}

StmtPtr IRCloner::mutate(const BlockPtr& v) {
  std::vector<StmtPtr> stmts_new;
  stmts_new.reserve(v->nstmts());
  for (const StmtPtr& stmt : *v) {
    stmts_new.push_back(stmt->accept_mutator(this));
  }
  return alloc<Block>(stmts_new);
}
```
- **EN**: This chunk defines `mutate`, which visits or transforms IR nodes while preserving the intended semantics. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `mutate`，其作用是访问或变换 IR 节点，同时保持预期语义。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 266-275
```cpp
StmtPtr IRCloner::mutate(const StorePtr& v) {
  std::vector<ExprPtr> indices_new;
  indices_new.reserve(v->indices().size());
  for (const auto& ind : v->indices()) {
    indices_new.push_back(ind->accept_mutator(this));
  }
  auto value_new = v->value()->accept_mutator(this);
  BufPtr buf_new = to<Buf>(v->buf()->accept_mutator(this));
  return alloc<Store>(buf_new, indices_new, value_new);
}
```
- **EN**: This chunk defines `mutate`, which visits or transforms IR nodes while preserving the intended semantics. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `mutate`，其作用是访问或变换 IR 节点，同时保持预期语义。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 277-291
```cpp
StmtPtr IRCloner::mutate(const AtomicAddPtr& v) {
  std::vector<ExprPtr> indices_new;
  indices_new.reserve(v->indices().size());
  for (const auto& ind : v->indices()) {
    indices_new.push_back(ind->accept_mutator(this));
  }
  auto value_new = v->value()->accept_mutator(this);
  BufPtr buf_new = to<Buf>(v->buf()->accept_mutator(this));
  return alloc<AtomicAdd>(buf_new, indices_new, value_new);
}

StmtPtr IRCloner::mutate(const AllocatePtr& v) {
  BufPtr buf_new = to<Buf>(v->buf()->accept_mutator(this));
  return alloc<Allocate>(buf_new);
}
```
- **EN**: This chunk defines `mutate`, which visits or transforms IR nodes while preserving the intended semantics. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `mutate`，其作用是访问或变换 IR 节点，同时保持预期语义。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 293-303
```cpp
StmtPtr IRCloner::mutate(const FreePtr& v) {
  BufPtr buf_new = to<Buf>(v->buf()->accept_mutator(this));
  return alloc<Free>(buf_new);
}

StmtPtr IRCloner::mutate(const SyncThreadsPtr& v) {
  return alloc<SyncThreads>();
}

StmtPtr IRCloner::mutate(const ExternalCallPtr& v) {
  BufPtr buf_new = to<Buf>(v->buf()->accept_mutator(this));
```
- **EN**: This chunk defines `mutate`, which visits or transforms IR nodes while preserving the intended semantics. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `mutate`，其作用是访问或变换 IR 节点，同时保持预期语义。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 305-317
```cpp
  std::vector<BufPtr> buf_args_new;
  buf_args_new.reserve(v->buf_args().size());
  for (const BufPtr& buf_arg : v->buf_args()) {
    buf_args_new.push_back(to<Buf>(buf_arg->accept_mutator(this)));
  }
  std::vector<ExprPtr> args_new;
  args_new.reserve(v->args().size());
  for (const ExprPtr& arg : v->args()) {
    args_new.push_back(arg->accept_mutator(this));
  }

  return alloc<ExternalCall>(buf_new, v->func_name(), buf_args_new, args_new);
}
```
- **EN**: This chunk continues `mutate` and expands its internal control flow or data movement. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `mutate`，进一步展开其内部控制流或数据流转。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 319-334
```cpp
StmtPtr IRCloner::mutate(const ExternalCallWithAllocPtr& v) {
  std::vector<BufPtr> buf_out_args_new;
  buf_out_args_new.reserve(v->buf_out_args().size());
  for (const auto& buf_out_arg : v->buf_out_args()) {
    buf_out_args_new.push_back(to<Buf>(buf_out_arg->accept_mutator(this)));
  }

  std::vector<BufPtr> buf_args_new;
  buf_args_new.reserve(v->buf_args().size());
  for (const auto& buf_arg : v->buf_args()) {
    buf_args_new.push_back(to<Buf>(buf_arg->accept_mutator(this)));
  }
  std::vector<ExprPtr> args_new;
  args_new.reserve(v->args().size());
  for (const auto& arg : v->args()) {
    args_new.push_back(arg->accept_mutator(this));
```
- **EN**: This chunk defines `mutate`, which visits or transforms IR nodes while preserving the intended semantics. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically.
- **CN**: 这一段定义了 `mutate`，其作用是访问或变换 IR 节点，同时保持预期语义。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。

### Lines 335-344
```cpp
  }

  return alloc<ExternalCallWithAlloc>(
      v->func_name(), buf_out_args_new, buf_args_new, args_new);
}

StmtPtr IRCloner::mutate(const LetPtr& v) {
  auto value_new = v->value()->accept_mutator(this);
  return alloc<Let>(v->var(), value_new);
}
```
- **EN**: This chunk defines `mutate`, which visits or transforms IR nodes while preserving the intended semantics. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `mutate`，其作用是访问或变换 IR 节点，同时保持预期语义。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 346-360
```cpp
StmtPtr IRCloner::mutate(const CondPtr& v) {
  auto condition_new = v->condition()->accept_mutator(this);
  StmtPtr true_old = v->true_stmt();
  StmtPtr false_old = v->false_stmt();
  StmtPtr true_new = true_old ? true_old->accept_mutator(this) : true_old;
  StmtPtr false_new = false_old ? false_old->accept_mutator(this) : false_old;
  return alloc<Cond>(condition_new, true_new, false_new);
}

StmtPtr Stmt::clone(const StmtPtr& s) {
  IRCloner cloner;
  StmtPtr cloned = s->accept_mutator(&cloner);
  set_parent(cloned, nullptr);
  return cloned;
}
```
- **EN**: This chunk defines `clone`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `clone`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 362-367
```cpp
ExprPtr Expr::clone(const ExprPtr& e) {
  IRCloner cloner;
  return e->accept_mutator(&cloner);
}

} // namespace torch::jit::tensorexpr
```
- **EN**: This chunk defines `clone`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `clone`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 末尾会返回累积结果，或把更新后的状态交还给调用方。

## Key Concepts / 关键概念

- **Tensor Expression IR**
  - EN: Implements Tensor Expression IR, analyses, scheduling, lowering, and backend code generation.
  - CN: 实现 Tensor Expression IR、分析、调度、降级以及后端代码生成。
- **unimplemented_lowering**
  - EN: `unimplemented_lowering` is a central symbol declared or implemented in this file.
  - CN: `unimplemented_lowering` 是本文件声明或实现的核心符号。
- **mutate**
  - EN: `mutate` is a central symbol declared or implemented in this file.
  - CN: `mutate` 是本文件声明或实现的核心符号。
- **IR simplification**
  - EN: Rewrites expressions and statements into simpler but equivalent forms.
  - CN: 把表达式和语句重写为更简单但等价的形式。
## Dependencies / 依赖关系

- **Internal PyTorch headers / 内部 PyTorch 头文件**: `torch/csrc/jit/tensorexpr/ir_cloner.h`, `torch/csrc/jit/tensorexpr/ir.h`, `torch/csrc/jit/tensorexpr/ir_simplifier.h`, `torch/csrc/jit/tensorexpr/reduction.h`
- **Primary symbols in this file / 本文件核心符号**: `unimplemented_lowering`, `mutate`
