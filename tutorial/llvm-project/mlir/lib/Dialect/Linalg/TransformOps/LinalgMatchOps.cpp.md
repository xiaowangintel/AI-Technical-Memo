# LinalgMatchOps.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/Linalg/TransformOps/LinalgMatchOps.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements transform-dialect operations, matchers, or extensions for the Linalg dialect and structured tensor computation.
  - **CN**: 实现 Linalg 方言与结构化张量计算 的 Transform Dialect 操作、匹配器或扩展。

## Line-by-Line Analysis / 逐行分析

### Lines 1-22
```cpp
//===- LinalgTransformOps.cpp - Implementation of Linalg match ops --------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "mlir/Dialect/Linalg/TransformOps/LinalgMatchOps.h"
#include "mlir/Analysis/SliceAnalysis.h"
#include "mlir/Dialect/Linalg/IR/Linalg.h"
#include "mlir/Dialect/Linalg/IR/LinalgInterfaces.h"
#include "mlir/Dialect/Linalg/TransformOps/Syntax.h"
#include "mlir/Dialect/Linalg/Utils/Utils.h"
#include "mlir/Dialect/Transform/IR/TransformTypes.h"
#include "mlir/Dialect/Transform/Interfaces/MatchInterfaces.h"
#include "mlir/IR/BuiltinAttributes.h"
#include "llvm/ADT/SmallVectorExtras.h"
#include "llvm/Support/DebugLog.h"
#include "llvm/Support/FormatVariadic.h"
#include "llvm/Support/InterleavedRange.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/Linalg/TransformOps/LinalgMatchOps.h`, `mlir/Analysis/SliceAnalysis.h`, `mlir/Dialect/Linalg/IR/Linalg.h`, `mlir/Dialect/Linalg/IR/LinalgInterfaces.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/Linalg/TransformOps/LinalgMatchOps.h`, `mlir/Analysis/SliceAnalysis.h`, `mlir/Dialect/Linalg/IR/Linalg.h`, `mlir/Dialect/Linalg/IR/LinalgInterfaces.h`。

### Lines 23-46
```cpp
using namespace mlir;

#define DEBUG_TYPE "linalg-transforms"

//===----------------------------------------------------------------------===//
// StructuredMatchOp
//===----------------------------------------------------------------------===//

DiagnosedSilenceableFailure transform::MatchStructuredOp::matchOperation(
    Operation *current, transform::TransformResults &results,
    transform::TransformState &state) {
  // First, check if the payload operation is a structured Linalg operation.
  if (!isa<linalg::LinalgOp>(current)) {
    if (getFailurePropagationMode().value_or(
            FailurePropagationMode::Propagate) ==
        FailurePropagationMode::Propagate) {
      return emitSilenceableError() << "expected a Linalg op";
    }
    // If errors are suppressed, succeed and set all results to empty lists.
    LDBG() << "optional nested matcher expected a Linalg op";
    results.setRemainingToEmpty(cast<TransformOpInterface>(getOperation()));
    return DiagnosedSilenceableFailure::success();
  }

```
- **EN**: Implements logic around `matchOperation`, `LinalgOp>`, `getFailurePropagationMode`, `emitSilenceableError`, and 3 more symbols.
- **CN**: 围绕 `matchOperation`, `LinalgOp>`, `getFailurePropagationMode`, `emitSilenceableError`, and 3 more symbols 实现具体逻辑。

### Lines 47-61
```cpp
  // Bind `current` to the block argument.
  auto scope = state.make_region_scope(getBodyRegion());
  if (failed(state.mapBlockArgument(getBody()->getArgument(0),
                                    MappedValue(current)))) {
    return DiagnosedSilenceableFailure::definiteFailure();
  }

  for (Operation &nested : getBody()->without_terminator()) {
    DiagnosedSilenceableFailure diag =
        state.applyTransform(cast<TransformOpInterface>(nested));
    if (diag.isDefiniteFailure())
      return diag;
    if (diag.succeeded())
      continue;

```
- **EN**: Implements logic around `make_region_scope`, `failed`, `MappedValue`, `definiteFailure`, and 4 more symbols; this block manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `make_region_scope`, `failed`, `MappedValue`, `definiteFailure`, and 4 more symbols 实现具体逻辑；该代码块处理 MLIR region、block 或控制流边。

### Lines 62-89
```cpp
    // If propagating errors, do this immediately.
    assert(diag.isSilenceableFailure());
    if (getFailurePropagationMode().value_or(
            FailurePropagationMode::Propagate) ==
        FailurePropagationMode::Propagate) {
      return diag;
    }

    // If suppressing errors, print the message into the debug stream before
    // silencing it. Then set all results value that are already known.
    // Results come from the terminator operands, which may be defined in the
    // (single) block of this operation or above it. When they are defined
    // above, they are known to be mapped at this point per SSA dominance.
    // When they are defined in this block, we additionally check if we have
    // already applied the operation that defines them. If not, the
    // corresponding results will be set to empty lists.
    LDBG() << "optional nested matcher failed: " << diag.getMessage();
    (void)diag.silence();
    SmallVector<OpOperand *> undefinedOperands;
    for (OpOperand &terminatorOperand :
         getBody()->getTerminator()->getOpOperands()) {
      Operation *definingOp = terminatorOperand.get().getDefiningOp();
      if (!definingOp)
        continue;
      if (definingOp->getBlock() != getBody())
        continue;
      if (definingOp->isBeforeInBlock(&nested))
        continue;
```
- **EN**: Implements logic around `assert`, `getFailurePropagationMode`, `LDBG`, `silence`, and 4 more symbols; this block implements verifier, folding, parsing, or printing hooks; manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `assert`, `getFailurePropagationMode`, `LDBG`, `silence`, and 4 more symbols 实现具体逻辑；该代码块实现验证、折叠、解析或打印钩子，并处理 MLIR region、block 或控制流边。

### Lines 90-109
```cpp

      undefinedOperands.push_back(&terminatorOperand);
    }

    SmallVector<SmallVector<transform::MappedValue>> mappings;
    auto filtered = llvm::make_filter_range(
        getBody()->getTerminator()->getOpOperands(), [&](OpOperand &opOperand) {
          return !llvm::is_contained(undefinedOperands, &opOperand);
        });
    SmallVector<Value> definedOperands = llvm::map_to_vector(
        filtered, [](OpOperand &opOperand) { return opOperand.get(); });
    detail::prepareValueMappings(mappings, definedOperands, state);
    for (auto &&[operand, mapping] : llvm::zip_equal(filtered, mappings)) {
      results.setMappedValues(getResults()[operand.getOperandNumber()],
                              mapping);
    }
    results.setRemainingToEmpty(cast<TransformOpInterface>(getOperation()));
    return DiagnosedSilenceableFailure::success();
  }

```
- **EN**: Implements logic around `push_back`, `make_filter_range`, `getBody`, `is_contained`, and 7 more symbols.
- **CN**: 围绕 `push_back`, `make_filter_range`, `getBody`, `is_contained`, and 7 more symbols 实现具体逻辑。

### Lines 110-137
```cpp
  // Set the results.
  detail::forwardTerminatorOperands(getBody(), state, results);
  return DiagnosedSilenceableFailure::success();
}

void transform::MatchStructuredOp::getEffects(
    SmallVectorImpl<MemoryEffects::EffectInstance> &effects) {
  onlyReadsHandle(getCurrentMutable(), effects);
  onlyReadsPayload(effects);
  producesHandle(getOperation()->getOpResults(), effects);
}

LogicalResult transform::MatchStructuredOp::verify() {
  if (getBody()->getNumArguments() != 1)
    return emitOpError() << "expected one body argument";
  if (!isa<TransformHandleTypeInterface>(getBody()->getArgument(0).getType())) {
    return emitOpError() << "expected body argument to implement "
                            "TransformHandleTypeInterface";
  }
  for (Operation &nested : getBody()->without_terminator()) {
    if (isa<MatchOpInterface>(nested))
      continue;
    InFlightDiagnostic diag =
        emitOpError()
        << "expects nested operations to implement MatchOpInterface";
    diag.attachNote(nested.getLoc()) << "offending operation";
    return diag;
  }
```
- **EN**: Implements logic around `forwardTerminatorOperands`, `success`, `getEffects`, `onlyReadsHandle`, and 7 more symbols; this block implements verifier, folding, parsing, or printing hooks.
- **CN**: 围绕 `forwardTerminatorOperands`, `success`, `getEffects`, `onlyReadsHandle`, and 7 more symbols 实现具体逻辑；该代码块实现验证、折叠、解析或打印钩子。

### Lines 138-151
```cpp
  return success();
}

//===----------------------------------------------------------------------===//
// StructuredOpPredicateOpTrait
//===----------------------------------------------------------------------===//

LogicalResult transform::detail::verifyStructuredOpPredicateOpTrait(
    Operation *op, Value structuredOpHandle) {
  if (!isa_and_nonnull<MatchStructuredOp>(op->getParentOp())) {
    return op->emitOpError() << "expects parent op to be '"
                             << MatchStructuredOp::getOperationName() << "'";
  }

```
- **EN**: Implements logic around `success`, `verifyStructuredOpPredicateOpTrait`, `isa_and_nonnull`, `emitOpError`, and 1 more symbols.
- **CN**: 围绕 `success`, `verifyStructuredOpPredicateOpTrait`, `isa_and_nonnull`, `emitOpError`, and 1 more symbols 实现具体逻辑。

### Lines 152-165
```cpp
  // Bail out here, let the verifier of the parent complain.
  Operation *parent = op->getParentOp();
  if (parent->getNumRegions() < 1 || parent->getRegion(0).empty() ||
      parent->getRegion(0).front().getNumArguments() < 1)
    return success();

  if (structuredOpHandle != parent->getRegion(0).front().getArgument(0)) {
    return op->emitOpError()
           << "expected predicate to apply to the surrounding structured op";
  }
  return success();
}

//===----------------------------------------------------------------------===//
```
- **EN**: Implements logic around `getParentOp`, `getNumRegions`, `getRegion`, `success`, and 1 more symbols.
- **CN**: 围绕 `getParentOp`, `getNumRegions`, `getRegion`, `success`, and 1 more symbols 实现具体逻辑。

### Lines 166-193
```cpp
// MatchStructuredBodyOp
//===----------------------------------------------------------------------===//

DiagnosedSilenceableFailure transform::MatchStructuredBodyOp::matchOperation(
    Operation *current, transform::TransformResults &results,
    transform::TransformState &state) {
  auto linalgOp = cast<linalg::LinalgOp>(current);
  if (std::optional<uint64_t> position = getReductionPosition()) {
    SmallVector<Operation *> combinerOps;
    if (!matchReduction(linalgOp.getRegionOutputArgs(), *position,
                        combinerOps)) {
      return emitSilenceableError() << "could not match reduction";
    }
    if (combinerOps.size() != 1) {
      return emitSilenceableError() << "reduction combiner is not a single op";
    }
    return DiagnosedSilenceableFailure::success();
  }
  if (getPassthrough()) {
    Block &body = linalgOp->getRegion(0).front();
    if (body.getTerminator()->getOperands() != linalgOp.getRegionInputArgs()) {
      return emitSilenceableError() << "not a passthrough";
    }
    return DiagnosedSilenceableFailure::success();
  }
  if (getElementwise()) {
    if (!isElementwise(linalgOp))
      return emitSilenceableError() << "not elementwise";
```
- **EN**: Implements logic around `matchOperation`, `LinalgOp>`, `getReductionPosition`, `matchReduction`, and 8 more symbols; this block manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `matchOperation`, `LinalgOp>`, `getReductionPosition`, `matchReduction`, and 8 more symbols 实现具体逻辑；该代码块处理 MLIR region、block 或控制流边。

### Lines 194-215
```cpp
    return DiagnosedSilenceableFailure::success();
  }
  if (std::optional<ArrayAttr> contractionOps = getContraction()) {
    Block &body = linalgOp->getRegion(0).front();
    std::string message;
    llvm::raw_string_ostream os(message);
    bool result = linalg::detail::isContractionBody(
        body,
        [&](Operation *elem, Operation *red) {
          return elem->getName().getStringRef() ==
                     cast<StringAttr>((*contractionOps)[0]).getValue() &&
                 red->getName().getStringRef() ==
                     cast<StringAttr>((*contractionOps)[1]).getValue();
        },
        os);
    if (result)
      return DiagnosedSilenceableFailure::success();
    return emitSilenceableError() << "contraction: " << message;
  }
  return emitDefiniteFailure() << "unknown body condition";
}

```
- **EN**: Implements logic around `success`, `getContraction`, `getRegion`, `os`, and 5 more symbols; this block manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `success`, `getContraction`, `getRegion`, `os`, and 5 more symbols 实现具体逻辑；该代码块处理 MLIR region、block 或控制流边。

### Lines 216-236
```cpp
LogicalResult transform::MatchStructuredBodyOp::verify() {
  int64_t numOptions = getReductionPosition().has_value() + getPassthrough() +
                       getElementwise() + getContraction().has_value();

  if (numOptions > 1) {
    StringAttr attributeNames[] = {
        getReductionPositionAttrName(), getPassthroughAttrName(),
        getElementwiseAttrName(), getContractionAttrName()};
    return emitOpError() << "only one of {" << llvm::interleaved(attributeNames)
                         << "} is allowed";
  }

  if (std::optional<ArrayAttr> contractionAttr = getContraction()) {
    if (contractionAttr->size() != 2) {
      return emitOpError() << "expects " << getContractionAttrName()
                           << " to contain two elements";
    }
  }
  return success();
}

```
- **EN**: Implements logic around `verify`, `getReductionPosition`, `getElementwise`, `getReductionPositionAttrName`, and 5 more symbols; this block implements verifier, folding, parsing, or printing hooks.
- **CN**: 围绕 `verify`, `getReductionPosition`, `getElementwise`, `getReductionPositionAttrName`, and 5 more symbols 实现具体逻辑；该代码块实现验证、折叠、解析或打印钩子。

### Lines 237-264
```cpp
//===----------------------------------------------------------------------===//
// MatchStructuredClassifyContractionDimsOp
//===----------------------------------------------------------------------===//

DiagnosedSilenceableFailure
transform::MatchStructuredClassifyContractionDimsOp::matchOperation(
    Operation *current, transform::TransformResults &results,
    transform::TransformState &state) {
  FailureOr<linalg::ContractionDimensions> contractionDims =
      linalg::inferContractionDims(cast<linalg::LinalgOp>(current));
  if (failed(contractionDims))
    return emitSilenceableError() << "could not infer contraction dimensions";

  MLIRContext *context = current->getContext();
  Builder builder(context);
  auto makeI64Attrs = [&](ArrayRef<unsigned> values) {
    return llvm::map_to_vector(values, [&](unsigned value) -> Attribute {
      return builder.getI64IntegerAttr(value);
    });
  };
  results.setParams(cast<OpResult>(getBatch()),
                    makeI64Attrs(contractionDims->batch));
  results.setParams(cast<OpResult>(getM()), makeI64Attrs(contractionDims->m));
  results.setParams(cast<OpResult>(getN()), makeI64Attrs(contractionDims->n));
  results.setParams(cast<OpResult>(getK()), makeI64Attrs(contractionDims->k));
  return DiagnosedSilenceableFailure::success();
}

```
- **EN**: Implements logic around `matchOperation`, `inferContractionDims`, `failed`, `emitSilenceableError`, and 7 more symbols; this block implements verifier, folding, parsing, or printing hooks.
- **CN**: 围绕 `matchOperation`, `inferContractionDims`, `failed`, `emitSilenceableError`, and 7 more symbols 实现具体逻辑；该代码块实现验证、折叠、解析或打印钩子。

### Lines 265-292
```cpp
//===----------------------------------------------------------------------===//
// MatchStructuredClassifyConvolutionDimsOp
//===----------------------------------------------------------------------===//

DiagnosedSilenceableFailure
transform::MatchStructuredClassifyConvolutionDimsOp::matchOperation(
    Operation *current, transform::TransformResults &results,
    transform::TransformState &state) {
  FailureOr<linalg::ConvolutionDimensions> convolutionDims =
      linalg::inferConvolutionDims(cast<linalg::LinalgOp>(current));
  if (failed(convolutionDims))
    return emitSilenceableError() << "could not infer convolution dimensions";

  MLIRContext *context = current->getContext();
  Builder builder(context);
  auto makeI64Attrs = [&](ArrayRef<unsigned> values) {
    return llvm::map_to_vector(values, [&](unsigned value) -> Attribute {
      return builder.getI64IntegerAttr(value);
    });
  };
  results.setParams(cast<OpResult>(getBatch()),
                    makeI64Attrs(convolutionDims->batch));
  results.setParams(cast<OpResult>(getOutputImage()),
                    makeI64Attrs(convolutionDims->outputImage));
  results.setParams(cast<OpResult>(getOutputChannel()),
                    makeI64Attrs(convolutionDims->outputChannel));
  results.setParams(cast<OpResult>(getFilterLoop()),
                    makeI64Attrs(convolutionDims->filterLoop));
```
- **EN**: Implements logic around `matchOperation`, `inferConvolutionDims`, `failed`, `emitSilenceableError`, and 6 more symbols; this block implements verifier, folding, parsing, or printing hooks.
- **CN**: 围绕 `matchOperation`, `inferConvolutionDims`, `failed`, `emitSilenceableError`, and 6 more symbols 实现具体逻辑；该代码块实现验证、折叠、解析或打印钩子。

### Lines 293-309
```cpp
  results.setParams(cast<OpResult>(getInputChannel()),
                    makeI64Attrs(convolutionDims->inputChannel));
  results.setParams(cast<OpResult>(getDepth()),
                    makeI64Attrs(convolutionDims->depth));

  auto makeI64AttrsFromI64 = [&](ArrayRef<int64_t> values) {
    return llvm::map_to_vector(values, [&](int64_t value) -> Attribute {
      return builder.getI64IntegerAttr(value);
    });
  };
  results.setParams(cast<OpResult>(getStrides()),
                    makeI64AttrsFromI64(convolutionDims->strides));
  results.setParams(cast<OpResult>(getDilations()),
                    makeI64AttrsFromI64(convolutionDims->dilations));
  return DiagnosedSilenceableFailure::success();
}

```
- **EN**: Implements logic around `setParams`, `makeI64Attrs`, `map_to_vector`, `getI64IntegerAttr`, and 2 more symbols.
- **CN**: 围绕 `setParams`, `makeI64Attrs`, `map_to_vector`, `getI64IntegerAttr`, and 2 more symbols 实现具体逻辑。

### Lines 310-333
```cpp
//===----------------------------------------------------------------------===//
// Utilities for structured match predicates.
//===----------------------------------------------------------------------===//

/// Checks if all values from `list` are also contained in `reference`. Returns
/// a silenceable error with the given message at the given location when it is
/// not the case. The error message must contain the "{0}" placeholder that
/// will be substituted with the value from `list` that is not contained in
/// `reference`.
static DiagnosedSilenceableFailure containsAll(ArrayRef<unsigned> reference,
                                               ArrayRef<int64_t> list,
                                               Location loc,
                                               const char *message) {
  for (int64_t value : list) {
    if (llvm::any_of(reference, [&](unsigned ref) {
          return static_cast<int64_t>(ref) == value;
        })) {
      continue;
    }
    return emitSilenceableFailure(loc) << llvm::formatv(message, value);
  }
  return DiagnosedSilenceableFailure::success();
}

```
- **EN**: Implements logic around `containsAll`, `any_of`, `static_cast`, `emitSilenceableFailure`, and 1 more symbols.
- **CN**: 围绕 `containsAll`, `any_of`, `static_cast`, `emitSilenceableFailure`, and 1 more symbols 实现具体逻辑。

### Lines 334-354
```cpp
//===----------------------------------------------------------------------===//
// MatchStructuredDimOp
//===----------------------------------------------------------------------===//

DiagnosedSilenceableFailure transform::MatchStructuredDimOp::matchOperation(
    Operation *current, transform::TransformResults &results,
    transform::TransformState &state) {
  auto linalgOp = cast<linalg::LinalgOp>(current);
  SmallVector<int64_t> dimensions;
  DiagnosedSilenceableFailure diag = getDimensionsFor(linalgOp, dimensions);
  if (!diag.succeeded())
    return diag;

  // If asked to check for the kind of dimension, perform the check.
  if (getParallel() || getReduction()) {
    SmallVector<unsigned> reference;
    if (getParallel())
      linalgOp.getParallelDims(reference);
    else if (getReduction())
      linalgOp.getReductionDims(reference);

```
- **EN**: Implements logic around `matchOperation`, `LinalgOp>`, `getDimensionsFor`, `succeeded`, and 4 more symbols.
- **CN**: 围绕 `matchOperation`, `LinalgOp>`, `getDimensionsFor`, `succeeded`, and 4 more symbols 实现具体逻辑。

### Lines 355-376
```cpp
    DiagnosedSilenceableFailure diag =
        containsAll(reference, dimensions, getLoc(),
                    getParallel() ? "expects dimension #{0} to be parallel"
                                  : "expects dimension #{0} to be reduction");
    if (!diag.succeeded())
      return diag;
  }

  // If not capturing, we are done here.
  if (!getResult())
    return diag;

  SmallVector<int64_t, 4> ranges = linalgOp.getStaticLoopRanges();
  Builder builder(current);
  SmallVector<Attribute> captured =
      llvm::map_to_vector(dimensions, [&](int64_t dim) -> Attribute {
        return builder.getI64IntegerAttr(ranges[dim]);
      });
  results.setParams(cast<OpResult>(getResult()), captured);
  return DiagnosedSilenceableFailure::success();
}

```
- **EN**: Implements logic around `containsAll`, `getParallel`, `succeeded`, `getResult`, and 6 more symbols.
- **CN**: 围绕 `containsAll`, `getParallel`, `succeeded`, `getResult`, and 6 more symbols 实现具体逻辑。

### Lines 377-397
```cpp
DiagnosedSilenceableFailure transform::MatchStructuredDimOp::getDimensionsFor(
    linalg::LinalgOp op, SmallVectorImpl<int64_t> &dims) {
  DiagnosedSilenceableFailure diag =
      expandTargetSpecification(getLoc(), getIsAll(), getIsInverted(),
                                getRawDimList(), op.getNumLoops(), dims);
  if (diag.isSilenceableFailure()) {
    diag.attachNote(op->getLoc())
        << "while considering dimensions of this payload operation";
  }
  return diag;
}

LogicalResult transform::MatchStructuredDimOp::verify() {
  if (getParallel() && getReduction()) {
    return emitOpError() << "cannot request the same dimension to be both "
                            "parallel and reduction";
  }
  return verifyTransformMatchDimsOp(getOperation(), getRawDimList(),
                                    getIsInverted(), getIsAll());
}

```
- **EN**: Implements logic around `getDimensionsFor`, `expandTargetSpecification`, `getRawDimList`, `isSilenceableFailure`, and 6 more symbols; this block implements verifier, folding, parsing, or printing hooks.
- **CN**: 围绕 `getDimensionsFor`, `expandTargetSpecification`, `getRawDimList`, `isSilenceableFailure`, and 6 more symbols 实现具体逻辑；该代码块实现验证、折叠、解析或打印钩子。

### Lines 398-411
```cpp
//===----------------------------------------------------------------------===//
// MatchStructuredElementalBitwidthOp
//===----------------------------------------------------------------------===//

DiagnosedSilenceableFailure
transform::MatchStructuredElementalBitwidthOp::matchValue(
    Value current, transform::TransformResults &results,
    transform::TransformState &state) {
  auto setupResult = [&](int64_t bitwidth) {
    Attribute attr = Builder(current.getContext()).getI64IntegerAttr(bitwidth);
    results.setParams(cast<OpResult>(getResult()), {attr});
    return DiagnosedSilenceableFailure::success();
  };

```
- **EN**: Implements logic around `matchValue`, `Builder`, `setParams`, `success`.
- **CN**: 围绕 `matchValue`, `Builder`, `setParams`, `success` 实现具体逻辑。

### Lines 412-426
```cpp
  Type type = current.getType();
  if (type.isIntOrFloat())
    return setupResult(type.getIntOrFloatBitWidth());

  if (auto shapedType = dyn_cast<ShapedType>(type)) {
    if (shapedType.getElementType().isIntOrFloat())
      return setupResult(shapedType.getElementTypeBitWidth());
  }
  return emitSilenceableError()
         << "unsupported type for bitwidth extraction: " << type;
}

//===----------------------------------------------------------------------===//
// MatchStructuredInputOp
//===----------------------------------------------------------------------===//
```
- **EN**: Implements logic around `getType`, `isIntOrFloat`, `setupResult`, `dyn_cast`, and 2 more symbols.
- **CN**: 围绕 `getType`, `isIntOrFloat`, `setupResult`, `dyn_cast`, and 2 more symbols 实现具体逻辑。

### Lines 427-451
```cpp

DiagnosedSilenceableFailure transform::MatchStructuredInputOp::matchOperation(
    Operation *current, transform::TransformResults &results,
    transform::TransformState &state) {
  auto linalgOp = cast<linalg::LinalgOp>(current);
  SmallVector<int64_t> positions;
  DiagnosedSilenceableFailure diag = getPositionsFor(linalgOp, positions);
  if (!diag.succeeded())
    return diag;

  SmallVector<MappedValue> operandMapping;
  operandMapping.reserve(positions.size());
  for (int64_t position : positions) {
    AffineMap indexingMap =
        linalgOp.getMatchingIndexingMap(linalgOp.getDpsInputOperand(position));
    if (getPermutation() && !indexingMap.isPermutation()) {
      return emitSilenceableError() << "the indexing map for input #"
                                    << position << " is not a permutation";
    }
    if (getProjectedPermutation() && !indexingMap.isProjectedPermutation()) {
      return emitSilenceableError()
             << "the indexing map for input #" << position
             << " is not a projected permutation";
    }

```
- **EN**: Implements logic around `matchOperation`, `LinalgOp>`, `getPositionsFor`, `succeeded`, and 5 more symbols; this block performs affine reasoning or shape/bounds manipulation.
- **CN**: 围绕 `matchOperation`, `LinalgOp>`, `getPositionsFor`, `succeeded`, and 5 more symbols 实现具体逻辑；该代码块执行仿射推理或形状/边界处理。

### Lines 452-466
```cpp
    // If capture not requested, skip it.
    if (!getResult())
      continue;

    if (isa<AffineMapParamType>(getResult().getType())) {
      operandMapping.emplace_back(AffineMapAttr::get(indexingMap));
      continue;
    }

    Value operand = linalgOp.getDpsInputOperand(position)->get();
    if (isa<TransformValueHandleTypeInterface>(getResult().getType())) {
      operandMapping.emplace_back(operand);
      continue;
    }

```
- **EN**: Implements logic around `getResult`, `isa`, `emplace_back`, `getDpsInputOperand`.
- **CN**: 围绕 `getResult`, `isa`, `emplace_back`, `getDpsInputOperand` 实现具体逻辑。

### Lines 467-490
```cpp
    Operation *operandProducer = operand.getDefiningOp();
    if (!operandProducer) {
      return emitSilenceableError()
             << "input #" << position << " is not produced by an operation";
    }
    operandMapping.emplace_back(operandProducer);
  }
  if (getResult())
    results.setMappedValues(cast<OpResult>(getResult()), operandMapping);
  return DiagnosedSilenceableFailure::success();
}

DiagnosedSilenceableFailure transform::MatchStructuredInputOp::getPositionsFor(
    linalg::LinalgOp op, SmallVectorImpl<int64_t> &positions) {
  DiagnosedSilenceableFailure diag = expandTargetSpecification(
      getLoc(), getIsAll(), getIsInverted(), getRawPositionList(),
      op.getNumDpsInputs(), positions);
  if (diag.isSilenceableFailure()) {
    diag.attachNote(op->getLoc())
        << "while considering DPS inputs of this payload operation";
  }
  return diag;
}

```
- **EN**: Implements logic around `getDefiningOp`, `emitSilenceableError`, `emplace_back`, `getResult`, and 8 more symbols.
- **CN**: 围绕 `getDefiningOp`, `emitSilenceableError`, `emplace_back`, `getResult`, and 8 more symbols 实现具体逻辑。

### Lines 491-504
```cpp
/// Verifies a matcher op for structured input or output, specifically the
/// attributes specifying the operand positions.
template <typename OpTy>
LogicalResult verifyStructuredOperandOp(OpTy op) {
  if (op.getPermutation() && op.getProjectedPermutation()) {
    return op.emitOpError()
           << op.getPermutationAttrName() << " and "
           << op.getProjectedPermutationAttrName() << " are mutually exclusive";
  }
  if (op.getRawPositionList().size() > 1 && op.getResult()) {
    return op.emitOpError()
           << "cannot bind multiple inputs/inits to the same value";
  }

```
- **EN**: Implements logic around `verifyStructuredOperandOp`, `getPermutation`, `emitOpError`, `getPermutationAttrName`, and 2 more symbols.
- **CN**: 围绕 `verifyStructuredOperandOp`, `getPermutation`, `emitOpError`, `getPermutationAttrName`, and 2 more symbols 实现具体逻辑。

### Lines 505-518
```cpp
  return success();
}

LogicalResult transform::MatchStructuredInputOp::verify() {
  if (failed(verifyStructuredOperandOp(*this)))
    return failure();
  return verifyTransformMatchDimsOp(getOperation(), getRawPositionList(),
                                    getIsInverted(), getIsAll());
}

//===----------------------------------------------------------------------===//
// MatchStructuredInitOp
//===----------------------------------------------------------------------===//

```
- **EN**: Implements logic around `success`, `verify`, `failed`, `failure`, and 2 more symbols; this block implements verifier, folding, parsing, or printing hooks.
- **CN**: 围绕 `success`, `verify`, `failed`, `failure`, and 2 more symbols 实现具体逻辑；该代码块实现验证、折叠、解析或打印钩子。

### Lines 519-541
```cpp
DiagnosedSilenceableFailure transform::MatchStructuredInitOp::matchOperation(
    Operation *current, transform::TransformResults &results,
    transform::TransformState &state) {
  auto linalgOp = cast<linalg::LinalgOp>(current);
  SmallVector<int64_t> positions;
  DiagnosedSilenceableFailure diag = getPositionsFor(linalgOp, positions);
  if (!diag.succeeded())
    return diag;

  SmallVector<MappedValue> operandMapping;
  operandMapping.reserve(positions.size());
  for (int64_t position : positions) {
    AffineMap indexingMap =
        linalgOp.getMatchingIndexingMap(linalgOp.getDpsInitOperand(position));
    if (getPermutation() && !indexingMap.isPermutation()) {
      return emitSilenceableError() << "the indexing map for output(init) #"
                                    << position << " is not a permutation";
    }
    if (getProjectedPermutation() && !indexingMap.isProjectedPermutation()) {
      return emitSilenceableError() << "the indexing map for output(init) #"
                                    << position << " is not a permutation";
    }

```
- **EN**: Implements logic around `matchOperation`, `LinalgOp>`, `getPositionsFor`, `succeeded`, and 5 more symbols; this block performs affine reasoning or shape/bounds manipulation.
- **CN**: 围绕 `matchOperation`, `LinalgOp>`, `getPositionsFor`, `succeeded`, and 5 more symbols 实现具体逻辑；该代码块执行仿射推理或形状/边界处理。

### Lines 542-556
```cpp
    // If capture not requested, skip it.
    if (!getResult())
      continue;

    if (isa<AffineMapParamType>(getResult().getType())) {
      operandMapping.emplace_back(AffineMapAttr::get(indexingMap));
      continue;
    }

    Value operand = linalgOp.getDpsInitOperand(position)->get();
    if (isa<TransformValueHandleTypeInterface>(getResult().getType())) {
      operandMapping.emplace_back(operand);
      continue;
    }

```
- **EN**: Implements logic around `getResult`, `isa`, `emplace_back`, `getDpsInitOperand`.
- **CN**: 围绕 `getResult`, `isa`, `emplace_back`, `getDpsInitOperand` 实现具体逻辑。

### Lines 557-580
```cpp
    Operation *operandProducer = operand.getDefiningOp();
    if (!operandProducer) {
      return emitSilenceableError() << "output(init) #" << position
                                    << " is not produced by an operation";
    }
    operandMapping.emplace_back(operandProducer);
  }
  if (getResult())
    results.setMappedValues(cast<OpResult>(getResult()), operandMapping);
  return DiagnosedSilenceableFailure::success();
}

DiagnosedSilenceableFailure transform::MatchStructuredInitOp::getPositionsFor(
    linalg::LinalgOp op, SmallVectorImpl<int64_t> &positions) {
  DiagnosedSilenceableFailure diag = expandTargetSpecification(
      getLoc(), getIsAll(), getIsInverted(), getRawPositionList(),
      op.getNumDpsInits(), positions);
  if (diag.isSilenceableFailure()) {
    diag.attachNote(op->getLoc())
        << "while considering DPS inits (outputs) of this payload operation";
  }
  return diag;
}

```
- **EN**: Implements logic around `getDefiningOp`, `emitSilenceableError`, `emplace_back`, `getResult`, and 9 more symbols.
- **CN**: 围绕 `getDefiningOp`, `emitSilenceableError`, `emplace_back`, `getResult`, and 9 more symbols 实现具体逻辑。

### Lines 581-602
```cpp
LogicalResult transform::MatchStructuredInitOp::verify() {
  if (failed(verifyStructuredOperandOp(*this)))
    return failure();
  return verifyTransformMatchDimsOp(getOperation(), getRawPositionList(),
                                    getIsInverted(), getIsAll());
}

//===----------------------------------------------------------------------===//
// MatchStructuredNumInputsOp
//===----------------------------------------------------------------------===//

DiagnosedSilenceableFailure
transform::MatchStructuredNumInputsOp::matchOperation(
    Operation *current, transform::TransformResults &results,
    transform::TransformState &state) {
  auto linalgOp = cast<linalg::LinalgOp>(current);
  Attribute attr =
      Builder(current).getI64IntegerAttr(linalgOp.getNumDpsInputs());
  results.setParams(cast<OpResult>(getResult()), {attr});
  return DiagnosedSilenceableFailure::success();
}

```
- **EN**: Implements logic around `verify`, `failed`, `failure`, `verifyTransformMatchDimsOp`, and 6 more symbols; this block implements verifier, folding, parsing, or printing hooks.
- **CN**: 围绕 `verify`, `failed`, `failure`, `verifyTransformMatchDimsOp`, and 6 more symbols 实现具体逻辑；该代码块实现验证、折叠、解析或打印钩子。

### Lines 603-617
```cpp
//===----------------------------------------------------------------------===//
// MatchStructuredNumInitsOp
//===----------------------------------------------------------------------===//

DiagnosedSilenceableFailure
transform::MatchStructuredNumInitsOp::matchOperation(
    Operation *current, transform::TransformResults &results,
    transform::TransformState &state) {
  auto linalgOp = cast<linalg::LinalgOp>(current);
  Attribute attr =
      Builder(current).getI64IntegerAttr(linalgOp.getNumDpsInits());
  results.setParams(cast<OpResult>(getResult()), {attr});
  return DiagnosedSilenceableFailure::success();
}

```
- **EN**: Implements logic around `matchOperation`, `LinalgOp>`, `Builder`, `setParams`, and 1 more symbols.
- **CN**: 围绕 `matchOperation`, `LinalgOp>`, `Builder`, `setParams`, and 1 more symbols 实现具体逻辑。

### Lines 618-631
```cpp
//===----------------------------------------------------------------------===//
// MatchStructuredRankOp
//===----------------------------------------------------------------------===//

DiagnosedSilenceableFailure transform::MatchStructuredRankOp::matchOperation(
    Operation *current, transform::TransformResults &results,
    transform::TransformState &state) {
  auto linalgOp = cast<linalg::LinalgOp>(current);
  int64_t numLoops = linalgOp.getNumLoops();
  Attribute attr = Builder(linalgOp->getContext()).getI64IntegerAttr(numLoops);
  results.setParams(cast<OpResult>(getRank()), {attr});
  return DiagnosedSilenceableFailure::success();
}

```
- **EN**: Implements logic around `matchOperation`, `LinalgOp>`, `getNumLoops`, `Builder`, and 2 more symbols.
- **CN**: 围绕 `matchOperation`, `LinalgOp>`, `getNumLoops`, `Builder`, and 2 more symbols 实现具体逻辑。

### Lines 632-650
```cpp
//===----------------------------------------------------------------------===//
// MatchStructuredResultOp
//===----------------------------------------------------------------------===//

DiagnosedSilenceableFailure transform::MatchStructuredResultOp::matchOperation(
    Operation *op, transform::TransformResults &results,
    transform::TransformState &state) {
  auto linalgOp = cast<linalg::LinalgOp>(op);
  int64_t position;
  DiagnosedSilenceableFailure diag = getPositionFor(linalgOp, position);
  if (!diag.succeeded())
    return diag;

  Value result = linalgOp.getTiedOpResult(linalgOp.getDpsInitOperand(position));
  if (isa<TransformValueHandleTypeInterface>(getResult().getType())) {
    results.setValues(cast<OpResult>(getResult()), {result});
    return DiagnosedSilenceableFailure::success();
  }

```
- **EN**: Implements logic around `matchOperation`, `LinalgOp>`, `getPositionFor`, `succeeded`, and 4 more symbols.
- **CN**: 围绕 `matchOperation`, `LinalgOp>`, `getPositionFor`, `succeeded`, and 4 more symbols 实现具体逻辑。

### Lines 651-668
```cpp
  if (result.getUsers().empty()) {
    return emitSilenceableError()
           << "no users of the result #" << getPosition();
  }
  Operation *firstUser = *result.getUsers().begin();
  if (getAny()) {
    results.set(cast<OpResult>(getResult()), {firstUser});
    return DiagnosedSilenceableFailure::success();
  }
  if (getSingle()) {
    if (!llvm::hasSingleElement(result.getUsers())) {
      return emitSilenceableError()
             << "more than one result user with single user requested";
    }
    results.set(cast<OpResult>(getResult()), {firstUser});
    return DiagnosedSilenceableFailure::success();
  }

```
- **EN**: Implements logic around `getUsers`, `emitSilenceableError`, `getPosition`, `getAny`, and 4 more symbols.
- **CN**: 围绕 `getUsers`, `emitSilenceableError`, `getPosition`, `getAny`, and 4 more symbols 实现具体逻辑。

### Lines 669-684
```cpp
  return emitDefiniteFailure() << "unknown sub-predicate";
}

DiagnosedSilenceableFailure
transform::MatchStructuredResultOp::getPositionFor(linalg::LinalgOp op,
                                                   int64_t &position) {
  auto rawPosition = static_cast<int64_t>(getPosition());
  position = rawPosition < 0 ? op.getNumDpsInits() + rawPosition : rawPosition;
  if (position >= op.getNumDpsInits() || position < 0) {
    return emitSilenceableError()
           << "position " << rawPosition
           << " overflows the number of results(ints) of the payload operation";
  }
  return DiagnosedSilenceableFailure::success();
}

```
- **EN**: Implements logic around `emitDefiniteFailure`, `getPositionFor`, `static_cast`, `getNumDpsInits`, and 3 more symbols.
- **CN**: 围绕 `emitDefiniteFailure`, `getPositionFor`, `static_cast`, `getNumDpsInits`, and 3 more symbols 实现具体逻辑。

### Lines 685-699
```cpp
LogicalResult transform::MatchStructuredResultOp::verify() {
  if ((getAny() || getSingle()) ^
      isa<TransformHandleTypeInterface>(getResult().getType())) {
    return emitOpError() << "expects either the any/single keyword or the type "
                            "value handle result type";
  }
  if (getAny() && getSingle()) {
    return emitOpError() << "'any' and 'single' are mutually exclusive";
  }
  return success();
}

//===----------------------------------------------------------------------===//
// MatchStructuredYieldOp
//===----------------------------------------------------------------------===//
```
- **EN**: Implements logic around `verify`, `getAny`, `isa`, `emitOpError`, and 1 more symbols; this block implements verifier, folding, parsing, or printing hooks.
- **CN**: 围绕 `verify`, `getAny`, `isa`, `emitOpError`, and 1 more symbols 实现具体逻辑；该代码块实现验证、折叠、解析或打印钩子。

### Lines 700-713
```cpp

void transform::MatchStructuredYieldOp::getEffects(
    SmallVectorImpl<MemoryEffects::EffectInstance> &effects) {
  onlyReadsHandle(getHandlesMutable(), effects);
  onlyReadsPayload(effects);
}

void transform::MatchStructuredYieldOp::build(OpBuilder &builder,
                                              OperationState &state) {
  build(builder, state, ValueRange());
}

#define GET_OP_CLASSES
#include "mlir/Dialect/Linalg/TransformOps/LinalgMatchOps.cpp.inc"
```
- **EN**: Implements logic around `getEffects`, `onlyReadsHandle`, `onlyReadsPayload`, `build`.
- **CN**: 围绕 `getEffects`, `onlyReadsHandle`, `onlyReadsPayload`, `build` 实现具体逻辑。

## Key Concepts / 关键概念

- **Transform dialect integration / Transform Dialect 集成**:
  - **EN**: Adds transform-dialect operations or extensions that steer other rewrites.
  - **CN**: 添加驱动其他重写的 Transform Dialect 操作或扩展。
- **Structured tensor ops / 结构化张量操作**:
  - **EN**: Represents loop-nest-like structured computations over tensors or buffers.
  - **CN**: 表示在张量或缓冲区上执行的类循环嵌套结构化计算。
- **Interface dispatch / 接口分派**:
  - **EN**: Attaches shared behavior to different ops or types through MLIR interfaces.
  - **CN**: 通过 MLIR 接口把共享行为附着到不同操作或类型上。
- **Type/attribute storage / 类型/属性存储**:
  - **EN**: Uses uniqued storage and parser/printer hooks for extensible compile-time metadata.
  - **CN**: 使用唯一化存储以及解析/打印钩子来表示可扩展的编译期元数据。
- **Region-based control flow / 基于 Region 的控制流**:
  - **EN**: Represents nested blocks and successors as first-class IR structure.
  - **CN**: 把嵌套 block 与后继边表示为一等 IR 结构。
- **LLVM interop / LLVM 互操作**:
  - **EN**: Bridges MLIR concepts to LLVM-compatible data structures, intrinsics, or codegen expectations.
  - **CN**: 把 MLIR 概念桥接到 LLVM 兼容的数据结构、intrinsic 或代码生成预期。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/Linalg/TransformOps/LinalgMatchOps.h`, `mlir/Analysis/SliceAnalysis.h`, `mlir/Dialect/Linalg/IR/Linalg.h`, `mlir/Dialect/Linalg/IR/LinalgInterfaces.h`, `mlir/Dialect/Linalg/TransformOps/Syntax.h`, `mlir/Dialect/Linalg/Utils/Utils.h`, `mlir/Dialect/Transform/IR/TransformTypes.h`, `mlir/Dialect/Transform/Interfaces/MatchInterfaces.h`, `mlir/IR/BuiltinAttributes.h`, `llvm/ADT/SmallVectorExtras.h` ... (+4 more)
- **Subsystem categories / 子系统类别**: other MLIR dialect declarations / 其他 MLIR 方言声明 (8), LLVM support-library helpers / LLVM Support 库辅助功能 (3), MLIR analysis interfaces / MLIR 分析接口 (1), MLIR IR core abstractions / MLIR IR 核心抽象 (1), LLVM ADT containers and utility types / LLVM ADT 容器与工具类型 (1)
- **Generated macros / 生成宏**: `GET_OP_CLASSES`
