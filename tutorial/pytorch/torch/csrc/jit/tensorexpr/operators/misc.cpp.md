# misc.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/csrc/jit/tensorexpr/operators/misc.cpp`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Defines operator wrappers, schemas, or dispatch helpers that connect TorchScript nodes to runtime kernels.
- **Purpose (CN)**: 定义算子包装、schema 或分发辅助逻辑，将 TorchScript 节点连接到运行时内核。
## Line-by-Line Analysis / 逐行分析

### Lines 1-23
```cpp
#include <torch/csrc/jit/tensorexpr/ir_simplifier.h>
#include <torch/csrc/jit/tensorexpr/kernel.h>
#include <torch/csrc/jit/tensorexpr/operators/misc.h>
#include <torch/csrc/jit/tensorexpr/tensor.h>

namespace torch::jit::tensorexpr {

int64_t normalizeAndCheckIndex(int64_t idx, int64_t list_size) {
  if (idx < 0) {
    // Handle negative indexing
    idx = list_size + idx;
  }

  if (idx < 0 || idx >= list_size) {
    TORCH_CHECK(false, "Invalid index ", idx, " for list_size", list_size);
  }
  return idx;
}

// Convert boolean to integer, if needed.
ExprHandle boolToInteger(const ExprHandle& x) {
  return x.dtype().scalar_type() == ScalarType::Bool ? cast<int>(x) : x;
}
```
- **EN**: This block assembles the compilation dependencies, pulling in local JIT headers such as torch/csrc/jit/tensorexpr/ir_simplifier.h, torch/csrc/jit/tensorexpr/kernel.h, torch/csrc/jit/tensorexpr/operators/misc.h, and 1 more. The namespace declarations place the code inside torch::jit::tensorexpr, matching the surrounding JIT subsystem. This chunk defines `boolToInteger`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段组织编译依赖，引入了本地 JIT 头文件，如 torch/csrc/jit/tensorexpr/ir_simplifier.h、torch/csrc/jit/tensorexpr/kernel.h、torch/csrc/jit/tensorexpr/operators/misc.h 等共 4 项。 命名空间声明把代码放入 torch::jit::tensorexpr 中，与周边 JIT 子系统保持一致。 这一段定义了 `boolToInteger`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 25-47
```cpp
ExprHandle promoteToDtype(ExprHandle e, ScalarType dt) {
  if (e.dtype().scalar_type() == dt) {
    return e;
  }

  switch (dt) {
#define TYPE_CASE(Type, Name) \
  case ScalarType::Name:      \
    e = cast<Type>(e);        \
    break;
    AT_FORALL_SCALAR_TYPES_AND3(Bool, Half, BFloat16, TYPE_CASE)
#undef TYPE_CASE
    case ScalarType::QUInt8:
      e = cast<c10::quint8>(e);
      break;
    case ScalarType::QInt8:
      e = cast<c10::qint8>(e);
      break;
    default:
      throw unsupported_dtype();
  }
  return e;
}
```
- **EN**: The preprocessor guard keeps declarations single-instanced when the header is included transitively. This chunk defines `unsupported_dtype`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Switch-based dispatch selects specialized behavior for each opcode, node kind, or schema case. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 预处理器保护用于避免头文件在传递包含时被重复展开。 这一段定义了 `unsupported_dtype`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 基于 switch 的分发为不同 opcode、节点类型或 schema 分支选择专门行为。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 49-68
```cpp
static bool checkTypes(const ScalarType highType, const int typeConstraints) {
  if (typeConstraints == kAllTypes) {
    return true;
  }

  if (c10::isIntegralType(highType, false)) {
    return (typeConstraints & kIntegralTypes) != 0;
  } else if (c10::isFloatingType(highType)) {
    return (typeConstraints & kFloatingPointTypes) != 0;
  } else if (highType == ScalarType::Bool) {
    return (typeConstraints & kBoolType) != 0;
  }

  // assume JIT not supporting complex and qint yet
  TORCH_INTERNAL_ASSERT(
      (typeConstraints & (kQintTypes | kComplexTypes)) == 0,
      buildErrorMessage(
          "Qint and Complex types are not supported in the fuser."));
  return false;
}
```
- **EN**: This chunk defines `checkTypes`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `checkTypes`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 70-90
```cpp
static bool isScalar(const ExprHandle& e) {
  auto n = e.node();
  return n->isConstant() || to<Var>(n);
}

ExprHandle promoteHalfToFloat(const ExprHandle& e) {
  auto scalarType = static_cast<c10::ScalarType>(e.dtype().scalar_type());
  auto floatType = static_cast<c10::ScalarType>(tensorexpr::ScalarType::Float);
  if (c10::isFloatingType(scalarType) &&
      (c10::elementSize(scalarType) < c10::elementSize(floatType))) {
    return Cast::make(
        Dtype(tensorexpr::ScalarType::Float, e.dtype().lanes()), e);
  } else {
    return e;
  }
}

void promoteInputs(std::vector<ExprHandle>& inputs, const int typeConstraints) {
  if (inputs.empty()) {
    return;
  }
```
- **EN**: This chunk defines `promoteInputs`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `promoteInputs`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 92-114
```cpp
  // Find the highest type among the inputs.
  ScalarType highType = inputs[0].dtype().scalar_type();
  for (const auto& input : inputs) {
    auto inputType = input.dtype().scalar_type();
    if (isScalar(input)) {
      if (isIntegralType(highType, false) && isFloatingType(inputType)) {
        highType = c10::get_default_dtype_as_scalartype();
      } else if (highType == c10::kBool) {
        highType = inputType;
      }
    } else {
      highType = promoteTypes(highType, inputType);
    }
  }

  if (!checkTypes(highType, typeConstraints)) {
    throw unsupported_dtype();
  }

  for (ExprHandle& e : inputs) {
    e = promoteToDtype(e, highType);
  }
}
```
- **EN**: This chunk defines `unsupported_dtype`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages.
- **CN**: 这一段定义了 `unsupported_dtype`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。

### Lines 116-132
```cpp
ExprHandle promoteIntegerToDefaultType(const ExprHandle& e) {
  auto scalarType = static_cast<c10::ScalarType>(e.dtype().scalar_type());
  if (!c10::isIntegralType(scalarType, /*includeBool*/ true)) {
    return e;
  }

  auto defaultType = c10::typeMetaToScalarType(c10::get_default_dtype());

  // We intend to promote Integers to floating-point types
  TORCH_INTERNAL_ASSERT(
      !c10::isIntegralType(defaultType, /*includeBool*/ true));

  return Cast::make(
      Dtype(
          static_cast<tensorexpr::ScalarType>(defaultType), e.dtype().lanes()),
      e);
}
```
- **EN**: This chunk defines `promoteIntegerToDefaultType`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `promoteIntegerToDefaultType`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 134-155
```cpp
ExprHandle demoteOutput(
    const ExprHandle& e,
    const std::optional<ScalarType> type) {
  if (!type.has_value()) {
    return e;
  }
  if (*type == e.dtype().scalar_type()) {
    return e;
  }

  switch (*type) {
#define TYPE_CASE(Type, Name) \
  case ScalarType::Name:      \
    return cast<Type>(e);
    AT_FORALL_SCALAR_TYPES_AND2(Half, BFloat16, TYPE_CASE);
#undef TYPE_CASE
    case ScalarType::Bool:
      return cast<bool>(e);
    default:
      throw unsupported_dtype();
  }
}
```
- **EN**: The preprocessor guard keeps declarations single-instanced when the header is included transitively. This chunk defines `unsupported_dtype`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Switch-based dispatch selects specialized behavior for each opcode, node kind, or schema case. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 预处理器保护用于避免头文件在传递包含时被重复展开。 这一段定义了 `unsupported_dtype`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 基于 switch 的分发为不同 opcode、节点类型或 schema 分支选择专门行为。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 157-177
```cpp
std::optional<TensorInfo> getTensorInfo(const BufHandle& b) {
  std::vector<int64_t> dims;
  auto b_dims = b.dims();
  dims.reserve(b_dims.size());
  for (auto dim : b_dims) {
    auto val = intValue(dim.node());
    if (!val) {
      return std::nullopt;
    }
    dims.push_back(*val);
  }
  return TensorInfo{dims, static_cast<at::ScalarType>(b.dtype().scalar_type())};
}

ExprHandle clamp(
    const ExprHandle& cmin,
    const ExprHandle& cmax,
    const ExprHandle& input) {
  auto mm = CompareSelect::make(input, cmin, cmin, input, kLT);
  return CompareSelect::make(mm, cmax, cmax, mm, kGT);
}
```
- **EN**: This chunk defines `clamp`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `clamp`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 179-202
```cpp
static bool isOne(const ExprHandle& e) {
  auto const& n = intValue(e);
  if (!n) {
    return false;
  }
  return *n == 1;
}

static std::pair<std::vector<ExprHandle>, bool> broadcastShapesImpl(
    const std::vector<ExprHandle>& a,
    const std::vector<ExprHandle>& b) {
  auto at = a.rbegin();
  auto bt = b.rbegin();
  std::vector<ExprHandle> ret;
  bool hasBroadcast = false;
  while (at != a.rend() || bt != b.rend()) {
    if (at == a.rend()) {
      hasBroadcast = true;
      ret.push_back(*bt++);
      continue;
    }
    if (bt == b.rend()) {
      hasBroadcast = true;
      ret.push_back(*at++);
```
- **EN**: This chunk defines `isOne`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `isOne`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 203-221
```cpp
      continue;
    }
    // TODO: if neither *at nor *bt is 1, ensure they are identical
    // expressions.  Nb: `==` doesn't work since that simply produces a new
    // ExprHandle.
    ExprHandle dim = *at;
    if (isOne(*at)) {
      if (!isOne(*bt)) {
        dim = *bt;
        hasBroadcast = true;
      }
    }
    ret.push_back(dim);
    at++;
    bt++;
  }
  std::reverse(ret.begin(), ret.end());
  return {ret, hasBroadcast};
}
```
- **EN**: This chunk continues `isOne` and expands its internal control flow or data movement. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `isOne`，进一步展开其内部控制流或数据流转。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 223-245
```cpp
static std::pair<std::vector<ExprHandle>, bool> broadcastShapesImpl(
    std::vector<std::vector<ExprHandle>> shapes) {
  size_t n = shapes.size();
  if (n == 1) {
    return {shapes[0], false};
  }
  auto res1 = broadcastShapesImpl(shapes[n - 2], shapes[n - 1]);
  shapes[n - 2] = res1.first;
  shapes.pop_back();
  auto res2 = broadcastShapesImpl(shapes);
  return {res2.first, (res1.second || res2.second)};
}

std::vector<ExprHandle> broadcastShapes(
    std::vector<std::vector<ExprHandle>> shapes) {
  return broadcastShapesImpl(std::move(shapes)).first;
}

std::vector<ExprHandle> broadcastShapes(
    const std::vector<ExprHandle>& a,
    const std::vector<ExprHandle>& b) {
  return broadcastShapesImpl(a, b).first;
}
```
- **EN**: This chunk defines `broadcastShapes`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `broadcastShapes`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 247-268
```cpp
std::vector<ExprHandle> valueShape(const ArgValue& v) {
  if (auto b = std::get_if<tensorexpr::BufHandle>(&v)) {
    return b->dims();
  }
  return {};
}

ExprHandle tensorOrConstant(
    const ArgValue& v,
    const std::vector<ExprHandle>& axes) {
  if (auto b = std::get_if<BufHandle>(&v)) {
    return broadcast(*b, axes);
  }
  return constant(v);
}

ExprHandle scalarOrConstant(const ArgValue& v) {
  if (auto vh = std::get_if<VarHandle>(&v)) {
    return *vh;
  }
  return constant(v);
}
```
- **EN**: This chunk defines `scalarOrConstant`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `scalarOrConstant`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 270-291
```cpp
ExprHandle broadcast(const BufHandle& b, const std::vector<ExprHandle>& axes) {
  return b.load(computeIndicesToBroadcast(axes, b.dims()));
}

ExprHandle constant(const ArgValue& v) {
  if (auto s = std::get_if<tensorexpr::VarHandle>(&v)) {
    return *s;
  } else if (auto d = std::get_if<double>(&v)) {
    return DoubleImm::make(*d);
  } else if (auto i = std::get_if<int64_t>(&v)) {
    return LongImm::make(*i);
  } else if (auto b = std::get_if<bool>(&v)) {
    return BoolImm::make(*b);
  } else if (std::get_if<ArgNone>(&v)) {
    // This is just a placeholder so we don't throw.  None-handling
    // is operator-specific and should be handled properly in
    // the operator-specific lowering code.
    return IntImm::make(0);
  } else {
    throw unsupported_dtype("Trying to convert unsupported dtype to constant");
  }
}
```
- **EN**: This chunk defines `unsupported_dtype`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `unsupported_dtype`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 293-314
```cpp
std::vector<ExprHandle> computeIndicesToBroadcast(
    const std::vector<ExprHandle>& outputAxes,
    const std::vector<ExprHandle>& inputSizes) {
  if (outputAxes.size() < inputSizes.size()) {
    throw malformed_input("Cannot broadcast to a lower rank tensor");
  }
  std::vector<ExprHandle> bcast;
  auto axisIt = outputAxes.rbegin();
  auto sizeIt = inputSizes.rbegin();
  while (sizeIt != inputSizes.rend()) {
    auto const& size = intValue(*sizeIt);
    if (size && *size == 1) {
      bcast.emplace_back(LongImm::make(0));
    } else {
      bcast.emplace_back(*axisIt);
    }
    ++axisIt;
    ++sizeIt;
  }
  std::reverse(bcast.begin(), bcast.end());
  return bcast;
}
```
- **EN**: This chunk defines `malformed_input`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `malformed_input`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 316-334
```cpp
Tensor computeChunk(
    const std::vector<ArgValue>& inputs,
    const std::vector<ExprHandle>& outputShape,
    const std::vector<ExprHandle>& outputStrides,
    const std::optional<ScalarType>& outputType,
    at::Device device) {
  return Compute(
      "prim_constantchunk",
      outputShape,
      [inputs](const std::vector<VarHandle>& axes) {
        const auto& b = std::get<BufHandle>(inputs[0]);
        int64_t chunkIdx = std::get<int64_t>(inputs[1]);
        int64_t dim = std::get<int64_t>(inputs[2]);
        int64_t chunks = std::get<int64_t>(inputs[3]);
        std::vector<ExprHandle> indices(axes.begin(), axes.end());

        auto norm_dim = normalizeAndCheckIndex(dim, indices.size());
        auto buf_info = getTensorInfo(b);
        size_t step = buf_info->dims[norm_dim] / chunks;
```
- **EN**: This chunk defines `indices`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `indices`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 336-359
```cpp
        std::vector<ExprHandle> new_indices;
        for (int64_t i = 0; i < static_cast<int64_t>(indices.size()); ++i) {
          if (i == norm_dim) {
            new_indices.push_back(
                indices[i] + ExprHandle(immLike(indices[i], chunkIdx * step)));
          } else {
            new_indices.push_back(indices[i]);
          }
        }

        return b.load(new_indices);
      });
}

Tensor computeTranspose(
    const std::vector<ArgValue>& inputs,
    const std::vector<ExprHandle>& outputShape,
    const std::vector<ExprHandle>& outputStrides,
    const std::optional<ScalarType>& outputType,
    at::Device device) {
  auto A = std::get<BufHandle>(inputs[0]);
  // Trivial case of 0-dim and 1-dim tensors: transpose is just a copy
  if (A.ndim() <= 1) {
    return Compute(
```
- **EN**: This chunk defines `computeTranspose`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `computeTranspose`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 360-375
```cpp
        "aten_transpose", outputShape, [&](const std::vector<VarHandle>& axes) {
          TORCH_INTERNAL_ASSERT(
              axes.size() <= 1,
              buildErrorMessage("Invalid axes size in transpose"));
          return A.load(axes);
        });
  }
  // Usual case where transpose actually swaps dimensions
  auto start_dim = at::maybe_wrap_dim(std::get<int64_t>(inputs[1]), A.ndim());
  auto to_dim = at::maybe_wrap_dim(std::get<int64_t>(inputs[2]), A.ndim());
  return Compute(
      "aten_transpose", outputShape, [&](std::vector<VarHandle> axes) {
        std::swap(axes[start_dim], axes[to_dim]);
        return A.load(axes);
      });
}
```
- **EN**: This chunk continues `computeTranspose` and expands its internal control flow or data movement. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `computeTranspose`，进一步展开其内部控制流或数据流转。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 377-400
```cpp
Tensor computeExpand(
    const std::vector<ArgValue>& inputs,
    const std::vector<ExprHandle>& outputShape,
    const std::vector<ExprHandle>& outputStrides,
    const std::optional<ScalarType>& outputType,
    at::Device device) {
  auto A = std::get<BufHandle>(inputs[0]);
  return Compute(
      "aten_expand", outputShape, [&](const std::vector<VarHandle>& axes) {
        std::vector<ExprHandle> indices(axes.begin(), axes.end());
        return broadcast(A, indices);
      });
}

Tensor computeReshape(
    const std::vector<ArgValue>& inputs,
    const std::vector<ExprHandle>& outputShape,
    const std::vector<ExprHandle>& outputStrides,
    const std::optional<ScalarType>& outputType,
    at::Device device) {
  auto A = std::get<BufHandle>(inputs[0]);
  if (A.ndim() == 0) {
    return Compute(
        "aten_view", outputShape, [&](const std::vector<VarHandle>& axes) {
```
- **EN**: This chunk defines `computeReshape`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `computeReshape`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 401-424
```cpp
          std::vector<ExprHandle> empty_indices;
          return A.load(empty_indices);
        });
  }
  return Compute(
      "aten_reshape", outputShape, [&](const std::vector<VarHandle>& axes) {
        std::vector<VarHandle> new_axes;
        assert(outputShape.size() == axes.size());
        /*
        Example for the index transformation. Assume we have a tensor A and
        its view B:
          A.size() = [6,2,3]
          B = A.view(2,1,9,1,2)

        In TE IR we would want to represent B as the following loopnest:
          for (i1 in 0..2)
            for (i2 in 0..1)
              for (i3 in 0..9)
                for (i4 in 0..1)
                  for (i5 in 0..2)
                    idx = i5 + i4*2 + i3*2 + i2*18 + i1*18
                    B[i1,i2,i3,i4,i5] = A[idx/(3*2), (idx/3)%2, idx%3]
        */
        std::vector<ExprPtr> dims, indices;
```
- **EN**: This chunk continues `computeReshape` and expands its internal control flow or data movement. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `computeReshape`，进一步展开其内部控制流或数据流转。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 425-448
```cpp
        for (size_t idx = 0; idx < outputShape.size(); idx++) {
          dims.push_back(outputShape[idx].node());
          indices.push_back(axes[idx].node());
        }

        auto ndim = dims.size();
        std::vector<ExprPtr> strides(ndim);
        strides[ndim - 1] = immLike(dims[ndim - 1], 1);
        for (size_t i = 1; i < ndim; i++) {
          strides[ndim - 1 - i] = alloc<Mul>(strides[ndim - i], dims[ndim - i]);
        }

        ExprHandle flat_idx = ExprHandle(flatten_index(dims, indices, strides));
        std::vector<ExprHandle> orig_buf_indexes(A.ndim(), ExprHandle(0));
        ExprHandle stride = ExprHandle(immLike(flat_idx, 1));
        for (size_t idx = 0; idx < A.ndim(); idx++) {
          size_t dim_idx = A.ndim() - idx - 1;
          // We don't need to generate mod-div for the first dimension -
          // ideally IRSimplifier would get rid of that for us, but for now
          // let's just avoid generating it in the first place.
          if (dim_idx > 0) {
            orig_buf_indexes[dim_idx] = flat_idx / stride % A.dim(dim_idx);
          } else {
            orig_buf_indexes[dim_idx] = flat_idx / stride;
```
- **EN**: This chunk defines `orig_buf_indexes`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段定义了 `orig_buf_indexes`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 449-472
```cpp
          }
          // In the example above the stride is initially 1 for dim_idx = 2,
          // then it's 3 for dim_idx = 1, and then it's 3*2 for dim_idx = 0.
          stride = stride * A.dim(dim_idx);
        }
        return A.load(orig_buf_indexes);
      });
}

Tensor computeFlatten(
    const std::vector<ArgValue>& inputs,
    const std::vector<ExprHandle>& outputShape,
    const std::vector<ExprHandle>& outputStrides,
    const std::optional<ScalarType>& outputType,
    at::Device device) {
  std::vector<int64_t> outputShapeVec;
  for (const auto dim : c10::irange(outputShape.size())) {
    outputShapeVec.push_back(outputShape[dim].AsNode<LongImm>()->value());
  }
  std::vector<ArgValue> reshapeInputs;
  reshapeInputs.push_back(inputs[0]);
  reshapeInputs.emplace_back(outputShapeVec);
  return computeReshape(
      reshapeInputs, outputShape, outputStrides, outputType, device);
```
- **EN**: This chunk defines `computeFlatten`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `computeFlatten`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 473-496
```cpp
}

static std::pair<ScalarType, std::vector<BufHandle>> processCatList(
    const std::vector<BufHandle>& bufList) {
  if (bufList.empty()) {
    throw std::runtime_error("Empty input list is passed to aten::cat");
  }
  std::vector<BufHandle> bufInputs;
  std::vector<BufHandle> nonEmptyInputs;
  for (auto buf : bufList) {
    bufInputs.push_back(buf);
    TORCH_INTERNAL_ASSERT(
        !buf.node()->dims().empty(), buildErrorMessage("Invalid buf rank"));
    // Ignore buffers that are 0-sized on any dimension.
    bool hasEmptyDims = false;
    for (const auto& dim : buf.dims()) {
      if (dim.AsNode<LongImm>() && immediateAs<int64_t>(dim) == 0ll) {
        hasEmptyDims = true;
        break;
      }
    }
    if (!hasEmptyDims) {
      nonEmptyInputs.push_back(buf);
    }
```
- **EN**: This chunk defines `runtime_error`, which executes prepared logic against runtime values or IR state. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages.
- **CN**: 这一段定义了 `runtime_error`，其作用是针对运行时数值或 IR 状态执行准备好的逻辑。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。

### Lines 497-514
```cpp
  }
  ScalarType highType = bufInputs[0].dtype().scalar_type();
  for (const auto& input : bufInputs) {
    auto maybe_dtype = input.dtype().scalar_type();
    highType = promoteTypes(highType, maybe_dtype);
  }
  return {highType, nonEmptyInputs};
}

static Tensor computeCatWoConditionals(
    const std::vector<ArgValue>& inputs,
    const std::vector<ExprHandle>& outputShape,
    const std::vector<ExprHandle>& outputStrides) {
  auto const& input_list = std::get<BufList>(inputs[0]);
  auto arg_dim = inputs[1];
  auto cat_info = processCatList(input_list);
  ScalarType high_type = cat_info.first;
  std::vector<BufHandle> non_empty_inputs = cat_info.second;
```
- **EN**: This chunk defines `computeCatWoConditionals`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `computeCatWoConditionals`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 516-539
```cpp
  // Now we build one loop per input:
  //
  // for i
  //   for j
  //     for k
  //       output[i,j,k] = inp1[i,j,k]
  // for i
  //   for j
  //     for k
  //       output[i,j+l1,k] = inp2[i,j,k]
  // for i
  //   for j
  //     for k
  //       output[i,j+l2,k] = inp3[i,j,k]

  auto output_sizes_expr = ExprHandleVectorToExprVector(outputShape);
  auto output_strides_expr = ExprHandleVectorToExprVector(outputStrides);
  auto output_buf = alloc<Buf>(
      "aten_cat",
      output_sizes_expr,
      ToDtype(high_type),
      nullptr,
      output_strides_expr);
  if (non_empty_inputs.empty()) {
```
- **EN**: This chunk continues `computeCatWoConditionals` and expands its internal control flow or data movement. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段延续了 `computeCatWoConditionals`，进一步展开其内部控制流或数据流转。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 540-562
```cpp
    return Tensor(
        output_buf, alloc<tensorexpr::Block>(std::vector<StmtPtr>({})));
  }

  int64_t concat_dim = std::get<int64_t>(arg_dim);
  auto norm_concat_dim = normalizeAndCheckIndex(concat_dim, outputShape.size());

  auto loop_order_fn = [&](const BufPtr& buf_) {
    std::vector<int32_t> loop_order;
    if (buf_->is_contiguous()) {
      for (int32_t i = buf_->ndim() - 1; i >= 0; i--) {
        loop_order.push_back(i);
      }
    } else if (buf_->is_contiguous(c10::MemoryFormat::ChannelsLast)) {
      loop_order = {1, 3, 2, 0};
    } else if (buf_->is_contiguous(c10::MemoryFormat::ChannelsLast3d)) {
      loop_order = {1, 4, 3, 2, 0};
    } else {
      loop_order = {1, 2, 0};
    }

    return loop_order;
  };
```
- **EN**: This chunk continues `computeCatWoConditionals` and expands its internal control flow or data movement. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `computeCatWoConditionals`，进一步展开其内部控制流或数据流转。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 564-585
```cpp
  auto gen_code_for_input = [&](const BufHandle& inp,
                                size_t inp_pos,
                                const ExprPtr& concat_dim_size,
                                const std::vector<ExprHandle>& dims) {
    std::vector<VarPtr> for_vars(dims.size());
    std::vector<ExprPtr> load_indices(dims.size());
    std::vector<ExprPtr> store_indices(dims.size());
    for (int64_t i = 0; i < static_cast<int64_t>(dims.size()); ++i) {
      for_vars[i] = alloc<Var>(
          "i" + std::to_string(inp_pos) + "_" + std::to_string(i),
          dims[i].dtype());
      load_indices[i] = for_vars[i];
      if (i == norm_concat_dim) {
        store_indices[i] = alloc<Add>(for_vars[i], concat_dim_size);
      } else {
        store_indices[i] = for_vars[i];
      }
    }
    auto inp_buf = inp.node();
    auto load_expr = alloc<Load>(inp_buf, load_indices);
    auto load_promoted = promoteToDtype(ExprHandle(load_expr), high_type);
    StmtPtr st = alloc<Store>(output_buf, store_indices, load_promoted.node());
```
- **EN**: This chunk defines `store_indices`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段定义了 `store_indices`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 587-610
```cpp
    auto loop_order = loop_order_fn(inp.node());
    for (auto dim_index : loop_order) {
      st = alloc<For>(
          for_vars[dim_index],
          immLike(dims[dim_index], 0),
          dims[dim_index].node(),
          st);
    }

    return st;
  };

  ExprPtr concat_dim_size = nullptr;
  auto block = alloc<tensorexpr::Block>(std::vector<StmtPtr>({}));
  for (size_t i = 0; i < non_empty_inputs.size(); ++i) {
    auto input_dims =
        ExprVectorToExprHandleVector(non_empty_inputs[i].node()->dims());
    if (concat_dim_size == nullptr) {
      concat_dim_size = immLike(input_dims[norm_concat_dim], 0);
    }
    block->append_stmt(gen_code_for_input(
        non_empty_inputs[i], i, concat_dim_size, input_dims));
    concat_dim_size =
        alloc<Add>(concat_dim_size, input_dims[norm_concat_dim].node());
```
- **EN**: This chunk continues `store_indices` and expands its internal control flow or data movement. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `store_indices`，进一步展开其内部控制流或数据流转。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 611-634
```cpp
  }
  return Tensor(output_buf, IRSimplifier::simplify(block));
}

Tensor computeCat(
    const std::vector<ArgValue>& inputs,
    const std::vector<ExprHandle>& outputShape,
    const std::vector<ExprHandle>& outputStrides,
    const std::optional<ScalarType>& outputType,
    at::Device device) {
  if (device == at::kCPU && getCatWoConditionals()) {
    return computeCatWoConditionals(inputs, outputShape, outputStrides);
  }
  auto const& inputList = std::get<BufList>(inputs[0]);
  auto argDim = inputs[1];
  auto catInfo = processCatList(inputList);
  ScalarType highType = catInfo.first;
  std::vector<BufHandle> nonEmptyInputs = catInfo.second;
  return Compute(
      "aten_cat",
      outputShape,
      outputStrides,
      [&](const std::vector<VarHandle>& axes) {
        if (nonEmptyInputs.empty()) {
```
- **EN**: This chunk defines `computeCat`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `computeCat`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 635-658
```cpp
          return ExprHandle(0);
        }

        int64_t dim_ = std::get<int64_t>(argDim);
        auto dim = normalizeAndCheckIndex(dim_, axes.size());
        // Promote input types.
        // Note that we need to consider all inputs, including empty - they
        // also affect the resultant dtype.

        // Now we know the final dtype, we know what inputs are non-empty,
        // and we know that there is at least one such an input. With all
        // that we construct a tensor expression performing the
        // concatenation.
        // The expression we build here is a cascading if-then-else that
        // essentially represents:
        //
        //              inp1[i, j, k]         if 0   < i < l1,
        // out[i,j,k] = inp2[i, j-l1, k]      if l1 =< i < l1 + l2,
        //              ...
        //              inpN[i, j-l_N_1, k]   if l1+l2+...l_N_1  < i
        // where l_i is the corresponding size of the i-th input.
        std::vector<ExprHandle> newAxes(axes.begin(), axes.end());
        ExprHandle load = promoteToDtype(
            tensorOrConstant(nonEmptyInputs[0], newAxes), highType);
```
- **EN**: This chunk declares `newAxes`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段声明了 `newAxes`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 659-675
```cpp
        auto offset = ExprHandle(nonEmptyInputs[0].node()->dim(dim));
        newAxes[dim] = newAxes[dim] - offset;

        for (size_t ii = 1; ii < nonEmptyInputs.size(); ++ii) {
          auto input = nonEmptyInputs[ii];
          load = ifThenElse(
              CompareSelect::make(axes[dim], offset, kLT),
              load,
              promoteToDtype(tensorOrConstant(input, newAxes), highType));

          offset = offset + ExprHandle(input.node()->dim(dim));
          newAxes[dim] = axes[dim] - offset;
        }

        return load;
      });
}
```
- **EN**: This chunk continues `newAxes` and expands its internal control flow or data movement. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `newAxes`，进一步展开其内部控制流或数据流转。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 677-697
```cpp
Tensor computeEmbedding(
    const std::vector<ArgValue>& inputs,
    const std::vector<ExprHandle>& outputShape,
    const std::vector<ExprHandle>& outputStrides,
    const std::optional<ScalarType>& outputType,
    at::Device device) {
  Dtype dtype = kFloat;
  if (outputType) {
    dtype = Dtype(*outputType);
  }

  BufHandle ResultBuf("emb", outputShape, dtype);
  const BufHandle& w = std::get<BufHandle>(inputs[0]);
  const BufHandle& indices = std::get<BufHandle>(inputs[1]);

  StmtPtr s =
      ExternalCall::make(ResultBuf, "nnc_aten_embedding", {w, indices}, {});
  return Tensor(ResultBuf.node(), s);
}

} // namespace torch::jit::tensorexpr
```
- **EN**: This chunk defines `ResultBuf`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `ResultBuf`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

## Key Concepts / 关键概念

- **Tensor Expression IR**
  - EN: Implements Tensor Expression IR, analyses, scheduling, lowering, and backend code generation.
  - CN: 实现 Tensor Expression IR、分析、调度、降级以及后端代码生成。
- **normalizeAndCheckIndex**
  - EN: `normalizeAndCheckIndex` is a central symbol declared or implemented in this file.
  - CN: `normalizeAndCheckIndex` 是本文件声明或实现的核心符号。
- **boolToInteger**
  - EN: `boolToInteger` is a central symbol declared or implemented in this file.
  - CN: `boolToInteger` 是本文件声明或实现的核心符号。
- **Operator dispatch**
  - EN: Bridges symbolic operator names or schemas to concrete C++ implementations.
  - CN: 把符号化算子名或 schema 连接到具体的 C++ 实现。
- **IR simplification**
  - EN: Rewrites expressions and statements into simpler but equivalent forms.
  - CN: 把表达式和语句重写为更简单但等价的形式。
## Dependencies / 依赖关系

- **Internal PyTorch headers / 内部 PyTorch 头文件**: `torch/csrc/jit/tensorexpr/ir_simplifier.h`, `torch/csrc/jit/tensorexpr/kernel.h`, `torch/csrc/jit/tensorexpr/operators/misc.h`, `torch/csrc/jit/tensorexpr/tensor.h`
- **Primary symbols in this file / 本文件核心符号**: `normalizeAndCheckIndex`, `boolToInteger`, `promoteToDtype`, `unsupported_dtype`, `checkTypes`, `isScalar`, `promoteHalfToFloat`, `promoteInputs`
