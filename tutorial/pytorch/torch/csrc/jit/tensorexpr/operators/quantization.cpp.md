# quantization.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/csrc/jit/tensorexpr/operators/quantization.cpp`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Defines operator wrappers, schemas, or dispatch helpers that connect TorchScript nodes to runtime kernels.
- **Purpose (CN)**: 定义算子包装、schema 或分发辅助逻辑，将 TorchScript 节点连接到运行时内核。
## Line-by-Line Analysis / 逐行分析

### Lines 1-18
```cpp
#include <c10/core/ScalarType.h>
#include <torch/csrc/jit/tensorexpr/ir_simplifier.h>
#include <torch/csrc/jit/tensorexpr/operators/misc.h>
#include <torch/csrc/jit/tensorexpr/operators/pointwise.h>
#include <torch/csrc/jit/tensorexpr/operators/quantization.h>

using namespace torch::jit::tensorexpr;

namespace torch::jit::tensorexpr {
namespace {
std::vector<int64_t> _pair_int(ArgValue v) {
  if (auto t = std::get_if<IntList>(&v)) {
    return {(*t)[0], (*t)[1]};
  }
  auto i = std::get<int64_t>(v);
  return {i, i};
}
} // namespace
```
- **EN**: This block assembles the compilation dependencies, pulling in local JIT headers such as torch/csrc/jit/tensorexpr/ir_simplifier.h, torch/csrc/jit/tensorexpr/operators/misc.h, torch/csrc/jit/tensorexpr/operators/pointwise.h, and 1 more; ATen/c10 facilities such as c10/core/ScalarType.h. The namespace declarations place the code inside torch::jit::tensorexpr, matching the surrounding JIT subsystem. This chunk defines `_pair_int`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段组织编译依赖，引入了本地 JIT 头文件，如 torch/csrc/jit/tensorexpr/ir_simplifier.h、torch/csrc/jit/tensorexpr/operators/misc.h、torch/csrc/jit/tensorexpr/operators/pointwise.h 等共 4 项；ATen/c10 基础设施，如 c10/core/ScalarType.h。 命名空间声明把代码放入 torch::jit::tensorexpr 中，与周边 JIT 子系统保持一致。 这一段定义了 `_pair_int`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 20-38
```cpp
double immQScale(const BufHandle& qx) {
  TORCH_INTERNAL_ASSERT(
      qx.node()->qscale(), buildErrorMessage("Expects BufHandle with qscale"));
  return to<DoubleImm>(IRSimplifier::simplify(qx.node()->qscale()))->value();
}

int64_t immQZero(const BufHandle& qx) {
  TORCH_INTERNAL_ASSERT(
      qx.node()->qzero(), buildErrorMessage("Expects BufHandle with qzero"));
  return to<LongImm>(IRSimplifier::simplify(qx.node()->qzero()))->value();
}

ScalarType immQDType(const BufHandle& qx) {
  return qx.dtype().scalar_type();
}

bool isQuantized(const BufHandle& qx) {
  return qx.node()->qscale() && qx.node()->qzero();
}
```
- **EN**: This chunk defines `isQuantized`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `isQuantized`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 40-63
```cpp
static BufHandle makeQBufHandleChannelsLast(
    const std::string& name,
    const std::vector<ExprHandle>& dims,
    Dtype dtype,
    const ExprPtr& qscale,
    const ExprPtr& qzero) {
  BufHandle ResultBuf(name, dims, dtype);
  ResultBuf.node()->set_qscale(qscale);
  ResultBuf.node()->set_qzero(qzero);
  ResultBuf.node()->set_strides(make_channels_last_strides(dims));
  return ResultBuf;
}

static BufHandle makeQBufHandleChannelsLast(
    const std::string& name,
    const std::vector<ExprHandle>& dims,
    Dtype dtype,
    const double qscale,
    const int64_t qzero) {
  return makeQBufHandleChannelsLast(
      name,
      dims,
      dtype,
      DoubleImm::make(qscale).node(),
```
- **EN**: This chunk defines `ResultBuf`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `ResultBuf`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 64-78
```cpp
      LongImm::make(qzero).node());
}

static BufHandle makeQBufHandleContiguous(
    const std::string& name,
    const std::vector<ExprHandle>& dims,
    Dtype dtype,
    const ExprPtr& qscale,
    const ExprPtr& qzero) {
  BufHandle ResultBuf(name, dims, dtype);
  ResultBuf.node()->set_qscale(qscale);
  ResultBuf.node()->set_qzero(qzero);
  ResultBuf.node()->set_strides(make_contiguous_strides(dims));
  return ResultBuf;
}
```
- **EN**: This chunk defines `ResultBuf`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `ResultBuf`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 80-103
```cpp
static BufHandle makeQBufHandleContiguous(
    const std::string& name,
    const std::vector<ExprHandle>& dims,
    Dtype dtype,
    const double qscale,
    const int64_t qzero) {
  return makeQBufHandleContiguous(
      name,
      dims,
      dtype,
      DoubleImm::make(qscale).node(),
      LongImm::make(qzero).node());
}

static bool isChannelsLast(const BufHandle& buf) {
  const auto& strides = buf.node()->strides();
  const auto& dims = buf.node()->dims();
  const auto rank = dims.size();
  if (rank < 3) {
    return false;
  }
  auto dimsC = to<LongImm>(IRSimplifier::simplify(dims[1]))->value();
  auto stridesC = to<LongImm>(IRSimplifier::simplify(strides[1]))->value();
  auto stridesLast =
```
- **EN**: This chunk defines `isChannelsLast`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `isChannelsLast`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 104-121
```cpp
      to<LongImm>(IRSimplifier::simplify(strides[rank - 1]))->value();

  return ((stridesLast == dimsC) && (stridesC == 1));
}

static ExprHandle quant(
    const ExprHandle& x,
    Dtype out_dtype,
    ExprHandle qscale,
    ExprHandle qzero) {
  auto promoted_qscale =
      promoteToDtype(std::move(qscale), x.dtype().scalar_type());
  auto promoted_qzero =
      promoteToDtype(std::move(qzero), x.dtype().scalar_type());
  return promoteToDtype(
      x / promoted_qscale + promoted_qzero + FloatImm::make(0.5f),
      out_dtype.scalar_type());
}
```
- **EN**: This chunk defines `quant`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `quant`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 123-146
```cpp
static ExprHandle dequant(
    ExprHandle qx,
    Dtype out_dtype,
    ExprHandle qscale,
    ExprHandle qzero) {
  auto qx_promoted = promoteToDtype(std::move(qx), out_dtype.scalar_type());
  auto qscale_promoted =
      promoteToDtype(std::move(qscale), out_dtype.scalar_type());
  auto qzero_promoted =
      promoteToDtype(std::move(qzero), out_dtype.scalar_type());
  return promoteToDtype(
      (qx_promoted - qzero_promoted) * qscale_promoted,
      out_dtype.scalar_type());
}

Tensor computeQuantizePerTensor(
    const std::vector<ArgValue>& inputs,
    const std::vector<ExprHandle>& outputShape,
    const std::vector<ExprHandle>& outputStrides,
    const std::optional<ScalarType>& /*unused*/,
    at::Device /*unused*/) {
  std::vector<VarPtr> vars;
  std::vector<ExprHandle> indices;
  for (const auto& os : outputShape) {
```
- **EN**: This chunk defines `computeQuantizePerTensor`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `computeQuantizePerTensor`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 147-165
```cpp
    auto var = alloc<Var>("", os.node()->dtype());
    vars.push_back(var);
    indices.push_back(VarHandle(var));
  }

  ExprHandle qscale = constant(inputs[1]);
  ExprHandle qzero = constant(inputs[2]);

  const auto dtype = [](auto qdtype) {
    if (static_cast<int64_t>(ScalarType::QInt8) == qdtype) {
      return Dtype(ScalarType::QInt8);
    } else if (static_cast<int64_t>(ScalarType::QUInt8) == qdtype) {
      return Dtype(ScalarType::QUInt8);
    }
    throw malformed_input("Expected quantized dtype");
  }(std::get<int64_t>(inputs[3]));

  ExprHandle e =
      quant(tensorOrConstant(inputs[0], indices), dtype, qscale, qzero);
```
- **EN**: This chunk defines `malformed_input`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `malformed_input`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 167-190
```cpp
  BufPtr buf = alloc<Buf>(
      "quantize_per_tensor",
      ExprHandleVectorToExprVector(outputShape),
      dtype,
      nullptr,
      std::nullopt,
      qscale.node(),
      qzero.node());
  return Tensor(buf, vars, e.node());
}

Tensor computeQuantizedAdd(
    const std::vector<ArgValue>& inputs,
    const std::vector<ExprHandle>& outputShape,
    const std::vector<ExprHandle>& outputStrides,
    const std::optional<ScalarType>& outputType,
    at::Device /*unused*/) {
  const BufHandle& QA = std::get<BufHandle>(inputs[0]);
  const BufHandle& QB = std::get<BufHandle>(inputs[1]);
  auto qa_scale = ExprHandle(QA.node()->qscale());
  auto qa_zero = ExprHandle(QA.node()->qzero());
  auto qb_scale = ExprHandle(QB.node()->qscale());
  auto qb_zero = ExprHandle(QB.node()->qzero());
  ExprHandle out_qscale = DoubleImm::make(std::get<double>(inputs[2]));
```
- **EN**: This chunk defines `computeQuantizedAdd`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `computeQuantizedAdd`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 191-214
```cpp
  ExprHandle out_qzero = LongImm::make(std::get<int64_t>(inputs[3]));
  Dtype dequant_dtype = kFloat;
  Dtype out_dtype = outputType ? Dtype(*outputType) : QA.dtype();
  std::vector<VarPtr> vars;
  std::vector<ExprHandle> indices;
  for (const auto& os : outputShape) {
    auto var = alloc<Var>("", os.node()->dtype());
    vars.push_back(var);
    indices.push_back(VarHandle(var));
  }
  auto lhs = tensorOrConstant(inputs[0], indices);
  auto rhs = tensorOrConstant(inputs[1], indices);
  ExprHandle exprHandle = quant(
      dequant(lhs, dequant_dtype, qa_scale, qa_zero) +
          dequant(rhs, dequant_dtype, qb_scale, qb_zero),
      out_dtype,
      out_qscale,
      out_qzero);
  BufPtr buf = alloc<Buf>(
      "quantized_add",
      ExprHandleVectorToExprVector(outputShape),
      out_dtype,
      nullptr,
      isChannelsLast(QA) ? make_channels_last_strides(outputShape)
```
- **EN**: This chunk continues `computeQuantizedAdd` and expands its internal control flow or data movement. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically.
- **CN**: 这一段延续了 `computeQuantizedAdd`，进一步展开其内部控制流或数据流转。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。

### Lines 215-230
```cpp
                         : make_contiguous_strides(outputShape),
      out_qscale.node(),
      out_qzero.node());
  return Tensor(buf, vars, exprHandle.node());
}

Tensor computeQuantizePerTensorExternalCall(
    const std::vector<ArgValue>& inputs,
    const std::vector<ExprHandle>& outputShape,
    const std::vector<ExprHandle>& outputStrides,
    const std::optional<ScalarType>& outputType,
    at::Device /*unused*/) {
  const BufHandle& x = std::get<BufHandle>(inputs[0]);
  const auto qscale = std::get<double>(inputs[1]);
  const auto qzero = std::get<int64_t>(inputs[2]);
  const auto qdtype = std::get<int64_t>(inputs[3]);
```
- **EN**: This chunk defines `computeQuantizePerTensorExternalCall`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `computeQuantizePerTensorExternalCall`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 232-251
```cpp
  const auto dtype = [](auto qdtype) {
    if (static_cast<int64_t>(ScalarType::QInt8) == qdtype) {
      return Dtype(ScalarType::QInt8);
    } else if (static_cast<int64_t>(ScalarType::QUInt8) == qdtype) {
      return Dtype(ScalarType::QUInt8);
    }
    throw malformed_input("Expected quantized dtype");
  }(qdtype);
  auto ResultBuf = [&]() {
    if (isChannelsLast(x)) {
      return makeQBufHandleChannelsLast(
          "quantize_per_tensor", outputShape, dtype, qscale, qzero);
    }
    return makeQBufHandleContiguous(
        "quantize_per_tensor", outputShape, dtype, qscale, qzero);
  }();
  StmtPtr s = ExternalCall::make(
      ResultBuf, "nnc_aten_quantize_per_tensor", {x}, {qscale, qzero, qdtype});
  return Tensor(ResultBuf.node(), s);
}
```
- **EN**: This chunk defines `malformed_input`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `malformed_input`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 253-276
```cpp
Tensor computeDequantizeExternalCall(
    const std::vector<ArgValue>& inputs,
    const std::vector<ExprHandle>& outputShape,
    const std::vector<ExprHandle>& outputStrides,
    const std::optional<ScalarType>& outputType,
    at::Device /*unused*/) {
  Dtype dtype = kFloat;
  if (outputType) {
    dtype = Dtype(*outputType);
  }

  const BufHandle& qx = std::get<BufHandle>(inputs[0]);
  const int64_t qdtype = (int64_t)immQDType(qx);

  BufHandle ResultBuf("dequantize", outputShape, dtype);
  StmtPtr s = ExternalCall::make(
      ResultBuf,
      "nnc_aten_dequantize",
      {qx},
      {ExprHandle(IRSimplifier::simplify(qx.node()->qscale())),
       ExprHandle(IRSimplifier::simplify(qx.node()->qzero())),
       qdtype});
  return Tensor(ResultBuf.node(), s);
}
```
- **EN**: This chunk defines `ResultBuf`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `ResultBuf`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 278-301
```cpp
Tensor computeQuantizedConv2dPrepack(
    const std::vector<ArgValue>& inputs,
    const std::vector<ExprHandle>& outputShape,
    const std::vector<ExprHandle>& outputStrides,
    const std::optional<ScalarType>& outputType,
    at::Device /*unused*/) {
  Dtype dtype = kFloat;
  if (outputType) {
    dtype = Dtype(*outputType);
  }

  BufHandle ResultBuf("quantized_conv2d_prepack", outputShape, dtype);
  const BufHandle& qw = std::get<BufHandle>(inputs[0]);
  const BufHandle& b = std::get<BufHandle>(inputs[1]);
  auto strides = _pair_int(inputs[2]);
  auto padding = _pair_int(inputs[3]);
  auto dilation = _pair_int(inputs[4]);
  auto groups = std::get<int64_t>(inputs[5]);
  TORCH_INTERNAL_ASSERT(
      qw.node()->qscale(),
      buildErrorMessage(
          "quantized_conv2d_prepack: Expects quantized weights, qscale is missing"));
  TORCH_INTERNAL_ASSERT(
      qw.node()->qzero(),
```
- **EN**: This chunk defines `ResultBuf`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages.
- **CN**: 这一段定义了 `ResultBuf`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。

### Lines 302-319
```cpp
      buildErrorMessage(
          "quantized_conv2d_prepack: Expects quantized weights, qzero is missing"));
  StmtPtr s = ExternalCall::make(
      ResultBuf,
      "nnc_aten_quantized_conv2d_prepack",
      {qw, b},
      {strides[0],
       strides[1],
       padding[0],
       padding[1],
       dilation[0],
       dilation[1],
       groups,
       immQScale(qw),
       immQZero(qw),
       (int64_t)immQDType(qw)});
  return Tensor(ResultBuf.node(), s);
}
```
- **EN**: This chunk continues `ResultBuf` and expands its internal control flow or data movement. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `ResultBuf`，进一步展开其内部控制流或数据流转。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 321-344
```cpp
Tensor computeQuantizedConv1d(
    const std::vector<ArgValue>& inputs,
    const std::vector<ExprHandle>& outputShape,
    const std::vector<ExprHandle>& outputStrides,
    const std::optional<ScalarType>& outputType,
    at::Device device) {
  const BufHandle& qx = std::get<BufHandle>(inputs[0]);
  const BufHandle& prepacked = std::get<BufHandle>(inputs[1]);
  const auto out_qscale = std::get<double>(inputs[2]);
  const auto out_qzero = std::get<int64_t>(inputs[3]);
  // Change to dtype based on outputType when dtype propagation implemented
  const auto out_qdtype = immQDType(qx);
  auto ResultBuf = makeQBufHandleChannelsLast(
      "quantized_conv1d",
      outputShape,
      Dtype(out_qdtype),
      out_qscale,
      out_qzero);
  StmtPtr s = ExternalCall::make(
      ResultBuf,
      "nnc_aten_quantized_conv1d",
      {qx, prepacked},
      {immQScale(qx),
       immQZero(qx),
```
- **EN**: This chunk defines `computeQuantizedConv1d`, which implements a focused step in Tensor Expression analysis, lowering, or code generation.
- **CN**: 这一段定义了 `computeQuantizedConv1d`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。

### Lines 345-368
```cpp
       (int64_t)immQDType(qx),
       out_qscale,
       out_qzero});
  return Tensor(ResultBuf.node(), s);
}

Tensor computeQuantizedConv2d(
    const std::vector<ArgValue>& inputs,
    const std::vector<ExprHandle>& outputShape,
    const std::vector<ExprHandle>& outputStrides,
    const std::optional<ScalarType>& outputType,
    at::Device device) {
  const BufHandle& qx = std::get<BufHandle>(inputs[0]);
  const BufHandle& prepacked = std::get<BufHandle>(inputs[1]);
  const auto out_qscale = std::get<double>(inputs[2]);
  const auto out_qzero = std::get<int64_t>(inputs[3]);
  // Change to dtype based on outputType when dtype propagation implemented
  const auto out_qdtype = immQDType(qx);
  auto ResultBuf = makeQBufHandleChannelsLast(
      "quantized_conv2d",
      outputShape,
      Dtype(out_qdtype),
      out_qscale,
      out_qzero);
```
- **EN**: This chunk defines `computeQuantizedConv2d`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `computeQuantizedConv2d`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 369-392
```cpp
  StmtPtr s = ExternalCall::make(
      ResultBuf,
      "nnc_aten_quantized_conv2d",
      {qx, prepacked},
      {immQScale(qx),
       immQZero(qx),
       (int64_t)immQDType(qx),
       out_qscale,
       out_qzero});
  return Tensor(ResultBuf.node(), s);
}

Tensor computeQuantizedConv2dRelu(
    const std::vector<ArgValue>& inputs,
    const std::vector<ExprHandle>& outputShape,
    const std::vector<ExprHandle>& outputStrides,
    const std::optional<ScalarType>& outputType,
    at::Device device) {
  const BufHandle& qx = std::get<BufHandle>(inputs[0]);
  const BufHandle& prepacked = std::get<BufHandle>(inputs[1]);
  const auto out_qscale = std::get<double>(inputs[2]);
  const auto out_qzero = std::get<int64_t>(inputs[3]);
  // Change to dtype based on outputType when dtype propagation implemented
  const auto out_qdtype = immQDType(qx);
```
- **EN**: This chunk defines `computeQuantizedConv2dRelu`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `computeQuantizedConv2dRelu`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 393-409
```cpp
  auto ResultBuf = makeQBufHandleChannelsLast(
      "quantized_conv2d_relu",
      outputShape,
      Dtype(out_qdtype),
      out_qscale,
      out_qzero);
  StmtPtr s = ExternalCall::make(
      ResultBuf,
      "nnc_aten_quantized_conv2d_relu",
      {qx, prepacked},
      {immQScale(qx),
       immQZero(qx),
       (int64_t)immQDType(qx),
       out_qscale,
       out_qzero});
  return Tensor(ResultBuf.node(), s);
}
```
- **EN**: This chunk continues `computeQuantizedConv2dRelu` and expands its internal control flow or data movement. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `computeQuantizedConv2dRelu`，进一步展开其内部控制流或数据流转。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 411-434
```cpp
Tensor computeQuantizedLinear(
    const std::vector<ArgValue>& inputs,
    const std::vector<ExprHandle>& outputShape,
    const std::vector<ExprHandle>& outputStrides,
    const std::optional<ScalarType>& outputType,
    at::Device device) {
  const BufHandle& qx = std::get<BufHandle>(inputs[0]);
  const BufHandle& prepacked = std::get<BufHandle>(inputs[1]);
  const auto out_qscale = std::get<double>(inputs[2]);
  const auto out_qzero = std::get<int64_t>(inputs[3]);
  // Change to dtype based on outputType when dtype propagation implemented
  const auto out_qdtype = immQDType(qx);
  auto ResultBuf = makeQBufHandleContiguous(
      "quantized_linear",
      outputShape,
      Dtype(out_qdtype),
      out_qscale,
      out_qzero);
  StmtPtr s = ExternalCall::make(
      ResultBuf,
      "nnc_aten_quantized_linear",
      {qx, prepacked},
      {immQScale(qx),
       immQZero(qx),
```
- **EN**: This chunk defines `computeQuantizedLinear`, which implements a focused step in Tensor Expression analysis, lowering, or code generation.
- **CN**: 这一段定义了 `computeQuantizedLinear`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。

### Lines 435-458
```cpp
       (int64_t)immQDType(qx),
       out_qscale,
       out_qzero});
  return Tensor(ResultBuf.node(), s);
}

Tensor computeQuantizedLinearRelu(
    const std::vector<ArgValue>& inputs,
    const std::vector<ExprHandle>& outputShape,
    const std::vector<ExprHandle>& outputStrides,
    const std::optional<ScalarType>& outputType,
    at::Device device) {
  const BufHandle& qx = std::get<BufHandle>(inputs[0]);
  const BufHandle& prepacked = std::get<BufHandle>(inputs[1]);
  const auto out_qscale = std::get<double>(inputs[2]);
  const auto out_qzero = std::get<int64_t>(inputs[3]);
  // Change to dtype based on outputType when dtype propagation implemented
  const auto out_qdtype = immQDType(qx);
  auto ResultBuf = makeQBufHandleContiguous(
      "quantized_linear_relu",
      outputShape,
      Dtype(out_qdtype),
      out_qscale,
      out_qzero);
```
- **EN**: This chunk defines `computeQuantizedLinearRelu`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `computeQuantizedLinearRelu`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 459-482
```cpp
  StmtPtr s = ExternalCall::make(
      ResultBuf,
      "nnc_aten_quantized_linear_relu",
      {qx, prepacked},
      {immQScale(qx),
       immQZero(qx),
       (int64_t)immQDType(qx),
       out_qscale,
       out_qzero});
  return Tensor(ResultBuf.node(), s);
}

Tensor computeQuantizedAddExternalCall(
    const std::vector<ArgValue>& inputs,
    const std::vector<ExprHandle>& outputShape,
    const std::vector<ExprHandle>& outputStrides,
    const std::optional<ScalarType>& outputType,
    at::Device device) {
  const BufHandle& qa = std::get<BufHandle>(inputs[0]);
  const BufHandle& qb = std::get<BufHandle>(inputs[1]);
  const auto out_qscale = std::get<double>(inputs[2]);
  const auto out_qzero = std::get<int64_t>(inputs[3]);
  // Change to dtype based on outputType when dtype propagation implemented
  const auto out_qdtype = immQDType(qa);
```
- **EN**: This chunk defines `computeQuantizedAddExternalCall`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `computeQuantizedAddExternalCall`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 483-506
```cpp
  const bool isQAChannelsLast = isChannelsLast(qa);
  const bool isQBChannelsLast = isChannelsLast(qb);
  auto ResultBuf = (isQAChannelsLast || isQBChannelsLast)
      ? makeQBufHandleChannelsLast(
            "quantized_add",
            outputShape,
            Dtype(out_qdtype),
            out_qscale,
            out_qzero)
      : makeQBufHandleContiguous(
            "quantized_add",
            outputShape,
            Dtype(out_qdtype),
            out_qscale,
            out_qzero);
  StmtPtr s = ExternalCall::make(
      ResultBuf,
      "nnc_aten_quantized_add",
      {qa, qb},
      {immQScale(qa),
       immQZero(qa),
       (int64_t)immQDType(qa),
       immQScale(qb),
       immQZero(qb),
```
- **EN**: This chunk continues `computeQuantizedAddExternalCall` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `computeQuantizedAddExternalCall`，进一步展开其内部控制流或数据流转。

### Lines 507-530
```cpp
       (int64_t)immQDType(qb),
       out_qscale,
       out_qzero});
  return Tensor(ResultBuf.node(), s);
}

Tensor computeQuantizedMul(
    const std::vector<ArgValue>& inputs,
    const std::vector<ExprHandle>& outputShape,
    const std::vector<ExprHandle>& outputStrides,
    const std::optional<ScalarType>& outputType,
    at::Device device) {
  const BufHandle& qa = std::get<BufHandle>(inputs[0]);
  const BufHandle& qb = std::get<BufHandle>(inputs[1]);
  const auto out_qscale = std::get<double>(inputs[2]);
  const auto out_qzero = std::get<int64_t>(inputs[3]);
  // Change to dtype based on outputType when dtype propagation implemented
  const auto out_qdtype = immQDType(qa);
  auto ResultBuf = makeQBufHandleContiguous(
      "quantized_mul", outputShape, Dtype(out_qdtype), out_qscale, out_qzero);
  StmtPtr s = ExternalCall::make(
      ResultBuf,
      "nnc_aten_quantized_mul",
      {qa, qb},
```
- **EN**: This chunk defines `computeQuantizedMul`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `computeQuantizedMul`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 531-554
```cpp
      {immQScale(qa),
       immQZero(qa),
       (int64_t)immQDType(qa),
       immQScale(qb),
       immQZero(qb),
       (int64_t)immQDType(qb),
       out_qscale,
       out_qzero});
  return Tensor(ResultBuf.node(), s);
}

Tensor computeQuantizedMulScalar(
    const std::vector<ArgValue>& inputs,
    const std::vector<ExprHandle>& outputShape,
    const std::vector<ExprHandle>& outputStrides,
    const std::optional<ScalarType>& outputType,
    at::Device device) {
  const BufHandle& qa = std::get<BufHandle>(inputs[0]);
  const auto scalar = std::get<double>(inputs[1]);
  // Change to dtype based on outputType when dtype propagation implemented
  const auto out_qdtype = immQDType(qa);
  double scale1 = immQScale(qa);
  auto ResultBuf = makeQBufHandleContiguous(
      "quantized_mul_scalar",
```
- **EN**: This chunk defines `computeQuantizedMulScalar`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `computeQuantizedMulScalar`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 555-578
```cpp
      outputShape,
      Dtype(out_qdtype),
      scale1 * scalar,
      immQZero(qa));
  StmtPtr s = ExternalCall::make(
      ResultBuf,
      "nnc_aten_quantized_mul_scalar",
      {qa},
      {scale1, immQZero(qa), (int64_t)immQDType(qa), scalar});
  return Tensor(ResultBuf.node(), s);
}

Tensor computeQuantizedRelu(
    const std::vector<ArgValue>& inputs,
    const std::vector<ExprHandle>& outputShape,
    const std::vector<ExprHandle>& outputStrides,
    const std::optional<ScalarType>& outputType,
    at::Device device) {
  const BufHandle& qa = std::get<BufHandle>(inputs[0]);
  const auto out_qdtype = immQDType(qa);
  const bool isQAChannelsLast = isChannelsLast(qa);
  auto ResultBuf = isQAChannelsLast ? makeQBufHandleChannelsLast(
                                          "quantized_relu",
                                          outputShape,
```
- **EN**: This chunk defines `computeQuantizedRelu`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `computeQuantizedRelu`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 579-594
```cpp
                                          Dtype(out_qdtype),
                                          immQScale(qa),
                                          immQZero(qa))
                                    : makeQBufHandleContiguous(
                                          "quantized_relu",
                                          outputShape,
                                          Dtype(out_qdtype),
                                          immQScale(qa),
                                          immQZero(qa));
  StmtPtr s = ExternalCall::make(
      ResultBuf,
      "nnc_aten_quantized_relu",
      {qa},
      {immQScale(qa), immQZero(qa), (int64_t)immQDType(qa)});
  return Tensor(ResultBuf.node(), s);
}
```
- **EN**: This chunk continues `computeQuantizedRelu` and expands its internal control flow or data movement. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `computeQuantizedRelu`，进一步展开其内部控制流或数据流转。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 596-619
```cpp
Tensor computeQuantizedCat(
    const std::vector<ArgValue>& inputs,
    const std::vector<ExprHandle>& outputShape,
    const std::vector<ExprHandle>& outputStrides,
    const std::optional<ScalarType>& outputType,
    at::Device device) {
  auto const& inputList = std::get<BufList>(inputs[0]);
  auto argDim = std::get<int64_t>(inputs[1]);
  auto n = inputList.size();
  // TODO: handle optional out_qscale, out_qzero
  const auto out_qscale = std::get<double>(inputs[2]);
  const auto out_qzero = std::get<int64_t>(inputs[3]);

  std::vector<BufHandle> args;
  std::vector<ExprHandle> extra_args;
  for (const auto i : c10::irange(n)) {
    const BufHandle& bh = inputList[i];
    args.emplace_back(bh);
    extra_args.emplace_back(immQScale(bh));
    extra_args.emplace_back(immQZero(bh));
    extra_args.emplace_back((int64_t)immQDType(bh));
  }
  extra_args.emplace_back(argDim);
  extra_args.emplace_back(out_qscale);
```
- **EN**: This chunk defines `computeQuantizedCat`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically.
- **CN**: 这一段定义了 `computeQuantizedCat`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。

### Lines 620-643
```cpp
  extra_args.emplace_back(out_qzero);
  auto ResultBuf = makeQBufHandleContiguous(
      "quantized_cat",
      outputShape,
      Dtype(immQDType(inputList[0])),
      out_qscale,
      out_qzero);
  StmtPtr s =
      ExternalCall::make(ResultBuf, "nnc_aten_quantized_cat", args, extra_args);
  return Tensor(ResultBuf.node(), s);
}

Tensor computeDequantize(
    const std::vector<ArgValue>& inputs,
    const std::vector<ExprHandle>& outputShape,
    const std::vector<ExprHandle>& outputStrides,
    const std::optional<ScalarType>& outputType,
    at::Device /*unused*/) {
  Dtype dtype = kFloat;
  if (outputType) {
    dtype = Dtype(*outputType);
  }
  auto const& qx = std::get<BufHandle>(inputs[0]);
  TORCH_INTERNAL_ASSERT(
```
- **EN**: This chunk defines `computeDequantize`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `computeDequantize`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 644-662
```cpp
      qx.node()->qscale(),
      buildErrorMessage("Missing quantized scale for dequantize"));
  TORCH_INTERNAL_ASSERT(
      qx.node()->qzero(),
      buildErrorMessage("Missing quantized zero point for dequantize"));
  auto qscale = ExprHandle(qx.node()->qscale());
  auto qzero = ExprHandle(qx.node()->qzero());
  std::vector<VarPtr> vars;
  std::vector<ExprHandle> indices;
  for (const auto& os : outputShape) {
    auto var = alloc<Var>("", os.node()->dtype());
    vars.push_back(var);
    indices.push_back(VarHandle(var));
  }
  auto y = dequant(tensorOrConstant(inputs[0], indices), dtype, qscale, qzero);
  BufPtr buf = alloc<Buf>(
      "dequantize", ExprHandleVectorToExprVector(outputShape), dtype);
  return Tensor(buf, vars, y.node());
}
```
- **EN**: This chunk continues `computeDequantize` and expands its internal control flow or data movement. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `computeDequantize`，进一步展开其内部控制流或数据流转。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 664-687
```cpp
Tensor computeUpsampleNearest2d(
    const std::vector<ArgValue>& inputs,
    const std::vector<ExprHandle>& outputShape,
    const std::vector<ExprHandle>& outputStrides,
    const std::optional<ScalarType>& outputType,
    at::Device /*unused*/) {
  const auto& A = std::get<BufHandle>(inputs[0]);
  const auto& output_height = outputShape[2];
  const auto& output_width = outputShape[3];
  auto input_height = ExprHandle(A.dim(2));
  auto input_width = ExprHandle(A.dim(3));

  std::vector<VarHandle> args = create_index_vars(outputShape);
  // Handle separately when scale is specified? as in 'scalar_t
  // compute_scales_value' in UpSample.h
  auto scale_h =
      promoteToDtype(input_height, ScalarType::Double) / output_height;
  auto scale_w = promoteToDtype(input_width, ScalarType::Double) / output_width;
  // TODO: will repetitive if in idx calculation will be taken out of the loop?
  auto compute_nearest_idx = [](const ExprHandle& scale,
                                const ExprHandle& dst_index,
                                const ExprHandle& input_size) {
    return Min::make(
        promoteToDtype(floor(dst_index * scale), ScalarType::Long),
```
- **EN**: This chunk defines `computeUpsampleNearest2d`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `computeUpsampleNearest2d`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 688-709
```cpp
        input_size - 1,
        true);
  };
  auto body_func = [&](std::vector<VarHandle> axes) {
    std::vector<ExprHandle> newAxes(axes.begin(), axes.end());
    newAxes[2] = compute_nearest_idx(scale_h, axes[2], input_height);
    newAxes[3] = compute_nearest_idx(scale_w, axes[3], input_width);
    return A.load(newAxes);
  };
  auto e = body_func(args);
  auto strides = isChannelsLast(A) ? make_channels_last_strides(outputShape)
                                   : make_contiguous_strides(outputShape);
  BufHandle buf = Buf::make(
      "upsample_nearest2d",
      outputShape,
      Dtype(*outputType),
      std::nullopt, // initializer
      fmap(strides, [&](const ExprPtr& stride) { return ExprHandle(stride); }),
      ExprHandle(A.node()->qscale()),
      ExprHandle(A.node()->qzero()));
  return Tensor(buf, args, e);
}
```
- **EN**: This chunk defines `newAxes`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `newAxes`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 711-726
```cpp
Tensor computeUpsampleNearest2dExternalCall(
    const std::vector<ArgValue>& inputs,
    const std::vector<ExprHandle>& outputShape,
    const std::vector<ExprHandle>& outputStrides,
    const std::optional<ScalarType>& outputType,
    at::Device /*unused*/) {
  Dtype dtype = kFloat;
  if (outputType) {
    dtype = Dtype(*outputType);
  }
  int64_t output_size_h = -1;
  int64_t output_size_w = -1;
  if (auto output_sizes = std::get_if<IntList>(&inputs[1])) {
    output_size_h = (*output_sizes)[0];
    output_size_w = (*output_sizes)[1];
  }
```
- **EN**: This chunk defines `computeUpsampleNearest2dExternalCall`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段定义了 `computeUpsampleNearest2dExternalCall`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 728-742
```cpp
  double scale_factor_h = -1.f;
  double scale_factor_w = -1.f;
  if (auto scale_factors = std::get_if<DoubleList>(&inputs[2])) {
    scale_factor_h = (*scale_factors)[0];
    scale_factor_w = (*scale_factors)[1];
  }
  const BufHandle& x = std::get<BufHandle>(inputs[0]);
  double qx_qscale = -1.f;
  int64_t qx_qzero = -1l;
  int64_t qx_qdtype = -1l;
  if (isQuantized(x)) {
    qx_qscale = immQScale(x);
    qx_qzero = immQZero(x);
    qx_qdtype = (int64_t)immQDType(x);
  }
```
- **EN**: This chunk continues `computeUpsampleNearest2dExternalCall` and expands its internal control flow or data movement. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段延续了 `computeUpsampleNearest2dExternalCall`，进一步展开其内部控制流或数据流转。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 744-767
```cpp
  BufHandle ResultBuf = [&]() {
    if (isQuantized(x)) {
      return makeQBufHandleChannelsLast(
          "upsample_nearest2d",
          outputShape,
          Dtype(immQDType(x)),
          qx_qscale,
          qx_qzero);
    }
    return BufHandle("upsample_nearest2d", outputShape, dtype);
  }();

  StmtPtr s = ExternalCall::make(
      ResultBuf,
      "nnc_aten_upsample_nearest2d",
      {x},
      {qx_qscale,
       qx_qzero,
       qx_qdtype,
       output_size_h,
       output_size_w,
       scale_factor_h,
       scale_factor_w});
  return Tensor(ResultBuf.node(), s);
```
- **EN**: This chunk continues `computeUpsampleNearest2dExternalCall` and expands its internal control flow or data movement. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `computeUpsampleNearest2dExternalCall`，进一步展开其内部控制流或数据流转。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 768-791
```cpp
}

Tensor computeQuantizedSigmoidExternalCall(
    const std::vector<ArgValue>& inputs,
    const std::vector<ExprHandle>& outputShape,
    const std::vector<ExprHandle>& outputStrides,
    const std::optional<ScalarType>& outputType,
    at::Device /*unused*/) {
  const BufHandle& qx = std::get<BufHandle>(inputs[0]);

  const auto out_qdtype = immQDType(qx);
  const double out_qscale = 1.0f / 256.0f;
  const int64_t out_qzero = (out_qdtype == ScalarType::QInt8) ? -128 : 0;

  auto ResultBuf = isChannelsLast(qx) ? makeQBufHandleChannelsLast(
                                            "quantized_sigmoid",
                                            outputShape,
                                            Dtype(out_qdtype),
                                            out_qscale,
                                            out_qzero)
                                      : makeQBufHandleContiguous(
                                            "quantized_sigmoid",
                                            outputShape,
                                            Dtype(out_qdtype),
```
- **EN**: This chunk defines `computeQuantizedSigmoidExternalCall`, which implements a focused step in Tensor Expression analysis, lowering, or code generation.
- **CN**: 这一段定义了 `computeQuantizedSigmoidExternalCall`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。

### Lines 792-806
```cpp
                                            out_qscale,
                                            out_qzero);
  StmtPtr s = ExternalCall::make(
      ResultBuf,
      "nnc_aten_quantized_sigmoid",
      {qx},
      {immQScale(qx),
       immQZero(qx),
       (int64_t)immQDType(qx),
       out_qscale,
       out_qzero});
  return Tensor(ResultBuf.node(), s);
}

} // namespace torch::jit::tensorexpr
```
- **EN**: This chunk continues `computeQuantizedSigmoidExternalCall` and expands its internal control flow or data movement. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `computeQuantizedSigmoidExternalCall`，进一步展开其内部控制流或数据流转。 末尾会返回累积结果，或把更新后的状态交还给调用方。

## Key Concepts / 关键概念

- **Tensor Expression IR**
  - EN: Implements Tensor Expression IR, analyses, scheduling, lowering, and backend code generation.
  - CN: 实现 Tensor Expression IR、分析、调度、降级以及后端代码生成。
- **_pair_int**
  - EN: `_pair_int` is a central symbol declared or implemented in this file.
  - CN: `_pair_int` 是本文件声明或实现的核心符号。
- **immQScale**
  - EN: `immQScale` is a central symbol declared or implemented in this file.
  - CN: `immQScale` 是本文件声明或实现的核心符号。
- **Operator dispatch**
  - EN: Bridges symbolic operator names or schemas to concrete C++ implementations.
  - CN: 把符号化算子名或 schema 连接到具体的 C++ 实现。
- **IR simplification**
  - EN: Rewrites expressions and statements into simpler but equivalent forms.
  - CN: 把表达式和语句重写为更简单但等价的形式。
## Dependencies / 依赖关系

- **Internal PyTorch headers / 内部 PyTorch 头文件**: `torch/csrc/jit/tensorexpr/ir_simplifier.h`, `torch/csrc/jit/tensorexpr/operators/misc.h`, `torch/csrc/jit/tensorexpr/operators/pointwise.h`, `torch/csrc/jit/tensorexpr/operators/quantization.h`
- **ATen/c10 foundations / ATen/c10 基础设施**: `c10/core/ScalarType.h`
- **Primary symbols in this file / 本文件核心符号**: `_pair_int`, `immQScale`, `immQZero`, `immQDType`, `isQuantized`, `makeQBufHandleChannelsLast`, `ResultBuf`, `makeQBufHandleContiguous`
