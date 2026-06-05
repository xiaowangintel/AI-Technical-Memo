# ir.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/csrc/jit/tensorexpr/ir.cpp`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Defines or manipulates expression nodes in the Tensor Expression subsystem.
- **Purpose (CN)**: 定义或操作 Tensor Expression 子系统中的表达式节点。
## Line-by-Line Analysis / 逐行分析

### Lines 1-12
```cpp
#include <torch/csrc/jit/tensorexpr/ir.h>
#include <torch/csrc/jit/tensorexpr/stmt.h>

#include <c10/util/irange.h>

#include <utility>

namespace torch::jit::tensorexpr {

static Dtype ChooseDtype(const Dtype& buffer_dtype, const Dtype& index_dtype) {
  return Dtype(buffer_dtype, index_dtype.lanes());
}
```
- **EN**: This block assembles the compilation dependencies, pulling in local JIT headers such as torch/csrc/jit/tensorexpr/ir.h, torch/csrc/jit/tensorexpr/stmt.h; ATen/c10 facilities such as c10/util/irange.h; standard-library headers such as utility. The namespace declarations place the code inside torch::jit::tensorexpr, matching the surrounding JIT subsystem. This chunk defines `ChooseDtype`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段组织编译依赖，引入了本地 JIT 头文件，如 torch/csrc/jit/tensorexpr/ir.h、torch/csrc/jit/tensorexpr/stmt.h；ATen/c10 基础设施，如 c10/util/irange.h；标准库头文件，如 utility。 命名空间声明把代码放入 torch::jit::tensorexpr 中，与周边 JIT 子系统保持一致。 这一段定义了 `ChooseDtype`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 14-29
```cpp
static Dtype dtypeOfIndices(const std::vector<ExprPtr>& indices) {
  if (indices.empty()) {
    // Return something so we can handle scalar buffers.
    return kInt;
  }
  return indices.at(0)->dtype();
}

static void castIndicesToInts(std::vector<ExprPtr>& indices) {
  // Cast all indices to either Int or Long
  auto index_dtype = ScalarType::Int;
  for (auto& index : indices) {
    if (index->dtype().scalar_type() == ScalarType::Long) {
      // If any of the indexes is Long, cast all of them to Long
      index_dtype = ScalarType::Long;
      break;
```
- **EN**: This chunk defines `castIndicesToInts`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `castIndicesToInts`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 30-40
```cpp
    }
  }

  for (auto& index : indices) {
    const Dtype& dt = index->dtype();
    if (c10::isIntegralType(dt.scalar_type(), true) &&
        dt.scalar_type() != index_dtype) {
      index = alloc<Cast>(Dtype(index_dtype, dt.lanes()), index);
    }
  }
}
```
- **EN**: This chunk continues `castIndicesToInts` and expands its internal control flow or data movement. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段延续了 `castIndicesToInts`，进一步展开其内部控制流或数据流转。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 42-56
```cpp
Load::Load(Dtype dtype, BufPtr buf, std::vector<ExprPtr> indices)
    : ExprNodeBase(dtype), buf_(std::move(buf)), indices_(std::move(indices)) {
  castIndicesToInts(indices_);
}

Load::Load(const BufPtr& buf, const std::vector<ExprPtr>& indices)
    : Load(ChooseDtype(buf->dtype(), dtypeOfIndices(indices)), buf, indices) {}

ExprHandle Load::make(
    Dtype dtype,
    const BufHandle& buf,
    const std::vector<ExprHandle>& indices) {
  return ExprHandle(
      alloc<Load>(dtype, buf.node(), ExprHandleVectorToExprVector(indices)));
}
```
- **EN**: This chunk defines `make`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `make`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 58-69
```cpp
ExprHandle Load::make(
    const BufHandle& buf,
    const std::vector<ExprHandle>& indices) {
  return Load::make(buf.dtype(), buf, indices);
}

Store::Store(BufPtr buf, std::vector<ExprPtr> indices, ExprPtr value)
    : buf_(std::move(buf)),
      indices_(std::move(indices)),
      value_(std::move(value)) {
  castIndicesToInts(indices_);
}
```
- **EN**: This chunk defines `make`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `make`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 71-83
```cpp
StorePtr Store::make(
    const BufHandle& buf,
    const std::vector<ExprHandle>& indices,
    const ExprHandle& value) {
  return alloc<Store>(
      buf.node(), ExprHandleVectorToExprVector(indices), value.node());
}

StorePtr BufHandle::store(
    const std::vector<ExprHandle>& args,
    const ExprHandle& value) const {
  return Store::make(*this, args, value);
}
```
- **EN**: This chunk defines `store`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `store`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 85-100
```cpp
ExprPtr flatten_index(
    const std::vector<ExprPtr>& dims,
    const std::vector<ExprPtr>& indices,
    const std::vector<ExprPtr>& strides) {
  // Handle already flattened indices first
  if (indices.size() == 1) {
    return indices[0];
  }

  size_t ndim = dims.size();
  if (ndim != indices.size()) {
    throw malformed_input("dimensions mismatch in flatten_index");
  }
  if (ndim != strides.size()) {
    throw malformed_input("strides mismatch in flatten_index");
  }
```
- **EN**: This chunk defines `malformed_input`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `malformed_input`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 101-116
```cpp
  if (ndim == 0) {
    return alloc<LongImm>(0);
  }
  ExprPtr total_index = immLike(indices[0], 0);
  for (const auto i : c10::irange(ndim)) {
    total_index = alloc<Add>(total_index, alloc<Mul>(indices[i], strides[i]));
  }
  return total_index;
}

Dtype Intrinsics::IntrinsicsDtype(IntrinsicsOp op_type, Dtype dt1) {
  if (op_type == kIsNan) {
    return dt1.cloneWithScalarType(ScalarType::Int);
  }
  // TODO: check the op_type and make a real decision
  return dt1;
```
- **EN**: This chunk defines `IntrinsicsDtype`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `IntrinsicsDtype`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 117-132
```cpp
}

Dtype Intrinsics::IntrinsicsDtype(IntrinsicsOp op_type, Dtype dt1, Dtype dt2) {
  // TODO: check the op_type and make a real decision
  return dt1;
}

Dtype Intrinsics::IntrinsicsDtype(
    IntrinsicsOp op_type,
    const std::vector<ExprPtr>& params) {
  // TODO: check the op_type and make a real decision
  // Doesn't this fail with kRand?
  if (params.empty()) {
    throw malformed_input("invalid params in Intrinsics");
  } else if (params.size() == 1) {
    return IntrinsicsDtype(op_type, params[0]->dtype());
```
- **EN**: This chunk defines `malformed_input`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `malformed_input`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 133-148
```cpp
  } else if (params.size() == 2) {
    return IntrinsicsDtype(op_type, params[0]->dtype(), params[1]->dtype());
  }
  return params[0]->dtype();
}

size_t Intrinsics::OpArgCount(IntrinsicsOp op_type) {
  switch (op_type) {
    case kSin:
    case kCos:
    case kTan:
    case kAsin:
    case kAcos:
    case kAtan:
    case kSinh:
    case kCosh:
```
- **EN**: This chunk defines `OpArgCount`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Switch-based dispatch selects specialized behavior for each opcode, node kind, or schema case. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `OpArgCount`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 基于 switch 的分发为不同 opcode、节点类型或 schema 分支选择专门行为。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 149-164
```cpp
    case kTanh:
    case kSigmoid:
    case kExp:
    case kExpm1:
    case kAbs:
    case kLog:
    case kLog2:
    case kLog10:
    case kLog1p:
    case kErf:
    case kErfc:
    case kSqrt:
    case kRsqrt:
    case kCeil:
    case kFloor:
    case kRound:
```
- **EN**: This chunk continues `OpArgCount` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `OpArgCount`，进一步展开其内部控制流或数据流转。

### Lines 165-180
```cpp
    case kTrunc:
    case kFrac:
    case kLgamma:
    case kIsNan:
      return 1;
    case kRand:
      return 0;
    case kAtan2:
    case kFmod:
    case kPow:
    case kRemainder:
      return 2;
    default:
      throw std::runtime_error("invalid op_type: " + std::to_string(op_type));
  }
}
```
- **EN**: This chunk declares `runtime_error`, which executes prepared logic against runtime values or IR state. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段声明了 `runtime_error`，其作用是针对运行时数值或 IR 状态执行准备好的逻辑。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 182-194
```cpp
ExternalCallPtr ExternalCall::make(
    BufHandle buf,
    const std::string& func_name,
    const std::vector<BufHandle>& buf_args,
    const std::vector<ExprHandle>& args) {
  std::vector<BufPtr> buf_arg_nodes;
  buf_arg_nodes.reserve(buf_args.size());
  for (const BufHandle& buf_arg : buf_args) {
    buf_arg_nodes.push_back(buf_arg.node());
  }
  return alloc<ExternalCall>(
      buf.node(), func_name, buf_arg_nodes, ExprHandleVectorToExprVector(args));
}
```
- **EN**: This chunk defines `make`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `make`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 196-205
```cpp
ExternalCallWithAllocPtr ExternalCallWithAlloc::make(
    const std::string& func_name,
    const std::vector<BufHandle>& buf_out_args,
    const std::vector<BufHandle>& buf_args,
    const std::vector<ExprHandle>& args) {
  std::vector<BufPtr> buf_out_arg_nodes;
  buf_out_arg_nodes.reserve(buf_out_args.size());
  for (const BufHandle& buf_out_arg : buf_out_args) {
    buf_out_arg_nodes.push_back(buf_out_arg.node());
  }
```
- **EN**: This chunk defines `make`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically.
- **CN**: 这一段定义了 `make`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。

### Lines 207-217
```cpp
  std::vector<BufPtr> buf_arg_nodes;
  buf_arg_nodes.reserve(buf_args.size());
  for (const BufHandle& buf_arg : buf_args) {
    buf_arg_nodes.push_back(buf_arg.node());
  }
  return alloc<ExternalCallWithAlloc>(
      func_name,
      buf_out_arg_nodes,
      buf_arg_nodes,
      ExprHandleVectorToExprVector(args));
}
```
- **EN**: This chunk continues `make` and expands its internal control flow or data movement. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `make`，进一步展开其内部控制流或数据流转。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 219-234
```cpp
FreeExtPtr FreeExt::make(const std::vector<BufHandle>& bufs) {
  std::vector<BufPtr> buf_nodes;
  buf_nodes.reserve(bufs.size());
  for (const BufHandle& buf : bufs) {
    buf_nodes.push_back(buf.node());
  }
  return alloc<FreeExt>(buf_nodes);
}

std::vector<ExprPtr> ExprHandleVectorToExprVector(
    const std::vector<ExprHandle>& v) {
  std::vector<ExprPtr> result(v.size());
  for (const auto i : c10::irange(v.size())) {
    result[i] = v[i].node();
  }
  return result;
```
- **EN**: This chunk defines `result`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `result`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 235-244
```cpp
}

std::vector<ExprHandle> ExprVectorToExprHandleVector(
    const std::vector<ExprPtr>& v) {
  std::vector<ExprHandle> result(v.size());
  for (const auto i : c10::irange(v.size())) {
    result[i] = ExprHandle(v[i]);
  }
  return result;
}
```
- **EN**: This chunk defines `result`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `result`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 246-261
```cpp
std::vector<VarPtr> VarHandleVectorToVarVector(
    const std::vector<VarHandle>& v) {
  std::vector<VarPtr> result(v.size());
  for (const auto i : c10::irange(v.size())) {
    result[i] = v[i].node();
  }
  return result;
}

std::vector<VarHandle> VarVectorToVarHandleVector(
    const std::vector<VarPtr>& v) {
  std::vector<VarHandle> result(v.size());
  for (const auto i : c10::irange(v.size())) {
    result[i] = VarHandle(v[i]);
  }
  return result;
```
- **EN**: This chunk defines `VarVectorToVarHandleVector`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `VarVectorToVarHandleVector`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 262-272
```cpp
}

bool immediateIsNegative(const ExprPtr& e) {
#define TYPE_CASE(Type, Name)                \
  if (Name##ImmPtr imm = to<Name##Imm>(e)) { \
    return imm->value() < 0;                 \
  }
  AT_FORALL_SCALAR_TYPES_AND2(Half, BFloat16, TYPE_CASE);
#undef TYPE_CASE
  return false;
}
```
- **EN**: The preprocessor guard keeps declarations single-instanced when the header is included transitively. This chunk defines `immediateIsNegative`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 预处理器保护用于避免头文件在传递包含时被重复展开。 这一段定义了 `immediateIsNegative`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 274-289
```cpp
bool immediateIsPositive(const ExprPtr& e) {
#define TYPE_CASE(Type, Name)                \
  if (Name##ImmPtr imm = to<Name##Imm>(e)) { \
    return imm->value() > 0;                 \
  }
  AT_FORALL_SCALAR_TYPES_AND3(Bool, Half, BFloat16, TYPE_CASE)
#undef TYPE_CASE
  return false;
}

bool immediateIsZero(const ExprPtr& e) {
#define TYPE_CASE(Type, Name)                \
  if (Name##ImmPtr imm = to<Name##Imm>(e)) { \
    return imm->value() == 0;                \
  }
  AT_FORALL_SCALAR_TYPES_AND3(Bool, Half, BFloat16, TYPE_CASE)
```
- **EN**: The preprocessor guard keeps declarations single-instanced when the header is included transitively. This chunk defines `immediateIsZero`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 预处理器保护用于避免头文件在传递包含时被重复展开。 这一段定义了 `immediateIsZero`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 290-294
```cpp
#undef TYPE_CASE
  return false;
}

} // namespace torch::jit::tensorexpr
```
- **EN**: This chunk continues `immediateIsZero` and expands its internal control flow or data movement. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `immediateIsZero`，进一步展开其内部控制流或数据流转。 末尾会返回累积结果，或把更新后的状态交还给调用方。

## Key Concepts / 关键概念

- **Tensor Expression IR**
  - EN: Implements Tensor Expression IR, analyses, scheduling, lowering, and backend code generation.
  - CN: 实现 Tensor Expression IR、分析、调度、降级以及后端代码生成。
- **ChooseDtype**
  - EN: `ChooseDtype` is a central symbol declared or implemented in this file.
  - CN: `ChooseDtype` 是本文件声明或实现的核心符号。
- **dtypeOfIndices**
  - EN: `dtypeOfIndices` is a central symbol declared or implemented in this file.
  - CN: `dtypeOfIndices` 是本文件声明或实现的核心符号。
- **TorchScript runtime**
  - EN: Coordinates execution plans, stacks, and operator dispatch for scripted programs.
  - CN: 协调脚本程序的执行计划、栈以及算子分发。
## Dependencies / 依赖关系

- **Internal PyTorch headers / 内部 PyTorch 头文件**: `torch/csrc/jit/tensorexpr/ir.h`, `torch/csrc/jit/tensorexpr/stmt.h`
- **ATen/c10 foundations / ATen/c10 基础设施**: `c10/util/irange.h`
- **Standard library / 标准库**: `utility`
- **Primary symbols in this file / 本文件核心符号**: `ChooseDtype`, `dtypeOfIndices`, `castIndicesToInts`, `make`, `store`, `flatten_index`, `malformed_input`, `IntrinsicsDtype`
