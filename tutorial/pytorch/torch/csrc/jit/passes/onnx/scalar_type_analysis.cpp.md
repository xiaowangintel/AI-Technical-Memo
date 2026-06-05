# scalar_type_analysis.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `torch/csrc/jit/passes/onnx/scalar_type_analysis.cpp`
- Repository: `pytorch`
- Purpose (EN): Implements the TorchScript JIT pass logic for scalar type analysis, including graph analysis and rewrites.
- 用途 (CN): 实现与 scalar type analysis 相关的 TorchScript JIT pass 逻辑，包括图分析与改写。

## Line-by-Line Analysis / 逐行分析
### Lines 1-7
```cpp
#include <c10/util/irange.h>
#include <torch/csrc/jit/jit_log.h>
#include <torch/csrc/jit/passes/dead_code_elimination.h>
#include <torch/csrc/jit/passes/onnx/helper.h>
#include <torch/csrc/jit/passes/onnx/scalar_type_analysis.h>

namespace torch::jit {
```
- EN: This block implements local helper logic for scalar type analysis. Key symbols: no dominant local symbol names.
- CN: 该代码块实现与 scalar type analysis 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 8-15
```cpp

namespace onnx {
using namespace ::c10::onnx;
}

namespace {
const int ONNX_OPSET_14 = 14;

```
- EN: This block implements local helper logic for scalar type analysis. Key symbols: `namespace`.
- CN: 该代码块实现与 scalar type analysis 相关的局部辅助逻辑。关键符号：`namespace`。

### Lines 16-29
```cpp
static const std::unordered_map<c10::ScalarType, int, ScalarTypeHashFunction>
    scalarTypeToONNXTypeMap = {
        {c10::kFloat, 1},
        {c10::kByte, 2},
        {c10::kChar, 3},
        {c10::kShort, 5},
        {c10::kInt, 6},
        {c10::kLong, 7},
        {c10::kBool, 9},
        {c10::kHalf, 10},
        {c10::kDouble, 11},
        {c10::kQInt8, 12},
        {c10::kQUInt8, 13},
        {c10::kQInt32, 14},
```
- EN: This block implements local helper logic for scalar type analysis. Key symbols: no dominant local symbol names.
- CN: 该代码块实现与 scalar type analysis 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 30-36
```cpp
        {c10::kBFloat16, 15},
        {c10::kFloat8_e4m3fn, 16},
        {c10::kFloat8_e5m2, 17},
        {c10::kFloat8_e4m3fnuz, 18},
        {c10::kFloat8_e5m2fnuz, 19},
};

```
- EN: This block implements local helper logic for scalar type analysis. Key symbols: no dominant local symbol names.
- CN: 该代码块实现与 scalar type analysis 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 37-43
```cpp
static int64_t ScalarTypeToONNXType(const c10::ScalarType& st) {
  int64_t onnx_type = -1;
  const auto it = scalarTypeToONNXTypeMap.find(st);
  if (it != scalarTypeToONNXTypeMap.end()) {
    onnx_type = it->second;
  }
  return onnx_type;
```
- EN: This block handles conditional branches; produces a result or forwards a computed value. Key symbols: `ScalarTypeToONNXType`, `find`, `end`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值。关键符号：`ScalarTypeToONNXType`, `find`, `end`。

### Lines 44-57
```cpp
}

// For these operators, all inputs and outputs share the same scalar type.
// There is no operator-wise special case handling needed.
static const std::unordered_set<NodeKind> standardOps = {
    onnx::Add,
    onnx::Concat,
    onnx::Div,
    onnx::Gemm,
    onnx::Min,
    onnx::Max,
    onnx::Mod,
    onnx::Mul,
    onnx::Pow,
```
- EN: This block manipulates TorchScript IR objects. Key symbols: no dominant local symbol names.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：无明显局部符号。

### Lines 58-65
```cpp
    onnx::Sub,
    onnx::MatMul,
    onnx::Conv,
};

// For these operators, all inputs share the same scalar type.
// The output scalar type is always Bool.
static const std::unordered_set<NodeKind> comparisonOps = {
```
- EN: This block manipulates TorchScript IR objects. Key symbols: no dominant local symbol names.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：无明显局部符号。

### Lines 66-72
```cpp
    onnx::Greater,
    onnx::Less,
    onnx::Equal,
    onnx::GreaterOrEqual,
    onnx::LessOrEqual,
};

```
- EN: This block implements local helper logic for scalar type analysis. Key symbols: no dominant local symbol names.
- CN: 该代码块实现与 scalar type analysis 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 73-79
```cpp
static const std::unordered_set<NodeKind> selectorOps = {onnx::Where};

static bool IsStandardOp(const NodeKind& nkind) {
  return standardOps.find(nkind) != standardOps.end();
}

static bool IsComparisonOp(const NodeKind& nkind) {
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `IsStandardOp`, `find`, `end`, `IsComparisonOp`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`IsStandardOp`, `find`, `end`, `IsComparisonOp`。

### Lines 80-86
```cpp
  return comparisonOps.find(nkind) != comparisonOps.end();
}

static bool IsSelectorOp(const NodeKind& nkind) {
  return selectorOps.find(nkind) != selectorOps.end();
}

```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `find`, `end`, `IsSelectorOp`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`find`, `end`, `IsSelectorOp`。

### Lines 87-93
```cpp
static TensorTypePtr CreateProfiledTensorTypeWithScalarType(
    const TensorTypePtr& typePtr,
    const c10::ScalarType& scalar_type) {
  TORCH_INTERNAL_ASSERT(typePtr != nullptr);
  return typePtr->withScalarType({scalar_type});
}

```
- EN: This block produces a result or forwards a computed value. Key symbols: `CreateProfiledTensorTypeWithScalarType`, `withScalarType`.
- CN: 该代码块返回结果或转发已计算的值。关键符号：`CreateProfiledTensorTypeWithScalarType`, `withScalarType`。

### Lines 94-101
```cpp
static bool IsImplicitCastSupported(const NodeKind& nodeKind) {
  return IsStandardOp(nodeKind) || IsComparisonOp(nodeKind) ||
      IsSelectorOp(nodeKind);
}

static std::optional<c10::ScalarType> PromoteScalarTypes(
    const std::vector<c10::ScalarType>& types) {
  if (types.empty()) {
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `IsImplicitCastSupported`, `IsStandardOp`, `IsComparisonOp`, `IsSelectorOp`, `PromoteScalarTypes`, `empty`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`IsImplicitCastSupported`, `IsStandardOp`, `IsComparisonOp`, `IsSelectorOp`, `PromoteScalarTypes`, `empty`。

### Lines 102-108
```cpp
    return std::nullopt;
  }
  auto st = types[0];
  for (const auto i : c10::irange(1, types.size())) {
    st = c10::promoteTypes(st, types[i]);
  }
  return st;
```
- EN: This block iterates over collections or graph structures; produces a result or forwards a computed value. Key symbols: `irange`, `size`, `promoteTypes`.
- CN: 该代码块遍历集合或图结构；返回结果或转发已计算的值。关键符号：`irange`, `size`, `promoteTypes`。

### Lines 109-117
```cpp
}

// Type promotion between scalars and tensors
// per logic here
// https://pytorch.org/docs/main/tensor_attributes.html#tensor-attributes
static std::optional<c10::ScalarType> PromoteScalarTypesWithCategory(
    const std::vector<c10::ScalarType>& typesFromTensors,
    const std::vector<c10::ScalarType>& typesFromScalars) {
  auto typeFromTensor = PromoteScalarTypes(typesFromTensors);
```
- EN: This block implements local helper logic for scalar type analysis. Key symbols: `PromoteScalarTypesWithCategory`, `PromoteScalarTypes`.
- CN: 该代码块实现与 scalar type analysis 相关的局部辅助逻辑。关键符号：`PromoteScalarTypesWithCategory`, `PromoteScalarTypes`。

### Lines 118-124
```cpp
  auto typeFromScalar = PromoteScalarTypes(typesFromScalars);

  auto getTypeCategory = [](c10::ScalarType t) {
    if (c10::kBool == t) {
      return 1;
    }
    if (c10::isIntegralType(t, /*includeBool=*/false)) {
```
- EN: This block handles conditional branches; produces a result or forwards a computed value. Key symbols: `PromoteScalarTypes`, `isIntegralType`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值。关键符号：`PromoteScalarTypes`, `isIntegralType`。

### Lines 125-131
```cpp
      return 2;
    }
    if (c10::isFloatingType(t)) {
      return 3;
    }
    return 0;
  };
```
- EN: This block handles conditional branches; produces a result or forwards a computed value. Key symbols: `isFloatingType`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值。关键符号：`isFloatingType`。

### Lines 132-138
```cpp

  if (std::nullopt == typeFromScalar) {
    return typeFromTensor;
  } else if (std::nullopt == typeFromTensor) {
    return typeFromScalar;
  }

```
- EN: This block handles conditional branches; produces a result or forwards a computed value. Key symbols: no dominant local symbol names.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值。关键符号：无明显局部符号。

### Lines 139-145
```cpp
  auto typeCategoryFromTensor = getTypeCategory(typeFromTensor.value());
  auto typeCategoryFromScalar = getTypeCategory(typeFromScalar.value());

  if (typeCategoryFromScalar > typeCategoryFromTensor) {
    return typeFromScalar;
  }
  return typeFromTensor;
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `getTypeCategory`, `value`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`getTypeCategory`, `value`。

### Lines 146-152
```cpp
}

static std::optional<c10::ScalarType> InferExpectedScalarType(const Node* n) {
  std::vector<c10::ScalarType> typesFromTensors;
  std::vector<c10::ScalarType> typesFromScalars;

  auto get_scalar_type =
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `InferExpectedScalarType`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`InferExpectedScalarType`。

### Lines 153-159
```cpp
      [](const Value* input) -> std::optional<at::ScalarType> {
    if (auto* tensor_type = input->type()->castRaw<TensorType>()) {
      return tensor_type->scalarType();
    }
    return std::nullopt;
  };
  auto emplace_type_from_scalar =
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `type`, `scalarType`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`type`, `scalarType`。

### Lines 160-168
```cpp
      [&typesFromTensors, &typesFromScalars](at::ScalarType scalar_type) {
        // Mimic PyTorch scalar type promotion logic
        // from https://github.com/pytorch/pytorch/issues/9515
        // Quoting:
        //    A Tensor is a considered a "wrapped number" if it is
        //    auto-wrapped from a C++ or Python number type. Integer types are
        //    wrapped as 0-dim int64 tensors and floating-point types are
        //    wrapped as 0-dim double tensors.
        auto default_scalar_type =
```
- EN: This block implements local helper logic for scalar type analysis. Key symbols: no dominant local symbol names.
- CN: 该代码块实现与 scalar type analysis 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 169-182
```cpp
            at::typeMetaToScalarType(at::get_default_dtype());
        switch (scalar_type) {
          case at::kDouble:
          case at::kFloat:
            // floating-point numbers wrapped as float32/float64 tensors are
            // considered to have default type, instead of double.
            typesFromScalars.emplace_back(default_scalar_type);
            break;
          case at::kLong:
          case at::kBool:
            // bool and integer numbers remain the same type.
            typesFromScalars.emplace_back(scalar_type);
            break;
          default:
```
- EN: This block implements local helper logic for scalar type analysis. Key symbols: `typeMetaToScalarType`, `get_default_dtype`, `emplace_back`.
- CN: 该代码块实现与 scalar type analysis 相关的局部辅助逻辑。关键符号：`typeMetaToScalarType`, `get_default_dtype`, `emplace_back`。

### Lines 183-189
```cpp
            // other types are not from wrapped numbers,
            // track them as types from tensors.
            typesFromTensors.emplace_back(scalar_type);
            break;
        }
      };

```
- EN: This block implements local helper logic for scalar type analysis. Key symbols: `emplace_back`.
- CN: 该代码块实现与 scalar type analysis 相关的局部辅助逻辑。关键符号：`emplace_back`。

### Lines 190-199
```cpp
  size_t input_idx = 0;
  std::for_each(
      n->inputs().begin(), n->inputs().end(), [&](const Value* input) {
        // We skip the 'condition' input (i.e., the first input) in case of
        // onnx::Where operator.
        if (IsSelectorOp(n->kind()) && input_idx == 0) {
          input_idx++;
          return;
        }

```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `for_each`, `inputs`, `begin`, `end`, `IsSelectorOp`, `kind`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`for_each`, `inputs`, `begin`, `end`, `IsSelectorOp`, `kind`。

### Lines 200-213
```cpp
        auto nkind = input->node()->kind();
        if (nkind == onnx::Gather &&
            input->node()->input(0)->node()->kind() == onnx::Shape) {
          // This is a special pattern generated by code like `dim_size =
          // x.size(0)`. It gets converted to the below ONNX IR graph
          //    %1 : Long() = onnx::Constant[value={0}]()
          //    %2 : Tensor = onnx::Shape(%x)
          //    %dim_size : Long() = onnx::Gather(%2, %1)
          // `dim_size` is treated in PyTorch as Scalar.
          // However, in the ONNX IR graph, it is an output of onnx::Gather,
          // which is by default considered as a tensor.
          typesFromScalars.emplace_back(c10::kLong);
        } else if (nkind == onnx::Constant) {
          auto tensor = input->node()->t(attr::value);
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `node`, `kind`, `input`, `emplace_back`, `t`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`node`, `kind`, `input`, `emplace_back`, `t`。

### Lines 214-223
```cpp
          auto rank = tensor.dim();
          auto scalar_type = tensor.scalar_type();

          if (rank == 0) {
            emplace_type_from_scalar(scalar_type);
          } else {
            typesFromTensors.emplace_back(scalar_type);
          }
        } else if (auto scalar_type = get_scalar_type(input)) {
          auto tensor_type = input->type()->castRaw<TensorType>();
```
- EN: This block handles conditional branches. Key symbols: `dim`, `scalar_type`, `emplace_type_from_scalar`, `emplace_back`, `get_scalar_type`, `type`.
- CN: 该代码块处理条件分支。关键符号：`dim`, `scalar_type`, `emplace_type_from_scalar`, `emplace_back`, `get_scalar_type`, `type`。

### Lines 224-232
```cpp
          // get_scalar_type returns non-null value already guarantees
          // that the input has a valid tensor_type.
          TORCH_INTERNAL_ASSERT(nullptr != tensor_type);
          // ONNX model track shape related computes that were done in pytorch
          // by python numbers as tensor computes. This is the only way for ONNX
          // to track them properly since ONNX only has tensor type, otherwise
          // the computation result will be tracked statically as constant, and
          // the model won't work for another input that differs in shape.

```
- EN: This block manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: no dominant local symbol names.
- CN: 该代码块操作 TorchScript IR 对象；执行面向优化的改写。关键符号：无明显局部符号。

### Lines 233-243
```cpp
          // Now for type promotion logic, scalars should be treated differently
          // with tensors. More info regarding type promotion logic commented at
          // `emplace_type_from_scalar`. Here we filter out rank 0 tensors and
          // run it with `emplace_type_from_scalar` to determine if they are
          // considered scalars for type promotion.

          // NOTE that this might introduce regression that a REAL 0-rank tensor
          // is now being recognized as scalar. The downside is the model will
          // drop in accuracy for these cases as certain computations will
          // happen in lower precision data types.
          auto rank = tensor_type->dim();
```
- EN: This block implements local helper logic for scalar type analysis. Key symbols: `dim`.
- CN: 该代码块实现与 scalar type analysis 相关的局部辅助逻辑。关键符号：`dim`。

### Lines 244-253
```cpp
          if (rank && rank.value() == 0) {
            emplace_type_from_scalar(scalar_type.value());
          } else {
            typesFromTensors.emplace_back(scalar_type.value());
          }

          input_idx++;
        }
      });

```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `value`, `emplace_type_from_scalar`, `emplace_back`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`value`, `emplace_type_from_scalar`, `emplace_back`。

### Lines 254-266
```cpp
  std::optional<c10::ScalarType> st = std::nullopt;
  const auto output_st = get_scalar_type(n->output());

  if (IsComparisonOp(n->kind())) {
    // For comparison ops, always promote scalar type to highest among inputs,
    // regardless if that input is a tensor or scalar.
    typesFromScalars.insert(
        typesFromScalars.end(),
        typesFromTensors.begin(),
        typesFromTensors.end());
    st = PromoteScalarTypes(typesFromScalars);
  } else {
    if (output_st) {
```
- EN: This block handles conditional branches. Key symbols: `get_scalar_type`, `output`, `IsComparisonOp`, `kind`, `insert`, `end`, `...`.
- CN: 该代码块处理条件分支。关键符号：`get_scalar_type`, `output`, `IsComparisonOp`, `kind`, `insert`, `end`, `...`。

### Lines 267-278
```cpp
      // If output scalar type is available, use that.
      st = output_st;
    } else {
      // PyTorch now does implicit type promotion regardless whether the inputs
      // are tensors or scalars. (Previously only scalars support implicit
      // casting).
      // Per logic here
      // https://pytorch.org/docs/main/tensor_attributes.html#tensor-attributes
      st = PromoteScalarTypesWithCategory(typesFromTensors, typesFromScalars);
    }
  }

```
- EN: This block implements local helper logic for scalar type analysis. Key symbols: `PromoteScalarTypesWithCategory`.
- CN: 该代码块实现与 scalar type analysis 相关的局部辅助逻辑。关键符号：`PromoteScalarTypesWithCategory`。

### Lines 279-289
```cpp
  return st;
}

static std::optional<c10::ScalarType> LowPrecisionCastForStandardOps(
    const Node* n,
    const c10::ScalarType& scalar_type) {
  // Some of standardOps do not support uint8\int8\int16 type for ONNX
  // opset version < 14.
  // Fix in this ONNX PR:
  // https://github.com/onnx/onnx/pull/3334
  if (n->kind() != onnx::Gemm && IsStandardOp(n->kind()) &&
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `LowPrecisionCastForStandardOps`, `kind`, `IsStandardOp`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`LowPrecisionCastForStandardOps`, `kind`, `IsStandardOp`。

### Lines 290-296
```cpp
      (scalar_type == c10::kByte || scalar_type == c10::kChar ||
       scalar_type == c10::kShort)) {
    return c10::kLong;
  }
  return scalar_type;
}

```
- EN: This block produces a result or forwards a computed value. Key symbols: no dominant local symbol names.
- CN: 该代码块返回结果或转发已计算的值。关键符号：无明显局部符号。

### Lines 297-310
```cpp
static void UpdateScalarTypeForInputs(
    Node* n,
    const c10::ScalarType& scalar_type) {
  const int64_t onnx_type = ScalarTypeToONNXType(scalar_type);
  if (onnx_type < 0) {
    TORCH_WARN(
        "ONNX Scalar Type Analysis - Scalar type: ",
        c10::toString(scalar_type),
        " of input tensor in operator: ",
        n->kind().toDisplayString(),
        " not supported in ONNX. ");
    return;
  }

```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `UpdateScalarTypeForInputs`, `ScalarTypeToONNXType`, `toString`, `kind`, `toDisplayString`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`UpdateScalarTypeForInputs`, `ScalarTypeToONNXType`, `toString`, `kind`, `toDisplayString`。

### Lines 311-319
```cpp
  size_t input_idx = 0;
  for (auto input : n->inputs()) {
    auto input_tensor_type = input->type()->cast<TensorType>();
    auto input_scalar_type =
        input_tensor_type ? input_tensor_type->scalarType() : std::nullopt;

    // We skip the 'condition' input (i.e., the first input) in case of
    // onnx:Where operator.
    if (IsSelectorOp(n->kind()) && input_idx == 0) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures. Key symbols: `inputs`, `type`, `scalarType`, `IsSelectorOp`, `kind`.
- CN: 该代码块处理条件分支；遍历集合或图结构。关键符号：`inputs`, `type`, `scalarType`, `IsSelectorOp`, `kind`。

### Lines 320-326
```cpp
      input_idx++;
      continue;
    }

    if ((input->node()->kind() == onnx::Constant) ||
        (input_scalar_type && (*input_scalar_type != scalar_type))) {
      if (input->node()->kind() == onnx::Constant) {
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `node`, `kind`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`node`, `kind`。

### Lines 327-339
```cpp
        // Fix up the scalar directly instead of inserting a cast operator.
        // TODO: Keep only the else branch once constant_folding is enabled by
        // default.
        at::Tensor val = input->node()->t(attr::value);
        at::Tensor new_val = val.to(scalar_type);
        Node* const_node = n->owningGraph()->create(onnx::Constant);
        const_node->t_(attr::value, new_val);
        const_node->insertBefore(n);
        const_node->output()->setType(TensorType::create(new_val));
        const_node->copyMetadata(n);
        n->replaceInputWith(input, const_node->output());
      } else {
        Node* cast_node = n->owningGraph()->create(onnx::Cast);
```
- EN: This block manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `node`, `t`, `to`, `owningGraph`, `create`, `t_`, `...`.
- CN: 该代码块操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`node`, `t`, `to`, `owningGraph`, `create`, `t_`, `...`。

### Lines 340-349
```cpp
        cast_node->addInput(input);
        cast_node->i_(attr::to, onnx_type);
        cast_node->insertBefore(n);
        cast_node->output()->setType(CreateProfiledTensorTypeWithScalarType(
            input_tensor_type, scalar_type));
        cast_node->copyMetadata(n);
        n->replaceInputWith(input, cast_node->output());
      }
    }

```
- EN: This block manipulates TorchScript IR objects. Key symbols: `addInput`, `i_`, `insertBefore`, `output`, `setType`, `CreateProfiledTensorTypeWithScalarType`, `...`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`addInput`, `i_`, `insertBefore`, `output`, `setType`, `CreateProfiledTensorTypeWithScalarType`, `...`。

### Lines 350-357
```cpp
    input_idx++;
  }
}

static void UpdateScalarTypeForOutput(
    Node* n,
    const c10::ScalarType& scalar_type) {
  if (auto output_tensor_type = n->output()->type()->cast<TensorType>()) {
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `UpdateScalarTypeForOutput`, `output`, `type`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`UpdateScalarTypeForOutput`, `output`, `type`。

### Lines 358-364
```cpp
    n->output()->setType(CreateProfiledTensorTypeWithScalarType(
        output_tensor_type, scalar_type));
  }
}

static void RecoverScalarTypeForOutput(
    Value* out,
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `output`, `setType`, `CreateProfiledTensorTypeWithScalarType`, `RecoverScalarTypeForOutput`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`output`, `setType`, `CreateProfiledTensorTypeWithScalarType`, `RecoverScalarTypeForOutput`。

### Lines 365-376
```cpp
    const c10::ScalarType& scalar_type) {
  Node* n = out->node();
  TORCH_INTERNAL_ASSERT(nullptr != n);
  const int64_t onnx_type = ScalarTypeToONNXType(scalar_type);
  Node* cast_node = n->owningGraph()->create(onnx::Cast, 1);
  cast_node->addInput(out);
  cast_node->i_(attr::to, onnx_type);
  cast_node->insertAfter(n);
  cast_node->copyMetadata(n);
  out->replaceAllUsesAfterNodeWith(cast_node, cast_node->output());
}

```
- EN: This block manipulates TorchScript IR objects. Key symbols: `node`, `ScalarTypeToONNXType`, `owningGraph`, `create`, `addInput`, `i_`, `...`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`node`, `ScalarTypeToONNXType`, `owningGraph`, `create`, `addInput`, `i_`, `...`。

### Lines 377-390
```cpp
// This example error found when exports transfo_xl model using add op in uint8
// type, as below:
// if self.same_length:
//     all_ones = word_emb.new_ones((qlen, klen), dtype=torch.uint8)
//     mask_len = klen - self.mem_len
//     if mask_len > 0:
//         mask_shift_len = qlen - mask_len
//     else:
//         mask_shift_len = qlen
//     dec_attn_mask = (torch.triu(all_ones, 1 + mlen) + torch.tril(all_ones,
//     -mask_shift_len))[:, :, None]  # -1
//
// `all_ones is` an uint8 tensor, But the calculation of `dec_attn_mask` using
// add(+) op to get the uint8 result. Reference Link:
```
- EN: Documents the intent, constraints, or compatibility notes that guide the surrounding implementation.
- CN: 说明周围实现所遵循的目标、限制条件或兼容性要求。

### Lines 391-399
```cpp
// https://github.com/huggingface/transformers/blob/b020a736c374460af1b34267283f957988350630/src/transformers/models/transfo_xl/modeling_transfo_xl.py#L936
static void LowPrecisionCastNodeForStandardOps(Node* n, int opset_version) {
  TORCH_INTERNAL_ASSERT(n->outputs().size() == 1);
  if (n->output()->type()->cast<TensorType>() == nullptr ||
      n->output()->type()->cast<TensorType>()->scalarType() == std::nullopt) {
    // skip LowPrecisionCast if op output type is null.
    return;
  }
  auto output_scalar_type =
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `LowPrecisionCastNodeForStandardOps`, `outputs`, `size`, `output`, `type`, `scalarType`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`LowPrecisionCastNodeForStandardOps`, `outputs`, `size`, `output`, `type`, `scalarType`。

### Lines 400-407
```cpp
      n->output()->type()->cast<TensorType>()->scalarType().value();
  for (size_t i = 0; i < n->inputs().size(); ++i) {
    if (n->input(i)->type()->cast<TensorType>() == nullptr ||
        n->input(i)->type()->cast<TensorType>()->scalarType() == std::nullopt) {
      // skip LowPrecisionCast if any op input type node is null.
      return;
    }
    auto input_tensor_type =
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `output`, `type`, `scalarType`, `value`, `inputs`, `size`, `...`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`output`, `type`, `scalarType`, `value`, `inputs`, `size`, `...`。

### Lines 408-414
```cpp
        n->input(i)->type()->cast<TensorType>()->scalarType().value();
    TORCH_INTERNAL_ASSERT(output_scalar_type == input_tensor_type);
  }

  // The LowPrecision problem will be fixed in ONNX opset 14.
  if (opset_version < ONNX_OPSET_14) {
    auto expected_scalar_type_cast =
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `input`, `type`, `scalarType`, `value`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`input`, `type`, `scalarType`, `value`。

### Lines 415-423
```cpp
        LowPrecisionCastForStandardOps(n, output_scalar_type);
    UpdateScalarTypeForInputs(n, *expected_scalar_type_cast);
    if (output_scalar_type != *expected_scalar_type_cast) {
      // If input type is changed, convert it to the original type.
      RecoverScalarTypeForOutput(n->output(), output_scalar_type);
    }
  }
}

```
- EN: This block handles conditional branches. Key symbols: `LowPrecisionCastForStandardOps`, `UpdateScalarTypeForInputs`, `RecoverScalarTypeForOutput`, `output`.
- CN: 该代码块处理条件分支。关键符号：`LowPrecisionCastForStandardOps`, `UpdateScalarTypeForInputs`, `RecoverScalarTypeForOutput`, `output`。

### Lines 424-435
```cpp
static void ImplicitCastNodeForONNX(Node* n) {
  if (IsImplicitCastSupported(n->kind())) {
    auto expected_scalar_type = InferExpectedScalarType(n);
    if (expected_scalar_type) {
      UpdateScalarTypeForInputs(n, *expected_scalar_type);
      if (!IsComparisonOp(n->kind())) {
        UpdateScalarTypeForOutput(n, *expected_scalar_type);
      }
    }
  }
}

```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `ImplicitCastNodeForONNX`, `IsImplicitCastSupported`, `kind`, `InferExpectedScalarType`, `UpdateScalarTypeForInputs`, `IsComparisonOp`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`ImplicitCastNodeForONNX`, `IsImplicitCastSupported`, `kind`, `InferExpectedScalarType`, `UpdateScalarTypeForInputs`, `IsComparisonOp`, `...`。

### Lines 436-447
```cpp
static void ImplicitCastForONNX(Block* block) {
  for (auto it = block->nodes().begin(); it != block->nodes().end(); ++it) {
    for (auto sub : it->blocks()) {
      ImplicitCastForONNX(sub);
    }

    ImplicitCastNodeForONNX(*it);
  }
  EliminateDeadCode(
      block, true, DCESideEffectPolicy::ALLOW_DELETING_NODES_WITH_SIDE_EFFECTS);
}

```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects; performs optimization-oriented rewriting; protects shared state or ordering assumptions. Key symbols: `ImplicitCastForONNX`, `nodes`, `begin`, `end`, `blocks`, `ImplicitCastNodeForONNX`, `...`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象；执行面向优化的改写；保护共享状态或执行顺序假设。关键符号：`ImplicitCastForONNX`, `nodes`, `begin`, `end`, `blocks`, `ImplicitCastNodeForONNX`, `...`。

### Lines 448-455
```cpp
static void LowPrecisionCastForStandardOpsONNX(
    Block* block,
    int opset_version) {
  for (auto it = block->nodes().begin(); it != block->nodes().end(); ++it) {
    for (auto sub : it->blocks()) {
      LowPrecisionCastForStandardOpsONNX(sub, opset_version);
    }

```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `LowPrecisionCastForStandardOpsONNX`, `nodes`, `begin`, `end`, `blocks`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`LowPrecisionCastForStandardOpsONNX`, `nodes`, `begin`, `end`, `blocks`。

### Lines 456-464
```cpp
    if (IsStandardOp(it->kind())) {
      LowPrecisionCastNodeForStandardOps(*it, opset_version);
    }
  }
  EliminateDeadCode(
      block, true, DCESideEffectPolicy::ALLOW_DELETING_NODES_WITH_SIDE_EFFECTS);
}
} // anonymous namespace

```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; performs optimization-oriented rewriting; protects shared state or ordering assumptions. Key symbols: `IsStandardOp`, `kind`, `LowPrecisionCastNodeForStandardOps`, `EliminateDeadCode`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；执行面向优化的改写；保护共享状态或执行顺序假设。关键符号：`IsStandardOp`, `kind`, `LowPrecisionCastNodeForStandardOps`, `EliminateDeadCode`。

### Lines 465-471
```cpp
void ScalarTypeAnalysisForONNX(
    const std::shared_ptr<Graph>& graph,
    bool lowprecision_cast,
    int opset_version) {
  GRAPH_DUMP("Before ScalarTypeAnalysisForONNX: ", graph);
  ImplicitCastForONNX(graph->block());
  if (lowprecision_cast) {
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `ScalarTypeAnalysisForONNX`, `ImplicitCastForONNX`, `block`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`ScalarTypeAnalysisForONNX`, `ImplicitCastForONNX`, `block`。

### Lines 472-480
```cpp
    LowPrecisionCastForStandardOpsONNX(graph->block(), opset_version);
  }
  GRAPH_DUMP("After ScalarTypeAnalysisForONNX: ", graph);
}

void ScalarTypeAnalysisNodeForONNX(Node* n) {
  ImplicitCastNodeForONNX(n);
}

```
- EN: This block manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `LowPrecisionCastForStandardOpsONNX`, `block`, `ScalarTypeAnalysisNodeForONNX`, `ImplicitCastNodeForONNX`.
- CN: 该代码块操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`LowPrecisionCastForStandardOpsONNX`, `block`, `ScalarTypeAnalysisNodeForONNX`, `ImplicitCastNodeForONNX`。

### Lines 481-481
```cpp
} // namespace torch::jit
```
- EN: Opens or closes namespace scopes so the remaining declarations live in the expected JIT/frontend module.
- CN: 打开或关闭命名空间作用域，使后续声明归属于预期的 JIT/前端模块。


## Key Concepts / 关键概念
- EN: IR graph manipulation — the file inspects or rewrites TorchScript graphs and nodes.
  CN: IR 图操作——该文件会检查或改写 TorchScript 的图、节点与块。
- EN: Optimization pass — the code simplifies graphs by removing redundant work or folding values.
  CN: 优化 pass——代码通过删除冗余计算或常量折叠来简化图。
- EN: Type and shape reasoning — the code propagates static information across the IR.
  CN: 类型与形状推理——代码会在 IR 中传播静态信息。
- EN: Export interop — the implementation helps bridge TorchScript graphs to ONNX-style representations.
  CN: 导出互操作——实现帮助 TorchScript 图与 ONNX 风格表示之间建立桥接。
- EN: Subsystem integration — the implementation collaborates with nearby JIT/frontend headers and runtime helpers.
  CN: 子系统集成——实现会与附近的 JIT/前端头文件以及运行时辅助模块协同工作。

## Dependencies / 依赖关系
- Internal includes / 内部头文件: `c10/util/irange.h`, `torch/csrc/jit/jit_log.h`, `torch/csrc/jit/passes/dead_code_elimination.h`, `torch/csrc/jit/passes/onnx/helper.h`, `torch/csrc/jit/passes/onnx/scalar_type_analysis.h`
- External includes / 外部头文件: 无
- Namespaces / 命名空间: `torch::jit`, `onnx`
- Representative symbols / 代表性符号: `namespace`, `ScalarTypeToONNXType`, `find`, `end`, `IsStandardOp`, `IsComparisonOp`, `IsSelectorOp`, `CreateProfiledTensorTypeWithScalarType`, `withScalarType`, `IsImplicitCastSupported`, `...`
