# Utils.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/ArmSME/IR/Utils.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This file implements utilities for the ArmSME dialect.
  - **CN**: 实现 ArmSME 方言与 Arm SME tile/vector 支持 的核心 IR、操作、类型、属性或接口。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- Utils.cpp - Utilities to support the ArmSME dialect ----------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件的高层描述。

### Lines 8-14
```cpp
//
// This file implements utilities for the ArmSME dialect.
//
//===----------------------------------------------------------------------===//

#include "mlir/Dialect/ArmSME/Utils/Utils.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/ArmSME/Utils/Utils.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/ArmSME/Utils/Utils.h`。

### Lines 15-28
```cpp
namespace mlir::arm_sme {

unsigned getSizeInBytes(TypeSize type) {
  switch (type) {
  case arm_sme::TypeSize::Byte:
    return 1;
  case arm_sme::TypeSize::Half:
    return 2;
  case arm_sme::TypeSize::Word:
    return 4;
  case arm_sme::TypeSize::Double:
    return 8;
  }
  llvm_unreachable("unknown type size");
```
- **EN**: Introduces declarations for `mlir::arm_sme`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `mlir::arm_sme` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 29-36
```cpp
  return 0;
}

unsigned getSMETileSliceMinNumElts(Type type) {
  assert(isValidSMETileElementType(type) && "invalid tile type!");
  return MinStreamingVectorLengthInBits / type.getIntOrFloatBitWidth();
}

```
- **EN**: Implements logic around `getSMETileSliceMinNumElts`, `assert`, `getIntOrFloatBitWidth`.
- **CN**: 围绕 `getSMETileSliceMinNumElts`, `assert`, `getIntOrFloatBitWidth` 实现具体逻辑。

### Lines 37-46
```cpp
bool isValidSMETileElementType(Type type) {
  return type.isInteger(8) || type.isInteger(16) || type.isInteger(32) ||
         type.isInteger(64) || type.isInteger(128) || type.isF16() ||
         type.isBF16() || type.isF32() || type.isF64() || type.isF128();
}

bool isValidSMETileVectorType(VectorType vType) {
  if ((vType.getRank() != 2) || !vType.allDimsScalable())
    return false;

```
- **EN**: Implements logic around `isValidSMETileElementType`, `isInteger`, `isBF16`, `isValidSMETileVectorType`, and 1 more symbols.
- **CN**: 围绕 `isValidSMETileElementType`, `isInteger`, `isBF16`, `isValidSMETileVectorType`, and 1 more symbols 实现具体逻辑。

### Lines 47-54
```cpp
  auto elemType = vType.getElementType();
  if (!isValidSMETileElementType(elemType))
    return false;

  unsigned minNumElts = getSMETileSliceMinNumElts(elemType);
  return vType.getShape() == ArrayRef<int64_t>({minNumElts, minNumElts});
}

```
- **EN**: Implements logic around `getElementType`, `isValidSMETileElementType`, `getSMETileSliceMinNumElts`, `getShape`.
- **CN**: 围绕 `getElementType`, `isValidSMETileElementType`, `getSMETileSliceMinNumElts`, `getShape` 实现具体逻辑。

### Lines 55-68
```cpp
std::optional<ArmSMETileType> getSMETileType(VectorType type) {
  if (!isValidSMETileVectorType(type))
    return {};
  switch (type.getElementTypeBitWidth()) {
  case 8:
    return ArmSMETileType::ZAB;
  case 16:
    return ArmSMETileType::ZAH;
  case 32:
    return ArmSMETileType::ZAS;
  case 64:
    return ArmSMETileType::ZAD;
  case 128:
    return ArmSMETileType::ZAQ;
```
- **EN**: Implements logic around `getSMETileType`, `isValidSMETileVectorType`, `getElementTypeBitWidth`.
- **CN**: 围绕 `getSMETileType`, `isValidSMETileVectorType`, `getElementTypeBitWidth` 实现具体逻辑。

### Lines 69-82
```cpp
  default:
    llvm_unreachable("unknown SME tile type");
  }
}

LogicalResult verifyOperationHasValidTileId(Operation *op) {
  auto tileOp = llvm::dyn_cast<ArmSMETileOpInterface>(op);
  if (!tileOp)
    return success(); // Not a tile op (no need to check).
  auto tileId = tileOp.getTileId();
  if (!tileId)
    return success(); // Not having a tile ID (yet) is okay.
  if (!tileId.getType().isSignlessInteger(32))
    return tileOp.emitOpError("tile ID should be a 32-bit signless integer");
```
- **EN**: Implements logic around `llvm_unreachable`, `verifyOperationHasValidTileId`, `dyn_cast`, `success`, and 3 more symbols.
- **CN**: 围绕 `llvm_unreachable`, `verifyOperationHasValidTileId`, `dyn_cast`, `success`, and 3 more symbols 实现具体逻辑。

### Lines 83-96
```cpp
  return success();
}

scf::ForOp createLoopOverTileSlices(
    PatternRewriter &rewriter, Location loc, Value initTile,
    std::function<Value(OpBuilder &, Location, Value, Value)> makeLoopBody) {
  OpBuilder::InsertionGuard g(rewriter);
  auto step = arith::ConstantIndexOp::create(rewriter, loc, 1);
  auto minTileSlices = arith::ConstantIndexOp::create(
      rewriter, loc, llvm::cast<VectorType>(initTile.getType()).getDimSize(0));
  auto vscale =
      vector::VectorScaleOp::create(rewriter, loc, rewriter.getIndexType());
  auto lowerBound = arith::ConstantIndexOp::create(rewriter, loc, 0);
  auto numTileSlices =
```
- **EN**: Implements logic around `success`, `createLoopOverTileSlices`, `function`, `g`, and 2 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `success`, `createLoopOverTileSlices`, `function`, `g`, and 2 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 97-107
```cpp
      arith::MulIOp::create(rewriter, loc, minTileSlices, vscale);
  auto forOp = scf::ForOp::create(rewriter, loc, lowerBound, numTileSlices,
                                  step, ValueRange{initTile});
  rewriter.setInsertionPointToStart(forOp.getBody());
  Value nextTile =
      makeLoopBody(rewriter, loc, /*tileSliceIndex=*/forOp.getInductionVar(),
                   /*currentTile=*/forOp.getRegionIterArg(0));
  scf::YieldOp::create(rewriter, loc, nextTile);
  return forOp;
}

```
- **EN**: Implements logic around `create`, `setInsertionPointToStart`, `makeLoopBody`, `getRegionIterArg`; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `create`, `setInsertionPointToStart`, `makeLoopBody`, `getRegionIterArg` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 108-115
```cpp
bool isMultipleOfSMETileVectorType(VectorType vType) {
  if (vType.getRank() != 2 || !vType.allDimsScalable())
    return false;

  auto elementType = vType.getElementType();
  if (!isValidSMETileElementType(elementType))
    return false;

```
- **EN**: Implements logic around `isMultipleOfSMETileVectorType`, `getRank`, `getElementType`, `isValidSMETileElementType`.
- **CN**: 围绕 `isMultipleOfSMETileVectorType`, `getRank`, `getElementType`, `isValidSMETileElementType` 实现具体逻辑。

### Lines 116-124
```cpp
  unsigned minNumElts = getSMETileSliceMinNumElts(elementType);

  int64_t vectorRows = vType.getDimSize(0);
  int64_t vectorCols = vType.getDimSize(1);

  return (vectorRows > minNumElts || vectorCols > minNumElts) &&
         vectorRows % minNumElts == 0 && vectorCols % minNumElts == 0;
}

```
- **EN**: Implements logic around `getSMETileSliceMinNumElts`, `getDimSize`.
- **CN**: 围绕 `getSMETileSliceMinNumElts`, `getDimSize` 实现具体逻辑。

### Lines 125-138
```cpp
VectorType getSMETileTypeForElement(Type elementType) {
  unsigned minNumElts = getSMETileSliceMinNumElts(elementType);
  return VectorType::get({minNumElts, minNumElts}, elementType, {true, true});
}

void eraseTriviallyDeadTileOps(IRRewriter &rewriter,
                               FunctionOpInterface function) {
  SmallVector<Operation *> worklist;
  function->walk([&](Operation *op) {
    auto armSMEOp = dyn_cast<arm_sme::ArmSMETileOpInterface>(op);
    if (armSMEOp && isOpTriviallyDead(armSMEOp))
      worklist.push_back(armSMEOp);
  });
  while (!worklist.empty()) {
```
- **EN**: Implements logic around `getSMETileTypeForElement`, `getSMETileSliceMinNumElts`, `get`, `eraseTriviallyDeadTileOps`, and 5 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `getSMETileTypeForElement`, `getSMETileSliceMinNumElts`, `get`, `eraseTriviallyDeadTileOps`, and 5 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 139-149
```cpp
    Operation *op = worklist.pop_back_val();
    if (!isOpTriviallyDead(op))
      continue;
    for (Value value : op->getOperands()) {
      if (auto armSMEOp = value.getDefiningOp<arm_sme::ArmSMETileOpInterface>())
        worklist.push_back(armSMEOp);
    }
    rewriter.eraseOp(op);
  }
}

```
- **EN**: Implements logic around `pop_back_val`, `isOpTriviallyDead`, `getOperands`, `ArmSMETileOpInterface>`, and 2 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `pop_back_val`, `isOpTriviallyDead`, `getOperands`, `ArmSMETileOpInterface>`, and 2 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 150-162
```cpp
bool isTriviallyCloneableTileOp(arm_sme::ArmSMETileOpInterface tileOp) {
  return tileOp && tileOp->getNumResults() == 1 &&
         tileOp->getNumOperands() == 0 && isPure(tileOp);
}

bool hasTileResult(arm_sme::ArmSMETileOpInterface tileOp) {
  for (Value result : tileOp->getResults()) {
    if (arm_sme::isValidSMETileVectorType(result.getType()))
      return true;
  }
  return false;
}

```
- **EN**: Implements logic around `isTriviallyCloneableTileOp`, `getNumResults`, `getNumOperands`, `hasTileResult`, and 2 more symbols.
- **CN**: 围绕 `isTriviallyCloneableTileOp`, `getNumResults`, `getNumOperands`, `hasTileResult`, and 2 more symbols 实现具体逻辑。

### Lines 163-176
```cpp
OpOperand *getTileOpOperand(arm_sme::ArmSMETileOpInterface tileOp) {
  if (!tileOp)
    return nullptr;
  auto isTileOperandType = [](OpOperand &operand) {
    return arm_sme::isValidSMETileVectorType(operand.get().getType());
  };
  assert(llvm::count_if(tileOp->getOpOperands(), isTileOperandType) <= 1 &&
         "expected at most one tile operand");
  OpOperand *tileOperand =
      llvm::find_if(tileOp->getOpOperands(), isTileOperandType);
  if (tileOperand == tileOp->getOpOperands().end())
    return nullptr;
  return tileOperand;
}
```
- **EN**: Implements logic around `getTileOpOperand`, `isValidSMETileVectorType`, `assert`, `find_if`, and 1 more symbols.
- **CN**: 围绕 `getTileOpOperand`, `isValidSMETileVectorType`, `assert`, `find_if`, and 1 more symbols 实现具体逻辑。

### Lines 177-183
```cpp

bool isTileTypeGreaterOrEqual(ArmSMETileType typeA, ArmSMETileType typeB) {
  // Note: This is <= due to how tile types are numbered in ArmSMEOps.td.
  return static_cast<unsigned>(typeA) <= static_cast<unsigned>(typeB);
}

} // namespace mlir::arm_sme
```
- **EN**: Implements logic around `isTileTypeGreaterOrEqual`, `static_cast`.
- **CN**: 围绕 `isTileTypeGreaterOrEqual`, `static_cast` 实现具体逻辑。

## Key Concepts / 关键概念

- **Dialect IR modeling / 方言 IR 建模**:
  - **EN**: Defines operations, attributes, types, verifiers, parsers, and printers for a dialect.
  - **CN**: 定义方言的操作、属性、类型、验证器、解析器与打印器。
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

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/ArmSME/Utils/Utils.h`
- **Subsystem categories / 子系统类别**: other MLIR dialect declarations / 其他 MLIR 方言声明 (1)
