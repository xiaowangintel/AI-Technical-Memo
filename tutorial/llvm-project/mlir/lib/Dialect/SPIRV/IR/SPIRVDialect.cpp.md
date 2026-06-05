# SPIRVDialect.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/SPIRV/IR/SPIRVDialect.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This file defines the SPIR-V dialect in MLIR.
  - **CN**: 该文件位于 `mlir/lib/Dialect/SPIRV/IR`，围绕 SPIRV 方言实现相关能力。

## Line-by-Line Analysis / 逐行分析

### Lines 1-14
```cpp
//===- LLVMDialect.cpp - MLIR SPIR-V dialect ------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file defines the SPIR-V dialect in MLIR.
//
//===----------------------------------------------------------------------===//

#include "mlir/Dialect/SPIRV/IR/SPIRVDialect.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/SPIRV/IR/SPIRVDialect.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/SPIRV/IR/SPIRVDialect.h`。

### Lines 15-31
```cpp
#include "SPIRVParsingUtils.h"

#include "mlir/Dialect/GPU/IR/CompilationInterfaces.h"
#include "mlir/Dialect/SPIRV/IR/SPIRVOps.h"
#include "mlir/Dialect/SPIRV/IR/SPIRVTypes.h"
#include "mlir/Dialect/SPIRV/IR/TargetAndABI.h"
#include "mlir/Dialect/UB/IR/UBOps.h"
#include "mlir/IR/Builders.h"
#include "mlir/IR/BuiltinTypes.h"
#include "mlir/IR/DialectImplementation.h"
#include "mlir/IR/MLIRContext.h"
#include "mlir/Parser/Parser.h"
#include "mlir/Transforms/InliningUtils.h"
#include "llvm/ADT/Sequence.h"
#include "llvm/ADT/StringExtras.h"
#include "llvm/ADT/TypeSwitch.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `SPIRVParsingUtils.h`, `mlir/Dialect/GPU/IR/CompilationInterfaces.h`, `mlir/Dialect/SPIRV/IR/SPIRVOps.h`, `mlir/Dialect/SPIRV/IR/SPIRVTypes.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `SPIRVParsingUtils.h`, `mlir/Dialect/GPU/IR/CompilationInterfaces.h`, `mlir/Dialect/SPIRV/IR/SPIRVOps.h`, `mlir/Dialect/SPIRV/IR/SPIRVTypes.h`。

### Lines 32-49
```cpp
using namespace mlir;
using namespace mlir::spirv;

#include "mlir/Dialect/SPIRV/IR/SPIRVOpsDialect.cpp.inc"

//===----------------------------------------------------------------------===//
// InlinerInterface
//===----------------------------------------------------------------------===//

/// Returns true if the given region contains spirv.Return or spirv.ReturnValue
/// ops.
static inline bool containsReturn(Region &region) {
  return llvm::any_of(region, [](Block &block) {
    Operation *terminator = block.getTerminator();
    return isa<spirv::ReturnOp, spirv::ReturnValueOp>(terminator);
  });
}

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/SPIRV/IR/SPIRVOpsDialect.cpp.inc`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/SPIRV/IR/SPIRVOpsDialect.cpp.inc`。

### Lines 50-70
```cpp
namespace {
/// This class defines the interface for inlining within the SPIR-V dialect.
struct SPIRVInlinerInterface : public DialectInlinerInterface {
  using DialectInlinerInterface::DialectInlinerInterface;

  /// All call operations within SPIRV can be inlined.
  bool isLegalToInline(Operation *call, Operation *callable,
                       bool wouldBeCloned) const final {
    return true;
  }

  /// Returns true if the given region 'src' can be inlined into the region
  /// 'dest' that is attached to an operation registered to the current dialect.
  bool isLegalToInline(Region *dest, Region *src, bool wouldBeCloned,
                       IRMapping &) const final {
    // Return true here when inlining into spirv.func, spirv.mlir.selection, and
    // spirv.mlir.loop operations.
    auto *op = dest->getParentOp();
    return isa<spirv::FuncOp, spirv::SelectionOp, spirv::LoopOp>(op);
  }

```
- **EN**: Introduces declarations for `defines`, `SPIRVInlinerInterface`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `defines`, `SPIRVInlinerInterface` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 71-87
```cpp
  /// Returns true if the given operation 'op', that is registered to this
  /// dialect, can be inlined into the region 'dest' that is attached to an
  /// operation registered to the current dialect.
  bool isLegalToInline(Operation *op, Region *dest, bool wouldBeCloned,
                       IRMapping &) const final {
    // TODO: Enable inlining structured control flows with return.
    if ((isa<spirv::SelectionOp, spirv::LoopOp>(op)) &&
        containsReturn(op->getRegion(0)))
      return false;
    // TODO: we need to filter OpKill here to avoid inlining it to
    // a loop continue construct:
    // https://github.com/KhronosGroup/SPIRV-Headers/issues/86
    // For now, we just disallow inlining OpKill anywhere in the code,
    // but this restriction should be relaxed, as pointed above.
    if (isa<spirv::KillOp>(op))
      return false;

```
- **EN**: Implements logic around `isLegalToInline`, `LoopOp>`, `containsReturn`, `KillOp>`; this block works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `isLegalToInline`, `LoopOp>`, `containsReturn`, `KillOp>` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义。

### Lines 88-105
```cpp
    return true;
  }

  /// Handle the given inlined terminator by replacing it with a new operation
  /// as necessary.
  void handleTerminator(Operation *op, Block *newDest) const final {
    if (auto returnOp = dyn_cast<spirv::ReturnOp>(op)) {
      auto builder = OpBuilder(op);
      spirv::BranchOp::create(builder, op->getLoc(), newDest);
      op->erase();
    } else if (auto retValOp = dyn_cast<spirv::ReturnValueOp>(op)) {
      auto builder = OpBuilder(op);
      spirv::BranchOp::create(builder, retValOp->getLoc(), newDest,
                              retValOp->getOperands());
      op->erase();
    }
  }

```
- **EN**: Implements logic around `handleTerminator`, `ReturnOp>`, `OpBuilder`, `create`, and 3 more symbols; this block touches target- or accelerator-specific semantics.
- **CN**: 围绕 `handleTerminator`, `ReturnOp>`, `OpBuilder`, `create`, and 3 more symbols 实现具体逻辑；该代码块涉及目标平台或加速器专用语义。

### Lines 106-121
```cpp
  /// Handle the given inlined terminator by replacing it with a new operation
  /// as necessary.
  void handleTerminator(Operation *op, ValueRange valuesToRepl) const final {
    // Only spirv.ReturnValue needs to be handled here.
    auto retValOp = dyn_cast<spirv::ReturnValueOp>(op);
    if (!retValOp)
      return;

    // Replace the values directly with the return operands.
    assert(valuesToRepl.size() == 1 &&
           "spirv.ReturnValue expected to only handle one result");
    valuesToRepl.front().replaceAllUsesWith(retValOp.getValue());
  }
};
} // namespace

```
- **EN**: Implements logic around `handleTerminator`, `ReturnValueOp>`, `assert`, `front`; this block works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `handleTerminator`, `ReturnValueOp>`, `assert`, `front` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义。

### Lines 122-135
```cpp
//===----------------------------------------------------------------------===//
// SPIR-V Dialect
//===----------------------------------------------------------------------===//

void SPIRVDialect::initialize() {
  registerAttributes();
  registerTypes();

  // Add SPIR-V ops.
  addOperations<
#define GET_OP_LIST
#include "mlir/Dialect/SPIRV/IR/SPIRVOps.cpp.inc"
      >();

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/SPIRV/IR/SPIRVOps.cpp.inc`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/SPIRV/IR/SPIRVOps.cpp.inc`。

### Lines 136-149
```cpp
  addInterfaces<SPIRVInlinerInterface>();

  // Allow unknown operations because SPIR-V is extensible.
  allowUnknownOperations();
  declarePromisedInterface<gpu::TargetAttrInterface, TargetEnvAttr>();
}

std::string SPIRVDialect::getAttributeName(Decoration decoration) {
  return getDecorationString(decoration);
}

//===----------------------------------------------------------------------===//
// Type Parsing
//===----------------------------------------------------------------------===//
```
- **EN**: Implements logic around `addInterfaces`, `allowUnknownOperations`, `TargetEnvAttr>`, `getAttributeName`, and 1 more symbols; this block works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `addInterfaces`, `allowUnknownOperations`, `TargetEnvAttr>`, `getAttributeName`, and 1 more symbols 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义。

### Lines 150-169
```cpp

// Forward declarations.
template <typename ValTy>
static std::optional<ValTy> parseAndVerify(SPIRVDialect const &dialect,
                                           DialectAsmParser &parser);
template <>
std::optional<Type> parseAndVerify<Type>(SPIRVDialect const &dialect,
                                         DialectAsmParser &parser);

template <>
std::optional<unsigned> parseAndVerify<unsigned>(SPIRVDialect const &dialect,
                                                 DialectAsmParser &parser);

static Type parseAndVerifyType(SPIRVDialect const &dialect,
                               DialectAsmParser &parser) {
  Type type;
  SMLoc typeLoc = parser.getCurrentLocation();
  if (parser.parseType(type))
    return Type();

```
- **EN**: Implements logic around `parseAndVerify`, `parseAndVerifyType`, `getCurrentLocation`, `parseType`, and 1 more symbols; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `parseAndVerify`, `parseAndVerifyType`, `getCurrentLocation`, `parseType`, and 1 more symbols 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性。

### Lines 170-197
```cpp
  // Allow SPIR-V dialect types.
  if (&type.getDialect() == &dialect)
    return type;

  // Check other allowed types.
  if (auto t = dyn_cast<FloatType>(type)) {
    // TODO: All float types are allowed for now, but this should be fixed.
  } else if (auto t = dyn_cast<IntegerType>(type)) {
    if (!ScalarType::isValid(t)) {
      parser.emitError(typeLoc,
                       "only 1/8/16/32/64-bit integer type allowed but found ")
          << type;
      return Type();
    }
  } else if (auto t = dyn_cast<VectorType>(type)) {
    if (t.getRank() != 1) {
      parser.emitError(typeLoc, "only 1-D vector allowed but found ") << t;
      return Type();
    }
    if (t.getNumElements() < 2) {
      parser.emitError(typeLoc, "SPIR-V does not allow one-element vectors");
      return Type();
    }
    if (t.getNumElements() > 4) {
      parser.emitError(
          typeLoc, "vector length has to be less than or equal to 4 but found ")
          << t.getNumElements();
      return Type();
```
- **EN**: Implements logic around `getDialect`, `isValid`, `emitError`, `Type`, and 2 more symbols; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getDialect`, `isValid`, `emitError`, `Type`, and 2 more symbols 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 198-218
```cpp
    }
    if (!isa<ScalarType>(t.getElementType())) {
      parser.emitError(
          typeLoc,
          "vector element type must be a SPIR-V scalar type but found ")
          << t.getElementType();
      return Type();
    }
  } else if (auto t = dyn_cast<TensorArmType>(type)) {
    if (!isa<ScalarType>(t.getElementType())) {
      parser.emitError(
          typeLoc, "only scalar element type allowed in tensor type but found ")
          << t.getElementType();
      return Type();
    }
  } else {
    parser.emitError(typeLoc, "cannot use ")
        << type << " to compose SPIR-V types";
    return Type();
  }

```
- **EN**: Implements logic around `getElementType`, `emitError`, `Type`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getElementType`, `emitError`, `Type` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 219-241
```cpp
  return type;
}

static Type parseAndVerifyMatrixType(SPIRVDialect const &dialect,
                                     DialectAsmParser &parser) {
  Type type;
  SMLoc typeLoc = parser.getCurrentLocation();
  if (parser.parseType(type))
    return Type();

  if (auto t = dyn_cast<VectorType>(type)) {
    if (t.getRank() != 1) {
      parser.emitError(typeLoc, "only 1-D vector allowed but found ") << t;
      return Type();
    }
    if (t.getNumElements() > 4 || t.getNumElements() < 2) {
      parser.emitError(typeLoc,
                       "matrix columns size has to be less than or equal "
                       "to 4 and greater than or equal 2, but found ")
          << t.getNumElements();
      return Type();
    }

```
- **EN**: Implements logic around `parseAndVerifyMatrixType`, `getCurrentLocation`, `parseType`, `Type`, and 3 more symbols; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `parseAndVerifyMatrixType`, `getCurrentLocation`, `parseType`, `Type`, and 3 more symbols 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 242-257
```cpp
    if (!isa<FloatType>(t.getElementType())) {
      parser.emitError(typeLoc, "matrix columns' elements must be of "
                                "Float type, got ")
          << t.getElementType();
      return Type();
    }
  } else {
    parser.emitError(typeLoc, "matrix must be composed using vector "
                              "type, got ")
        << type;
    return Type();
  }

  return type;
}

```
- **EN**: Implements logic around `getElementType`, `emitError`, `Type`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getElementType`, `emitError`, `Type` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 258-272
```cpp
static Type parseAndVerifySampledImageType(SPIRVDialect const &dialect,
                                           DialectAsmParser &parser) {
  Type type;
  SMLoc typeLoc = parser.getCurrentLocation();
  if (parser.parseType(type))
    return Type();

  auto imageType = dyn_cast<ImageType>(type);
  if (!imageType) {
    parser.emitError(typeLoc,
                     "sampled image must be composed using image type, got ")
        << type;
    return Type();
  }

```
- **EN**: Implements logic around `parseAndVerifySampledImageType`, `getCurrentLocation`, `parseType`, `Type`, and 1 more symbols; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `parseAndVerifySampledImageType`, `getCurrentLocation`, `parseType`, `Type`, and 1 more symbols 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性。

### Lines 273-293
```cpp
  if (llvm::is_contained({Dim::SubpassData, Dim::Buffer}, imageType.getDim())) {
    parser.emitError(
        typeLoc, "sampled image Dim must not be SubpassData or Buffer, got ")
        << stringifyDim(imageType.getDim());
    return Type();
  }

  return type;
}

/// Parses an optional `, stride = N` assembly segment. If no parsing failure
/// occurs, writes `N` to `stride` if existing and writes 0 to `stride` if
/// missing.
static LogicalResult parseOptionalArrayStride(const SPIRVDialect &dialect,
                                              DialectAsmParser &parser,
                                              unsigned &stride) {
  if (failed(parser.parseOptionalComma())) {
    stride = 0;
    return success();
  }

```
- **EN**: Implements logic around `is_contained`, `emitError`, `stringifyDim`, `Type`, and 3 more symbols; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `is_contained`, `emitError`, `stringifyDim`, `Type`, and 3 more symbols 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性。

### Lines 294-308
```cpp
  if (parser.parseKeyword("stride") || parser.parseEqual())
    return failure();

  SMLoc strideLoc = parser.getCurrentLocation();
  std::optional<unsigned> optStride = parseAndVerify<unsigned>(dialect, parser);
  if (!optStride)
    return failure();

  if (!(stride = *optStride)) {
    parser.emitError(strideLoc, "ArrayStride must be greater than zero");
    return failure();
  }
  return success();
}

```
- **EN**: Implements logic around `parseKeyword`, `failure`, `getCurrentLocation`, `parseAndVerify`, and 2 more symbols; this block handles textual assembly parsing or printing concerns.
- **CN**: 围绕 `parseKeyword`, `failure`, `getCurrentLocation`, `parseAndVerify`, and 2 more symbols 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题。

### Lines 309-330
```cpp
// element-type ::= integer-type
//                | floating-point-type
//                | vector-type
//                | spirv-type
//
// array-type ::= `!spirv.array` `<` integer-literal `x` element-type
//                (`,` `stride` `=` integer-literal)? `>`
static Type parseArrayType(SPIRVDialect const &dialect,
                           DialectAsmParser &parser) {
  if (parser.parseLess())
    return Type();

  SmallVector<int64_t, 1> countDims;
  SMLoc countLoc = parser.getCurrentLocation();
  if (parser.parseDimensionList(countDims, /*allowDynamic=*/false))
    return Type();
  if (countDims.size() != 1) {
    parser.emitError(countLoc,
                     "expected single integer for array element count");
    return Type();
  }

```
- **EN**: Implements logic around `parseArrayType`, `parseLess`, `Type`, `getCurrentLocation`, and 3 more symbols; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `parseArrayType`, `parseLess`, `Type`, `getCurrentLocation`, and 3 more symbols 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义，并协调核心结构化 MLIR 方言之间的行为。

### Lines 331-346
```cpp
  // According to the SPIR-V spec:
  // "Length is the number of elements in the array. It must be at least 1."
  int64_t count = countDims[0];
  if (count == 0) {
    parser.emitError(countLoc, "expected array length greater than 0");
    return Type();
  }

  Type elementType = parseAndVerifyType(dialect, parser);
  if (!elementType)
    return Type();

  unsigned stride = 0;
  if (failed(parseOptionalArrayStride(dialect, parser, stride)))
    return Type();

```
- **EN**: Implements logic around `emitError`, `Type`, `parseAndVerifyType`, `failed`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `emitError`, `Type`, `parseAndVerifyType`, `failed` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性。

### Lines 347-364
```cpp
  if (parser.parseGreater())
    return Type();
  return ArrayType::get(elementType, count, stride);
}

// cooperative-matrix-type ::=
//   `!spirv.coopmatrix` `<` rows `x` columns `x` element-type `,`
//                           scope `,` use `>`
static Type parseCooperativeMatrixType(SPIRVDialect const &dialect,
                                       DialectAsmParser &parser) {
  if (parser.parseLess())
    return {};

  SmallVector<int64_t, 2> dims;
  SMLoc countLoc = parser.getCurrentLocation();
  if (parser.parseDimensionList(dims, /*allowDynamic=*/false))
    return {};

```
- **EN**: Implements logic around `parseGreater`, `Type`, `get`, `parseCooperativeMatrixType`, and 3 more symbols; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `parseGreater`, `Type`, `get`, `parseCooperativeMatrixType`, and 3 more symbols 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义。

### Lines 365-378
```cpp
  if (dims.size() != 2) {
    parser.emitError(countLoc, "expected row and column count");
    return {};
  }

  auto elementTy = parseAndVerifyType(dialect, parser);
  if (!elementTy)
    return {};

  Scope scope;
  if (parser.parseComma() ||
      spirv::parseEnumKeywordAttr(scope, parser, "scope <id>"))
    return {};

```
- **EN**: Implements logic around `size`, `emitError`, `parseAndVerifyType`, `parseComma`, and 1 more symbols; this block handles textual assembly parsing or printing concerns; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `size`, `emitError`, `parseAndVerifyType`, `parseComma`, and 1 more symbols 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并涉及目标平台或加速器专用语义。

### Lines 379-396
```cpp
  CooperativeMatrixUseKHR use;
  if (parser.parseComma() ||
      spirv::parseEnumKeywordAttr(use, parser, "use <id>"))
    return {};

  if (parser.parseGreater())
    return {};

  return CooperativeMatrixType::get(elementTy, dims[0], dims[1], scope, use);
}

// tensor-arm-type ::=
//   `!spirv.arm.tensor` `<` dim0 `x` dim1 `x` ... `x` dimN `x` element-type`>`
static Type parseTensorArmType(SPIRVDialect const &dialect,
                               DialectAsmParser &parser) {
  if (parser.parseLess())
    return {};

```
- **EN**: Implements logic around `parseComma`, `parseEnumKeywordAttr`, `parseGreater`, `get`, and 2 more symbols; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `parseComma`, `parseEnumKeywordAttr`, `parseGreater`, `get`, and 2 more symbols 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义，并协调核心结构化 MLIR 方言之间的行为。

### Lines 397-413
```cpp
  bool unranked = false;
  SmallVector<int64_t, 4> dims;
  SMLoc countLoc = parser.getCurrentLocation();

  if (parser.parseOptionalStar().succeeded()) {
    unranked = true;
    if (parser.parseXInDimensionList())
      return {};
  } else if (parser.parseDimensionList(dims, /*allowDynamic=*/true)) {
    return {};
  }

  if (!unranked && dims.empty()) {
    parser.emitError(countLoc, "arm.tensors do not support rank zero");
    return {};
  }

```
- **EN**: Implements logic around `getCurrentLocation`, `parseOptionalStar`, `parseXInDimensionList`, `parseDimensionList`, and 2 more symbols; this block handles textual assembly parsing or printing concerns.
- **CN**: 围绕 `getCurrentLocation`, `parseOptionalStar`, `parseXInDimensionList`, `parseDimensionList`, and 2 more symbols 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题。

### Lines 414-429
```cpp
  if (llvm::is_contained(dims, 0)) {
    parser.emitError(countLoc, "arm.tensors do not support zero dimensions");
    return {};
  }

  if (llvm::any_of(dims, [](int64_t dim) { return dim < 0; }) &&
      llvm::any_of(dims, [](int64_t dim) { return dim > 0; })) {
    parser.emitError(countLoc, "arm.tensor shape dimensions must be either "
                               "fully dynamic or completed shaped");
    return {};
  }

  auto elementTy = parseAndVerifyType(dialect, parser);
  if (!elementTy)
    return {};

```
- **EN**: Implements logic around `is_contained`, `emitError`, `any_of`, `parseAndVerifyType`; this block handles textual assembly parsing or printing concerns; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `is_contained`, `emitError`, `any_of`, `parseAndVerifyType` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并协调核心结构化 MLIR 方言之间的行为。

### Lines 430-449
```cpp
  if (parser.parseGreater())
    return {};

  return TensorArmType::get(dims, elementTy);
}

// TODO: Reorder methods to be utilities first and parse*Type
// methods in alphabetical order
//
// storage-class ::= `UniformConstant`
//                 | `Uniform`
//                 | `Workgroup`
//                 | <and other storage classes...>
//
// pointer-type ::= `!spirv.ptr<` element-type `,` storage-class `>`
static Type parsePointerType(SPIRVDialect const &dialect,
                             DialectAsmParser &parser) {
  if (parser.parseLess())
    return Type();

```
- **EN**: Implements logic around `parseGreater`, `get`, `parsePointerType`, `parseLess`, and 1 more symbols; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `parseGreater`, `get`, `parsePointerType`, `parseLess`, and 1 more symbols 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义。

### Lines 450-469
```cpp
  auto pointeeType = parseAndVerifyType(dialect, parser);
  if (!pointeeType)
    return Type();

  StringRef storageClassSpec;
  SMLoc storageClassLoc = parser.getCurrentLocation();
  if (parser.parseComma() || parser.parseKeyword(&storageClassSpec))
    return Type();

  auto storageClass = symbolizeStorageClass(storageClassSpec);
  if (!storageClass) {
    parser.emitError(storageClassLoc, "unknown storage class: ")
        << storageClassSpec;
    return Type();
  }
  if (parser.parseGreater())
    return Type();
  return PointerType::get(pointeeType, *storageClass);
}

```
- **EN**: Implements logic around `parseAndVerifyType`, `Type`, `getCurrentLocation`, `parseComma`, and 4 more symbols; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `parseAndVerifyType`, `Type`, `getCurrentLocation`, `parseComma`, and 4 more symbols 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性。

### Lines 470-484
```cpp
// runtime-array-type ::= `!spirv.rtarray` `<` element-type
//                        (`,` `stride` `=` integer-literal)? `>`
static Type parseRuntimeArrayType(SPIRVDialect const &dialect,
                                  DialectAsmParser &parser) {
  if (parser.parseLess())
    return Type();

  Type elementType = parseAndVerifyType(dialect, parser);
  if (!elementType)
    return Type();

  unsigned stride = 0;
  if (failed(parseOptionalArrayStride(dialect, parser, stride)))
    return Type();

```
- **EN**: Implements logic around `parseRuntimeArrayType`, `parseLess`, `Type`, `parseAndVerifyType`, and 1 more symbols; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `parseRuntimeArrayType`, `parseLess`, `Type`, `parseAndVerifyType`, and 1 more symbols 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义。

### Lines 485-505
```cpp
  if (parser.parseGreater())
    return Type();
  return RuntimeArrayType::get(elementType, stride);
}

// matrix-type ::= `!spirv.matrix` `<` integer-literal `x` element-type `>`
static Type parseMatrixType(SPIRVDialect const &dialect,
                            DialectAsmParser &parser) {
  if (parser.parseLess())
    return Type();

  SmallVector<int64_t, 1> countDims;
  SMLoc countLoc = parser.getCurrentLocation();
  if (parser.parseDimensionList(countDims, /*allowDynamic=*/false))
    return Type();
  if (countDims.size() != 1) {
    parser.emitError(countLoc, "expected single unsigned "
                               "integer for number of columns");
    return Type();
  }

```
- **EN**: Implements logic around `parseGreater`, `Type`, `get`, `parseMatrixType`, and 5 more symbols; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `parseGreater`, `Type`, `get`, `parseMatrixType`, and 5 more symbols 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义。

### Lines 506-520
```cpp
  int64_t columnCount = countDims[0];
  // According to the specification, Matrices can have 2, 3, or 4 columns
  if (columnCount < 2 || columnCount > 4) {
    parser.emitError(countLoc, "matrix is expected to have 2, 3, or 4 "
                               "columns");
    return Type();
  }

  Type columnType = parseAndVerifyMatrixType(dialect, parser);
  if (!columnType)
    return Type();

  if (parser.parseGreater())
    return Type();

```
- **EN**: Implements logic around `emitError`, `Type`, `parseAndVerifyMatrixType`, `parseGreater`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `emitError`, `Type`, `parseAndVerifyMatrixType`, `parseGreater` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性。

### Lines 521-534
```cpp
  return MatrixType::get(columnType, columnCount);
}

// Specialize this function to parse each of the parameters that define an
// ImageType. By default it assumes this is an enum type.
template <typename ValTy>
static std::optional<ValTy> parseAndVerify(SPIRVDialect const &dialect,
                                           DialectAsmParser &parser) {
  StringRef enumSpec;
  SMLoc enumLoc = parser.getCurrentLocation();
  if (parser.parseKeyword(&enumSpec)) {
    return std::nullopt;
  }

```
- **EN**: Introduces declarations for `type`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `type` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 535-550
```cpp
  auto val = spirv::symbolizeEnum<ValTy>(enumSpec);
  if (!val)
    parser.emitError(enumLoc, "unknown attribute: '") << enumSpec << "'";
  return val;
}

template <>
std::optional<Type> parseAndVerify<Type>(SPIRVDialect const &dialect,
                                         DialectAsmParser &parser) {
  // TODO: Further verify that the element type can be sampled
  auto ty = parseAndVerifyType(dialect, parser);
  if (!ty)
    return std::nullopt;
  return ty;
}

```
- **EN**: Implements logic around `symbolizeEnum`, `emitError`, `parseAndVerify`, `parseAndVerifyType`; this block handles textual assembly parsing or printing concerns; checks operation invariants or verification rules; works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `symbolizeEnum`, `emitError`, `parseAndVerify`, `parseAndVerifyType` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并检查操作不变式或验证规则，并处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义。

### Lines 551-565
```cpp
template <typename IntTy>
static std::optional<IntTy> parseAndVerifyInteger(SPIRVDialect const &dialect,
                                                  DialectAsmParser &parser) {
  IntTy offsetVal = std::numeric_limits<IntTy>::max();
  if (parser.parseInteger(offsetVal))
    return std::nullopt;
  return offsetVal;
}

template <>
std::optional<unsigned> parseAndVerify<unsigned>(SPIRVDialect const &dialect,
                                                 DialectAsmParser &parser) {
  return parseAndVerifyInteger<unsigned>(dialect, parser);
}

```
- **EN**: Implements logic around `parseAndVerifyInteger`, `max`, `parseInteger`, `parseAndVerify`; this block handles textual assembly parsing or printing concerns.
- **CN**: 围绕 `parseAndVerifyInteger`, `max`, `parseInteger`, `parseAndVerify` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题。

### Lines 566-589
```cpp
namespace {
// Functor object to parse a comma separated list of specs. The function
// parseAndVerify does the actual parsing and verification of individual
// elements. This is a functor since parsing the last element of the list
// (termination condition) needs partial specialization.
template <typename ParseType, typename... Args>
struct ParseCommaSeparatedList {
  std::optional<std::tuple<ParseType, Args...>>
  operator()(SPIRVDialect const &dialect, DialectAsmParser &parser) const {
    auto parseVal = parseAndVerify<ParseType>(dialect, parser);
    if (!parseVal)
      return std::nullopt;

    auto numArgs = std::tuple_size<std::tuple<Args...>>::value;
    if (numArgs != 0 && failed(parser.parseComma()))
      return std::nullopt;
    auto remainingValues = ParseCommaSeparatedList<Args...>{}(dialect, parser);
    if (!remainingValues)
      return std::nullopt;
    return std::tuple_cat(std::tuple<ParseType>(parseVal.value()),
                          remainingValues.value());
  }
};

```
- **EN**: Introduces declarations for `ParseCommaSeparatedList`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `ParseCommaSeparatedList` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 590-617
```cpp
// Partial specialization of the function to parse a comma separated list of
// specs to parse the last element of the list.
template <typename ParseType>
struct ParseCommaSeparatedList<ParseType> {
  std::optional<std::tuple<ParseType>>
  operator()(SPIRVDialect const &dialect, DialectAsmParser &parser) const {
    if (auto value = parseAndVerify<ParseType>(dialect, parser))
      return std::tuple<ParseType>(*value);
    return std::nullopt;
  }
};
} // namespace

// dim ::= `1D` | `2D` | `3D` | `Cube` | <and other SPIR-V Dim specifiers...>
//
// depth-info ::= `NoDepth` | `IsDepth` | `DepthUnknown`
//
// arrayed-info ::= `NonArrayed` | `Arrayed`
//
// sampling-info ::= `SingleSampled` | `MultiSampled`
//
// sampler-use-info ::= `SamplerUnknown` | `NeedSampler` |  `NoSampler`
//
// format ::= `Unknown` | `Rgba32f` | <and other SPIR-V Image formats...>
//
// image-type ::= `!spirv.image<` element-type `,` dim `,` depth-info `,`
//                              arrayed-info `,` sampling-info `,`
//                              sampler-use-info `,` format `>`
```
- **EN**: Introduces declarations for `ParseCommaSeparatedList`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `ParseCommaSeparatedList` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 618-634
```cpp
static Type parseImageType(SPIRVDialect const &dialect,
                           DialectAsmParser &parser) {
  if (parser.parseLess())
    return Type();

  auto value =
      ParseCommaSeparatedList<Type, Dim, ImageDepthInfo, ImageArrayedInfo,
                              ImageSamplingInfo, ImageSamplerUseInfo,
                              ImageFormat>{}(dialect, parser);
  if (!value)
    return Type();

  if (parser.parseGreater())
    return Type();
  return ImageType::get(*value);
}

```
- **EN**: Implements logic around `parseImageType`, `parseLess`, `Type`, `parseGreater`, and 1 more symbols; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `parseImageType`, `parseLess`, `Type`, `parseGreater`, and 1 more symbols 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性。

### Lines 635-649
```cpp
// sampledImage-type :: = `!spirv.sampledImage<` image-type `>`
static Type parseSampledImageType(SPIRVDialect const &dialect,
                                  DialectAsmParser &parser) {
  if (parser.parseLess())
    return Type();

  Type parsedType = parseAndVerifySampledImageType(dialect, parser);
  if (!parsedType)
    return Type();

  if (parser.parseGreater())
    return Type();
  return SampledImageType::get(parsedType);
}

```
- **EN**: Implements logic around `parseSampledImageType`, `parseLess`, `Type`, `parseAndVerifySampledImageType`, and 2 more symbols; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `parseSampledImageType`, `parseLess`, `Type`, `parseAndVerifySampledImageType`, and 2 more symbols 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义。

### Lines 650-664
```cpp
// Parse decorations associated with a member.
static ParseResult parseStructMemberDecorations(
    SPIRVDialect const &dialect, DialectAsmParser &parser,
    ArrayRef<Type> memberTypes,
    SmallVectorImpl<StructType::OffsetInfo> &offsetInfo,
    SmallVectorImpl<StructType::MemberDecorationInfo> &memberDecorationInfo) {

  // Check if the first element is offset.
  SMLoc offsetLoc = parser.getCurrentLocation();
  StructType::OffsetInfo offset = 0;
  OptionalParseResult offsetParseResult = parser.parseOptionalInteger(offset);
  if (offsetParseResult.has_value()) {
    if (failed(*offsetParseResult))
      return failure();

```
- **EN**: Implements logic around `parseStructMemberDecorations`, `getCurrentLocation`, `parseOptionalInteger`, `has_value`, and 2 more symbols; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `parseStructMemberDecorations`, `getCurrentLocation`, `parseOptionalInteger`, `has_value`, and 2 more symbols 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性。

### Lines 665-680
```cpp
    if (offsetInfo.size() != memberTypes.size() - 1) {
      return parser.emitError(offsetLoc,
                              "offset specification must be given for "
                              "all members");
    }
    offsetInfo.push_back(offset);
  }

  // Check for no spirv::Decorations.
  if (succeeded(parser.parseOptionalRSquare()))
    return success();

  // If there was an offset, make sure to parse the comma.
  if (offsetParseResult.has_value() && parser.parseComma())
    return failure();

```
- **EN**: Implements logic around `size`, `emitError`, `push_back`, `succeeded`, and 3 more symbols; this block handles textual assembly parsing or printing concerns; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `size`, `emitError`, `push_back`, `succeeded`, and 3 more symbols 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并涉及目标平台或加速器专用语义。

### Lines 681-706
```cpp
  // Check for spirv::Decorations.
  auto parseDecorations = [&]() {
    auto memberDecoration = parseAndVerify<spirv::Decoration>(dialect, parser);
    if (!memberDecoration)
      return failure();

    // Parse member decoration value if it exists.
    if (succeeded(parser.parseOptionalEqual())) {
      Attribute memberDecorationValue;
      if (failed(parser.parseAttribute(memberDecorationValue)))
        return failure();

      memberDecorationInfo.emplace_back(
          static_cast<uint32_t>(memberTypes.size() - 1),
          memberDecoration.value(), memberDecorationValue);
    } else {
      memberDecorationInfo.emplace_back(
          static_cast<uint32_t>(memberTypes.size() - 1),
          memberDecoration.value(), UnitAttr::get(dialect.getContext()));
    }
    return success();
  };
  if (failed(parser.parseCommaSeparatedList(parseDecorations)) ||
      failed(parser.parseRSquare()))
    return failure();

```
- **EN**: Implements logic around `Decoration>`, `failure`, `succeeded`, `failed`, and 4 more symbols; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `Decoration>`, `failure`, `succeeded`, `failed`, and 4 more symbols 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义。

### Lines 707-721
```cpp
  return success();
}

// struct-member-decoration ::= integer-literal? spirv-decoration*
// struct-type ::=
//             `!spirv.struct<` (id `,`)?
//                          `(`
//                            (spirv-type (`[` struct-member-decoration `]`)?)*
//                          `)`
//                            (`,` struct-decoration)?
//                          `>`
static Type parseStructType(SPIRVDialect const &dialect,
                            DialectAsmParser &parser) {
  // TODO: This function is quite lengthy. Break it down into smaller chunks.

```
- **EN**: Implements logic around `success`, `parseStructType`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `success`, `parseStructType` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义。

### Lines 722-739
```cpp
  if (parser.parseLess())
    return Type();

  StringRef identifier;
  FailureOr<DialectAsmParser::CyclicParseReset> cyclicParse;

  // Check if this is an identified struct type.
  if (succeeded(parser.parseOptionalKeyword(&identifier))) {
    // Check if this is a possible recursive reference.
    auto structType =
        StructType::getIdentified(dialect.getContext(), identifier);
    cyclicParse = parser.tryStartCyclicParse(structType);
    if (succeeded(parser.parseOptionalGreater())) {
      if (succeeded(cyclicParse)) {
        parser.emitError(
            parser.getNameLoc(),
            "recursive struct reference not nested in struct definition");

```
- **EN**: Introduces declarations for `type`, `reference`, `definition`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `type`, `reference`, `definition` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 740-755
```cpp
        return Type();
      }

      return structType;
    }

    if (failed(parser.parseComma()))
      return Type();

    if (failed(cyclicParse)) {
      parser.emitError(parser.getNameLoc(),
                       "identifier already used for an enclosing struct");
      return Type();
    }
  }

```
- **EN**: Implements logic around `Type`, `failed`, `emitError`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `Type`, `failed`, `emitError` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性。

### Lines 756-772
```cpp
  if (failed(parser.parseLParen()))
    return Type();

  if (succeeded(parser.parseOptionalRParen()) &&
      succeeded(parser.parseOptionalGreater())) {
    return StructType::getEmpty(dialect.getContext(), identifier);
  }

  StructType idStructTy;

  if (!identifier.empty())
    idStructTy = StructType::getIdentified(dialect.getContext(), identifier);

  SmallVector<Type, 4> memberTypes;
  SmallVector<StructType::OffsetInfo, 4> offsetInfo;
  SmallVector<StructType::MemberDecorationInfo, 4> memberDecorationInfo;

```
- **EN**: Implements logic around `failed`, `Type`, `succeeded`, `getEmpty`, and 2 more symbols; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `failed`, `Type`, `succeeded`, `getEmpty`, and 2 more symbols 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性。

### Lines 773-789
```cpp
  do {
    Type memberType;
    if (parser.parseType(memberType))
      return Type();
    if (!isa<SPIRVType>(memberType)) {
      parser.emitError(parser.getNameLoc(),
                       "member type must be a valid SPIR-V type");
      return Type();
    }
    memberTypes.push_back(memberType);

    if (succeeded(parser.parseOptionalLSquare()))
      if (parseStructMemberDecorations(dialect, parser, memberTypes, offsetInfo,
                                       memberDecorationInfo))
        return Type();
  } while (succeeded(parser.parseOptionalComma()));

```
- **EN**: Implements logic around `parseType`, `Type`, `emitError`, `push_back`, and 2 more symbols; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `parseType`, `Type`, `emitError`, `push_back`, and 2 more symbols 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性。

### Lines 790-806
```cpp
  if (!offsetInfo.empty() && memberTypes.size() != offsetInfo.size()) {
    parser.emitError(parser.getNameLoc(),
                     "offset specification must be given for all members");
    return Type();
  }

  if (failed(parser.parseRParen()))
    return Type();

  SmallVector<StructType::StructDecorationInfo, 1> structDecorationInfo;

  auto parseStructDecoration = [&]() {
    std::optional<spirv::Decoration> decoration =
        parseAndVerify<spirv::Decoration>(dialect, parser);
    if (!decoration)
      return failure();

```
- **EN**: Implements logic around `empty`, `emitError`, `Type`, `failed`, and 2 more symbols; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `empty`, `emitError`, `Type`, `failed`, and 2 more symbols 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义。

### Lines 807-820
```cpp
    // Parse decoration value if it exists.
    if (succeeded(parser.parseOptionalEqual())) {
      Attribute decorationValue;
      if (failed(parser.parseAttribute(decorationValue)))
        return failure();

      structDecorationInfo.emplace_back(decoration.value(), decorationValue);
    } else {
      structDecorationInfo.emplace_back(decoration.value(),
                                        UnitAttr::get(dialect.getContext()));
    }
    return success();
  };

```
- **EN**: Implements logic around `succeeded`, `failed`, `failure`, `emplace_back`, and 2 more symbols; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `succeeded`, `failed`, `failure`, `emplace_back`, and 2 more symbols 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性。

### Lines 821-835
```cpp
  while (succeeded(parser.parseOptionalComma()))
    if (failed(parseStructDecoration()))
      return Type();

  if (failed(parser.parseGreater()))
    return Type();

  if (!identifier.empty()) {
    if (failed(idStructTy.trySetBody(memberTypes, offsetInfo,
                                     memberDecorationInfo,
                                     structDecorationInfo)))
      return Type();
    return idStructTy;
  }

```
- **EN**: Implements logic around `succeeded`, `failed`, `Type`, `empty`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `succeeded`, `failed`, `Type`, `empty` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性。

### Lines 836-851
```cpp
  return StructType::get(memberTypes, offsetInfo, memberDecorationInfo,
                         structDecorationInfo);
}

// spirv-type ::= array-type
//              | element-type
//              | image-type
//              | pointer-type
//              | runtime-array-type
//              | sampled-image-type
//              | struct-type
Type SPIRVDialect::parseType(DialectAsmParser &parser) const {
  StringRef keyword;
  if (parser.parseKeyword(&keyword))
    return Type();

```
- **EN**: Implements logic around `get`, `parseType`, `parseKeyword`, `Type`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `get`, `parseType`, `parseKeyword`, `Type` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义。

### Lines 852-877
```cpp
  if (keyword == "array")
    return parseArrayType(*this, parser);
  if (keyword == "coopmatrix")
    return parseCooperativeMatrixType(*this, parser);
  if (keyword == "image")
    return parseImageType(*this, parser);
  if (keyword == "ptr")
    return parsePointerType(*this, parser);
  if (keyword == "rtarray")
    return parseRuntimeArrayType(*this, parser);
  if (keyword == "sampled_image")
    return parseSampledImageType(*this, parser);
  if (keyword == "sampler")
    return SamplerType::get(getContext());
  if (keyword == "named_barrier")
    return NamedBarrierType::get(getContext());
  if (keyword == "struct")
    return parseStructType(*this, parser);
  if (keyword == "matrix")
    return parseMatrixType(*this, parser);
  if (keyword == "arm.tensor")
    return parseTensorArmType(*this, parser);
  parser.emitError(parser.getNameLoc(), "unknown SPIR-V type: ") << keyword;
  return Type();
}

```
- **EN**: Implements logic around `parseArrayType`, `parseCooperativeMatrixType`, `parseImageType`, `parsePointerType`, and 8 more symbols; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `parseArrayType`, `parseCooperativeMatrixType`, `parseImageType`, `parsePointerType`, and 8 more symbols 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 878-895
```cpp
//===----------------------------------------------------------------------===//
// Type Printing
//===----------------------------------------------------------------------===//

static void print(ArrayType type, DialectAsmPrinter &os) {
  os << "array<" << type.getNumElements() << " x " << type.getElementType();
  if (unsigned stride = type.getArrayStride())
    os << ", stride=" << stride;
  os << ">";
}

static void print(RuntimeArrayType type, DialectAsmPrinter &os) {
  os << "rtarray<" << type.getElementType();
  if (unsigned stride = type.getArrayStride())
    os << ", stride=" << stride;
  os << ">";
}

```
- **EN**: Implements logic around `print`, `getNumElements`, `getArrayStride`, `getElementType`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `print`, `getNumElements`, `getArrayStride`, `getElementType` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性。

### Lines 896-909
```cpp
static void print(PointerType type, DialectAsmPrinter &os) {
  os << "ptr<" << type.getPointeeType() << ", "
     << stringifyStorageClass(type.getStorageClass()) << ">";
}

static void print(ImageType type, DialectAsmPrinter &os) {
  os << "image<" << type.getElementType() << ", " << stringifyDim(type.getDim())
     << ", " << stringifyImageDepthInfo(type.getDepthInfo()) << ", "
     << stringifyImageArrayedInfo(type.getArrayedInfo()) << ", "
     << stringifyImageSamplingInfo(type.getSamplingInfo()) << ", "
     << stringifyImageSamplerUseInfo(type.getSamplerUseInfo()) << ", "
     << stringifyImageFormat(type.getImageFormat()) << ">";
}

```
- **EN**: Implements logic around `print`, `getPointeeType`, `stringifyStorageClass`, `getElementType`, and 5 more symbols; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `print`, `getPointeeType`, `stringifyStorageClass`, `getElementType`, and 5 more symbols 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性。

### Lines 910-924
```cpp
static void print(SampledImageType type, DialectAsmPrinter &os) {
  os << "sampled_image<" << type.getImageType() << ">";
}

static void print(SamplerType type, DialectAsmPrinter &os) { os << "sampler"; }

static void print(NamedBarrierType type, DialectAsmPrinter &os) {
  os << "named_barrier";
}

static void print(StructType type, DialectAsmPrinter &os) {
  FailureOr<AsmPrinter::CyclicPrintReset> cyclicPrint;

  os << "struct<";

```
- **EN**: Implements logic around `print`, `getImageType`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `print`, `getImageType` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性。

### Lines 925-938
```cpp
  if (type.isIdentified()) {
    os << type.getIdentifier();

    cyclicPrint = os.tryStartCyclicPrint(type);
    if (failed(cyclicPrint)) {
      os << ">";
      return;
    }

    os << ", ";
  }

  os << "(";

```
- **EN**: Implements logic around `isIdentified`, `getIdentifier`, `tryStartCyclicPrint`, `failed`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `isIdentified`, `getIdentifier`, `tryStartCyclicPrint`, `failed` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 939-964
```cpp
  auto printMember = [&](unsigned i) {
    os << type.getElementType(i);
    SmallVector<spirv::StructType::MemberDecorationInfo, 0> decorations;
    type.getMemberDecorations(i, decorations);
    if (type.hasOffset() || !decorations.empty()) {
      os << " [";
      if (type.hasOffset()) {
        os << type.getMemberOffset(i);
        if (!decorations.empty())
          os << ", ";
      }
      auto eachFn = [&os](spirv::StructType::MemberDecorationInfo decoration) {
        os << stringifyDecoration(decoration.decoration);
        if (decoration.hasValue()) {
          os << "=";
          os.printAttributeWithoutType(decoration.decorationValue);
        }
      };
      llvm::interleaveComma(decorations, os, eachFn);
      os << "]";
    }
  };
  llvm::interleaveComma(llvm::seq<unsigned>(0, type.getNumElements()), os,
                        printMember);
  os << ")";

```
- **EN**: Implements logic around `getElementType`, `getMemberDecorations`, `hasOffset`, `getMemberOffset`, and 5 more symbols; this block works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `getElementType`, `getMemberDecorations`, `hasOffset`, `getMemberOffset`, and 5 more symbols 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义。

### Lines 965-978
```cpp
  SmallVector<spirv::StructType::StructDecorationInfo, 1> decorations;
  type.getStructDecorations(decorations);
  if (!decorations.empty()) {
    os << ", ";
    auto eachFn = [&os](spirv::StructType::StructDecorationInfo decoration) {
      os << stringifyDecoration(decoration.decoration);
      if (decoration.hasValue()) {
        os << "=";
        os.printAttributeWithoutType(decoration.decorationValue);
      }
    };
    llvm::interleaveComma(decorations, os, eachFn);
  }

```
- **EN**: Implements logic around `getStructDecorations`, `empty`, `stringifyDecoration`, `hasValue`, and 2 more symbols; this block works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `getStructDecorations`, `empty`, `stringifyDecoration`, `hasValue`, and 2 more symbols 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义。

### Lines 979-992
```cpp
  os << ">";
}

static void print(CooperativeMatrixType type, DialectAsmPrinter &os) {
  os << "coopmatrix<" << type.getRows() << "x" << type.getColumns() << "x"
     << type.getElementType() << ", " << type.getScope() << ", "
     << type.getUse() << ">";
}

static void print(MatrixType type, DialectAsmPrinter &os) {
  os << "matrix<" << type.getNumColumns() << " x " << type.getColumnType();
  os << ">";
}

```
- **EN**: Implements logic around `print`, `getRows`, `getElementType`, `getUse`, and 1 more symbols; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `print`, `getRows`, `getElementType`, `getUse`, and 1 more symbols 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性。

### Lines 993-1010
```cpp
static void print(TensorArmType type, DialectAsmPrinter &os) {
  os << "arm.tensor<";

  llvm::interleave(
      type.getShape(), os,
      [&](int64_t dim) {
        if (ShapedType::isDynamic(dim))
          os << '?';
        else
          os << dim;
      },
      "x");
  if (!type.hasRank()) {
    os << "*";
  }
  os << "x" << type.getElementType() << ">";
}

```
- **EN**: Implements logic around `print`, `interleave`, `getShape`, `isDynamic`, and 2 more symbols; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `print`, `interleave`, `getShape`, `isDynamic`, and 2 more symbols 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 1011-1029
```cpp
void SPIRVDialect::printType(Type type, DialectAsmPrinter &os) const {
  TypeSwitch<Type>(type)
      .Case<ArrayType, CooperativeMatrixType, PointerType, RuntimeArrayType,
            ImageType, SampledImageType, SamplerType, NamedBarrierType,
            StructType, MatrixType, TensorArmType>(
          [&](auto type) { print(type, os); })
      .DefaultUnreachable("Unhandled SPIR-V type");
}

//===----------------------------------------------------------------------===//
// Constant
//===----------------------------------------------------------------------===//

Operation *SPIRVDialect::materializeConstant(OpBuilder &builder,
                                             Attribute value, Type type,
                                             Location loc) {
  if (auto poison = dyn_cast<ub::PoisonAttr>(value))
    return ub::PoisonOp::create(builder, loc, type, poison);

```
- **EN**: Implements logic around `printType`, `TypeSwitch`, `TensorArmType>`, `print`, and 4 more symbols; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `printType`, `TypeSwitch`, `TensorArmType>`, `print`, and 4 more symbols 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性。

### Lines 1030-1044
```cpp
  if (!spirv::ConstantOp::isBuildableWith(type))
    return nullptr;

  return spirv::ConstantOp::create(builder, loc, type, value);
}

//===----------------------------------------------------------------------===//
// Shader Interface ABI
//===----------------------------------------------------------------------===//

LogicalResult SPIRVDialect::verifyOperationAttribute(Operation *op,
                                                     NamedAttribute attribute) {
  StringRef symbol = attribute.getName().strref();
  Attribute attr = attribute.getValue();

```
- **EN**: Implements logic around `isBuildableWith`, `create`, `verifyOperationAttribute`, `getName`, and 1 more symbols; this block expresses reusable interface-based behavior; works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `isBuildableWith`, `create`, `verifyOperationAttribute`, `getName`, and 1 more symbols 实现具体逻辑；该代码块表达基于接口的可复用行为，并处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义。

### Lines 1045-1065
```cpp
  if (symbol == spirv::getEntryPointABIAttrName()) {
    if (!isa<spirv::EntryPointABIAttr>(attr)) {
      return op->emitError("'")
             << symbol << "' attribute must be an entry point ABI attribute";
    }
  } else if (symbol == spirv::getTargetEnvAttrName()) {
    if (!isa<spirv::TargetEnvAttr>(attr))
      return op->emitError("'") << symbol << "' must be a spirv::TargetEnvAttr";
  } else if (symbol == spirv::getLoopControlAttrName()) {
    if (!isa<spirv::LoopControlAttr>(attr))
      return op->emitError("'")
             << symbol << "' must be a spirv::LoopControlAttr";
  } else if (symbol == spirv::getSelectionControlAttrName()) {
    if (!isa<spirv::SelectionControlAttr>(attr))
      return op->emitError("'")
             << symbol << "' must be a spirv::SelectionControlAttr";
  } else {
    return op->emitError("found unsupported '")
           << symbol << "' attribute on operation";
  }

```
- **EN**: Implements logic around `getEntryPointABIAttrName`, `EntryPointABIAttr>`, `emitError`, `getTargetEnvAttrName`, and 5 more symbols; this block works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `getEntryPointABIAttrName`, `EntryPointABIAttr>`, `emitError`, `getTargetEnvAttrName`, and 5 more symbols 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义。

### Lines 1066-1081
```cpp
  return success();
}

/// Verifies the given SPIR-V `attribute` attached to a value of the given
/// `valueType` is valid.
static LogicalResult verifyRegionAttribute(Location loc, Type valueType,
                                           NamedAttribute attribute) {
  StringRef symbol = attribute.getName().strref();
  Attribute attr = attribute.getValue();

  if (symbol == spirv::getInterfaceVarABIAttrName()) {
    auto varABIAttr = dyn_cast<spirv::InterfaceVarABIAttr>(attr);
    if (!varABIAttr)
      return emitError(loc, "'")
             << symbol << "' must be a spirv::InterfaceVarABIAttr";

```
- **EN**: Implements logic around `success`, `verifyRegionAttribute`, `getName`, `getValue`, and 3 more symbols; this block works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `success`, `verifyRegionAttribute`, `getName`, `getValue`, and 3 more symbols 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义。

### Lines 1082-1098
```cpp
    if (varABIAttr.getStorageClass() && !valueType.isIntOrIndexOrFloat())
      return emitError(loc, "'") << symbol
                                 << "' attribute cannot specify storage class "
                                    "when attaching to a non-scalar value";
    return success();
  }
  if (symbol == spirv::DecorationAttr::name) {
    if (!isa<spirv::DecorationAttr>(attr))
      return emitError(loc, "'")
             << symbol << "' must be a spirv::DecorationAttr";
    return success();
  }

  return emitError(loc, "found unsupported '")
         << symbol << "' attribute on region argument";
}

```
- **EN**: Implements logic around `getStorageClass`, `emitError`, `success`, `DecorationAttr>`; this block works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `getStorageClass`, `emitError`, `success`, `DecorationAttr>` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义。

### Lines 1099-1120
```cpp
LogicalResult SPIRVDialect::verifyRegionArgAttribute(Operation *op,
                                                     unsigned regionIndex,
                                                     unsigned argIndex,
                                                     NamedAttribute attribute) {
  auto funcOp = dyn_cast<FunctionOpInterface>(op);
  if (!funcOp)
    return success();
  Type argType = funcOp.getArgumentTypes()[argIndex];

  return verifyRegionAttribute(op->getLoc(), argType, attribute);
}

LogicalResult SPIRVDialect::verifyRegionResultAttribute(
    Operation *op, unsigned /*regionIndex*/, unsigned resultIndex,
    NamedAttribute attribute) {
  if (auto graphOp = dyn_cast<spirv::GraphARMOp>(op))
    return verifyRegionAttribute(
        op->getLoc(), graphOp.getResultTypes()[resultIndex], attribute);
  return op->emitError(
      "cannot attach SPIR-V attributes to region result which is "
      "not part of a spirv::GraphARMOp type");
}
```
- **EN**: Implements logic around `verifyRegionArgAttribute`, `success`, `getArgumentTypes`, `verifyRegionAttribute`, and 4 more symbols; this block works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `verifyRegionArgAttribute`, `success`, `getArgumentTypes`, `verifyRegionAttribute`, and 4 more symbols 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义。

## Key Concepts / 关键概念

- **Dialect IR definitions / 方言 IR 定义**:
  - **EN**: Defines or implements dialect operations, attributes, types, traits, and registration hooks.
  - **CN**: 定义或实现方言操作、属性、类型、trait 以及注册钩子。
- **Textual assembly syntax / 文本汇编语法**:
  - **EN**: Implements dialect-specific parsing and printing hooks for MLIR assembly.
  - **CN**: 实现 MLIR 汇编的方言专用解析与打印钩子。
- **Interface-based extensibility / 基于接口的可扩展性**:
  - **EN**: Models reusable capabilities that can be queried across dialect boundaries.
  - **CN**: 建模可跨方言查询的可复用能力。
- **Structured IR coordination / 结构化 IR 协同**:
  - **EN**: Interacts with structured MLIR dialects that model loops, tensors, memory, and vector semantics.
  - **CN**: 与建模循环、张量、内存和向量语义的结构化 MLIR 方言交互。
- **Target-specific semantics / 目标相关语义**:
  - **EN**: Encodes rules tied to accelerator, GPU, or binary target environments.
  - **CN**: 编码与加速器、GPU 或二进制目标环境绑定的规则。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/SPIRV/IR/SPIRVDialect.h`, `SPIRVParsingUtils.h`, `mlir/Dialect/GPU/IR/CompilationInterfaces.h`, `mlir/Dialect/SPIRV/IR/SPIRVOps.h`, `mlir/Dialect/SPIRV/IR/SPIRVTypes.h`, `mlir/Dialect/SPIRV/IR/TargetAndABI.h`, `mlir/Dialect/UB/IR/UBOps.h`, `mlir/IR/Builders.h`, `mlir/IR/BuiltinTypes.h`, `mlir/IR/DialectImplementation.h` ... (+8 more)
- **Subsystem categories / 子系统类别**: dialect-specific operation, type, attribute, or transform declarations / 方言相关的操作、类型、属性或变换声明 (8), MLIR core IR types and infrastructural utilities / MLIR 核心 IR 类型与基础设施工具 (4), LLVM ADT containers and low-level helpers / LLVM ADT 容器与底层辅助工具 (3), parsing utilities and textual IR loading support / 解析工具与文本 IR 加载支持 (1), generic transformation and conversion helpers / 通用变换与转换辅助逻辑 (1)
- **Generated macros / 生成宏**: `GET_OP_LIST`
