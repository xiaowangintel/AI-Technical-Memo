# AMDGPUOps.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/AMDGPU/IR/AMDGPUOps.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This file implements the AMDGPU dialect operations, their verifiers, and their canonicalizations.
  - **CN**: 实现 AMDGPU 方言与目标专用 GPU 支持 的核心 IR、操作、类型、属性或接口。

## Line-by-Line Analysis / 逐行分析

### Lines 1-32
```cpp
//===- AMDGPUOps.cpp - MLIR AMDGPU dialect operations ---------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements the AMDGPU dialect operations, their verifiers, and
// their canonicalizations.
//
//===----------------------------------------------------------------------===//

#include "mlir/Dialect/AMDGPU/IR/AMDGPUDialect.h"

#include "mlir/Dialect/Arith/IR/Arith.h"
#include "mlir/Dialect/GPU/IR/GPUDialect.h"
#include "mlir/Dialect/LLVMIR/ROCDLDialect.h"
#include "mlir/Dialect/MemRef/IR/MemRef.h"
#include "mlir/Dialect/MemRef/Utils/MemRefUtils.h"
#include "mlir/Dialect/Utils/IndexingUtils.h"
#include "mlir/Dialect/Vector/IR/VectorOps.h"
#include "mlir/IR/Builders.h"
#include "mlir/IR/BuiltinTypes.h"
#include "mlir/IR/Diagnostics.h"
#include "mlir/IR/Matchers.h"
#include "mlir/IR/OpImplementation.h"
#include "mlir/IR/PatternMatch.h"
#include "mlir/IR/TypeUtilities.h"
#include "llvm/ADT/DenseMap.h"
#include "llvm/ADT/SmallVector.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/AMDGPU/IR/AMDGPUDialect.h`, `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/GPU/IR/GPUDialect.h`, `mlir/Dialect/LLVMIR/ROCDLDialect.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/AMDGPU/IR/AMDGPUDialect.h`, `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/GPU/IR/GPUDialect.h`, `mlir/Dialect/LLVMIR/ROCDLDialect.h`。

### Lines 33-53
```cpp
#include <algorithm>
#include <cstdint>
#include <limits>
#include <optional>

using namespace mlir;
using namespace mlir::amdgpu;

/// Verifies that the number of indices matches the rank of the indexed memref,
/// emitting an op error mentioning `indexName` on mismatch.
template <typename OpTy>
static LogicalResult verifyIndexCount(OpTy op, StringRef indexName,
                                      MemRefType memrefType,
                                      int64_t numIndices) {
  int64_t rank = memrefType.getRank();
  if (rank != numIndices)
    return op.emitOpError("expected ")
           << rank << " " << indexName << " indices, got " << numIndices;
  return success();
}

```
- **EN**: Implements logic around `verifyIndexCount`, `getRank`, `emitOpError`, `success`; this block moves data between tensor-style values and explicit buffers; encodes accelerator-specific execution or lowering rules.
- **CN**: 围绕 `verifyIndexCount`, `getRank`, `emitOpError`, `success` 实现具体逻辑；该代码块在张量风格值与显式缓冲区之间移动数据，并编码加速器专用执行或 lowering 规则。

### Lines 54-71
```cpp
//===----------------------------------------------------------------------===//
// 8-bit float ops
//===----------------------------------------------------------------------===//
LogicalResult PackedTrunc2xFp8Op::verify() {
  if (getExisting() && getExisting().getType() != getResult().getType())
    return emitOpError("existing values must have same type as result");
  return success();
}

LogicalResult PackedStochRoundFp8Op::verify() {
  if (getExisting() && getExisting().getType() != getResult().getType())
    return emitOpError("existing values must have same type as result");
  return success();
}

//===----------------------------------------------------------------------===//
// mxfp float ops
//===----------------------------------------------------------------------===//
```
- **EN**: Implements logic around `verify`, `getExisting`, `emitOpError`, `success`; this block implements verifier, folding, parsing, or printing hooks.
- **CN**: 围绕 `verify`, `getExisting`, `emitOpError`, `success` 实现具体逻辑；该代码块实现验证、折叠、解析或打印钩子。

### Lines 72-107
```cpp
LogicalResult PackedScaledTruncOp::verify() {
  if (getExisting() && getExisting().getType() != getResult().getType())
    return emitOpError("existing values must have same type as result");
  return success();
}

//===----------------------------------------------------------------------===//
// FatRawBufferCastOp
//===----------------------------------------------------------------------===//

/// Convert the type `source` to one with the same sizes and strides - and
/// offset, unless `stripOffset` is true, in which case the offset is reset to
/// 0, if the offset should be reset but the layout of `source` isn't either the
/// identity layout or a strided layout, this function fails.
static FailureOr<MemRefType> getFatRawBufferTypeLike(MemRefType source,
                                                     bool resetOffset) {
  MLIRContext *ctx = source.getContext();
  MemRefType::Builder mb(source);
  mb.setMemorySpace(
      amdgpu::AddressSpaceAttr::get(ctx, amdgpu::AddressSpace::FatRawBuffer));
  MemRefLayoutAttrInterface layout = source.getLayout();
  if (resetOffset && !layout.isIdentity()) {
    auto stridedLayout = dyn_cast<StridedLayoutAttr>(layout);
    if (!stridedLayout)
      return failure();
    MemRefLayoutAttrInterface newLayout =
        StridedLayoutAttr::get(ctx, 0, stridedLayout.getStrides());
    // Special case: if resetting the offset causes the strided layout to become
    // the identity layout, then reset to the identity layout.
    // TODO: this'll get a lot simpler when we have the contiguous layout.
    SmallVector<int64_t> stridesIfIdentity;
    if (source.hasStaticShape()) {
      stridesIfIdentity = computeSuffixProduct(source.getShape());
    } else if (source.getRank() <= 1) {
      stridesIfIdentity = SmallVector<int64_t>(source.getRank(), 1);
    }
```
- **EN**: Implements logic around `verify`, `getExisting`, `emitOpError`, `success`, and 13 more symbols; this block implements verifier, folding, parsing, or printing hooks; encodes accelerator-specific execution or lowering rules.
- **CN**: 围绕 `verify`, `getExisting`, `emitOpError`, `success`, and 13 more symbols 实现具体逻辑；该代码块实现验证、折叠、解析或打印钩子，并编码加速器专用执行或 lowering 规则。

### Lines 108-133
```cpp
    if (stridesIfIdentity == stridedLayout.getStrides()) {
      newLayout = AffineMapAttr::get(
          AffineMap::getMultiDimIdentityMap(source.getRank(), ctx));
    }
    mb.setLayout(newLayout);
  }
  return (MemRefType)(mb);
}

LogicalResult FatRawBufferCastOp::inferReturnTypes(
    MLIRContext *context, std::optional<Location> location, ValueRange operands,
    DictionaryAttr attributes, PropertyRef properties, RegionRange regions,
    SmallVectorImpl<Type> &inferredReturnTypes) {
  Adaptor adaptor(operands, attributes, properties, regions);
  auto sourceType =
      dyn_cast_if_present<MemRefType>(adaptor.getSource().getType());
  if (!sourceType)
    return failure();
  FailureOr<MemRefType> resultType =
      getFatRawBufferTypeLike(sourceType, adaptor.getResetOffset());
  if (failed(resultType))
    return failure();
  inferredReturnTypes = SmallVector<Type>{*resultType};
  return success();
}

```
- **EN**: Implements logic around `getStrides`, `get`, `getMultiDimIdentityMap`, `setLayout`, and 7 more symbols; this block implements verifier, folding, parsing, or printing hooks; performs affine reasoning or shape/bounds manipulation.
- **CN**: 围绕 `getStrides`, `get`, `getMultiDimIdentityMap`, `setLayout`, and 7 more symbols 实现具体逻辑；该代码块实现验证、折叠、解析或打印钩子，并执行仿射推理或形状/边界处理。

### Lines 134-152
```cpp
FailureOr<OpFoldResult> FatRawBufferCastOp::reifyDimOfResult(OpBuilder &builder,
                                                             int resultIndex,
                                                             int dim) {
  assert(resultIndex == 0 && "FatRawBufferCastOp has a single result");
  return memref::getMixedSize(builder, getLoc(), getSource(), dim);
}

LogicalResult FatRawBufferCastOp::verify() {
  FailureOr<MemRefType> expectedResultType =
      getFatRawBufferTypeLike(getSource().getType(), getResetOffset());
  if (failed(expectedResultType))
    return emitOpError("source type ")
           << getSource().getType() << " can't have its offset reset";
  if (getResult().getType() != *expectedResultType)
    return emitOpError("expected result type to be ")
           << *expectedResultType << " but got " << getResult().getType();
  return success();
}

```
- **EN**: Implements logic around `reifyDimOfResult`, `assert`, `getMixedSize`, `verify`, and 6 more symbols; this block implements verifier, folding, parsing, or printing hooks; moves data between tensor-style values and explicit buffers.
- **CN**: 围绕 `reifyDimOfResult`, `assert`, `getMixedSize`, `verify`, and 6 more symbols 实现具体逻辑；该代码块实现验证、折叠、解析或打印钩子，并在张量风格值与显式缓冲区之间移动数据。

### Lines 153-172
```cpp
static bool hasGlobalMemorySpace(Attribute memorySpace) {
  if (!memorySpace)
    return true;
  if (auto intMemorySpace = dyn_cast<IntegerAttr>(memorySpace))
    return intMemorySpace.getInt() == 0 || intMemorySpace.getInt() == 1;
  if (auto gpuMemorySpace = dyn_cast<gpu::AddressSpaceAttr>(memorySpace))
    return gpuMemorySpace.getValue() == gpu::AddressSpace::Global;
  return false;
}

static bool hasWorkgroupMemorySpace(Attribute memorySpace) {
  if (!memorySpace)
    return false;
  if (auto intMemorySpace = dyn_cast<IntegerAttr>(memorySpace))
    return intMemorySpace.getInt() == 3;
  if (auto gpuMemorySpace = dyn_cast<gpu::AddressSpaceAttr>(memorySpace))
    return gpuMemorySpace.getValue() == gpu::AddressSpace::Workgroup;
  return false;
}

```
- **EN**: Implements logic around `hasGlobalMemorySpace`, `dyn_cast`, `getInt`, `AddressSpaceAttr>`, and 2 more symbols; this block encodes accelerator-specific execution or lowering rules.
- **CN**: 围绕 `hasGlobalMemorySpace`, `dyn_cast`, `getInt`, `AddressSpaceAttr>`, and 2 more symbols 实现具体逻辑；该代码块编码加速器专用执行或 lowering 规则。

### Lines 173-190
```cpp
static bool hasFatRawBufferMemorySpace(Attribute memorySpace) {
  if (!memorySpace)
    return false;
  if (auto intMemorySpace = dyn_cast<IntegerAttr>(memorySpace))
    return intMemorySpace.getInt() == 7;
  if (auto gpuMemorySpace = dyn_cast<amdgpu::AddressSpaceAttr>(memorySpace))
    return gpuMemorySpace.getValue() == amdgpu::AddressSpace::FatRawBuffer;
  return false;
}

//===----------------------------------------------------------------------===//
// RawBuffer*Op
//===----------------------------------------------------------------------===//
template <typename T>
static LogicalResult verifyRawBufferOp(T &op) {
  MemRefType bufferType = llvm::cast<MemRefType>(op.getMemref().getType());
  bool isGlobal = hasGlobalMemorySpace(bufferType.getMemorySpace());

```
- **EN**: Implements logic around `hasFatRawBufferMemorySpace`, `dyn_cast`, `getInt`, `AddressSpaceAttr>`, and 4 more symbols; this block encodes accelerator-specific execution or lowering rules.
- **CN**: 围绕 `hasFatRawBufferMemorySpace`, `dyn_cast`, `getInt`, `AddressSpaceAttr>`, and 4 more symbols 实现具体逻辑；该代码块编码加速器专用执行或 lowering 规则。

### Lines 191-211
```cpp
  if (!isGlobal)
    return op.emitOpError(
        "buffer ops must operate on a memref in global memory");
  if (!bufferType.hasRank())
    return op.emitOpError(
        "cannot meaningfully buffer_store to an unranked memref");
  return verifyIndexCount(op, "buffer", bufferType, op.getIndices().size());
}

LogicalResult RawBufferLoadOp::verify() { return verifyRawBufferOp(*this); }

LogicalResult RawBufferStoreOp::verify() { return verifyRawBufferOp(*this); }

LogicalResult RawBufferAtomicFaddOp::verify() {
  return verifyRawBufferOp(*this);
}

LogicalResult RawBufferAtomicFmaxOp::verify() {
  return verifyRawBufferOp(*this);
}

```
- **EN**: Implements logic around `emitOpError`, `hasRank`, `verifyIndexCount`, `verify`, and 1 more symbols; this block implements verifier, folding, parsing, or printing hooks; moves data between tensor-style values and explicit buffers.
- **CN**: 围绕 `emitOpError`, `hasRank`, `verifyIndexCount`, `verify`, and 1 more symbols 实现具体逻辑；该代码块实现验证、折叠、解析或打印钩子，并在张量风格值与显式缓冲区之间移动数据。

### Lines 212-232
```cpp
LogicalResult RawBufferAtomicSmaxOp::verify() {
  return verifyRawBufferOp(*this);
}

LogicalResult RawBufferAtomicUminOp::verify() {
  return verifyRawBufferOp(*this);
}

LogicalResult RawBufferAtomicCmpswapOp::verify() {
  return verifyRawBufferOp(*this);
}

static std::optional<uint32_t> getConstantUint32(Value v) {
  APInt cst;
  if (!v.getType().isInteger(32))
    return std::nullopt;
  if (matchPattern(v, m_ConstantInt(&cst)))
    return cst.getZExtValue();
  return std::nullopt;
}

```
- **EN**: Implements logic around `verify`, `verifyRawBufferOp`, `getConstantUint32`, `getType`, and 2 more symbols; this block implements verifier, folding, parsing, or printing hooks.
- **CN**: 围绕 `verify`, `verifyRawBufferOp`, `getConstantUint32`, `getType`, and 2 more symbols 实现具体逻辑；该代码块实现验证、折叠、解析或打印钩子。

### Lines 233-268
```cpp
template <typename OpType>
static bool staticallyOutOfBounds(OpType op) {
  if (!op.getBoundsCheck())
    return false;
  MemRefType bufferType = op.getMemref().getType();
  if (!bufferType.hasStaticShape())
    return false;
  int64_t offset;
  SmallVector<int64_t> strides;
  if (failed(bufferType.getStridesAndOffset(strides, offset)))
    return false;
  int64_t result = offset + op.getIndexOffset().value_or(0);
  if (op.getSgprOffset()) {
    std::optional<uint32_t> sgprOffset = getConstantUint32(op.getSgprOffset());
    if (!sgprOffset)
      return false;
    result += *sgprOffset;
  }
  if (strides.size() != op.getIndices().size())
    return false;
  int64_t indexVal = 0;
  for (auto pair : llvm::zip(strides, op.getIndices())) {
    int64_t stride = std::get<0>(pair);
    Value idx = std::get<1>(pair);
    std::optional<uint32_t> idxVal = getConstantUint32(idx);
    if (!idxVal)
      return false;
    indexVal += stride * *idxVal;
  }
  result += indexVal;
  if (result > std::numeric_limits<uint32_t>::max())
    // Overflow means don't drop
    return false;
  return result >= bufferType.getNumElements();
}

```
- **EN**: Implements logic around `staticallyOutOfBounds`, `getBoundsCheck`, `getMemref`, `hasStaticShape`, and 9 more symbols.
- **CN**: 围绕 `staticallyOutOfBounds`, `getBoundsCheck`, `getMemref`, `hasStaticShape`, and 9 more symbols 实现具体逻辑。

### Lines 269-287
```cpp
namespace {
template <typename OpType>
struct RemoveStaticallyOobBufferLoads final : public OpRewritePattern<OpType> {
  using OpRewritePattern<OpType>::OpRewritePattern;

  LogicalResult matchAndRewrite(OpType op, PatternRewriter &rw) const override {
    if (!staticallyOutOfBounds(op))
      return failure();
    Type loadType = op.getResult().getType();
    rw.replaceOpWithNewOp<arith::ConstantOp>(op, loadType,
                                             rw.getZeroAttr(loadType));
    return success();
  }
};

template <typename OpType>
struct RemoveStaticallyOobBufferWrites final : public OpRewritePattern<OpType> {
  using OpRewritePattern<OpType>::OpRewritePattern;

```
- **EN**: Introduces declarations for `RemoveStaticallyOobBufferLoads`, `RemoveStaticallyOobBufferWrites`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `RemoveStaticallyOobBufferLoads`, `RemoveStaticallyOobBufferWrites` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 288-307
```cpp
  LogicalResult matchAndRewrite(OpType op, PatternRewriter &rw) const override {
    if (!staticallyOutOfBounds(op))
      return failure();

    rw.eraseOp(op);
    return success();
  }
};
} // end namespace

void RawBufferLoadOp::getCanonicalizationPatterns(RewritePatternSet &results,
                                                  MLIRContext *context) {
  results.add<RemoveStaticallyOobBufferLoads<RawBufferLoadOp>>(context);
}

void RawBufferStoreOp::getCanonicalizationPatterns(RewritePatternSet &results,
                                                   MLIRContext *context) {
  results.add<RemoveStaticallyOobBufferWrites<RawBufferStoreOp>>(context);
}

```
- **EN**: Implements logic around `matchAndRewrite`, `staticallyOutOfBounds`, `failure`, `eraseOp`, and 3 more symbols; this block registers dialect entities or dialect-level hooks.
- **CN**: 围绕 `matchAndRewrite`, `staticallyOutOfBounds`, `failure`, `eraseOp`, and 3 more symbols 实现具体逻辑；该代码块注册方言实体或方言级钩子。

### Lines 308-327
```cpp
void RawBufferAtomicFaddOp::getCanonicalizationPatterns(
    RewritePatternSet &results, MLIRContext *context) {
  results.add<RemoveStaticallyOobBufferWrites<RawBufferAtomicFaddOp>>(context);
}

void RawBufferAtomicFmaxOp::getCanonicalizationPatterns(
    RewritePatternSet &results, MLIRContext *context) {
  results.add<RemoveStaticallyOobBufferWrites<RawBufferAtomicFmaxOp>>(context);
}

void RawBufferAtomicSmaxOp::getCanonicalizationPatterns(
    RewritePatternSet &results, MLIRContext *context) {
  results.add<RemoveStaticallyOobBufferWrites<RawBufferAtomicSmaxOp>>(context);
}

void RawBufferAtomicUminOp::getCanonicalizationPatterns(
    RewritePatternSet &results, MLIRContext *context) {
  results.add<RemoveStaticallyOobBufferWrites<RawBufferAtomicUminOp>>(context);
}

```
- **EN**: Implements logic around `getCanonicalizationPatterns`, `add`; this block registers dialect entities or dialect-level hooks.
- **CN**: 围绕 `getCanonicalizationPatterns`, `add` 实现具体逻辑；该代码块注册方言实体或方言级钩子。

### Lines 328-347
```cpp
void RawBufferAtomicCmpswapOp::getCanonicalizationPatterns(
    RewritePatternSet &results, MLIRContext *context) {
  results.add<RemoveStaticallyOobBufferLoads<RawBufferAtomicCmpswapOp>>(
      context);
}

//===----------------------------------------------------------------------===//
// ScaledExtPackedMatrixOp
//===----------------------------------------------------------------------===//
LogicalResult ScaledExtPackedMatrixOp::verify() {
  int blockSize = getBlockSize();
  assert(llvm::is_contained({16, 32}, blockSize) && "invalid block size");

  int firstScaleByte = getFirstScaleByte();
  int firstScaleLane = getFirstScaleLane();
  auto sourceType = cast<VectorType>(getSource().getType());
  Type elementType = sourceType.getElementType();
  auto floatType = cast<FloatType>(elementType);
  unsigned bitWidth = floatType.getWidth();

```
- **EN**: Implements logic around `getCanonicalizationPatterns`, `add`, `verify`, `getBlockSize`, and 6 more symbols; this block implements verifier, folding, parsing, or printing hooks; registers dialect entities or dialect-level hooks; manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `getCanonicalizationPatterns`, `add`, `verify`, `getBlockSize`, and 6 more symbols 实现具体逻辑；该代码块实现验证、折叠、解析或打印钩子，并注册方言实体或方言级钩子，并处理 MLIR region、block 或控制流边。

### Lines 348-375
```cpp
  assert(llvm::is_contained(llvm::ArrayRef<unsigned>{4, 6, 8}, bitWidth));

  const bool is_fp8 = bitWidth == 8;
  const bool is_block_16 = blockSize == 16;

  if (!is_fp8) {
    if (is_block_16) {
      if (!llvm::is_contained({0, 1}, firstScaleByte)) {
        return emitOpError("blockSize of 16 can only have firstScaleByte be 0 "
                           "or 1 for f4 and f6.");
      }
    } else {
      if (!llvm::is_contained({0, 2}, firstScaleByte)) {
        return emitOpError("blockSize of 32 can only have firstScaleByte be 0 "
                           "or 2 for f4 and f6.");
      }
    }
  } else {
    if (is_block_16) {
      bool is_valid = ((firstScaleLane == 0) && (firstScaleByte == 0)) ||
                      ((firstScaleLane == 16) && (firstScaleByte == 2));
      if (!is_valid) {
        return emitOpError("blockSize of 16 can only have (firstScaleLane, "
                           "firstScaleByte) be (0, 0) or (16, 2) for f8.");
      }
    }
  }

```
- **EN**: Implements logic around `assert`, `is_contained`, `emitOpError`, `be`.
- **CN**: 围绕 `assert`, `is_contained`, `emitOpError`, `be` 实现具体逻辑。

### Lines 376-398
```cpp
  return success();
}

//===----------------------------------------------------------------------===//
// WMMAOp
//===----------------------------------------------------------------------===//

ParseResult mlir::amdgpu::parseMNKDimensionList(OpAsmParser &parser,
                                                IntegerAttr &m, IntegerAttr &n,
                                                IntegerAttr &k) {
  SmallVector<int64_t, 3> dimensions;
  if (parser.parseDimensionList(dimensions, false, false))
    return failure();
  if (dimensions.size() != 3)
    return parser.emitError(parser.getCurrentLocation())
           << "expected 3 dimensions in MNK dimension list";

  m = parser.getBuilder().getI32IntegerAttr(dimensions[0]);
  n = parser.getBuilder().getI32IntegerAttr(dimensions[1]);
  k = parser.getBuilder().getI32IntegerAttr(dimensions[2]);
  return success();
}

```
- **EN**: Implements logic around `success`, `parseMNKDimensionList`, `parseDimensionList`, `failure`, and 3 more symbols; this block implements verifier, folding, parsing, or printing hooks; encodes accelerator-specific execution or lowering rules.
- **CN**: 围绕 `success`, `parseMNKDimensionList`, `parseDimensionList`, `failure`, and 3 more symbols 实现具体逻辑；该代码块实现验证、折叠、解析或打印钩子，并编码加速器专用执行或 lowering 规则。

### Lines 399-418
```cpp
LogicalResult WMMAOp::verify() {
  auto sourceAType = cast<VectorType>(getSourceA().getType());
  auto sourceBType = cast<VectorType>(getSourceB().getType());
  auto destType = cast<VectorType>(getDestC().getType());

  Type sourceAElemType = sourceAType.getElementType();
  Type sourceBElemType = sourceBType.getElementType();
  if (sourceAType.getNumElements() != sourceBType.getNumElements()) {
    return emitOpError("source vectors have different lengths: ")
           << sourceAType << " vs. " << sourceBType;
  }

  bool isDestFloat = destType.getElementType().isFloat();
  bool isSrcFloat = sourceAElemType.isFloat();

  if (isDestFloat && !isSrcFloat)
    return emitOpError("expected float sources with float destination");
  if (!isDestFloat && isSrcFloat)
    return emitOpError("expected int sources with int destination");

```
- **EN**: Implements logic around `verify`, `cast`, `getElementType`, `getNumElements`, and 2 more symbols; this block implements verifier, folding, parsing, or printing hooks.
- **CN**: 围绕 `verify`, `cast`, `getElementType`, `getNumElements`, and 2 more symbols 实现具体逻辑；该代码块实现验证、折叠、解析或打印钩子。

### Lines 419-436
```cpp
  if (!sourceAElemType.isFloat(8) && sourceAElemType != sourceBElemType) {
    return emitOpError(
               "source element types must match (except for fp8/bf8) but have ")
           << sourceAType << " and " << sourceBType;
  }

  if (isSrcFloat) {
    if (getClamp())
      return emitOpError("clamp flag is not supported for float types");
    if (getUnsignedA() || getUnsignedB())
      return emitOpError("unsigned flags are not supported for float types");
  }
  return success();
}

//===----------------------------------------------------------------------===//
// ScaledWMMAOp
//===----------------------------------------------------------------------===//
```
- **EN**: Implements logic around `isFloat`, `emitOpError`, `match`, `getClamp`, and 2 more symbols.
- **CN**: 围绕 `isFloat`, `emitOpError`, `match`, `getClamp`, and 2 more symbols 实现具体逻辑。

### Lines 437-454
```cpp

LogicalResult ScaledWMMAOp::verify() {
  // Helper functions for type classification.
  auto isF8 = llvm::IsaPred<Float8E4M3FNType, Float8E5M2Type>;
  auto isF6 = llvm::IsaPred<Float6E2M3FNType, Float6E3M2FNType>;
  auto isF4 = llvm::IsaPred<Float4E2M1FNType>;
  auto isScaleF8 = llvm::IsaPred<Float8E8M0FNUType, Float8E4M3FNType>;
  auto isE8M0 = llvm::IsaPred<Float8E8M0FNUType>;
  auto isE4M3 = llvm::IsaPred<Float8E4M3FNType>;

  auto sourceAType = cast<VectorType>(getSourceA().getType());
  auto sourceBType = cast<VectorType>(getSourceB().getType());
  auto destType = cast<VectorType>(getDestC().getType());

  // Validate source element types are small floats (fp4/fp6/fp8).
  Type aElemType = sourceAType.getElementType();
  Type bElemType = sourceBType.getElementType();

```
- **EN**: Implements logic around `verify`, `cast`, `getElementType`; this block implements verifier, folding, parsing, or printing hooks.
- **CN**: 围绕 `verify`, `cast`, `getElementType` 实现具体逻辑；该代码块实现验证、折叠、解析或打印钩子。

### Lines 455-479
```cpp
  // Validate vector lengths based on dimensions.
  int64_t m = getM();
  int64_t aLen = sourceAType.getNumElements();
  int64_t bLen = sourceBType.getNumElements();
  int64_t expectedOutLen = (m == 16) ? 8 : 16;

  if (destType.getNumElements() != expectedOutLen)
    return emitOpError("expected output vector of length ")
           << expectedOutLen << " but got " << destType.getNumElements();

  if (m == 16) {
    // For 16×16×128: both A and B must be 64 elements.
    if (aLen != 64)
      return emitOpError(
                 "for 16x16x128, sourceA must have 64 elements but got ")
             << aLen;
    if (bLen != 64)
      return emitOpError(
                 "for 16x16x128, sourceB must have 64 elements but got ")
             << bLen;
  } else { // m == 32
    // For 32×16×128: only fp4 is supported, A is 128, B is 64.
    if (!isF4(aElemType) && !isF4(bElemType))
      return emitOpError("32x16x128 only supports fp4 element types");

```
- **EN**: Implements logic around `getM`, `getNumElements`, `emitOpError`, `isF4`.
- **CN**: 围绕 `getM`, `getNumElements`, `emitOpError`, `isF4` 实现具体逻辑。

### Lines 480-500
```cpp
    if (aLen != 128)
      return emitOpError(
                 "for 32x16x128, sourceA must have 128 elements but got ")
             << aLen;
    if (bLen != 64)
      return emitOpError(
                 "for 32x16x128, sourceB must have 64 elements but got ")
             << bLen;

    // For 32x16x128, matrix A uses all 32 lanes so a_first_scale_lane must be
    // 0.
    if (getAFirstScaleLane() != 0)
      return emitOpError("for 32x16x128, a_first_scale_lane must be 0");
  }

  // Validate scale types and their compatibility with matrix element types.
  auto scaleAType = cast<VectorType>(getScaleA().getType());
  auto scaleBType = cast<VectorType>(getScaleB().getType());
  Type scaleAElemType = scaleAType.getElementType();
  Type scaleBElemType = scaleBType.getElementType();

```
- **EN**: Implements logic around `emitOpError`, `getAFirstScaleLane`, `cast`, `getElementType`.
- **CN**: 围绕 `emitOpError`, `getAFirstScaleLane`, `cast`, `getElementType` 实现具体逻辑。

### Lines 501-519
```cpp
  // Validate scale element types are valid scale f8 types (E8M0FNU or E4M3FN).
  if (!isScaleF8(scaleAElemType) || !isScaleF8(scaleBElemType))
    return emitOpError(
        "scale operands must have f8 element types (E8M0FNU or E4M3FN)");

  // Any matrices A/B (fp8|fp6|fp4) with E8M0 scales for matrix A/B are valid.
  if (isE8M0(scaleAElemType) && isE8M0(scaleBElemType))
    return success();

  // Matrix A (F8|F6) x Matrix B (F4) with Scale A (E8M0), Scale B (E5M3|E4M3).
  if ((isF8(aElemType) || isF6(aElemType)) && isE8M0(scaleAElemType) &&
      isF4(bElemType) && isE4M3(scaleBElemType))
    return success();

  // Matrix A (F4) x Matrix B (F8|F6) with Scale A (E5M3|E4M3), Scale B (E8M0).
  if (isF4(aElemType) && isE4M3(scaleAElemType) &&
      (isF8(bElemType) || isF6(bElemType)) && isE8M0(scaleBElemType))
    return success();

```
- **EN**: Implements logic around `isScaleF8`, `emitOpError`, `types`, `isE8M0`, and 3 more symbols.
- **CN**: 围绕 `isScaleF8`, `emitOpError`, `types`, `isE8M0`, and 3 more symbols 实现具体逻辑。

### Lines 520-537
```cpp
  // Matrix A (F4) x Matrix B (F4) with Scale A (E4M3), Scale B (E4M3).
  if (isF4(aElemType) && isF4(bElemType) && isE4M3(scaleAElemType) &&
      isE4M3(scaleBElemType))
    return success();

  // No valid combination matched.
  return emitOpError("invalid combination of matrix and scale types: ")
         << "sourceA=" << aElemType << ", scaleA=" << scaleAElemType
         << ", sourceB=" << bElemType << ", scaleB=" << scaleBElemType;
}

//===----------------------------------------------------------------------===//
// MFMAOp
//===----------------------------------------------------------------------===//
LogicalResult MFMAOp::verify() {
  constexpr uint32_t waveSize = 64;
  Builder b(getContext());

```
- **EN**: Implements logic around `isF4`, `isE4M3`, `success`, `emitOpError`, and 2 more symbols; this block implements verifier, folding, parsing, or printing hooks.
- **CN**: 围绕 `isF4`, `isE4M3`, `success`, `emitOpError`, and 2 more symbols 实现具体逻辑；该代码块实现验证、折叠、解析或打印钩子。

### Lines 538-573
```cpp
  Type sourceType = getSourceA().getType();
  Type destType = getDestC().getType();

  Type sourceElem = sourceType, destElem = destType;
  uint32_t sourceLen = 1, destLen = 1;
  if (auto sourceVector = dyn_cast<VectorType>(sourceType)) {
    sourceLen = sourceVector.getNumElements();
    sourceElem = sourceVector.getElementType();
  }
  if (auto destVector = dyn_cast<VectorType>(destType)) {
    destLen = destVector.getNumElements();
    destElem = destVector.getElementType();
  }

  Type sourceBType = getSourceB().getType();
  if (sourceElem.isFloat(8) || sourceElem.isFloat(6) || sourceElem.isFloat(4)) {
    int64_t sourceBLen = 1;
    Type sourceBElem = sourceBType;
    if (auto sourceBVector = llvm::dyn_cast<VectorType>(sourceBType)) {
      sourceBLen = sourceBVector.getNumElements();
      sourceBElem = sourceBVector.getElementType();
    }
    if (!sourceBElem.isFloat(8) && !sourceBElem.isFloat(6) &&
        !sourceBElem.isFloat(4))
      return emitOpError("expected both source operands to have small-float "
                         "elements if one does");
    if (sourceLen != sourceBLen)
      return emitOpError(
          "expected both small-float source vectors to have the same length");
  } else {
    if (sourceType != sourceBType)
      return emitOpError("expected both non-small-float source operand types "
                         "to match exactly");
  }
  // Normalize the wider integer types the compiler expects to i8.
  if (sourceElem.isInteger(32)) {
```
- **EN**: Implements logic around `getSourceA`, `getDestC`, `dyn_cast`, `getNumElements`, and 5 more symbols.
- **CN**: 围绕 `getSourceA`, `getDestC`, `dyn_cast`, `getNumElements`, and 5 more symbols 实现具体逻辑。

### Lines 574-593
```cpp
    sourceLen *= 4;
    sourceElem = b.getI8Type();
  }
  if (sourceElem.isInteger(64)) {
    sourceLen *= 8;
    sourceElem = b.getI8Type();
  }

  int64_t numSourceElems = (getM() * getK() * getBlocks()) / waveSize;
  if (sourceLen != numSourceElems)
    return emitOpError("expected " + Twine(numSourceElems) +
                       " source values for this operation but got " +
                       Twine(sourceLen));

  int64_t numDestElems = (getM() * getN() * getBlocks()) / waveSize;
  if (destLen != numDestElems)
    return emitOpError("expected " + Twine(numDestElems) +
                       " result values for this operation but got " +
                       Twine(destLen));

```
- **EN**: Implements logic around `getI8Type`, `isInteger`, `getM`, `emitOpError`, and 1 more symbols.
- **CN**: 围绕 `getI8Type`, `isInteger`, `getM`, `emitOpError`, and 1 more symbols 实现具体逻辑。

### Lines 594-611
```cpp
  if (destElem.isF64() && getBlgp() != MFMAPermB::none)
    return emitOpError(
        "double-precision ops do not support permuting lanes of B");
  if (destElem.isF64() && getCbsz() != 0)
    return emitOpError(
        "double-precision ops do not support permuting lanes of A");
  if (getAbid() >= (1u << getCbsz()))
    return emitOpError(
        "block ID for permuting A (abid) must be below 2 ** cbsz");

  if ((getNegateA() || getNegateB() || getNegateC()) && !destElem.isF64())
    return emitOpError(
        "negation flags only available for double-precision operations");

  return success();
}

//===----------------------------------------------------------------------===//
```
- **EN**: Implements logic around `isF64`, `emitOpError`, `getAbid`, `A`, and 2 more symbols; this block manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `isF64`, `emitOpError`, `getAbid`, `A`, and 2 more symbols 实现具体逻辑；该代码块处理 MLIR region、block 或控制流边。

### Lines 612-631
```cpp
// SparseMFMAOp
//===----------------------------------------------------------------------===//

LogicalResult SparseMFMAOp::verify() {
  constexpr uint32_t waveSize = 64;

  auto sparseType = cast<VectorType>(getSourceA().getType());
  auto denseType = cast<VectorType>(getSourceB().getType());
  auto destType = cast<VectorType>(getDestC().getType());

  Type sparseElem = sparseType.getElementType();
  Type denseElem = denseType.getElementType();
  int64_t sparseLen = sparseType.getNumElements();
  int64_t denseLen = denseType.getNumElements();
  int64_t destLen = destType.getNumElements();

  if (denseLen != 2 * sparseLen)
    return emitOpError("expected dense source operand to have exactly double "
                       "the number of elements of the sparse source operand");

```
- **EN**: Implements logic around `verify`, `cast`, `getElementType`, `getNumElements`, and 1 more symbols; this block implements verifier, folding, parsing, or printing hooks.
- **CN**: 围绕 `verify`, `cast`, `getElementType`, `getNumElements`, and 1 more symbols 实现具体逻辑；该代码块实现验证、折叠、解析或打印钩子。

### Lines 632-651
```cpp
  // Check that source element types are compatible.
  // For fp8/bf8 mixed operations, element types can differ (e.g., fp8 * bf8).
  // For other types, element types must match exactly.
  bool bothFloat8 = sparseElem.isFloat(8) && denseElem.isFloat(8);
  if (!bothFloat8 && sparseElem != denseElem)
    return emitOpError(
        "expected source operands to have the same element type");

  // Classify the sparse MFMA variant. The three flavors differ in CBSZ/ABID
  // handling and in the sparse-index layout:
  //   - gfx942 16-bit:              max ABID = 3, sparse idx = vector<4xi8>
  //   - gfx950 16-bit / gfx942 8-bit: max ABID = 1, sparse idx = vector<2xi16>
  //   - gfx950 8-bit:  CBSZ/ABID ignored by hw,   sparse idx = i32
  uint32_t m = getM(), k = getK();
  bool is8BitSource = sparseElem.isFloat(8) || sparseElem.isInteger(8);
  bool is16BitGfx942 =
      !is8BitSource && ((m == 16 && k == 32) || (m == 32 && k == 16));
  bool is8BitGfx950 =
      is8BitSource && ((m == 16 && k == 128) || (m == 32 && k == 64));

```
- **EN**: Implements logic around `isFloat`, `emitOpError`, `getM`.
- **CN**: 围绕 `isFloat`, `emitOpError`, `getM` 实现具体逻辑。

### Lines 652-669
```cpp
  // CBSZ/ABID range check. On gfx950 8-bit the hardware always uses the first
  // set and ignores these fields, so require zeros in IR. Otherwise ABID is
  // only meaningful when CBSZ == 0 (when CBSZ != 0 the first set is always
  // used and ABID is irrelevant, so the verifier accepts any value).
  if (is8BitGfx950) {
    if (getCbsz() != 0)
      return emitOpError(
          "CBSZ must be 0 for this variant (field is ignored by hardware)");
    if (getAbid() != 0)
      return emitOpError(
          "ABID must be 0 for this variant (field is ignored by hardware)");
  } else if (getCbsz() == 0) {
    unsigned maxAbid = is16BitGfx942 ? 3u : 1u;
    if (getAbid() > maxAbid)
      return emitOpError("ABID must be in [0, ")
             << maxAbid << "] for this variant";
  }

```
- **EN**: Implements logic around `getCbsz`, `emitOpError`, `variant`, `getAbid`.
- **CN**: 围绕 `getCbsz`, `emitOpError`, `variant`, `getAbid` 实现具体逻辑。

### Lines 670-692
```cpp
  Type sparseIdxType = getSparseIdx().getType();
  if (is8BitGfx950) {
    if (!sparseIdxType.isInteger(32))
      return emitOpError("expected i32 sparse indices for this variant "
                         "(no internal set structure), but got ")
             << sparseIdxType;
  } else {
    unsigned expectedIdxElems = is16BitGfx942 ? 4 : 2;
    unsigned expectedIdxBits = is16BitGfx942 ? 8 : 16;
    auto vecType = dyn_cast<VectorType>(sparseIdxType);
    if (!vecType || vecType.getNumElements() != expectedIdxElems ||
        !vecType.getElementType().isInteger(expectedIdxBits))
      return emitOpError("expected vector<")
             << expectedIdxElems << "xi" << expectedIdxBits
             << "> sparse indices for this variant, but got " << sparseIdxType;
  }

  int64_t expectedSourceElems = (getM() * getK()) / waveSize;
  if (denseLen != expectedSourceElems)
    return emitOpError("expected " + Twine(expectedSourceElems) +
                       " source values for this operation but got " +
                       Twine(denseLen));

```
- **EN**: Implements logic around `getSparseIdx`, `isInteger`, `emitOpError`, `dyn_cast`, and 4 more symbols.
- **CN**: 围绕 `getSparseIdx`, `isInteger`, `emitOpError`, `dyn_cast`, and 4 more symbols 实现具体逻辑。

### Lines 693-710
```cpp
  int64_t expectedDestElems = (getM() * getN()) / waveSize;
  if (destLen != expectedDestElems)
    return emitOpError("expected " + Twine(expectedDestElems) +
                       " result values for this operation but got " +
                       Twine(destLen));

  return success();
}

//===----------------------------------------------------------------------===//
// SparseWMMAOp
//===----------------------------------------------------------------------===//

LogicalResult SparseWMMAOp::verify() {
  auto sparseType = cast<VectorType>(getSourceA().getType());
  auto denseType = cast<VectorType>(getSourceB().getType());
  auto destType = cast<VectorType>(getDestC().getType());

```
- **EN**: Implements logic around `getM`, `emitOpError`, `Twine`, `success`, and 2 more symbols; this block implements verifier, folding, parsing, or printing hooks.
- **CN**: 围绕 `getM`, `emitOpError`, `Twine`, `success`, and 2 more symbols 实现具体逻辑；该代码块实现验证、折叠、解析或打印钩子。

### Lines 711-729
```cpp
  Type sparseElem = sparseType.getElementType();
  Type denseElem = denseType.getElementType();
  Type destElem = destType.getElementType();
  int64_t sparseLen = sparseType.getNumElements();
  int64_t denseLen = denseType.getNumElements();
  int64_t destLen = destType.getNumElements();

  uint32_t m = getM(), n = getN(), k = getK();
  if ((m != 16) || (n != 16))
    return emitOpError("expected MxN to be exactly 16x16");

  const bool isWavesize64 = getWave64();
  const bool isInt4Input = sparseElem.isInteger(4) && denseElem.isInteger(4);
  const bool isEqualLengthAllowed = isWavesize64 && isInt4Input && k == 32;

  if ((denseLen != 2 * sparseLen) && !isEqualLengthAllowed)
    return emitOpError("expected dense source operand to have exactly double "
                       "the number of elements of the sparse source operand");

```
- **EN**: Implements logic around `getElementType`, `getNumElements`, `getM`, `emitOpError`, and 2 more symbols.
- **CN**: 围绕 `getElementType`, `getNumElements`, `getM`, `emitOpError`, and 2 more symbols 实现具体逻辑。

### Lines 730-747
```cpp
  if (isEqualLengthAllowed && (denseLen != sparseLen))
    return emitOpError("expected dense source operand to have exactly the "
                       "same the number of elements");

  if (destElem.isInteger()) {
    if (!(sparseElem.isInteger() && denseElem.isInteger())) {
      return emitOpError("source operand and destination operands must all be "
                         "either integer or float types");
    }
  }

  if (destElem.isFloat()) {
    if (!(sparseElem.isFloat() && denseElem.isFloat())) {
      return emitOpError("source operand and destination operands must all be "
                         "either integer or float types");
    }
  }

```
- **EN**: Implements logic around `emitOpError`, `isInteger`, `isFloat`.
- **CN**: 围绕 `emitOpError`, `isInteger`, `isFloat` 实现具体逻辑。

### Lines 748-769
```cpp
  // Check that source element types are compatible.
  // For fp8/bf8 mixed operations, element types can differ (e.g., fp8 * bf8).
  // For other types, element types must match exactly.
  bool bothFloat8 = sparseElem.isFloat(8) && denseElem.isFloat(8);
  if (!bothFloat8 && sparseElem != denseElem)
    return emitOpError(
        "expected source operands to have the same element type");

  const int64_t waveSize = isWavesize64 ? 64 : 32;

  int64_t expectedSourceElems = (getM() * getK()) / waveSize;
  if (denseLen != expectedSourceElems)
    return emitOpError("expected " + Twine(expectedSourceElems) +
                       " source values for this operation but got " +
                       Twine(denseLen));

  int64_t expectedDestElems = (getM() * getN()) / waveSize;
  if (destLen != expectedDestElems)
    return emitOpError("expected " + Twine(expectedDestElems) +
                       " result values for this operation but got " +
                       Twine(destLen));

```
- **EN**: Implements logic around `isFloat`, `emitOpError`, `getM`, `Twine`.
- **CN**: 围绕 `isFloat`, `emitOpError`, `getM`, `Twine` 实现具体逻辑。

### Lines 770-789
```cpp
  return success();
}

//===----------------------------------------------------------------------===//
// DotOp
//===----------------------------------------------------------------------===//
LogicalResult DotOp::verify() {
  Type aElem = cast<VectorType>(getSourceA().getType()).getElementType();
  Type bElem = cast<VectorType>(getSourceB().getType()).getElementType();
  Type dest = getDestC().getType();

  bool aIsFloat8 = aElem.isFloat(8);
  bool bIsFloat8 = bElem.isFloat(8);
  bool aIsInteger = isa<IntegerType>(aElem);

  bool bothFloat8 = aIsFloat8 && bIsFloat8;
  if (!bothFloat8 && aElem != bElem)
    return emitOpError(
        "expected source operands to have the same element type");

```
- **EN**: Implements logic around `success`, `verify`, `cast`, `getDestC`, and 3 more symbols; this block implements verifier, folding, parsing, or printing hooks.
- **CN**: 围绕 `success`, `verify`, `cast`, `getDestC`, and 3 more symbols 实现具体逻辑；该代码块实现验证、折叠、解析或打印钩子。

### Lines 790-807
```cpp
  if (aElem.isF16()) {
    if (!dest.isF32() && !dest.isF16())
      return emitOpError("expected f32 or f16 accumulator for f16 sources");
  } else if (aElem.isBF16()) {
    if (!dest.isF32() && !dest.isBF16())
      return emitOpError("expected f32 or bf16 accumulator for bf16 sources");
  } else if (aIsInteger) {
    if (!dest.isInteger(32))
      return emitOpError("expected i32 accumulator for integer sources");
  } else if (aIsFloat8) {
    if (!dest.isF32())
      return emitOpError("expected f32 accumulator for fp8 sources");
  }

  if ((getUnsignedA() || getUnsignedB()) && !aIsInteger)
    return emitOpError(
        "unsignedA/unsignedB are only valid for integer source types");

```
- **EN**: Implements logic around `isF16`, `isF32`, `emitOpError`, `isBF16`, and 2 more symbols.
- **CN**: 围绕 `isF16`, `isF32`, `emitOpError`, `isBF16`, and 2 more symbols 实现具体逻辑。

### Lines 808-825
```cpp
  if (aElem.isInteger(16) && getUnsignedA() != getUnsignedB())
    return emitOpError(
        "mixed-sign dot is not supported for 16-bit integer sources");

  if (getClamp()) {
    bool noClamp = (aElem.isF16() && dest.isF16()) ||
                   (aElem.isBF16() && dest.isBF16()) || aIsFloat8;
    if (noClamp)
      return emitOpError(
          "clamp is not supported for this (source, accumulator) combination");
  }

  return success();
}

//===----------------------------------------------------------------------===//
// DPPOp
//===----------------------------------------------------------------------===//
```
- **EN**: Implements logic around `isInteger`, `emitOpError`, `getClamp`, `isF16`, and 3 more symbols.
- **CN**: 围绕 `isInteger`, `emitOpError`, `getClamp`, `isF16`, and 3 more symbols 实现具体逻辑。

### Lines 826-845
```cpp
LogicalResult DPPOp::verify() {
  DPPPerm kind = getKind();
  Attribute permArgument = getPermArgument().value_or(Attribute{});

  switch (kind) {

  case DPPPerm::quad_perm: {
    auto quadPermAttr = dyn_cast_or_null<ArrayAttr>(permArgument);
    if (!quadPermAttr || quadPermAttr.size() != 4) {
      return emitOpError("quad_perm attribute must have exactly 4 elements");
    }
    for (auto elem : quadPermAttr.getAsRange<IntegerAttr>()) {
      int32_t num = elem.getInt();
      if (num < 0 || num > 3) {
        return emitOpError(
            "Each element of quad_perm must be in the range [0, 3]");
      }
    }
  } break;

```
- **EN**: Implements logic around `verify`, `getKind`, `getPermArgument`, `dyn_cast_or_null`, and 4 more symbols; this block implements verifier, folding, parsing, or printing hooks.
- **CN**: 围绕 `verify`, `getKind`, `getPermArgument`, `dyn_cast_or_null`, and 4 more symbols 实现具体逻辑；该代码块实现验证、折叠、解析或打印钩子。

### Lines 846-878
```cpp
  case DPPPerm::row_shl:
  case DPPPerm::row_shr:
  case DPPPerm::row_ror: {
    if (!permArgument) {
      return emitOpError("Attribute '" + Twine(stringifyDPPPerm(kind)) +
                         "' value not specified");
    }
    if (auto intAttr = dyn_cast<IntegerAttr>(permArgument)) {
      uint32_t attrValue = intAttr.getInt();
      if (attrValue < 1 || attrValue > 15) {
        return emitOpError("Attribute value must be between 1 and 15");
      }
    }
  } break;

  case DPPPerm::wave_shl:
  case DPPPerm::wave_shr:
  case DPPPerm::wave_rol:
  case DPPPerm::wave_ror:
  case DPPPerm::row_mirror:
  case DPPPerm::row_half_mirror:
  case DPPPerm::row_bcast_15:
  case DPPPerm::row_bcast_31: {
    if (permArgument && !isa<UnitAttr>(permArgument)) {
      return emitOpError("Expected unit attribute for permArgument, but found "
                         "non-trivial argument");
    }
    break;
  }
  }
  return success();
}

```
- **EN**: Implements logic around `emitOpError`, `dyn_cast`, `getInt`, `isa`, and 1 more symbols.
- **CN**: 围绕 `emitOpError`, `dyn_cast`, `getInt`, `isa`, and 1 more symbols 实现具体逻辑。

### Lines 879-900
```cpp
//===----------------------------------------------------------------------===//
// PermlaneSwapOp
//===----------------------------------------------------------------------===//
LogicalResult PermlaneSwapOp::verify() {
  unsigned rowLength = getRowLength();

  if (rowLength != 16 && rowLength != 32)
    return emitOpError("row_length attribute must either be 16 or 32.");

  return success();
}

/// Remove amdgpu.lds_barrier after amdgpu.lds_barrier.
static LogicalResult eraseRedundantLDSBarrierOps(LDSBarrierOp op,
                                                 PatternRewriter &rewriter) {
  if (isa_and_nonnull<LDSBarrierOp>(op->getNextNode())) {
    rewriter.eraseOp(op);
    return success();
  }
  return failure();
}

```
- **EN**: Implements logic around `verify`, `getRowLength`, `emitOpError`, `success`, and 4 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure; implements verifier, folding, parsing, or printing hooks; encodes accelerator-specific execution or lowering rules.
- **CN**: 围绕 `verify`, `getRowLength`, `emitOpError`, `success`, and 4 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并实现验证、折叠、解析或打印钩子，并编码加速器专用执行或 lowering 规则。

### Lines 901-921
```cpp
void LDSBarrierOp::getCanonicalizationPatterns(RewritePatternSet &results,
                                               MLIRContext *context) {
  results.add(eraseRedundantLDSBarrierOps);
}

//===----------------------------------------------------------------------===//
// MemoryCounterWaitOp
//===----------------------------------------------------------------------===//

namespace {
/// Fuse adjacent memory counter wait ops, taking the minimum value of the
/// counters.
struct FuseMemoryCounterWaitOp final : OpRewritePattern<MemoryCounterWaitOp> {
  using Base::Base;

  LogicalResult matchAndRewrite(MemoryCounterWaitOp op,
                                PatternRewriter &rewriter) const override {
    auto next = dyn_cast<MemoryCounterWaitOp>(op->getNextNode());
    if (!next)
      return failure();

```
- **EN**: Introduces declarations for `FuseMemoryCounterWaitOp`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `FuseMemoryCounterWaitOp` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 922-947
```cpp
    auto setters = {&MemoryCounterWaitOp::setLoad,
                    &MemoryCounterWaitOp::setStore, &MemoryCounterWaitOp::setDs,
                    &MemoryCounterWaitOp::setExp,
                    &MemoryCounterWaitOp::setTensor};
    auto lhsVals = {op.getLoad(), op.getStore(), op.getDs(), op.getExp(),
                    op.getTensor()};
    auto rhsVals = {next.getLoad(), next.getStore(), next.getDs(),
                    next.getExp(), next.getTensor()};
    rewriter.modifyOpInPlace(op, [&] {
      for (auto [setter, lhs, rhs] :
           llvm::zip_equal(setters, lhsVals, rhsVals)) {
        if (lhs && rhs) {
          (op.*setter)(std::min(*lhs, *rhs));
        } else if (lhs) {
          (op.*setter)(*lhs);
        } else if (rhs) {
          (op.*setter)(*rhs);
        }
      }
    });
    rewriter.eraseOp(next);
    return success();
  }
};
} // namespace

```
- **EN**: Implements logic around `getLoad`, `getTensor`, `getExp`, `modifyOpInPlace`, and 4 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `getLoad`, `getTensor`, `getExp`, `modifyOpInPlace`, and 4 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 948-966
```cpp
void MemoryCounterWaitOp::getCanonicalizationPatterns(
    RewritePatternSet &results, MLIRContext *context) {
  results.add<FuseMemoryCounterWaitOp>(context);
}

//===----------------------------------------------------------------------===//
// GatherToLDSOp
//===----------------------------------------------------------------------===//

LogicalResult GatherToLDSOp::verify() {
  MemRefType srcType = cast<MemRefType>(getSrc().getType());
  MemRefType dstType = cast<MemRefType>(getDst().getType());

  if (failed(
          verifyIndexCount(*this, "source", srcType, getSrcIndices().size())) ||
      failed(verifyIndexCount(*this, "destination", dstType,
                              getDstIndices().size())))
    return failure();

```
- **EN**: Implements logic around `getCanonicalizationPatterns`, `add`, `verify`, `cast`, and 4 more symbols; this block implements verifier, folding, parsing, or printing hooks; registers dialect entities or dialect-level hooks.
- **CN**: 围绕 `getCanonicalizationPatterns`, `add`, `verify`, `cast`, and 4 more symbols 实现具体逻辑；该代码块实现验证、折叠、解析或打印钩子，并注册方言实体或方言级钩子。

### Lines 967-987
```cpp
  if (dstType.getRank() > 0 && !dstType.areTrailingDimsContiguous(1))
    return emitOpError("destination type inner most dim must be contiguous");

  auto elemType = srcType.getElementType();
  // Check $src and $dst element types are the same.
  if (elemType != dstType.getElementType())
    return emitOpError("source and destination element types must match");

  // copy type sizes should be 1, 2, 4, 12 or 16 bytes.
  auto transferType = getTransferType();
  int transferSize;
  if (auto vectorTransfer = dyn_cast<VectorType>(transferType)) {
    transferSize = vectorTransfer.getNumElements() *
                   vectorTransfer.getElementTypeBitWidth();
  } else {
    transferSize = transferType.getIntOrFloatBitWidth();
  }
  if (!llvm::is_contained({8, 16, 32, 96, 128}, transferSize))
    return emitOpError(
        "Transfering type size must be 8, 16, 32, 96 or 128 bits");

```
- **EN**: Implements logic around `getRank`, `emitOpError`, `getElementType`, `getTransferType`, and 5 more symbols.
- **CN**: 围绕 `getRank`, `emitOpError`, `getElementType`, `getTransferType`, and 5 more symbols 实现具体逻辑。

### Lines 988-1017
```cpp
  if (!hasGlobalMemorySpace(srcType.getMemorySpace()) &&
      !hasFatRawBufferMemorySpace(srcType.getMemorySpace()))
    return emitOpError(
        "source memory address space must be global or fat raw buffer");

  if (!hasWorkgroupMemorySpace(dstType.getMemorySpace()))
    return emitOpError("destination memory address space must be Workgroup");

  return success();
}

namespace {
/// If the source/target of a GatherToLDSOp is a CastOp that only removes static
/// information or changes layout, the cast can be skipped.
struct FoldGatherToLDSOfCast final : OpRewritePattern<GatherToLDSOp> {
  using OpRewritePattern::OpRewritePattern;

  LogicalResult matchAndRewrite(GatherToLDSOp gatherOp,
                                PatternRewriter &rewriter) const override {
    bool modified = false;
    auto foldCast = [&](OpOperand &operand) {
      if (auto castOp = operand.get().getDefiningOp<memref::CastOp>()) {
        if (memref::CastOp::canFoldIntoConsumerOp(castOp)) {
          rewriter.modifyOpInPlace(gatherOp,
                                   [&] { operand.assign(castOp.getSource()); });
          modified = true;
        }
      }
    };

```
- **EN**: Introduces declarations for `FoldGatherToLDSOfCast`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `FoldGatherToLDSOfCast` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 1018-1038
```cpp
    foldCast(gatherOp.getSrcMutable());
    foldCast(gatherOp.getDstMutable());

    return success(modified);
  }
};
} // namespace

void GatherToLDSOp::getCanonicalizationPatterns(RewritePatternSet &results,
                                                MLIRContext *context) {
  results.add<FoldGatherToLDSOfCast>(context);
}

//===----------------------------------------------------------------------===//
// GlobalLoadAsyncToLDSOp
//===----------------------------------------------------------------------===//

LogicalResult GlobalLoadAsyncToLDSOp::verify() {
  MemRefType srcType = cast<MemRefType>(getSrc().getType());
  MemRefType dstType = cast<MemRefType>(getDst().getType());

```
- **EN**: Implements logic around `foldCast`, `success`, `getCanonicalizationPatterns`, `add`, and 2 more symbols; this block implements verifier, folding, parsing, or printing hooks; registers dialect entities or dialect-level hooks.
- **CN**: 围绕 `foldCast`, `success`, `getCanonicalizationPatterns`, `add`, and 2 more symbols 实现具体逻辑；该代码块实现验证、折叠、解析或打印钩子，并注册方言实体或方言级钩子。

### Lines 1039-1058
```cpp
  if (failed(
          verifyIndexCount(*this, "source", srcType, getSrcIndices().size())) ||
      failed(verifyIndexCount(*this, "destination", dstType,
                              getDstIndices().size())))
    return failure();

  if (srcType.getElementType() != dstType.getElementType())
    return emitOpError("source and destination element types must match");

  Type transferType = getTransferType();
  int transferSize;
  if (auto vectorTransfer = dyn_cast<VectorType>(transferType)) {
    transferSize = vectorTransfer.getNumElements() *
                   vectorTransfer.getElementTypeBitWidth();
  } else {
    transferSize = transferType.getIntOrFloatBitWidth();
  }
  if (!llvm::is_contained({8, 32, 64, 128}, transferSize))
    return emitOpError("transfer type size must be 8, 32, 64, or 128 bits");

```
- **EN**: Implements logic around `failed`, `verifyIndexCount`, `getDstIndices`, `failure`, and 8 more symbols.
- **CN**: 围绕 `failed`, `verifyIndexCount`, `getDstIndices`, `failure`, and 8 more symbols 实现具体逻辑。

### Lines 1059-1078
```cpp
  if (!hasGlobalMemorySpace(srcType.getMemorySpace()))
    return emitOpError("source memory address space must be global");

  if (!hasWorkgroupMemorySpace(dstType.getMemorySpace()))
    return emitOpError("destination memory address space must be Workgroup");

  return success();
}

static LogicalResult
foldGlobalLoadAsyncToLDSConstantMask(GlobalLoadAsyncToLDSOp op,
                                     PatternRewriter &rewriter) {
  Value mask = op.getMask();
  if (!mask)
    return failure();

  APInt maskValue;
  if (!matchPattern(mask, m_ConstantInt(&maskValue)))
    return failure();

```
- **EN**: Implements logic around `hasGlobalMemorySpace`, `emitOpError`, `hasWorkgroupMemorySpace`, `success`, and 4 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure; implements verifier, folding, parsing, or printing hooks; encodes accelerator-specific execution or lowering rules.
- **CN**: 围绕 `hasGlobalMemorySpace`, `emitOpError`, `hasWorkgroupMemorySpace`, `success`, and 4 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并实现验证、折叠、解析或打印钩子，并编码加速器专用执行或 lowering 规则。

### Lines 1079-1096
```cpp
  if (maskValue.isZero()) {
    rewriter.eraseOp(op);
    return success();
  }

  rewriter.modifyOpInPlace(op, [&]() { op.getMaskMutable().clear(); });
  return success();
}

void GlobalLoadAsyncToLDSOp::getCanonicalizationPatterns(
    RewritePatternSet &results, MLIRContext *context) {
  results.add(foldGlobalLoadAsyncToLDSConstantMask);
}

//===----------------------------------------------------------------------===//
// TransposeLoadOp
//===----------------------------------------------------------------------===//

```
- **EN**: Implements logic around `isZero`, `eraseOp`, `success`, `modifyOpInPlace`, and 2 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure; implements verifier, folding, parsing, or printing hooks; registers dialect entities or dialect-level hooks.
- **CN**: 围绕 `isZero`, `eraseOp`, `success`, `modifyOpInPlace`, and 2 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并实现验证、折叠、解析或打印钩子，并注册方言实体或方言级钩子。

### Lines 1097-1119
```cpp
LogicalResult TransposeLoadOp::verify() {
  MemRefType srcType = cast<MemRefType>(getSrc().getType());

  if (failed(
          verifyIndexCount(*this, "source", srcType, getSrcIndices().size())))
    return failure();

  if (!hasWorkgroupMemorySpace(srcType.getMemorySpace()))
    return emitOpError("source memory address space must be Workgroup");

  auto transferType = cast<VectorType>(getType());
  size_t numElements = transferType.getNumElements();
  size_t elementTypeSize =
      transferType.getElementType().getIntOrFloatBitWidth();

  // ElementSize -> NumElements
  const llvm::SmallDenseMap<size_t, size_t> kValidLoadSizeMap = {
      {4, 16},
      {6, 16},
      {8, 8},
      {16, 4},
  };

```
- **EN**: Implements logic around `verify`, `cast`, `failed`, `verifyIndexCount`, and 5 more symbols; this block implements verifier, folding, parsing, or printing hooks; encodes accelerator-specific execution or lowering rules.
- **CN**: 围绕 `verify`, `cast`, `failed`, `verifyIndexCount`, and 5 more symbols 实现具体逻辑；该代码块实现验证、折叠、解析或打印钩子，并编码加速器专用执行或 lowering 规则。

### Lines 1120-1139
```cpp
  auto validNumElems = kValidLoadSizeMap.find(elementTypeSize);
  if (validNumElems == kValidLoadSizeMap.end())
    return emitOpError("Unsupported element type size for transpose load: ")
           << elementTypeSize << " bits";

  if (numElements != validNumElems->second)
    return emitOpError(
               "Transferring type size mismatch: expected num of elements: ")
           << validNumElems->second;

  return success();
}

//===----------------------------------------------------------------------===//
// GlobalTransposeLoadOp
//===----------------------------------------------------------------------===//

LogicalResult GlobalTransposeLoadOp::verify() {
  MemRefType srcType = cast<MemRefType>(getSrc().getType());

```
- **EN**: Implements logic around `find`, `end`, `emitOpError`, `success`, and 2 more symbols; this block implements verifier, folding, parsing, or printing hooks.
- **CN**: 围绕 `find`, `end`, `emitOpError`, `success`, and 2 more symbols 实现具体逻辑；该代码块实现验证、折叠、解析或打印钩子。

### Lines 1140-1159
```cpp
  if (failed(
          verifyIndexCount(*this, "source", srcType, getSrcIndices().size())))
    return failure();

  if (!hasGlobalMemorySpace(srcType.getMemorySpace()))
    return emitOpError("source memory address space must be Global");

  auto resultType = cast<VectorType>(getType());
  size_t numElements = resultType.getNumElements();
  size_t elementTypeSize = resultType.getElementType().getIntOrFloatBitWidth();

  // ElementSize -> NumElements. Chipset gating (gfx1200 vs gfx1250) is
  // enforced in the lowering.
  static const llvm::SmallDenseMap<size_t, size_t> kValidLoadSizeMap = {
      {4, 16}, // global_load_tr4_b64  (gfx1250+)
      {6, 16}, // global_load_tr6_b96  (gfx1250+)
      {8, 8},  // global_load_tr_b64   (gfx1200+)
      {16, 8}, // global_load_tr_b128  (gfx1200+)
  };

```
- **EN**: Implements logic around `failed`, `verifyIndexCount`, `failure`, `hasGlobalMemorySpace`, and 8 more symbols.
- **CN**: 围绕 `failed`, `verifyIndexCount`, `failure`, `hasGlobalMemorySpace`, and 8 more symbols 实现具体逻辑。

### Lines 1160-1177
```cpp
  auto validNumElems = kValidLoadSizeMap.find(elementTypeSize);
  if (validNumElems == kValidLoadSizeMap.end())
    return emitOpError(
               "unsupported element type size for global transpose load: ")
           << elementTypeSize << " bits";

  if (numElements != validNumElems->second)
    return emitOpError(
               "transferring type size mismatch: expected num of elements: ")
           << validNumElems->second;

  return success();
}

//===----------------------------------------------------------------------===//
// MakeDmaBaseOp
//===----------------------------------------------------------------------===//

```
- **EN**: Implements logic around `find`, `end`, `emitOpError`, `success`.
- **CN**: 围绕 `find`, `end`, `emitOpError`, `success` 实现具体逻辑。

### Lines 1178-1196
```cpp
template <typename BaseOp>
static LogicalResult verifyBase(BaseOp op) {
  auto ldsType = cast<MemRefType>(op.getLds().getType());
  auto globalType = cast<MemRefType>(op.getGlobal().getType());
  if (failed(verifyIndexCount(op, "global", globalType,
                              op.getGlobalIndices().size())) ||
      failed(verifyIndexCount(op, "lds", ldsType, op.getLdsIndices().size())))
    return failure();

  if (!hasWorkgroupMemorySpace(ldsType.getMemorySpace()))
    return op.emitOpError(
        "lds memref must have workgroup address space attribute.");
  if (!hasGlobalMemorySpace(globalType.getMemorySpace()))
    return op.emitOpError(
        "global memref must have global address space attribute.");

  Type elementType = ldsType.getElementType();
  unsigned width = elementType.getIntOrFloatBitWidth();

```
- **EN**: Implements logic around `verifyBase`, `cast`, `failed`, `getGlobalIndices`, and 6 more symbols; this block moves data between tensor-style values and explicit buffers; encodes accelerator-specific execution or lowering rules.
- **CN**: 围绕 `verifyBase`, `cast`, `failed`, `getGlobalIndices`, and 6 more symbols 实现具体逻辑；该代码块在张量风格值与显式缓冲区之间移动数据，并编码加速器专用执行或 lowering 规则。

### Lines 1197-1226
```cpp
  if (!llvm::is_contained({8u, 16u, 32u, 64u}, width))
    return op.emitOpError(
               "element type must be 1, 2, 4, or 8 bytes long but type was ")
           << width << " bits long.";
  return success();
}

LogicalResult MakeDmaBaseOp::verify() { return verifyBase(*this); }

//===----------------------------------------------------------------------===//
// MakeGatherDmaBaseOp
//===----------------------------------------------------------------------===//

LogicalResult
TDMGatherBaseType::verify(function_ref<InFlightDiagnostic()> emitError,
                          Type elementType, Type indexType) {
  unsigned width = elementType.getIntOrFloatBitWidth();
  if (!llvm::is_contained({8u, 16u, 32u, 64u}, width))
    return emitError()
           << "element type must be 1, 2, 4, or 8 bytes wide but type "
           << elementType << " is " << width / 8 << " bytes wide.";
  MLIRContext *ctx = elementType.getContext();
  Type i16 = IntegerType::get(ctx, 32);
  Type i32 = IntegerType::get(ctx, 16);
  if (!llvm::is_contained({i16, i32}, indexType))
    return emitError() << "index type must be i16 or i32 but index type is "
                       << indexType << ".";
  return success();
}

```
- **EN**: Implements logic around `is_contained`, `emitOpError`, `success`, `verify`, and 4 more symbols; this block implements verifier, folding, parsing, or printing hooks.
- **CN**: 围绕 `is_contained`, `emitOpError`, `success`, `verify`, and 4 more symbols 实现具体逻辑；该代码块实现验证、折叠、解析或打印钩子。

### Lines 1227-1248
```cpp
LogicalResult MakeGatherDmaBaseOp::verify() { return verifyBase(*this); }

//===----------------------------------------------------------------------===//
// MakeDmaDescriptorOp
//===----------------------------------------------------------------------===//

template <typename DescriptorOp>
static LogicalResult verifyDescriptorOp(DescriptorOp op) {
  ArrayRef<int64_t> globalStaticStrides = op.getGlobalStaticStrides();

  if (globalStaticStrides.empty())
    return op.emitOpError("strides must not be empty.");
  if (globalStaticStrides.back() != 1)
    return op.emitOpError("strides for the innermost dimension must be 1.");

  ArrayRef<int64_t> globalStaticSizes = op.getGlobalStaticSizes();
  size_t rank = globalStaticSizes.size();
  if (rank > 5)
    return op.emitOpError("tensor and tile must be at most of rank 5.");
  if (rank != globalStaticStrides.size())
    return op.emitOpError("strides and sizes must have same rank.");

```
- **EN**: Implements logic around `verify`, `verifyDescriptorOp`, `getGlobalStaticStrides`, `empty`, and 4 more symbols; this block implements verifier, folding, parsing, or printing hooks; moves data between tensor-style values and explicit buffers.
- **CN**: 围绕 `verify`, `verifyDescriptorOp`, `getGlobalStaticStrides`, `empty`, and 4 more symbols 实现具体逻辑；该代码块实现验证、折叠、解析或打印钩子，并在张量风格值与显式缓冲区之间移动数据。

### Lines 1249-1269
```cpp
  ArrayRef<int64_t> sharedStaticSizes = op.getSharedStaticSizes();
  if (rank != sharedStaticSizes.size())
    return op.emitOpError("tensor must have same rank as tile.");

  unsigned elementTypeWidth = op.getElementTypeWidth();
  if (!llvm::is_contained({8u, 16u, 32u, 64u}, elementTypeWidth))
    return op.emitOpError(
               "element type width must be 1, 2, 4 or 8 bytes, but was ")
           << elementTypeWidth << " bits long";

  if (!op.getAtomicBarrierAddress() && !op.getAtomicBarrierIndices().empty())
    return op.emitOpError(
        "atomic barrier indices require an atomic barrier address");

  if (Value atomicBarrierAddress = op.getAtomicBarrierAddress()) {
    auto atomicBarrierAddressType =
        cast<MemRefType>(atomicBarrierAddress.getType());
    if (failed(verifyIndexCount(op, "atomic barrier", atomicBarrierAddressType,
                                op.getAtomicBarrierIndices().size())))
      return failure();

```
- **EN**: Implements logic around `getSharedStaticSizes`, `size`, `emitOpError`, `getElementTypeWidth`, and 6 more symbols; this block moves data between tensor-style values and explicit buffers.
- **CN**: 围绕 `getSharedStaticSizes`, `size`, `emitOpError`, `getElementTypeWidth`, and 6 more symbols 实现具体逻辑；该代码块在张量风格值与显式缓冲区之间移动数据。

### Lines 1270-1287
```cpp
    bool barrierInLDS =
        hasWorkgroupMemorySpace(atomicBarrierAddressType.getMemorySpace());
    if (!barrierInLDS)
      return op.emitOpError("atomic barrier address must be in LDS.");
  }

  if (op.getEarlyTimeout() && !op.getWorkgroupMask())
    return op.emitOpError(
        "early timeout does not apply when workgroup_mask is not set.");
  return success();
}

template <typename DescriptorOp, typename FoldAdaptor>
static OpFoldResult foldDescriptorOp(DescriptorOp op, FoldAdaptor adaptor) {
  SmallVector<OpFoldResult> mixedGlobalSizes(op.getMixedGlobalSizes());
  SmallVector<OpFoldResult> mixedGlobalStrides(op.getMixedGlobalStrides());
  SmallVector<OpFoldResult> mixedSharedSizes(op.getMixedSharedSizes());

```
- **EN**: Implements logic around `hasWorkgroupMemorySpace`, `emitOpError`, `getEarlyTimeout`, `success`, and 4 more symbols; this block implements verifier, folding, parsing, or printing hooks.
- **CN**: 围绕 `hasWorkgroupMemorySpace`, `emitOpError`, `getEarlyTimeout`, `success`, and 4 more symbols 实现具体逻辑；该代码块实现验证、折叠、解析或打印钩子。

### Lines 1288-1305
```cpp
  if (failed(foldDynamicIndexList(mixedGlobalSizes, /*onlyNonNegative=*/true,
                                  /*onlyNonZero=*/true)) &&
      failed(foldDynamicIndexList(mixedGlobalStrides, /*onlyNonNegative=*/true,
                                  /*onlyNonZero=*/true)) &&
      failed(foldDynamicIndexList(mixedSharedSizes, /*onlyNonNegative=*/true,
                                  /*onlyNonZero=*/true)))
    return nullptr;

  SmallVector<Value> dynamicGlobalSizes, dynamicGlobalStrides,
      dynamicSharedSizes;
  SmallVector<int64_t> staticGlobalSizes, staticGlobalStrides,
      staticSharedSizes;

  dispatchIndexOpFoldResults(mixedGlobalSizes, dynamicGlobalSizes,
                             staticGlobalSizes);
  op.setGlobalStaticSizes(staticGlobalSizes);
  op.getGlobalDynamicSizesMutable().assign(dynamicGlobalSizes);

```
- **EN**: Implements logic around `failed`, `dispatchIndexOpFoldResults`, `setGlobalStaticSizes`, `getGlobalDynamicSizesMutable`; this block implements verifier, folding, parsing, or printing hooks.
- **CN**: 围绕 `failed`, `dispatchIndexOpFoldResults`, `setGlobalStaticSizes`, `getGlobalDynamicSizesMutable` 实现具体逻辑；该代码块实现验证、折叠、解析或打印钩子。

### Lines 1306-1325
```cpp
  dispatchIndexOpFoldResults(mixedGlobalStrides, dynamicGlobalStrides,
                             staticGlobalStrides);
  op.setGlobalStaticStrides(staticGlobalStrides);
  op.getGlobalDynamicStridesMutable().assign(dynamicGlobalStrides);

  dispatchIndexOpFoldResults(mixedSharedSizes, dynamicSharedSizes,
                             staticSharedSizes);
  op.setSharedStaticSizes(staticSharedSizes);
  op.getSharedDynamicSizesMutable().assign(dynamicSharedSizes);
  return op.getResult();
}

LogicalResult MakeDmaDescriptorOp::verify() {
  return verifyDescriptorOp(*this);
}

OpFoldResult MakeDmaDescriptorOp::fold(FoldAdaptor adaptor) {
  return foldDescriptorOp(*this, adaptor);
}

```
- **EN**: Implements logic around `dispatchIndexOpFoldResults`, `setGlobalStaticStrides`, `getGlobalDynamicStridesMutable`, `setSharedStaticSizes`, and 6 more symbols; this block implements verifier, folding, parsing, or printing hooks.
- **CN**: 围绕 `dispatchIndexOpFoldResults`, `setGlobalStaticStrides`, `getGlobalDynamicStridesMutable`, `setSharedStaticSizes`, and 6 more symbols 实现具体逻辑；该代码块实现验证、折叠、解析或打印钩子。

### Lines 1326-1343
```cpp
//===----------------------------------------------------------------------===//
// MakeGatherDmaDescriptorOp
//===----------------------------------------------------------------------===//

LogicalResult MakeGatherDmaDescriptorOp::verify() {
  ArrayRef<int64_t> globalStaticSizes = getGlobalStaticSizes();
  size_t rank = globalStaticSizes.size();
  if (rank > 2)
    return emitOpError(
        "tensor and tile must be at most of rank two in gather mode.");
  Value indices = getIndices();
  Type elementType = cast<VectorType>(indices.getType()).getElementType();
  if (elementType != getBase().getType().getIndexType())
    return emitOpError("indices' element type must match base's element type.");

  return verifyDescriptorOp(*this);
}

```
- **EN**: Implements logic around `verify`, `getGlobalStaticSizes`, `size`, `emitOpError`, and 4 more symbols; this block implements verifier, folding, parsing, or printing hooks; moves data between tensor-style values and explicit buffers.
- **CN**: 围绕 `verify`, `getGlobalStaticSizes`, `size`, `emitOpError`, and 4 more symbols 实现具体逻辑；该代码块实现验证、折叠、解析或打印钩子，并在张量风格值与显式缓冲区之间移动数据。

### Lines 1344-1373
```cpp
OpFoldResult MakeGatherDmaDescriptorOp::fold(FoldAdaptor adaptor) {
  return foldDescriptorOp(*this, adaptor);
}

//===----------------------------------------------------------------------===//
// ScaledMFMAOp
//===----------------------------------------------------------------------===//

namespace {
/// Check if the scales input is used in other scaled mfma's while they exist.
/// If theyre unused then pack the scales.
struct PackScales final : OpRewritePattern<ScaledMFMAOp> {
  using OpRewritePattern::OpRewritePattern;

  LogicalResult matchAndRewrite(ScaledMFMAOp op,
                                PatternRewriter &rewriter) const override {
    Location loc = op.getLoc();
    auto setOpsel = [&op](unsigned idx, int64_t val) {
      switch (idx) {
      case 3:
        op.setScalesIdxA(val);
        break;
      case 4:
        op.setScalesIdxB(val);
        break;
      default:
        break;
      }
    };

```
- **EN**: Introduces declarations for `PackScales`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `PackScales` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 1374-1402
```cpp
    // For every scale operand of this ScaledMFMAOp, if the scale is produced by
    // the extraction of a single scale from some vector, then attempt to
    // extract 4 values from that vector instead.
    //
    // Example: (f8 here means f8E8M0FNU)
    // %unit = vector.extract %ScaleSrc[offsets] : f8 from vector<...>
    // %scale = vector.insert %unit, ... : f8 into vector<4xf8>
    // amdgpu.scaled_mfma(%scale[0] * ...
    //
    // rewrite to:
    //
    // %reshaped = vector.shape_cast %ScaleSrc : vector<...> to vector<?xf8>
    // %scale = vector.extract %reshaped[?] : vector<4xf8> from vector<?xf8>
    // amdgpu.scaled_mfma(%scale[0-3] * ...
    //
    // This creates duplicate shape_casts for every use but these will be
    // removed in CSE.
    for (auto opIdx : std::array<int64_t, 2>({3, 4})) {
      auto insertOp = op.getOperand(opIdx).getDefiningOp<vector::InsertOp>();
      if (!insertOp) {
        return rewriter.notifyMatchFailure(op,
                                           "defining op not a vector.insert");
      }
      // If the extracted value is not a single scalar, then it has been packed.
      if (isa<VectorType>(insertOp.getValueToStore().getType())) {
        return rewriter.notifyMatchFailure(
            op, "scaled mfma operand already packed");
      }

```
- **EN**: Implements logic around `getOperand`, `notifyMatchFailure`, `isa`; this block applies MLIR pattern-rewrite or conversion infrastructure; encodes accelerator-specific execution or lowering rules.
- **CN**: 围绕 `getOperand`, `notifyMatchFailure`, `isa` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并编码加速器专用执行或 lowering 规则。

### Lines 1403-1422
```cpp
      auto extractOp =
          insertOp.getValueToStore().getDefiningOp<vector::ExtractOp>();
      if (!extractOp) {
        return rewriter.notifyMatchFailure(op,
                                           "defining op not a vector.extract");
      }

      Value scaleSrc = extractOp.getOperand(0);
      auto scaleSrcType = dyn_cast<VectorType>(scaleSrc.getType());
      if (!scaleSrcType) {
        return rewriter.notifyMatchFailure(op, "not a vector type");
      }

      // We do not handle dynamic dims yet, assume that the input is padded to
      // a static shape now.
      if (!scaleSrcType.hasStaticShape()) {
        return rewriter.notifyMatchFailure(op,
                                           "dynamic dims not yet supported");
      }

```
- **EN**: Implements logic around `getValueToStore`, `notifyMatchFailure`, `getOperand`, `dyn_cast`, and 1 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `getValueToStore`, `notifyMatchFailure`, `getOperand`, `dyn_cast`, and 1 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 1423-1458
```cpp
      int64_t numElements = scaleSrcType.getNumElements();
      if (numElements < 4) {
        return rewriter.notifyMatchFailure(
            op, "do not pack if # of scales less than four");
      }

      // Find a linearized idx using the size and offsets of the extract op.
      auto extractedPos = llvm::to_vector_of<int64_t>(
          llvm::reverse(extractOp.getStaticPosition()));
      ArrayRef<int64_t> scaleSrcShape = scaleSrcType.getShape();
      int64_t scaleSrcRank = scaleSrcType.getRank();
      SmallVector<int64_t> extractSizes(scaleSrcRank, 1);
      for (int64_t i = 1; i < scaleSrcRank; ++i) {
        extractSizes[i] = extractSizes[i - 1] * scaleSrcShape[scaleSrcRank - i];
      }
      int64_t idx = linearize(extractedPos, extractSizes);

      // All n scales (where n is the total number of scales) must now be
      // extracted in chunks of 4 elements. This is done by dividing the
      // original vector of scales into groups of 4 elements
      // at offsets 0, 4, ..., m (where m = n/4). All extractions of a
      // scale at a particular index are now replaced with an extraction
      // of the entire group of 4 elements to which that index belongs.
      //
      // If the number of scales happens to be indivisible by 4, extract
      // the remaining n - m scales in a chunk of 4 elements starting at
      // offset n - 4.
      int64_t offset = idx - (idx % 4);
      int64_t opsel = idx - offset;
      int64_t size = 4l;
      // Accomdate remaining elements in the case of non-4-divisible vectors.
      if (numElements - offset < size) {
        opsel = size - (numElements - idx);
        offset = numElements - 4l;
      }
      Type scaleSrcElemType = scaleSrcType.getElementType();
```
- **EN**: Implements logic around `getNumElements`, `notifyMatchFailure`, `to_vector_of`, `reverse`, and 5 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `getNumElements`, `notifyMatchFailure`, `to_vector_of`, `reverse`, and 5 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 1459-1480
```cpp
      auto newSrcType =
          VectorType::get(ArrayRef{numElements}, scaleSrcElemType);
      Value newScaleSrc =
          vector::ShapeCastOp::create(rewriter, loc, newSrcType, scaleSrc);
      auto extract = vector::ExtractStridedSliceOp::create(
          rewriter, loc, newScaleSrc, ArrayRef{offset}, ArrayRef{size},
          ArrayRef{int64_t(1)});
      rewriter.modifyOpInPlace(op, [&] {
        op->setOperand(opIdx, extract);
        setOpsel(opIdx, opsel);
      });
    }
    return success();
  }
};
} // namespace

void ScaledMFMAOp::getCanonicalizationPatterns(RewritePatternSet &results,
                                               MLIRContext *context) {
  results.add<PackScales>(context);
}

```
- **EN**: Implements logic around `get`, `create`, `int64_t`, `modifyOpInPlace`, and 5 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure; registers dialect entities or dialect-level hooks.
- **CN**: 围绕 `get`, `create`, `int64_t`, `modifyOpInPlace`, and 5 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并注册方言实体或方言级钩子。

### Lines 1481-1501
```cpp
//===----------------------------------------------------------------------===//
// In-LDS Barrier Operations (gfx1250+)
//===----------------------------------------------------------------------===//

template <typename T>
static LogicalResult verifyDsBarrierOpCommon(T &op) {
  MemRefType memrefType = llvm::cast<MemRefType>(op.getBase().getType());
  if (failed(
          verifyIndexCount(op, "barrier", memrefType, op.getIndices().size())))
    return failure();

  if (!hasWorkgroupMemorySpace(memrefType.getMemorySpace()))
    return op.emitOpError("barrier must be in workgroup (LDS) memory");

  return success();
}

LogicalResult DsBarrierInitOp::verify() {
  return verifyDsBarrierOpCommon(*this);
}

```
- **EN**: Implements logic around `verifyDsBarrierOpCommon`, `cast`, `failed`, `verifyIndexCount`, and 5 more symbols; this block implements verifier, folding, parsing, or printing hooks; encodes accelerator-specific execution or lowering rules.
- **CN**: 围绕 `verifyDsBarrierOpCommon`, `cast`, `failed`, `verifyIndexCount`, and 5 more symbols 实现具体逻辑；该代码块实现验证、折叠、解析或打印钩子，并编码加速器专用执行或 lowering 规则。

### Lines 1502-1520
```cpp
LogicalResult DsBarrierPollStateOp::verify() {
  return verifyDsBarrierOpCommon(*this);
}

LogicalResult DsAsyncBarrierArriveOp::verify() {
  return verifyDsBarrierOpCommon(*this);
}

LogicalResult DsBarrierArriveOp::verify() {
  return verifyDsBarrierOpCommon(*this);
}

//===----------------------------------------------------------------------===//
// GlobalPrefetchOp
//===----------------------------------------------------------------------===//

LogicalResult GlobalPrefetchOp::verify() {
  auto src = cast<MemRefType>(getSrc().getType());

```
- **EN**: Implements logic around `verify`, `verifyDsBarrierOpCommon`, `cast`; this block implements verifier, folding, parsing, or printing hooks.
- **CN**: 围绕 `verify`, `verifyDsBarrierOpCommon`, `cast` 实现具体逻辑；该代码块实现验证、折叠、解析或打印钩子。

### Lines 1521-1538
```cpp
  if (failed(verifyIndexCount(*this, "source", src, getIndices().size())))
    return failure();

  Attribute memSpace = src.getMemorySpace();
  if (!memSpace)
    return this->emitOpError("the source must have address space attribute");
  if (!hasGlobalMemorySpace(memSpace))
    return this->emitOpError("the source must reside in global address space");

  const LoadTemporalHint temporalHint = getTemporalHint();
  const Scope scope = getCacheScope();
  const bool isSpeculative = getSpeculative();

  // See GFX1250 SPG for a detail explanation
  if (isSpeculative && scope == Scope::WGP)
    return this->emitOpError(
        "does not support speculative prefetch in WGP scope");

```
- **EN**: Implements logic around `failed`, `failure`, `getMemorySpace`, `emitOpError`, and 4 more symbols.
- **CN**: 围绕 `failed`, `failure`, `getMemorySpace`, `emitOpError`, and 4 more symbols 实现具体逻辑。

### Lines 1539-1557
```cpp
  // Note that temporal hints are shared between load, store,
  // prefetch, etc. instructions. However, some instructions
  // operate only with a subset of hints according to the ISA
  // documentation. In case of global prefetch, non-temporal (NT)
  // and last-use (LU) hints are not used. The extra bits of encoding
  // are used to encode speculative or non-speculative instruction behavior
  if (llvm::is_contained({LoadTemporalHint::NT, LoadTemporalHint::LU},
                         temporalHint))
    return this->emitOpError("does not support NT and LU modes");

  if (llvm::is_contained({LoadTemporalHint::NT_RT, LoadTemporalHint::RT_NT,
                          LoadTemporalHint::NT_HT},
                         temporalHint) &&
      !isSpeculative) {
    return this->emitOpError("operates only in the speculative mode");
  }
  return success();
}

```
- **EN**: Implements logic around `is_contained`, `emitOpError`, `success`.
- **CN**: 围绕 `is_contained`, `emitOpError`, `success` 实现具体逻辑。

### Lines 1558-1559
```cpp
#define GET_OP_CLASSES
#include "mlir/Dialect/AMDGPU/IR/AMDGPU.cpp.inc"
```
- **EN**: Bridges to TableGen-generated declarations or definitions that expand MLIR op/type/interface metadata.
- **CN**: 桥接到由 TableGen 生成的声明或定义，用于展开 MLIR 操作/类型/接口元数据。

## Key Concepts / 关键概念

- **Dialect IR modeling / 方言 IR 建模**:
  - **EN**: Defines operations, attributes, types, verifiers, parsers, and printers for a dialect.
  - **CN**: 定义方言的操作、属性、类型、验证器、解析器与打印器。
- **GPU execution model / GPU 执行模型**:
  - **EN**: Models kernels, memory spaces, launch geometry, and accelerator-specific rewrites.
  - **CN**: 建模 kernel、内存空间、启动几何以及加速器专用重写。
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

## Dependencies / 依赖关系

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/AMDGPU/IR/AMDGPUDialect.h`, `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/GPU/IR/GPUDialect.h`, `mlir/Dialect/LLVMIR/ROCDLDialect.h`, `mlir/Dialect/MemRef/IR/MemRef.h`, `mlir/Dialect/MemRef/Utils/MemRefUtils.h`, `mlir/Dialect/Utils/IndexingUtils.h`, `mlir/Dialect/Vector/IR/VectorOps.h`, `mlir/IR/Builders.h`, `mlir/IR/BuiltinTypes.h` ... (+8 more)
- **Standard-library headers / 标准库头文件**: `<algorithm>`, `<cstdint>`, `<limits>`, `<optional>`
- **Subsystem categories / 子系统类别**: other MLIR dialect declarations / 其他 MLIR 方言声明 (9), MLIR IR core abstractions / MLIR IR 核心抽象 (7), LLVM ADT containers and utility types / LLVM ADT 容器与工具类型 (2)
- **Generated macros / 生成宏**: `GET_OP_CLASSES`
