# ElementwiseToLinalg.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/Linalg/Transforms/ElementwiseToLinalg.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements Linalg dialect support for rewrite patterns, passes, and IR-to-IR transformation logic, centered on `ElementwiseToLinalg`.
  - **CN**: 实现 Linalg 方言中围绕 `ElementwiseToLinalg` 的重写模式、Pass 以及 IR 到 IR 的变换逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- ElementwiseToLinalg.cpp - conversion of elementwise to linalg ------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and file-level summary.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件级摘要。

### Lines 8-16
```cpp

#include "mlir/Dialect/Linalg/Passes.h"

#include "mlir/Dialect/Linalg/IR/Linalg.h"
#include "mlir/Dialect/Linalg/Transforms/Transforms.h"
#include "mlir/Dialect/Linalg/Utils/Utils.h"
#include "mlir/Transforms/DialectConversion.h"
#include "llvm/ADT/SmallVectorExtras.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/Linalg/Passes.h`, `mlir/Dialect/Linalg/IR/Linalg.h`, `mlir/Dialect/Linalg/Transforms/Transforms.h`, `mlir/Dialect/Linalg/Utils/Utils.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/Linalg/Passes.h`, `mlir/Dialect/Linalg/IR/Linalg.h`, `mlir/Dialect/Linalg/Transforms/Transforms.h`, `mlir/Dialect/Linalg/Utils/Utils.h`。

### Lines 17-23
```cpp
namespace mlir {
#define GEN_PASS_DEF_CONVERTELEMENTWISETOLINALGPASS
#include "mlir/Dialect/Linalg/Passes.h.inc"
} // namespace mlir

using namespace mlir;

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/Linalg/Passes.h.inc`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/Linalg/Passes.h.inc`。

### Lines 24-31
```cpp
static inline bool isScalarLike(Type t) {
  return isa<IntegerType, FloatType, IndexType, ComplexType>(t);
}

static bool isElementwiseMappableOpOnRankedTensors(Operation *op) {
  if (!OpTrait::hasElementwiseMappableTraits(op))
    return false;

```
- **EN**: Implements logic around `isScalarLike`, `ComplexType>`, `isElementwiseMappableOpOnRankedTensors`, `hasElementwiseMappableTraits`.
- **CN**: 围绕 `isScalarLike`, `ComplexType>`, `isElementwiseMappableOpOnRankedTensors`, `hasElementwiseMappableTraits` 实现具体逻辑。

### Lines 32-42
```cpp
  auto types = op->getOperandTypes();

  // We want at least one ranked tensor.
  bool anyRankedTensor = llvm::any_of(types, llvm::IsaPred<RankedTensorType>);

  // No invalid operands (i.e., every operand is a ranked tensor or
  // scalar-like).
  bool noneInvalid = llvm::none_of(types, [](Type t) {
    return !(isa<RankedTensorType>(t) || isScalarLike(t));
  });

```
- **EN**: Implements logic around `getOperandTypes`, `any_of`, `none_of`, `isScalarLike`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getOperandTypes`, `any_of`, `none_of`, `isScalarLike` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 43-49
```cpp
  return anyRankedTensor && noneInvalid;
}

/// Given `op` assumed `isElementwiseMappableOpOnRankedTensors`, iterate over
/// the result types and return a list of values such that, for each result type
/// `t` and value `v` at the same index `idx`:
///   1. `v.getType() == t`
```
- **EN**: Finalizes a local computation, invariant check, or helper decision.
- **CN**: 完成局部计算、不变式检查或辅助决策。

### Lines 50-56
```cpp
///   2. If an operand of `op` has type `t`, let `operand_first` be the first
///      such operand. Then`v == operand_first`.
///   3. Otherwise, v is a newly created `tensor::EmptyOp` with:
///        a. Static and dynamic dims extracted from the first operand of `op`.
///        b. Elemental type equal to the elemental type of `t`.
///
/// This is sufficient because ElementwiseMappable guarantees that "The static
```
- **EN**: Documents the next declarations, transformation intent, or design constraints for this file.
- **CN**: 为后续声明、变换意图或设计约束提供说明。

### Lines 57-70
```cpp
/// types of all vector (resp. tensor) operands and results must have the same
/// shape".
static SmallVector<Value, 4>
getOrCreateOperandsMatchingResultTypes(OpBuilder &b, Operation *op) {
  assert(isElementwiseMappableOpOnRankedTensors(op));
  Location loc = op->getLoc();
  ValueRange operands = op->getOperands();
  TypeRange rankedTensorTypes = op->getResultTypes();
  SmallVector<Value, 4> res;
  res.reserve(rankedTensorTypes.size());
  for (Type t : rankedTensorTypes) {
    // Try to find an operand with type matching the result tensor.
    bool found = false;
    for (Value v : operands) {
```
- **EN**: Implements logic around `getOrCreateOperandsMatchingResultTypes`, `assert`, `getLoc`, `getOperands`, and 2 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getOrCreateOperandsMatchingResultTypes`, `assert`, `getLoc`, `getOperands`, and 2 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 71-79
```cpp
      if (v.getType() == t) {
        found = true;
        res.push_back(v);
        break;
      }
    }
    if (found)
      continue;

```
- **EN**: Implements logic around `getType`, `push_back`.
- **CN**: 围绕 `getType`, `push_back` 实现具体逻辑。

### Lines 80-87
```cpp
    // Extract static / dynamic shape mix from the first operand.
    res.push_back(tensor::EmptyOp::create(
        b, loc, tensor::getMixedSizes(b, loc, operands.front()),
        cast<RankedTensorType>(t).getElementType()));
  }
  return res;
}

```
- **EN**: Implements logic around `push_back`, `getMixedSizes`, `getElementType`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `push_back`, `getMixedSizes`, `getElementType` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 88-97
```cpp
namespace {
struct ConvertAnyElementwiseMappableOpOnRankedTensors : public RewritePattern {
  ConvertAnyElementwiseMappableOpOnRankedTensors(MLIRContext *context)
      : RewritePattern(MatchAnyOpTypeTag(), /*benefit=*/1, context) {}
  LogicalResult matchAndRewrite(Operation *op,
                                PatternRewriter &rewriter) const final {
    if (!isElementwiseMappableOpOnRankedTensors(op))
      return rewriter.notifyMatchFailure(
          op, "requires elementwise op on ranked tensors");

```
- **EN**: Introduces declarations for `ConvertAnyElementwiseMappableOpOnRankedTensors`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `ConvertAnyElementwiseMappableOpOnRankedTensors` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 98-105
```cpp
    auto resTy = cast<RankedTensorType>(op->getResult(0).getType());
    auto rank = resTy.getRank();

    // Maps: identity for tensors (rank > 0), scalar map for scalars.
    AffineMap scalarMap = AffineMap::get(/*dimCount=*/rank, /*symbolCount=*/0,
                                         /*results=*/{}, rewriter.getContext());
    AffineMap idMap = rewriter.getMultiDimIdentityMap(rank);

```
- **EN**: Implements logic around `getResult`, `getRank`, `get`, `getContext`, and 1 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `getResult`, `getRank`, `get`, `getContext`, and 1 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 106-119
```cpp
    // Match phase.
    SmallVector<bool> isScalarOperand;
    isScalarOperand.reserve(op->getNumOperands());
    for (Type ty : op->getOperandTypes()) {
      if (isScalarLike(ty))
        isScalarOperand.push_back(true);
      else if (auto rt = dyn_cast<RankedTensorType>(ty))
        isScalarOperand.push_back(false);
      else
        return rewriter.notifyMatchFailure(
            op,
            "unsupported operand type (expected scalar-like or ranked tensor)");
    }

```
- **EN**: Implements logic around `reserve`, `getOperandTypes`, `isScalarLike`, `push_back`, and 2 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `reserve`, `getOperandTypes`, `isScalarLike`, `push_back`, and 2 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 120-126
```cpp
    // Create indexing maps.
    SmallVector<AffineMap> indexingMaps;
    indexingMaps.reserve(op->getNumOperands() + op->getNumResults());

    for (bool isScalar : isScalarOperand)
      indexingMaps.push_back(isScalar ? scalarMap : idMap);

```
- **EN**: Implements logic around `reserve`, `push_back`.
- **CN**: 围绕 `reserve`, `push_back` 实现具体逻辑。

### Lines 127-140
```cpp
    indexingMaps.append(op->getNumResults(), idMap);

    SmallVector<utils::IteratorType> iteratorTypes(
        rank, utils::IteratorType::parallel);
    SmallVector<Value> outputs =
        getOrCreateOperandsMatchingResultTypes(rewriter, op);
    rewriter.replaceOpWithNewOp<linalg::GenericOp>(
        op, /*resultTensorTypes=*/op->getResultTypes(),
        /*inputs=*/op->getOperands(),
        /*outputs=*/outputs,
        /*indexingMaps=*/indexingMaps,
        /*iteratorTypes=*/iteratorTypes,
        /*bodyBuilder=*/
        [&](OpBuilder &builder, Location loc, ValueRange regionArgs) {
```
- **EN**: Implements logic around `append`, `iteratorTypes`, `getOrCreateOperandsMatchingResultTypes`, `GenericOp>`, and 2 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `append`, `iteratorTypes`, `getOrCreateOperandsMatchingResultTypes`, `GenericOp>`, and 2 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 141-154
```cpp
          SmallVector<Type> resultEltTys =
              llvm::map_to_vector<6>(op->getResultTypes(), [](Type type) {
                return cast<TensorType>(type).getElementType();
              });
          Operation *scalarOp =
              builder.create(loc, op->getName().getIdentifier(),
                             regionArgs.take_front(op->getNumOperands()),
                             resultEltTys, op->getAttrs());
          linalg::YieldOp::create(builder, loc, scalarOp->getResults());
        });
    return success();
  }
};
} // namespace
```
- **EN**: Implements logic around `map_to_vector`, `getElementType`, `create`, `take_front`, and 2 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `map_to_vector`, `getElementType`, `create`, `take_front`, and 2 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 155-161
```cpp

void mlir::linalg::populateElementwiseToLinalgConversionPatterns(
    RewritePatternSet &patterns) {
  patterns.add<ConvertAnyElementwiseMappableOpOnRankedTensors>(
      patterns.getContext());
}

```
- **EN**: Implements logic around `populateElementwiseToLinalgConversionPatterns`, `add`, `getContext`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `populateElementwiseToLinalgConversionPatterns`, `add`, `getContext` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 162-168
```cpp
namespace {
class ConvertElementwiseToLinalgPass
    : public impl::ConvertElementwiseToLinalgPassBase<
          ConvertElementwiseToLinalgPass> {
  using impl::ConvertElementwiseToLinalgPassBase<
      ConvertElementwiseToLinalgPass>::ConvertElementwiseToLinalgPassBase;

```
- **EN**: Introduces declarations for `ConvertElementwiseToLinalgPass`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `ConvertElementwiseToLinalgPass` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 169-179
```cpp
  void runOnOperation() final {
    auto *func = getOperation();
    auto *context = &getContext();
    ConversionTarget target(*context);
    RewritePatternSet patterns(context);

    mlir::linalg::populateElementwiseToLinalgConversionPatterns(patterns);
    target.markUnknownOpDynamicallyLegal([](Operation *op) {
      return !isElementwiseMappableOpOnRankedTensors(op);
    });

```
- **EN**: Implements logic around `runOnOperation`, `getOperation`, `getContext`, `target`, and 4 more symbols; this block packages logic as an MLIR pass or pass helper; coordinates dialect conversion or lowering decisions; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `runOnOperation`, `getOperation`, `getContext`, `target`, and 4 more symbols 实现具体逻辑；该代码块将逻辑组织为 MLIR Pass 或 Pass 辅助组件，并协调方言转换或 lowering 决策，并协调核心结构化 MLIR 方言之间的行为。

### Lines 180-184
```cpp
    if (failed(applyPartialConversion(func, target, std::move(patterns))))
      signalPassFailure();
  }
};
} // namespace
```
- **EN**: Implements logic around `failed`, `signalPassFailure`.
- **CN**: 围绕 `failed`, `signalPassFailure` 实现具体逻辑。

## Key Concepts / 关键概念

- **Rewrite and lowering / 重写与 lowering**:
  - **EN**: Uses MLIR pattern infrastructure or passes to canonicalize, legalize, or lower operations.
  - **CN**: 使用 MLIR 模式基础设施或 Pass 对操作进行规范化、合法化或 lowering。
- **Pattern rewriting / 模式重写**:
  - **EN**: Uses MLIR rewrite patterns to match operations and replace them with improved forms.
  - **CN**: 使用 MLIR 重写模式匹配操作，并将其替换为更合适的形式。
- **Dialect conversion pipeline / 方言转换流水线**:
  - **EN**: Coordinates legality checks, type adaptation, and target-specific lowering.
  - **CN**: 协调合法性检查、类型适配以及目标相关 lowering。
- **Pass infrastructure / Pass 基础设施**:
  - **EN**: Wraps transformations as reusable passes that can be scheduled in MLIR pipelines.
  - **CN**: 将变换封装为可在 MLIR 流水线中调度的可复用 Pass。
- **Structured IR coordination / 结构化 IR 协同**:
  - **EN**: Interacts with structured MLIR dialects that model loops, tensors, memory, and vector semantics.
  - **CN**: 与建模循环、张量、内存和向量语义的结构化 MLIR 方言交互。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/Linalg/Passes.h`, `mlir/Dialect/Linalg/IR/Linalg.h`, `mlir/Dialect/Linalg/Transforms/Transforms.h`, `mlir/Dialect/Linalg/Utils/Utils.h`, `mlir/Transforms/DialectConversion.h`, `llvm/ADT/SmallVectorExtras.h`, `mlir/Dialect/Linalg/Passes.h.inc`
- **Subsystem categories / 子系统类别**: dialect-specific operation, type, attribute, or transform declarations / 方言相关的操作、类型、属性或变换声明 (5), generic transformation and conversion helpers / 通用变换与转换辅助逻辑 (1), LLVM ADT containers and low-level helpers / LLVM ADT 容器与底层辅助工具 (1)
