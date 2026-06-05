# IRDLLoading.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/IRDL/IRDLLoading.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This file is licensed under the Apache License v2.0 with LLVM Exceptions.
  - **CN**: 实现 IRDL 方言与声明式方言定义支持 中与 `IRDLLoading` 相关的支持逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-23
```cpp
//===- IRDLLoading.cpp - IRDL dialect loading --------------------- C++ -*-===//
//
// This file is licensed under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// Manages the loading of MLIR objects from IRDL operations.
//
//===----------------------------------------------------------------------===//

#include "mlir/Dialect/IRDL/IRDLLoading.h"
#include "mlir/Dialect/IRDL/IR/IRDL.h"
#include "mlir/Dialect/IRDL/IR/IRDLInterfaces.h"
#include "mlir/Dialect/IRDL/IRDLSymbols.h"
#include "mlir/Dialect/IRDL/IRDLVerifiers.h"
#include "mlir/IR/Attributes.h"
#include "mlir/IR/BuiltinOps.h"
#include "mlir/IR/ExtensibleDialect.h"
#include "mlir/IR/OperationSupport.h"
#include "llvm/ADT/STLExtras.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/IRDL/IRDLLoading.h`, `mlir/Dialect/IRDL/IR/IRDL.h`, `mlir/Dialect/IRDL/IR/IRDLInterfaces.h`, `mlir/Dialect/IRDL/IRDLSymbols.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/IRDL/IRDLLoading.h`, `mlir/Dialect/IRDL/IR/IRDL.h`, `mlir/Dialect/IRDL/IR/IRDLInterfaces.h`, `mlir/Dialect/IRDL/IRDLSymbols.h`。

### Lines 24-40
```cpp
using namespace mlir;
using namespace mlir::irdl;

/// Verify that the given list of parameters satisfy the given constraints.
/// This encodes the logic of the verification method for attributes and types
/// defined with IRDL.
static LogicalResult
irdlAttrOrTypeVerifier(function_ref<InFlightDiagnostic()> emitError,
                       ArrayRef<Attribute> params,
                       ArrayRef<std::unique_ptr<Constraint>> constraints,
                       ArrayRef<size_t> paramConstraints) {
  if (params.size() != paramConstraints.size()) {
    emitError() << "expected " << paramConstraints.size()
                << " type arguments, but had " << params.size();
    return failure();
  }

```
- **EN**: Implements logic around `irdlAttrOrTypeVerifier`, `size`, `emitError`, `failure`; this block implements verifier, folding, parsing, or printing hooks.
- **CN**: 围绕 `irdlAttrOrTypeVerifier`, `size`, `emitError`, `failure` 实现具体逻辑；该代码块实现验证、折叠、解析或打印钩子。

### Lines 41-62
```cpp
  ConstraintVerifier verifier(constraints);

  // Check that each parameter satisfies its constraint.
  for (auto [i, param] : enumerate(params))
    if (failed(verifier.verify(emitError, param, paramConstraints[i])))
      return failure();

  return success();
}

/// Get the operand segment sizes from the attribute dictionary.
LogicalResult getSegmentSizesFromAttr(Operation *op, StringRef elemName,
                                      StringRef attrName, unsigned numElements,
                                      ArrayRef<Variadicity> variadicities,
                                      SmallVectorImpl<int> &segmentSizes) {
  // Get the segment sizes attribute, and check that it is of the right type.
  Attribute segmentSizesAttr = op->getAttr(attrName);
  if (!segmentSizesAttr) {
    return op->emitError() << "'" << attrName
                           << "' attribute is expected but not provided";
  }

```
- **EN**: Implements logic around `verifier`, `enumerate`, `failed`, `failure`, and 4 more symbols; this block implements verifier, folding, parsing, or printing hooks.
- **CN**: 围绕 `verifier`, `enumerate`, `failed`, `failure`, and 4 more symbols 实现具体逻辑；该代码块实现验证、折叠、解析或打印钩子。

### Lines 63-86
```cpp
  auto denseSegmentSizes = dyn_cast<DenseI32ArrayAttr>(segmentSizesAttr);
  if (!denseSegmentSizes) {
    return op->emitError() << "'" << attrName
                           << "' attribute is expected to be a dense i32 array";
  }

  if (denseSegmentSizes.size() != (int64_t)variadicities.size()) {
    return op->emitError() << "'" << attrName << "' attribute for specifying "
                           << elemName << " segments must have "
                           << variadicities.size() << " elements, but got "
                           << denseSegmentSizes.size();
  }

  // Check that the segment sizes are corresponding to the given variadicities,
  for (auto [i, segmentSize, variadicity] :
       enumerate(denseSegmentSizes.asArrayRef(), variadicities)) {
    if (segmentSize < 0)
      return op->emitError()
             << "'" << attrName << "' attribute for specifying " << elemName
             << " segments must have non-negative values";
    if (variadicity == Variadicity::single && segmentSize != 1)
      return op->emitError() << "element " << i << " in '" << attrName
                             << "' attribute must be equal to 1";

```
- **EN**: Implements logic around `dyn_cast`, `emitError`, `size`, `enumerate`.
- **CN**: 围绕 `dyn_cast`, `emitError`, `size`, `enumerate` 实现具体逻辑。

### Lines 87-102
```cpp
    if (variadicity == Variadicity::optional && segmentSize > 1)
      return op->emitError() << "element " << i << " in '" << attrName
                             << "' attribute must be equal to 0 or 1";

    segmentSizes.push_back(segmentSize);
  }

  // Check that the sum of the segment sizes is equal to the number of elements.
  int32_t sum = 0;
  for (int32_t segmentSize : denseSegmentSizes.asArrayRef())
    sum += segmentSize;
  if (sum != static_cast<int32_t>(numElements))
    return op->emitError() << "sum of elements in '" << attrName
                           << "' attribute must be equal to the number of "
                           << elemName << "s";

```
- **EN**: Implements logic around `emitError`, `push_back`, `asArrayRef`, `static_cast`.
- **CN**: 围绕 `emitError`, `push_back`, `asArrayRef`, `static_cast` 实现具体逻辑。

### Lines 103-122
```cpp
  return success();
}

/// Compute the segment sizes of the given element (operands, results).
/// If the operation has more than two non-single elements (optional or
/// variadic), then get the segment sizes from the attribute dictionary.
/// Otherwise, compute the segment sizes from the number of elements.
/// `elemName` should be either `"operand"` or `"result"`.
LogicalResult getSegmentSizes(Operation *op, StringRef elemName,
                              StringRef attrName, unsigned numElements,
                              ArrayRef<Variadicity> variadicities,
                              SmallVectorImpl<int> &segmentSizes) {
  // If we have more than one non-single variadicity, we need to get the
  // segment sizes from the attribute dictionary.
  int numberNonSingle = count_if(
      variadicities, [](Variadicity v) { return v != Variadicity::single; });
  if (numberNonSingle > 1)
    return getSegmentSizesFromAttr(op, elemName, attrName, numElements,
                                   variadicities, segmentSizes);

```
- **EN**: Implements logic around `success`, `getSegmentSizes`, `count_if`, `getSegmentSizesFromAttr`.
- **CN**: 围绕 `success`, `getSegmentSizes`, `count_if`, `getSegmentSizesFromAttr` 实现具体逻辑。

### Lines 123-140
```cpp
  // If we only have single variadicities, the segments sizes are all 1.
  if (numberNonSingle == 0) {
    if (numElements != variadicities.size()) {
      return op->emitError() << "op expects exactly " << variadicities.size()
                             << " " << elemName << "s, but got " << numElements;
    }
    for (size_t i = 0, e = variadicities.size(); i < e; ++i)
      segmentSizes.push_back(1);
    return success();
  }

  assert(numberNonSingle == 1);

  // There is exactly one non-single element, so we can
  // compute its size and check that it is valid.
  int nonSingleSegmentSize = static_cast<int>(numElements) -
                             static_cast<int>(variadicities.size()) + 1;

```
- **EN**: Implements logic around `size`, `emitError`, `push_back`, `success`, and 2 more symbols.
- **CN**: 围绕 `size`, `emitError`, `push_back`, `success`, and 2 more symbols 实现具体逻辑。

### Lines 141-158
```cpp
  if (nonSingleSegmentSize < 0) {
    return op->emitError() << "op expects at least " << variadicities.size() - 1
                           << " " << elemName << "s, but got " << numElements;
  }

  // Add the segment sizes.
  for (Variadicity variadicity : variadicities) {
    if (variadicity == Variadicity::single) {
      segmentSizes.push_back(1);
      continue;
    }

    // If we have an optional element, we should check that it represents
    // zero or one elements.
    if (nonSingleSegmentSize > 1 && variadicity == Variadicity::optional)
      return op->emitError() << "op expects at most " << variadicities.size()
                             << " " << elemName << "s, but got " << numElements;

```
- **EN**: Implements logic around `emitError`, `push_back`.
- **CN**: 围绕 `emitError`, `push_back` 实现具体逻辑。

### Lines 159-175
```cpp
    segmentSizes.push_back(nonSingleSegmentSize);
  }

  return success();
}

/// Compute the segment sizes of the given operands.
/// If the operation has more than two non-single operands (optional or
/// variadic), then get the segment sizes from the attribute dictionary.
/// Otherwise, compute the segment sizes from the number of operands.
LogicalResult getOperandSegmentSizes(Operation *op,
                                     ArrayRef<Variadicity> variadicities,
                                     SmallVectorImpl<int> &segmentSizes) {
  return getSegmentSizes(op, "operand", "operandSegmentSizes",
                         op->getNumOperands(), variadicities, segmentSizes);
}

```
- **EN**: Implements logic around `push_back`, `success`, `getOperandSegmentSizes`, `getSegmentSizes`, and 1 more symbols.
- **CN**: 围绕 `push_back`, `success`, `getOperandSegmentSizes`, `getSegmentSizes`, and 1 more symbols 实现具体逻辑。

### Lines 176-189
```cpp
/// Compute the segment sizes of the given results.
/// If the operation has more than two non-single results (optional or
/// variadic), then get the segment sizes from the attribute dictionary.
/// Otherwise, compute the segment sizes from the number of results.
LogicalResult getResultSegmentSizes(Operation *op,
                                    ArrayRef<Variadicity> variadicities,
                                    SmallVectorImpl<int> &segmentSizes) {
  return getSegmentSizes(op, "result", "resultSegmentSizes",
                         op->getNumResults(), variadicities, segmentSizes);
}

/// Verify that the given operation satisfies the given constraints.
/// This encodes the logic of the verification method for operations defined
/// with IRDL.
```
- **EN**: Implements logic around `getResultSegmentSizes`, `getSegmentSizes`, `getNumResults`; this block implements verifier, folding, parsing, or printing hooks.
- **CN**: 围绕 `getResultSegmentSizes`, `getSegmentSizes`, `getNumResults` 实现具体逻辑；该代码块实现验证、折叠、解析或打印钩子。

### Lines 190-207
```cpp
static LogicalResult irdlOpVerifier(
    Operation *op, ConstraintVerifier &verifier,
    ArrayRef<size_t> operandConstrs, ArrayRef<Variadicity> operandVariadicity,
    ArrayRef<size_t> resultConstrs, ArrayRef<Variadicity> resultVariadicity,
    const DenseMap<StringAttr, size_t> &attributeConstrs) {
  // Get the segment sizes for the operands.
  // This will check that the number of operands is correct.
  SmallVector<int> operandSegmentSizes;
  if (failed(
          getOperandSegmentSizes(op, operandVariadicity, operandSegmentSizes)))
    return failure();

  // Get the segment sizes for the results.
  // This will check that the number of results is correct.
  SmallVector<int> resultSegmentSizes;
  if (failed(getResultSegmentSizes(op, resultVariadicity, resultSegmentSizes)))
    return failure();

```
- **EN**: Implements logic around `irdlOpVerifier`, `failed`, `getOperandSegmentSizes`, `failure`.
- **CN**: 围绕 `irdlOpVerifier`, `failed`, `getOperandSegmentSizes`, `failure` 实现具体逻辑。

### Lines 208-221
```cpp
  auto emitError = [op] { return op->emitError(); };

  /// Сheck that we have all needed attributes passed
  /// and they satisfy the constraints.
  DictionaryAttr actualAttrs = op->getAttrDictionary();

  for (auto [name, constraint] : attributeConstrs) {
    /// First, check if the attribute actually passed.
    std::optional<NamedAttribute> actual = actualAttrs.getNamed(name);
    if (!actual.has_value())
      return op->emitOpError()
             << "attribute " << name << " is expected but not provided";

    /// Then, check if the attribute value satisfies the constraint.
```
- **EN**: Implements logic around `emitError`, `getAttrDictionary`, `getNamed`, `has_value`, and 1 more symbols.
- **CN**: 围绕 `emitError`, `getAttrDictionary`, `getNamed`, `has_value`, and 1 more symbols 实现具体逻辑。

### Lines 222-237
```cpp
    if (failed(verifier.verify({emitError}, actual->getValue(), constraint)))
      return failure();
  }

  // Check that all operands satisfy the constraints
  int operandIdx = 0;
  for (auto [defIndex, segmentSize] : enumerate(operandSegmentSizes)) {
    for (int i = 0; i < segmentSize; i++) {
      if (failed(verifier.verify(
              {emitError}, TypeAttr::get(op->getOperandTypes()[operandIdx]),
              operandConstrs[defIndex])))
        return failure();
      ++operandIdx;
    }
  }

```
- **EN**: Implements logic around `failed`, `failure`, `enumerate`, `get`; this block implements verifier, folding, parsing, or printing hooks.
- **CN**: 围绕 `failed`, `failure`, `enumerate`, `get` 实现具体逻辑；该代码块实现验证、折叠、解析或打印钩子。

### Lines 238-252
```cpp
  // Check that all results satisfy the constraints
  int resultIdx = 0;
  for (auto [defIndex, segmentSize] : enumerate(resultSegmentSizes)) {
    for (int i = 0; i < segmentSize; i++) {
      if (failed(verifier.verify({emitError},
                                 TypeAttr::get(op->getResultTypes()[resultIdx]),
                                 resultConstrs[defIndex])))
        return failure();
      ++resultIdx;
    }
  }

  return success();
}

```
- **EN**: Implements logic around `enumerate`, `failed`, `get`, `failure`, and 1 more symbols; this block implements verifier, folding, parsing, or printing hooks.
- **CN**: 围绕 `enumerate`, `failed`, `get`, `failure`, and 1 more symbols 实现具体逻辑；该代码块实现验证、折叠、解析或打印钩子。

### Lines 253-266
```cpp
static LogicalResult irdlRegionVerifier(
    Operation *op, ConstraintVerifier &verifier,
    ArrayRef<std::unique_ptr<RegionConstraint>> regionsConstraints) {
  if (op->getNumRegions() != regionsConstraints.size()) {
    return op->emitOpError()
           << "unexpected number of regions: expected "
           << regionsConstraints.size() << " but got " << op->getNumRegions();
  }

  for (auto [constraint, region] :
       llvm::zip(regionsConstraints, op->getRegions()))
    if (failed(constraint->verify(region, verifier)))
      return failure();

```
- **EN**: Implements logic around `irdlRegionVerifier`, `getNumRegions`, `emitOpError`, `size`, and 3 more symbols; this block implements verifier, folding, parsing, or printing hooks; manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `irdlRegionVerifier`, `getNumRegions`, `emitOpError`, `size`, and 3 more symbols 实现具体逻辑；该代码块实现验证、折叠、解析或打印钩子，并处理 MLIR region、block 或控制流边。

### Lines 267-294
```cpp
  return success();
}

llvm::unique_function<LogicalResult(Operation *) const>
mlir::irdl::createVerifier(
    OperationOp op,
    const DenseMap<irdl::TypeOp, std::unique_ptr<DynamicTypeDefinition>> &types,
    const DenseMap<irdl::AttributeOp, std::unique_ptr<DynamicAttrDefinition>>
        &attrs) {
  // Resolve SSA values to verifier constraint slots
  SmallVector<Value> constrToValue;
  SmallVector<Value> regionToValue;
  for (Operation &op : op->getRegion(0).getOps()) {
    if (isa<VerifyConstraintInterface>(op)) {
      if (op.getNumResults() != 1) {
        op.emitError()
            << "IRDL constraint operations must have exactly one result";
        return nullptr;
      }
      constrToValue.push_back(op.getResult(0));
    }
    if (isa<VerifyRegionInterface>(op)) {
      if (op.getNumResults() != 1) {
        op.emitError()
            << "IRDL constraint operations must have exactly one result";
        return nullptr;
      }
      regionToValue.push_back(op.getResult(0));
```
- **EN**: Implements logic around `success`, `unique_function`, `createVerifier`, `getRegion`, and 4 more symbols.
- **CN**: 围绕 `success`, `unique_function`, `createVerifier`, `getRegion`, and 4 more symbols 实现具体逻辑。

### Lines 295-309
```cpp
    }
  }

  // Build the verifiers for each constraint slot
  SmallVector<std::unique_ptr<Constraint>> constraints;
  for (Value v : constrToValue) {
    VerifyConstraintInterface op =
        cast<VerifyConstraintInterface>(v.getDefiningOp());
    std::unique_ptr<Constraint> verifier =
        op.getVerifier(constrToValue, types, attrs);
    if (!verifier)
      return nullptr;
    constraints.push_back(std::move(verifier));
  }

```
- **EN**: Implements logic around `cast`, `getVerifier`, `push_back`.
- **CN**: 围绕 `cast`, `getVerifier`, `push_back` 实现具体逻辑。

### Lines 310-334
```cpp
  // Build region constraints
  SmallVector<std::unique_ptr<RegionConstraint>> regionConstraints;
  for (Value v : regionToValue) {
    VerifyRegionInterface op = cast<VerifyRegionInterface>(v.getDefiningOp());
    std::unique_ptr<RegionConstraint> verifier =
        op.getVerifier(constrToValue, types, attrs);
    regionConstraints.push_back(std::move(verifier));
  }

  SmallVector<size_t> operandConstraints;
  SmallVector<Variadicity> operandVariadicity;

  // Gather which constraint slots correspond to operand constraints
  auto operandsOp = op.getOp<OperandsOp>();
  if (operandsOp.has_value()) {
    operandConstraints.reserve(operandsOp->getArgs().size());
    for (Value operand : operandsOp->getArgs()) {
      for (auto [i, constr] : enumerate(constrToValue)) {
        if (constr == operand) {
          operandConstraints.push_back(i);
          break;
        }
      }
    }

```
- **EN**: Implements logic around `cast`, `getVerifier`, `push_back`, `getOp`, and 4 more symbols; this block manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `cast`, `getVerifier`, `push_back`, `getOp`, and 4 more symbols 实现具体逻辑；该代码块处理 MLIR region、block 或控制流边。

### Lines 335-355
```cpp
    // Gather the variadicities of each operand
    for (VariadicityAttr attr : operandsOp->getVariadicity())
      operandVariadicity.push_back(attr.getValue());
  }

  SmallVector<size_t> resultConstraints;
  SmallVector<Variadicity> resultVariadicity;

  // Gather which constraint slots correspond to result constraints
  auto resultsOp = op.getOp<ResultsOp>();
  if (resultsOp.has_value()) {
    resultConstraints.reserve(resultsOp->getArgs().size());
    for (Value result : resultsOp->getArgs()) {
      for (auto [i, constr] : enumerate(constrToValue)) {
        if (constr == result) {
          resultConstraints.push_back(i);
          break;
        }
      }
    }

```
- **EN**: Implements logic around `getVariadicity`, `push_back`, `getOp`, `has_value`, and 3 more symbols.
- **CN**: 围绕 `getVariadicity`, `push_back`, `getOp`, `has_value`, and 3 more symbols 实现具体逻辑。

### Lines 356-377
```cpp
    // Gather the variadicities of each result
    for (Attribute attr : resultsOp->getVariadicity())
      resultVariadicity.push_back(cast<VariadicityAttr>(attr).getValue());
  }

  // Gather which constraint slots correspond to attributes constraints
  DenseMap<StringAttr, size_t> attributeConstraints;
  auto attributesOp = op.getOp<AttributesOp>();
  if (attributesOp.has_value()) {
    const Operation::operand_range values = attributesOp->getAttributeValues();
    const ArrayAttr names = attributesOp->getAttributeValueNames();

    for (const auto &[name, value] : llvm::zip(names, values)) {
      for (auto [i, constr] : enumerate(constrToValue)) {
        if (constr == value) {
          attributeConstraints[cast<StringAttr>(name)] = i;
          break;
        }
      }
    }
  }

```
- **EN**: Implements logic around `getVariadicity`, `push_back`, `getOp`, `has_value`, and 5 more symbols.
- **CN**: 围绕 `getVariadicity`, `push_back`, `getOp`, `has_value`, and 5 more symbols 实现具体逻辑。

### Lines 378-396
```cpp
  return
      [constraints{std::move(constraints)},
       regionConstraints{std::move(regionConstraints)},
       operandConstraints{std::move(operandConstraints)},
       operandVariadicity{std::move(operandVariadicity)},
       resultConstraints{std::move(resultConstraints)},
       resultVariadicity{std::move(resultVariadicity)},
       attributeConstraints{std::move(attributeConstraints)}](Operation *op) {
        ConstraintVerifier verifier(constraints);
        const LogicalResult opVerifierResult = irdlOpVerifier(
            op, verifier, operandConstraints, operandVariadicity,
            resultConstraints, resultVariadicity, attributeConstraints);
        const LogicalResult opRegionVerifierResult =
            irdlRegionVerifier(op, verifier, regionConstraints);
        return LogicalResult::success(opVerifierResult.succeeded() &&
                                      opRegionVerifierResult.succeeded());
      };
}

```
- **EN**: Implements logic around `move`, `verifier`, `irdlOpVerifier`, `irdlRegionVerifier`, and 2 more symbols.
- **CN**: 围绕 `move`, `verifier`, `irdlOpVerifier`, `irdlRegionVerifier`, and 2 more symbols 实现具体逻辑。

### Lines 397-412
```cpp
/// Define and load an operation represented by a `irdl.operation`
/// operation.
static WalkResult loadOperation(
    OperationOp op, ExtensibleDialect *dialect,
    const DenseMap<TypeOp, std::unique_ptr<DynamicTypeDefinition>> &types,
    const DenseMap<AttributeOp, std::unique_ptr<DynamicAttrDefinition>>
        &attrs) {

  // IRDL does not support defining custom parsers or printers.
  auto parser = [](OpAsmParser &parser, OperationState &result) {
    return failure();
  };
  auto printer = [](Operation *op, OpAsmPrinter &printer, StringRef) {
    printer.printGenericOp(op);
  };

```
- **EN**: Implements logic around `loadOperation`, `failure`, `printGenericOp`; this block implements verifier, folding, parsing, or printing hooks.
- **CN**: 围绕 `loadOperation`, `failure`, `printGenericOp` 实现具体逻辑；该代码块实现验证、折叠、解析或打印钩子。

### Lines 413-428
```cpp
  auto verifier = createVerifier(op, types, attrs);
  if (!verifier)
    return WalkResult::interrupt();

  // IRDL supports only checking number of blocks and argument constraints
  // It is done in the main verifier to reuse `ConstraintVerifier` context
  auto regionVerifier = [](Operation *op) { return LogicalResult::success(); };

  auto opDef = DynamicOpDefinition::get(
      op.getName(), dialect, std::move(verifier), std::move(regionVerifier),
      std::move(parser), std::move(printer));
  dialect->registerDynamicOp(std::move(opDef));

  return WalkResult::advance();
}

```
- **EN**: Implements logic around `createVerifier`, `interrupt`, `success`, `get`, and 4 more symbols; this block implements verifier, folding, parsing, or printing hooks.
- **CN**: 围绕 `createVerifier`, `interrupt`, `success`, `get`, and 4 more symbols 实现具体逻辑；该代码块实现验证、折叠、解析或打印钩子。

### Lines 429-447
```cpp
/// Get the verifier of a type or attribute definition.
/// Return nullptr if the definition is invalid.
static DynamicAttrDefinition::VerifierFn getAttrOrTypeVerifier(
    Operation *attrOrTypeDef, ExtensibleDialect *dialect,
    DenseMap<TypeOp, std::unique_ptr<DynamicTypeDefinition>> &types,
    DenseMap<AttributeOp, std::unique_ptr<DynamicAttrDefinition>> &attrs) {
  assert((isa<AttributeOp>(attrOrTypeDef) || isa<TypeOp>(attrOrTypeDef)) &&
         "Expected an attribute or type definition");

  // Resolve SSA values to verifier constraint slots
  SmallVector<Value> constrToValue;
  for (Operation &op : attrOrTypeDef->getRegion(0).getOps()) {
    if (isa<VerifyConstraintInterface>(op)) {
      assert(op.getNumResults() == 1 &&
             "IRDL constraint operations must have exactly one result");
      constrToValue.push_back(op.getResult(0));
    }
  }

```
- **EN**: Implements logic around `getAttrOrTypeVerifier`, `assert`, `getRegion`, `isa`, and 1 more symbols.
- **CN**: 围绕 `getAttrOrTypeVerifier`, `assert`, `getRegion`, `isa`, and 1 more symbols 实现具体逻辑。

### Lines 448-466
```cpp
  // Build the verifiers for each constraint slot
  SmallVector<std::unique_ptr<Constraint>> constraints;
  for (Value v : constrToValue) {
    VerifyConstraintInterface op =
        cast<VerifyConstraintInterface>(v.getDefiningOp());
    std::unique_ptr<Constraint> verifier =
        op.getVerifier(constrToValue, types, attrs);
    if (!verifier)
      return {};
    constraints.push_back(std::move(verifier));
  }

  // Get the parameter definitions.
  std::optional<ParametersOp> params;
  if (auto attr = dyn_cast<AttributeOp>(attrOrTypeDef))
    params = attr.getOp<ParametersOp>();
  else if (auto type = dyn_cast<TypeOp>(attrOrTypeDef))
    params = type.getOp<ParametersOp>();

```
- **EN**: Implements logic around `cast`, `getVerifier`, `push_back`, `dyn_cast`, and 1 more symbols.
- **CN**: 围绕 `cast`, `getVerifier`, `push_back`, `dyn_cast`, and 1 more symbols 实现具体逻辑。

### Lines 467-480
```cpp
  // Gather which constraint slots correspond to parameter constraints
  SmallVector<size_t> paramConstraints;
  if (params.has_value()) {
    paramConstraints.reserve(params->getArgs().size());
    for (Value param : params->getArgs()) {
      for (auto [i, constr] : enumerate(constrToValue)) {
        if (constr == param) {
          paramConstraints.push_back(i);
          break;
        }
      }
    }
  }

```
- **EN**: Implements logic around `has_value`, `reserve`, `getArgs`, `enumerate`, and 1 more symbols.
- **CN**: 围绕 `has_value`, `reserve`, `getArgs`, `enumerate`, and 1 more symbols 实现具体逻辑。

### Lines 481-494
```cpp
  auto verifier = [paramConstraints{std::move(paramConstraints)},
                   constraints{std::move(constraints)}](
                      function_ref<InFlightDiagnostic()> emitError,
                      ArrayRef<Attribute> params) {
    return irdlAttrOrTypeVerifier(emitError, params, constraints,
                                  paramConstraints);
  };

  // While the `std::move` is not required, not adding it triggers a bug in
  // clang-10.
  return std::move(verifier);
}

/// Get the possible bases of a constraint. Return `true` if all bases can
```
- **EN**: Implements logic around `move`, `function_ref`, `irdlAttrOrTypeVerifier`.
- **CN**: 围绕 `move`, `function_ref`, `irdlAttrOrTypeVerifier` 实现具体逻辑。

### Lines 495-513
```cpp
/// potentially be matched.
/// A base is a type or an attribute definition. For instance, the base of
/// `irdl.parametric "!builtin.complex"(...)` is `builtin.complex`.
/// This function returns the following information through arguments:
/// - `paramIds`: the set of type or attribute IDs that are used as bases.
/// - `paramIrdlOps`: the set of IRDL operations that are used as bases.
/// - `isIds`: the set of type or attribute IDs that are used in `irdl.is`
///   constraints.
static bool getBases(Operation *op, SmallPtrSet<TypeID, 4> &paramIds,
                     SmallPtrSet<Operation *, 4> &paramIrdlOps,
                     SmallPtrSet<TypeID, 4> &isIds) {
  // For `irdl.any_of`, we get the bases from all its arguments.
  if (auto anyOf = dyn_cast<AnyOfOp>(op)) {
    bool hasAny = false;
    for (Value arg : anyOf.getArgs())
      hasAny &= getBases(arg.getDefiningOp(), paramIds, paramIrdlOps, isIds);
    return hasAny;
  }

```
- **EN**: Implements logic around `getBases`, `dyn_cast`, `getArgs`.
- **CN**: 围绕 `getBases`, `dyn_cast`, `getArgs` 实现具体逻辑。

### Lines 514-528
```cpp
  // For `irdl.all_of`, we get the bases from the first argument.
  // This is restrictive, but we can relax it later if needed.
  if (auto allOf = dyn_cast<AllOfOp>(op))
    return getBases(allOf.getArgs()[0].getDefiningOp(), paramIds, paramIrdlOps,
                    isIds);

  // For `irdl.parametric`, we get directly the base from the operation.
  if (auto params = dyn_cast<ParametricOp>(op)) {
    SymbolRefAttr symRef = params.getBaseType();
    Operation *defOp = irdl::lookupSymbolNearDialect(op, symRef);
    assert(defOp && "symbol reference should refer to an existing operation");
    paramIrdlOps.insert(defOp);
    return false;
  }

```
- **EN**: Implements logic around `dyn_cast`, `getBases`, `getBaseType`, `lookupSymbolNearDialect`, and 2 more symbols; this block works with symbol tables or function-like operations.
- **CN**: 围绕 `dyn_cast`, `getBases`, `getBaseType`, `lookupSymbolNearDialect`, and 2 more symbols 实现具体逻辑；该代码块处理符号表或类函数操作。

### Lines 529-556
```cpp
  // For `irdl.is`, we get the base TypeID directly.
  if (auto is = dyn_cast<IsOp>(op)) {
    Attribute expected = is.getExpected();
    isIds.insert(expected.getTypeID());
    return false;
  }

  if (auto base = dyn_cast<BaseOp>(op)) {
    if (base.getBaseName()) {
      StringRef baseName = *base.getBaseName();
      if (baseName[0] == '!') {
        auto abstractType =
            AbstractType::lookup(baseName.drop_front(1), op->getContext());
        assert(abstractType && "type name should refer to an existing type");
        paramIds.insert(abstractType->get().getTypeID());
      } else if (baseName[0] == '#') {
        auto abstractAttr =
            AbstractAttribute::lookup(baseName.drop_front(1), op->getContext());
        assert(abstractAttr && "attribute name should refer to an existing "
                               "attribute");
        paramIds.insert(abstractAttr->get().getTypeID());
      } else {
        llvm_unreachable(
            "invalid `irdl.base` operation: base name should start "
            "with '!' for types or '#' for attributes");
      }
      return false;
    }
```
- **EN**: Implements logic around `dyn_cast`, `getExpected`, `insert`, `getBaseName`, and 3 more symbols.
- **CN**: 围绕 `dyn_cast`, `getExpected`, `insert`, `getBaseName`, and 3 more symbols 实现具体逻辑。

### Lines 557-570
```cpp

    if (base.getBaseRef()) {
      SymbolRefAttr symRef = *base.getBaseRef();
      Operation *defOp = irdl::lookupSymbolNearDialect(op, symRef);
      assert(defOp && "symbol reference should refer to an existing operation");
      paramIrdlOps.insert(defOp);
      return false;
    }

    llvm_unreachable(
        "invalid `irdl.base` operation: expected either a base name "
        "or a base symbol reference");
  }

```
- **EN**: Implements logic around `getBaseRef`, `lookupSymbolNearDialect`, `assert`, `insert`, and 1 more symbols; this block works with symbol tables or function-like operations.
- **CN**: 围绕 `getBaseRef`, `lookupSymbolNearDialect`, `assert`, `insert`, and 1 more symbols 实现具体逻辑；该代码块处理符号表或类函数操作。

### Lines 571-584
```cpp
  // For `irdl.any`, we return `false` since we can match any type or attribute
  // base.
  if (auto isA = dyn_cast<AnyOp>(op))
    return true;

  llvm_unreachable("unknown IRDL constraint");
}

/// Check that an any_of is in the subset IRDL can handle.
/// IRDL uses a greedy algorithm to match constraints. This means that if we
/// encounter an `any_of` with multiple constraints, we will match the first
/// constraint that is satisfied. Thus, the order of constraints matter in
/// `any_of` with our current algorithm.
/// In order to make the order of constraints irrelevant, we require that
```
- **EN**: Implements logic around `dyn_cast`, `llvm_unreachable`.
- **CN**: 围绕 `dyn_cast`, `llvm_unreachable` 实现具体逻辑。

### Lines 585-600
```cpp
/// all `any_of` constraint parameters are disjoint. For this, we check that
/// the base parameters are all disjoints between `parametric` operations, and
/// that they are disjoint between `parametric` and `is` operations.
/// This restriction will be relaxed in the future, when we will change our
/// algorithm to be non-greedy.
static LogicalResult checkCorrectAnyOf(AnyOfOp anyOf) {
  SmallPtrSet<TypeID, 4> paramIds;
  SmallPtrSet<Operation *, 4> paramIrdlOps;
  SmallPtrSet<TypeID, 4> isIds;

  for (Value arg : anyOf.getArgs()) {
    Operation *argOp = arg.getDefiningOp();
    SmallPtrSet<TypeID, 4> argParamIds;
    SmallPtrSet<Operation *, 4> argParamIrdlOps;
    SmallPtrSet<TypeID, 4> argIsIds;

```
- **EN**: Implements logic around `checkCorrectAnyOf`, `getArgs`, `getDefiningOp`.
- **CN**: 围绕 `checkCorrectAnyOf`, `getArgs`, `getDefiningOp` 实现具体逻辑。

### Lines 601-616
```cpp
    // Get the bases of this argument. If it can match any type or attribute,
    // then our `any_of` should not be allowed.
    if (getBases(argOp, argParamIds, argParamIrdlOps, argIsIds))
      return failure();

    // We check that the base parameters are all disjoints between `parametric`
    // operations, and that they are disjoint between `parametric` and `is`
    // operations.
    for (TypeID id : argParamIds) {
      if (isIds.count(id))
        return failure();
      bool inserted = paramIds.insert(id).second;
      if (!inserted)
        return failure();
    }

```
- **EN**: Implements logic around `getBases`, `failure`, `count`, `insert`.
- **CN**: 围绕 `getBases`, `failure`, `count`, `insert` 实现具体逻辑。

### Lines 617-635
```cpp
    // We check that the base parameters are all disjoints with `irdl.is`
    // operations.
    for (TypeID id : isIds) {
      if (paramIds.count(id))
        return failure();
      isIds.insert(id);
    }

    // We check that all `parametric` operations are disjoint. We do not
    // need to check that they are disjoint with `is` operations, since
    // `is` operations cannot refer to attributes defined with `irdl.parametric`
    // operations.
    for (Operation *op : argParamIrdlOps) {
      bool inserted = paramIrdlOps.insert(op).second;
      if (!inserted)
        return failure();
    }
  }

```
- **EN**: Implements logic around `count`, `failure`, `insert`.
- **CN**: 围绕 `count`, `failure`, `insert` 实现具体逻辑。

### Lines 636-649
```cpp
  return success();
}

/// Load all dialects in the given module, without loading any operation, type
/// or attribute definitions.
static DenseMap<DialectOp, ExtensibleDialect *> loadEmptyDialects(ModuleOp op) {
  DenseMap<DialectOp, ExtensibleDialect *> dialects;
  op.walk([&](DialectOp dialectOp) {
    MLIRContext *ctx = dialectOp.getContext();
    StringRef dialectName = dialectOp.getName();

    DynamicDialect *dialect = ctx->getOrLoadDynamicDialect(
        dialectName, [](DynamicDialect *dialect) {});

```
- **EN**: Implements logic around `success`, `loadEmptyDialects`, `walk`, `getContext`, and 2 more symbols.
- **CN**: 围绕 `success`, `loadEmptyDialects`, `walk`, `getContext`, and 2 more symbols 实现具体逻辑。

### Lines 650-672
```cpp
    dialects.insert({dialectOp, dialect});
  });
  return dialects;
}

/// Preallocate type definitions objects with empty verifiers.
/// This in particular allocates a TypeID for each type definition.
static DenseMap<TypeOp, std::unique_ptr<DynamicTypeDefinition>>
preallocateTypeDefs(ModuleOp op,
                    DenseMap<DialectOp, ExtensibleDialect *> dialects) {
  DenseMap<TypeOp, std::unique_ptr<DynamicTypeDefinition>> typeDefs;
  op.walk([&](TypeOp typeOp) {
    ExtensibleDialect *dialect = dialects[typeOp.getParentOp()];
    auto typeDef = DynamicTypeDefinition::get(
        typeOp.getName(), dialect,
        [](function_ref<InFlightDiagnostic()>, ArrayRef<Attribute>) {
          return success();
        });
    typeDefs.try_emplace(typeOp, std::move(typeDef));
  });
  return typeDefs;
}

```
- **EN**: Implements logic around `insert`, `preallocateTypeDefs`, `walk`, `getParentOp`, and 5 more symbols.
- **CN**: 围绕 `insert`, `preallocateTypeDefs`, `walk`, `getParentOp`, and 5 more symbols 实现具体逻辑。

### Lines 673-690
```cpp
/// Preallocate attribute definitions objects with empty verifiers.
/// This in particular allocates a TypeID for each attribute definition.
static DenseMap<AttributeOp, std::unique_ptr<DynamicAttrDefinition>>
preallocateAttrDefs(ModuleOp op,
                    DenseMap<DialectOp, ExtensibleDialect *> dialects) {
  DenseMap<AttributeOp, std::unique_ptr<DynamicAttrDefinition>> attrDefs;
  op.walk([&](AttributeOp attrOp) {
    ExtensibleDialect *dialect = dialects[attrOp.getParentOp()];
    auto attrDef = DynamicAttrDefinition::get(
        attrOp.getName(), dialect,
        [](function_ref<InFlightDiagnostic()>, ArrayRef<Attribute>) {
          return success();
        });
    attrDefs.try_emplace(attrOp, std::move(attrDef));
  });
  return attrDefs;
}

```
- **EN**: Implements logic around `preallocateAttrDefs`, `walk`, `getParentOp`, `get`, and 4 more symbols.
- **CN**: 围绕 `preallocateAttrDefs`, `walk`, `getParentOp`, `get`, and 4 more symbols 实现具体逻辑。

### Lines 691-707
```cpp
LogicalResult mlir::irdl::loadDialects(ModuleOp op) {
  // First, check that all any_of constraints are in a correct form.
  // This is to ensure we can do the verification correctly.
  WalkResult anyOfCorrects = op.walk(
      [](AnyOfOp anyOf) { return (WalkResult)checkCorrectAnyOf(anyOf); });
  if (anyOfCorrects.wasInterrupted())
    return op.emitError("any_of constraints are not in the correct form");

  // Preallocate all dialects, and type and attribute definitions.
  // In particular, this allocates TypeIDs so type and attributes can have
  // verifiers that refer to each other.
  DenseMap<DialectOp, ExtensibleDialect *> dialects = loadEmptyDialects(op);
  DenseMap<TypeOp, std::unique_ptr<DynamicTypeDefinition>> types =
      preallocateTypeDefs(op, dialects);
  DenseMap<AttributeOp, std::unique_ptr<DynamicAttrDefinition>> attrs =
      preallocateAttrDefs(op, dialects);

```
- **EN**: Implements logic around `loadDialects`, `walk`, `checkCorrectAnyOf`, `wasInterrupted`, and 4 more symbols.
- **CN**: 围绕 `loadDialects`, `walk`, `checkCorrectAnyOf`, `wasInterrupted`, and 4 more symbols 实现具体逻辑。

### Lines 708-731
```cpp
  // Set the verifier for types.
  WalkResult res = op.walk([&](TypeOp typeOp) {
    DynamicAttrDefinition::VerifierFn verifier = getAttrOrTypeVerifier(
        typeOp, dialects[typeOp.getParentOp()], types, attrs);
    if (!verifier)
      return WalkResult::interrupt();
    types[typeOp]->setVerifyFn(std::move(verifier));
    return WalkResult::advance();
  });
  if (res.wasInterrupted())
    return failure();

  // Set the verifier for attributes.
  res = op.walk([&](AttributeOp attrOp) {
    DynamicAttrDefinition::VerifierFn verifier = getAttrOrTypeVerifier(
        attrOp, dialects[attrOp.getParentOp()], types, attrs);
    if (!verifier)
      return WalkResult::interrupt();
    attrs[attrOp]->setVerifyFn(std::move(verifier));
    return WalkResult::advance();
  });
  if (res.wasInterrupted())
    return failure();

```
- **EN**: Implements logic around `walk`, `getAttrOrTypeVerifier`, `getParentOp`, `interrupt`, and 4 more symbols.
- **CN**: 围绕 `walk`, `getAttrOrTypeVerifier`, `getParentOp`, `interrupt`, and 4 more symbols 实现具体逻辑。

### Lines 732-750
```cpp
  // Define and load all operations.
  res = op.walk([&](OperationOp opOp) {
    return loadOperation(opOp, dialects[opOp.getParentOp()], types, attrs);
  });
  if (res.wasInterrupted())
    return failure();

  // Load all types in their dialects.
  for (auto &pair : types) {
    ExtensibleDialect *dialect = dialects[pair.first.getParentOp()];
    dialect->registerDynamicType(std::move(pair.second));
  }

  // Load all attributes in their dialects.
  for (auto &pair : attrs) {
    ExtensibleDialect *dialect = dialects[pair.first.getParentOp()];
    dialect->registerDynamicAttr(std::move(pair.second));
  }

```
- **EN**: Implements logic around `walk`, `loadOperation`, `wasInterrupted`, `failure`, and 3 more symbols.
- **CN**: 围绕 `walk`, `loadOperation`, `wasInterrupted`, `failure`, and 3 more symbols 实现具体逻辑。

### Lines 751-752
```cpp
  return success();
}
```
- **EN**: Implements logic around `success`.
- **CN**: 围绕 `success` 实现具体逻辑。

## Key Concepts / 关键概念

- **Declarative dialect definitions / 声明式方言定义**:
  - **EN**: Encodes dialect structure and constraints as IR that can define other IR.
  - **CN**: 把方言结构与约束编码成可定义其他 IR 的 IR。
- **Assembly format hooks / 汇编格式钩子**:
  - **EN**: Custom parsers and printers define how operations or attributes appear in MLIR assembly.
  - **CN**: 自定义解析器和打印器定义操作或属性在 MLIR 汇编中的呈现方式。
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
- **Streaming output / 流式输出**:
  - **EN**: Writes diagnostics or assembly-like textual forms through LLVM/MLIR stream APIs.
  - **CN**: 通过 LLVM/MLIR 流式 API 输出诊断或类汇编文本。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/IRDL/IRDLLoading.h`, `mlir/Dialect/IRDL/IR/IRDL.h`, `mlir/Dialect/IRDL/IR/IRDLInterfaces.h`, `mlir/Dialect/IRDL/IRDLSymbols.h`, `mlir/Dialect/IRDL/IRDLVerifiers.h`, `mlir/IR/Attributes.h`, `mlir/IR/BuiltinOps.h`, `mlir/IR/ExtensibleDialect.h`, `mlir/IR/OperationSupport.h`, `llvm/ADT/STLExtras.h`
- **Subsystem categories / 子系统类别**: other MLIR dialect declarations / 其他 MLIR 方言声明 (5), MLIR IR core abstractions / MLIR IR 核心抽象 (4), LLVM ADT containers and utility types / LLVM ADT 容器与工具类型 (1)
