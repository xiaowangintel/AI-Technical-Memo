# Promotion.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/Linalg/Transforms/Promotion.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This file implements the linalg dialect Promotion pass.
  - **CN**: 该文件位于 `mlir/lib/Dialect/Linalg/Transforms`，围绕 Linalg 方言实现相关能力。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```cpp
//===- Promotion.cpp - Implementation of linalg Promotion -----------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements the linalg dialect Promotion pass.
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and file-level summary.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件级摘要。

### Lines 12-30
```cpp

#include "mlir/Dialect/Arith/IR/Arith.h"
#include "mlir/Dialect/Arith/Utils/Utils.h"
#include "mlir/Dialect/Complex/IR/Complex.h"
#include "mlir/Dialect/Func/IR/FuncOps.h"
#include "mlir/Dialect/GPU/IR/GPUDialect.h"
#include "mlir/Dialect/Linalg/IR/Linalg.h"
#include "mlir/Dialect/Linalg/Passes.h"
#include "mlir/Dialect/Linalg/Transforms/Transforms.h"
#include "mlir/IR/AffineMap.h"
#include "mlir/Interfaces/ValueBoundsOpInterface.h"
#include "mlir/Support/LLVM.h"
#include "mlir/Transforms/FoldUtils.h"
#include "llvm/ADT/MapVector.h"
#include "llvm/ADT/SmallBitVector.h"
#include "llvm/ADT/SmallSet.h"
#include "llvm/ADT/TypeSwitch.h"
#include "llvm/Support/Debug.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/Arith/Utils/Utils.h`, `mlir/Dialect/Complex/IR/Complex.h`, `mlir/Dialect/Func/IR/FuncOps.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/Arith/Utils/Utils.h`, `mlir/Dialect/Complex/IR/Complex.h`, `mlir/Dialect/Func/IR/FuncOps.h`。

### Lines 31-40
```cpp
using namespace mlir;
using namespace mlir::linalg;
using namespace mlir::scf;

using llvm::MapVector;

#define DEBUG_TYPE "linalg-promotion"

/// Alloc a new buffer of `size` * `width` i8; where `width` is given by the
/// data `layout` for `elementType`.
```
- **EN**: Defines preprocessor-controlled structure, generated hook points, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、生成式钩子或编译期常量。

### Lines 41-53
```cpp
/// Use AllocOp or AllocaOp depending on `options`.
/// Take an optional alignment.
static Value allocBuffer(ImplicitLocOpBuilder &b,
                         const LinalgPromotionOptions &options,
                         Type elementType, Value allocSize, DataLayout &layout,
                         std::optional<unsigned> alignment = std::nullopt) {
  llvm::TypeSize width = layout.getTypeSize(elementType);
  assert(!width.isScalable() && "cannot allocate buffer for a scalable vector");

  IntegerAttr alignmentAttr;
  if (alignment.has_value())
    alignmentAttr = b.getI64IntegerAttr(alignment.value());

```
- **EN**: Implements logic around `allocBuffer`, `getTypeSize`, `assert`, `has_value`, and 1 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `allocBuffer`, `getTypeSize`, `assert`, `has_value`, and 1 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 54-71
```cpp
  Attribute memorySpaceAttr;
  if (options.memorySpace.has_value())
    memorySpaceAttr = *options.memorySpace;

  // Static buffer.
  if (std::optional<int64_t> cst = getConstantIntValue(allocSize)) {
    auto staticBufferType = MemRefType::get(width.getFixedValue() * cst.value(),
                                            b.getIntegerType(8));
    staticBufferType =
        MemRefType::Builder(staticBufferType).setMemorySpace(memorySpaceAttr);
    if (options.useAlloca) {
      return memref::AllocaOp::create(b, staticBufferType, ValueRange{},
                                      alignmentAttr);
    }
    return memref::AllocOp::create(b, staticBufferType, ValueRange{},
                                   alignmentAttr);
  }

```
- **EN**: Implements logic around `has_value`, `getConstantIntValue`, `get`, `getIntegerType`, and 2 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `has_value`, `getConstantIntValue`, `get`, `getIntegerType`, and 2 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 72-83
```cpp
  // Fallback dynamic buffer.
  auto dynamicBufferType =
      MemRefType::get(ShapedType::kDynamic, b.getIntegerType(8));
  dynamicBufferType =
      MemRefType::Builder(dynamicBufferType).setMemorySpace(memorySpaceAttr);
  Value mul = b.createOrFold<arith::MulIOp>(
      arith::ConstantIndexOp::create(b, width), allocSize);
  if (options.useAlloca)
    return memref::AllocaOp::create(b, dynamicBufferType, mul, alignmentAttr);
  return memref::AllocOp::create(b, dynamicBufferType, mul, alignmentAttr);
}

```
- **EN**: Implements logic around `get`, `Builder`, `MulIOp>`, `create`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `get`, `Builder`, `MulIOp>`, `create` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 84-96
```cpp
/// Default allocation callback function. This allocates a promoted buffer when
/// no call back to do so is provided. The default is to allocate a
/// memref<..xi8> and return a view to get a memref type of shape
/// boundingSubViewSize.
static std::optional<Value> defaultAllocBufferCallBack(
    const LinalgPromotionOptions &options, OpBuilder &builder,
    memref::SubViewOp subView, ArrayRef<Value> boundingSubViewSize,
    std::optional<unsigned> alignment, DataLayout &layout) {
  ShapedType viewType = subView.getType();
  ImplicitLocOpBuilder b(subView.getLoc(), builder);
  auto zero = arith::ConstantIndexOp::create(b, 0);
  auto one = arith::ConstantIndexOp::create(b, 1);

```
- **EN**: Implements logic around `defaultAllocBufferCallBack`, `getType`, `b`, `create`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `defaultAllocBufferCallBack`, `getType`, `b`, `create` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 97-108
```cpp
  Attribute memorySpaceAttr;
  if (options.memorySpace.has_value())
    memorySpaceAttr = *options.memorySpace;

  Value allocSize = one;
  for (const auto &size : llvm::enumerate(boundingSubViewSize))
    allocSize = b.createOrFold<arith::MulIOp>(allocSize, size.value());
  Value buffer = allocBuffer(b, options, viewType.getElementType(), allocSize,
                             layout, alignment);
  SmallVector<int64_t, 4> dynSizes(boundingSubViewSize.size(),
                                   ShapedType::kDynamic);

```
- **EN**: Implements logic around `has_value`, `enumerate`, `MulIOp>`, `allocBuffer`, and 1 more symbols.
- **CN**: 围绕 `has_value`, `enumerate`, `MulIOp>`, `allocBuffer`, and 1 more symbols 实现具体逻辑。

### Lines 109-118
```cpp
  auto viewMemRefType = MemRefType::get(dynSizes, viewType.getElementType());
  viewMemRefType =
      MemRefType::Builder(viewMemRefType).setMemorySpace(memorySpaceAttr);
  Value view = b.createOrFold<memref::ViewOp>(viewMemRefType, buffer, zero,
                                              boundingSubViewSize);
  return view;
}

/// Default implementation of deallocation of the buffer use for promotion. It
/// expects to get the same value that the default allocation method returned,
```
- **EN**: Implements logic around `get`, `Builder`, `ViewOp>`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `get`, `Builder`, `ViewOp>` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 119-130
```cpp
/// i.e. result of a ViewOp.
static LogicalResult
defaultDeallocBufferCallBack(const LinalgPromotionOptions &options,
                             OpBuilder &b, Value fullLocalView) {
  if (!options.useAlloca) {
    auto viewOp = cast<memref::ViewOp>(fullLocalView.getDefiningOp());
    memref::DeallocOp::create(b, viewOp.getSource().getLoc(),
                              viewOp.getSource());
  }
  return success();
}

```
- **EN**: Implements logic around `defaultDeallocBufferCallBack`, `ViewOp>`, `create`, `getSource`, and 1 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `defaultDeallocBufferCallBack`, `ViewOp>`, `create`, `getSource`, and 1 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 131-140
```cpp
namespace {

/// Helper struct that captures the information required to apply the
/// transformation on each op. This bridges the abstraction gap with the
/// user-facing API which exposes positional arguments to control which operands
/// are promoted.
struct LinalgOpInstancePromotionOptions {
  LinalgOpInstancePromotionOptions(LinalgOp op,
                                   const LinalgPromotionOptions &options);
  /// SubViews to promote.
```
- **EN**: Introduces declarations for `that`, `LinalgOpInstancePromotionOptions`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `that`, `LinalgOpInstancePromotionOptions` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 141-150
```cpp
  MapVector<int64_t, Value> subViews;
  /// Subviews operand numbers to copy in using copyInFn.
  llvm::SmallSet<int64_t, 4> operandsNumbersToCopyIn;
  /// True if the full view should be used for the promoted buffer.
  DenseMap<Value, bool> useFullTileBuffers;
  /// True if the original subview size should be used. This means the full tile
  /// buffer is the same size as the partial view.
  bool useOriginalSubviewSize;

  /// Callback functions for allocation and deallocation of promoted buffers, as
```
- **EN**: Contains supporting implementation details for the surrounding MLIR dialect component.
- **CN**: 包含周边 MLIR 方言组件所需的辅助实现细节。

### Lines 151-161
```cpp
  /// well as to copy the data into and out of these buffers.
  AllocBufferCallbackFn allocationFn;
  DeallocBufferCallbackFn deallocationFn;
  CopyCallbackFn copyInFn;
  CopyCallbackFn copyOutFn;

  /// Alignment of promoted buffer.
  std::optional<unsigned> alignment;
};
} // namespace

```
- **EN**: Contains supporting implementation details for the surrounding MLIR dialect component.
- **CN**: 包含周边 MLIR 方言组件所需的辅助实现细节。

### Lines 162-172
```cpp
LinalgOpInstancePromotionOptions::LinalgOpInstancePromotionOptions(
    LinalgOp linalgOp, const LinalgPromotionOptions &options)
    : subViews(), alignment(options.alignment) {
  assert(linalgOp.hasPureBufferSemantics() &&
         "revisit usage of shaped operand");
  auto vUseFullTileBuffers =
      options.useFullTileBuffers.value_or(llvm::SmallBitVector());
  vUseFullTileBuffers.resize(linalgOp->getNumOperands(),
                             options.useFullTileBuffersDefault);
  useOriginalSubviewSize = options.useOriginalSubviewSize;

```
- **EN**: Implements logic around `LinalgOpInstancePromotionOptions`, `subViews`, `assert`, `value_or`, and 1 more symbols.
- **CN**: 围绕 `LinalgOpInstancePromotionOptions`, `subViews`, `assert`, `value_or`, and 1 more symbols 实现具体逻辑。

### Lines 173-189
```cpp
  for (OpOperand &opOperand : linalgOp->getOpOperands()) {
    int64_t operandNumber = opOperand.getOperandNumber();
    if (options.operandsToPromote &&
        !options.operandsToPromote->count(operandNumber))
      continue;
    Operation *op = opOperand.get().getDefiningOp();
    if (auto sv = dyn_cast_or_null<memref::SubViewOp>(op)) {
      subViews[operandNumber] = sv;
      // In case of linalg generic, copy in only if subview is used in linalg
      // payload.
      if (!isa<linalg::GenericOp>(linalgOp) ||
          linalgOp.payloadUsesValueFromOperand(&opOperand))
        operandsNumbersToCopyIn.insert(operandNumber);
      useFullTileBuffers[sv] = vUseFullTileBuffers[operandNumber];
    }
  }

```
- **EN**: Implements logic around `getOpOperands`, `getOperandNumber`, `count`, `get`, and 4 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getOpOperands`, `getOperandNumber`, `count`, `get`, and 4 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 190-200
```cpp
  if (options.allocationFn) {
    allocationFn = *options.allocationFn;
  } else {
    allocationFn = [&](OpBuilder &b, memref::SubViewOp subViewOp,
                       ArrayRef<Value> boundingSubViewSize,
                       DataLayout &layout) -> std::optional<Value> {
      return defaultAllocBufferCallBack(options, b, subViewOp,
                                        boundingSubViewSize, alignment, layout);
    };
  }

```
- **EN**: Implements logic around `defaultAllocBufferCallBack`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `defaultAllocBufferCallBack` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 201-219
```cpp
  if (options.deallocationFn) {
    deallocationFn = *options.deallocationFn;
  } else {
    deallocationFn = [&](OpBuilder &b, Value buffer) {
      return defaultDeallocBufferCallBack(options, b, buffer);
    };
  }

  // Save the loc because `linalgOp` goes out of scope.
  Location loc = linalgOp.getLoc();
  auto defaultCopyCallBack = [loc](OpBuilder &b, Value src,
                                   Value dst) -> LogicalResult {
    linalg::CopyOp::create(b, loc, src, dst);
    return success();
  };
  copyInFn = (options.copyInFn ? *(options.copyInFn) : defaultCopyCallBack);
  copyOutFn = (options.copyOutFn ? *(options.copyOutFn) : defaultCopyCallBack);
}

```
- **EN**: Implements logic around `defaultDeallocBufferCallBack`, `getLoc`, `create`, `success`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `defaultDeallocBufferCallBack`, `getLoc`, `create`, `success` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 220-239
```cpp
// Performs promotion of a `subView` into a local buffer of the size of the
// *ranges* of the `subView`. This produces a buffer whose size may be bigger
// than the actual size of the `subView` at the boundaries.
// This is related to the full/partial tile problem.
// Returns a PromotionInfo containing a `buffer`, `fullLocalView` and
// `partialLocalView` such that:
//   * `buffer` is always the size of the full tile.
//   * `fullLocalView` is a dense contiguous view into that buffer.
//   * `partialLocalView` is a dense non-contiguous slice of `fullLocalView`
//     that corresponds to the size of `subView` and accounting for boundary
//     effects.
// The point of the full tile buffer is that constant static tile sizes are
// folded and result in a buffer type with statically known size and alignment
// properties.
// To account for general boundary effects, padding must be performed on the
// boundary tiles. For now this is done with an unconditional `fill` op followed
// by a partial `copy` op.
FailureOr<PromotionInfo> mlir::linalg::promoteSubviewAsNewBuffer(
    OpBuilder &b, Location loc, memref::SubViewOp subView,
    bool useOriginalSubviewSize, const AllocBufferCallbackFn &allocationFn,
```
- **EN**: Implements logic around `promoteSubviewAsNewBuffer`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `promoteSubviewAsNewBuffer` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 240-259
```cpp
    DataLayout &layout) {
  auto viewType = subView.getType();
  auto rank = viewType.getRank();
  SmallVector<Value, 4> fullSizes;
  SmallVector<OpFoldResult> partialSizes;
  fullSizes.reserve(rank);
  partialSizes.reserve(rank);
  llvm::SmallBitVector droppedDims = subView.getDroppedDims();
  int64_t resultDimIdx = 0;
  for (const auto &en : llvm::enumerate(subView.getOrCreateRanges(b, loc))) {
    if (droppedDims[en.index()])
      continue;
    auto rangeValue = en.value();
    // Try to extract a tight constant. If the size is known statically, no need
    // to look for the bound.
    LLVM_DEBUG(llvm::dbgs() << "Extract tightest: " << rangeValue.size << "\n");
    Value size;
    if (llvm::isa_and_present<Attribute>(rangeValue.size) ||
        useOriginalSubviewSize) {
      size = getValueOrCreateConstantIndexOp(b, loc, rangeValue.size);
```
- **EN**: Implements logic around `getType`, `getRank`, `reserve`, `getDroppedDims`, and 6 more symbols.
- **CN**: 围绕 `getType`, `getRank`, `reserve`, `getDroppedDims`, and 6 more symbols 实现具体逻辑。

### Lines 260-279
```cpp
    } else {
      FailureOr<int64_t> upperBound =
          ValueBoundsConstraintSet::computeConstantBound(
              presburger::BoundType::UB, rangeValue.size,
              /*stopCondition=*/nullptr, ValueBoundsOptions{/*closedUB=*/true});
      size = failed(upperBound)
                 ? getValueOrCreateConstantIndexOp(b, loc, rangeValue.size)
                 : arith::ConstantIndexOp::create(b, loc, *upperBound);
    }
    LLVM_DEBUG(llvm::dbgs() << "Extracted tightest: " << size << "\n");
    fullSizes.push_back(size);
    partialSizes.push_back(
        b.createOrFold<memref::DimOp>(loc, subView, resultDimIdx++));
  }
  // If a callback is not specified, then use the default implementation for
  // allocating the promoted buffer.
  std::optional<Value> fullLocalView =
      allocationFn(b, subView, fullSizes, layout);
  if (!fullLocalView)
    return failure();
```
- **EN**: Implements logic around `computeConstantBound`, `failed`, `getValueOrCreateConstantIndexOp`, `create`, and 5 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `computeConstantBound`, `failed`, `getValueOrCreateConstantIndexOp`, `create`, and 5 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 280-292
```cpp
  SmallVector<OpFoldResult, 4> zeros(fullSizes.size(), b.getIndexAttr(0));
  SmallVector<OpFoldResult, 4> ones(fullSizes.size(), b.getIndexAttr(1));
  auto partialLocalView = b.createOrFold<memref::SubViewOp>(
      loc, *fullLocalView, zeros, partialSizes, ones);
  return PromotionInfo{*fullLocalView, partialLocalView};
}

static FailureOr<MapVector<int64_t, PromotionInfo>>
promoteSubViews(ImplicitLocOpBuilder &b,
                LinalgOpInstancePromotionOptions options, DataLayout &layout) {
  if (options.subViews.empty())
    return failure();

```
- **EN**: Implements logic around `zeros`, `ones`, `SubViewOp>`, `promoteSubViews`, and 2 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `zeros`, `ones`, `SubViewOp>`, `promoteSubViews`, and 2 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 293-304
```cpp
  MapVector<int64_t, PromotionInfo> promotionInfoMap;

  for (auto v : options.subViews) {
    memref::SubViewOp subView =
        cast<memref::SubViewOp>(v.second.getDefiningOp());
    auto promotionInfo = promoteSubviewAsNewBuffer(
        b, b.getLoc(), subView, options.useOriginalSubviewSize,
        options.allocationFn, layout);
    if (failed(promotionInfo))
      return failure();
    promotionInfoMap[v.first] = *promotionInfo;

```
- **EN**: Implements logic around `SubViewOp>`, `promoteSubviewAsNewBuffer`, `getLoc`, `failed`, and 1 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `SubViewOp>`, `promoteSubviewAsNewBuffer`, `getLoc`, `failed`, and 1 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 305-324
```cpp
    // Only fill the buffer if the full local view is used
    if (!options.useFullTileBuffers[v.second])
      continue;
    Type subviewEltType = subView.getType().getElementType();
    Value fillVal =
        llvm::TypeSwitch<Type, Value>(subviewEltType)
            .Case([&](FloatType t) {
              return arith::ConstantOp::create(b, FloatAttr::get(t, 0.0));
            })
            .Case([&](IntegerType t) {
              return arith::ConstantOp::create(b, IntegerAttr::get(t, 0));
            })
            .Case([&](ComplexType t) {
              Value tmp;
              if (auto et = dyn_cast<FloatType>(t.getElementType()))
                tmp = arith::ConstantOp::create(b, FloatAttr::get(et, 0.0));
              else if (auto et = cast<IntegerType>(t.getElementType()))
                tmp = arith::ConstantOp::create(b, IntegerAttr::get(et, 0));
              return complex::CreateOp::create(b, t, tmp, tmp);
            })
```
- **EN**: Implements logic around `getType`, `Value>`, `Case`, `create`, and 1 more symbols.
- **CN**: 围绕 `getType`, `Value>`, `Case`, `create`, and 1 more symbols 实现具体逻辑。

### Lines 325-344
```cpp
            .Default(nullptr);
    if (!fillVal)
      return failure();
    linalg::FillOp::create(b, fillVal, promotionInfo->fullLocalView);
  }

  // Copy data into the promoted buffers. Use callback if provided.
  for (auto v : options.subViews) {
    auto *info = promotionInfoMap.find(v.first);
    if (info == promotionInfoMap.end())
      continue;
    if (options.operandsNumbersToCopyIn.count(v.first) == 0)
      continue;
    if (failed(options.copyInFn(
            b, cast<memref::SubViewOp>(v.second.getDefiningOp()),
            info->second.partialLocalView)))
      return failure();
  }
  return promotionInfoMap;
}
```
- **EN**: Implements logic around `Default`, `failure`, `create`, `find`, and 4 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `Default`, `failure`, `create`, `find`, and 4 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 345-357
```cpp

static FailureOr<LinalgOp>
promoteSubViews(ImplicitLocOpBuilder &b, LinalgOp op,
                LinalgOpInstancePromotionOptions options, DataLayout &layout) {
  assert(op.hasPureBufferSemantics() &&
         "expected linalg op with buffer semantics");

  // 1. Promote the specified views and use them in the new op.
  auto promotedBuffersAndViews = promoteSubViews(b, options, layout);
  if (failed(promotedBuffersAndViews) ||
      promotedBuffersAndViews->size() != options.subViews.size())
    return failure();

```
- **EN**: Implements logic around `promoteSubViews`, `assert`, `failed`, `size`, and 1 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `promoteSubViews`, `assert`, `failed`, `size`, and 1 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 358-377
```cpp
  // 2. Append all other operands as they appear, this enforces that such
  // operands are not views. This is to support cases such as FillOp taking
  // extra scalars etc.  Keep a reference to output buffers;
  SmallVector<Value, 8> opViews;
  opViews.reserve(op->getNumOperands());
  SmallVector<std::pair<Value, Value>, 8> writebackViews;
  writebackViews.reserve(promotedBuffersAndViews->size());
  for (OpOperand &opOperand : op->getOpOperands()) {
    int64_t operandNumber = opOperand.getOperandNumber();
    if (options.subViews.count(operandNumber) != 0) {
      if (options.useFullTileBuffers[opOperand.get()])
        opViews.push_back(
            (*promotedBuffersAndViews)[operandNumber].fullLocalView);
      else
        opViews.push_back(
            (*promotedBuffersAndViews)[operandNumber].partialLocalView);
      if (operandNumber >= op.getNumDpsInputs())
        writebackViews.emplace_back(std::make_pair(
            opOperand.get(),
            (*promotedBuffersAndViews)[operandNumber].partialLocalView));
```
- **EN**: Implements logic around `reserve`, `getOpOperands`, `getOperandNumber`, `count`, and 4 more symbols.
- **CN**: 围绕 `reserve`, `getOpOperands`, `getOperandNumber`, `count`, and 4 more symbols 实现具体逻辑。

### Lines 378-392
```cpp
    } else {
      opViews.push_back(opOperand.get());
    }
  }
  op->setOperands(0, opViews.size(), opViews);

  OpBuilder::InsertionGuard guard(b);
  b.setInsertionPointAfter(op);
  // 3. Emit write-back for the promoted output views: copy the partial view.
  for (auto viewAndPartialLocalView : writebackViews) {
    if (failed(options.copyOutFn(b, viewAndPartialLocalView.second,
                                 viewAndPartialLocalView.first)))
      return failure();
  }

```
- **EN**: Implements logic around `push_back`, `setOperands`, `guard`, `setInsertionPointAfter`, and 2 more symbols.
- **CN**: 围绕 `push_back`, `setOperands`, `guard`, `setInsertionPointAfter`, and 2 more symbols 实现具体逻辑。

### Lines 393-412
```cpp
  // 4. Dealloc all local buffers.
  for (const auto &pi : *promotedBuffersAndViews)
    (void)options.deallocationFn(b, pi.second.fullLocalView);
  return op;
}

LogicalResult
mlir::linalg::promoteSubviewsPrecondition(Operation *op,
                                          LinalgPromotionOptions options) {
  LinalgOp linalgOp = dyn_cast<LinalgOp>(op);
  // Transformation applies to buffers only.
  if (!linalgOp || !linalgOp.hasPureBufferSemantics())
    return failure();
  // Check that at least one of the requested operands is indeed a subview.
  for (OpOperand &opOperand : linalgOp->getOpOperands()) {
    auto sv =
        isa_and_nonnull<memref::SubViewOp>(opOperand.get().getDefiningOp());
    if (sv) {
      if (!options.operandsToPromote ||
          options.operandsToPromote->count(opOperand.getOperandNumber()))
```
- **EN**: Implements logic around `deallocationFn`, `promoteSubviewsPrecondition`, `hasPureBufferSemantics`, `failure`, and 3 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `deallocationFn`, `promoteSubviewsPrecondition`, `hasPureBufferSemantics`, `failure`, and 3 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 413-432
```cpp
        return success();
    }
  }
  // TODO: Check all subviews requested are bound by a static constant.
  // TODO: Check that the total footprint fits within a given size.
  return failure();
}

FailureOr<LinalgOp>
mlir::linalg::promoteSubViews(OpBuilder &builder, LinalgOp linalgOp,
                              const LinalgPromotionOptions &options) {
  LinalgOpInstancePromotionOptions linalgOptions(linalgOp, options);
  auto layout = DataLayout::closest(linalgOp);
  ImplicitLocOpBuilder b(linalgOp.getLoc(), builder);
  auto res = ::promoteSubViews(b, linalgOp, linalgOptions, layout);
  if (failed(res))
    return failure();
  return res;
}

```
- **EN**: Implements logic around `success`, `failure`, `promoteSubViews`, `linalgOptions`, and 3 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `success`, `failure`, `promoteSubViews`, `linalgOptions`, and 3 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 433-444
```cpp
/// Allocate the given subview to a memory address space in GPU by creating a
/// allocation operation and setting the memref type address space to desired
/// address space.
static std::optional<Value> allocateSubviewGPUMemoryInAddressSpace(
    OpBuilder &builder, memref::SubViewOp subview, ArrayRef<Value> sizeBounds,
    gpu::AddressSpace addressSpace) {
  OpBuilder::InsertionGuard guard(builder);

  func::FuncOp funcOp = subview->getParentOfType<func::FuncOp>();
  if (!funcOp)
    return std::nullopt;

```
- **EN**: Implements logic around `allocateSubviewGPUMemoryInAddressSpace`, `guard`, `FuncOp>`; this block touches target- or accelerator-specific semantics; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `allocateSubviewGPUMemoryInAddressSpace`, `guard`, `FuncOp>` 实现具体逻辑；该代码块涉及目标平台或加速器专用语义，并协调核心结构化 MLIR 方言之间的行为。

### Lines 445-454
```cpp
  // The subview size bounds are expected to be constant; they specify the shape
  // of the allocation.
  SmallVector<int64_t> shape;
  for (Value bound : sizeBounds) {
    APInt value;
    if (!matchPattern(bound, m_ConstantInt(&value)))
      return std::nullopt;
    shape.push_back(value.getSExtValue());
  }

```
- **EN**: Implements logic around `matchPattern`, `push_back`.
- **CN**: 围绕 `matchPattern`, `push_back` 实现具体逻辑。

### Lines 455-469
```cpp
  builder.setInsertionPointToStart(&funcOp.front());
  auto type = MemRefType::get(
      shape, subview.getType().getElementType(), MemRefLayoutAttrInterface{},
      gpu::AddressSpaceAttr::get(builder.getContext(), addressSpace));
  Value buffer;
  if (addressSpace == gpu::GPUDialect::getWorkgroupAddressSpace()) {
    buffer = memref::AllocOp::create(builder, funcOp.getLoc(), type);
  } else if (addressSpace == gpu::GPUDialect::getPrivateAddressSpace()) {
    buffer = memref::AllocaOp::create(builder, funcOp.getLoc(), type);
  } else {
    return std::nullopt;
  }
  return buffer;
}

```
- **EN**: Implements logic around `setInsertionPointToStart`, `get`, `getType`, `getWorkgroupAddressSpace`, and 2 more symbols; this block touches target- or accelerator-specific semantics; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `setInsertionPointToStart`, `get`, `getType`, `getWorkgroupAddressSpace`, and 2 more symbols 实现具体逻辑；该代码块涉及目标平台或加速器专用语义，并协调核心结构化 MLIR 方言之间的行为。

### Lines 470-479
```cpp
/// Allocate the subview in the GPU workgroup memory.
std::optional<Value> mlir::linalg::allocateWorkgroupMemory(
    OpBuilder &builder, memref::SubViewOp subview, ArrayRef<Value> sizeBounds,
    DataLayout &) {
  return allocateSubviewGPUMemoryInAddressSpace(
      builder, subview, sizeBounds,
      gpu::GPUDialect::getWorkgroupAddressSpace());
}

/// In case of GPU group memory there is no need to deallocate.
```
- **EN**: Implements logic around `allocateWorkgroupMemory`, `allocateSubviewGPUMemoryInAddressSpace`, `getWorkgroupAddressSpace`; this block touches target- or accelerator-specific semantics; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `allocateWorkgroupMemory`, `allocateSubviewGPUMemoryInAddressSpace`, `getWorkgroupAddressSpace` 实现具体逻辑；该代码块涉及目标平台或加速器专用语义，并协调核心结构化 MLIR 方言之间的行为。

### Lines 480-494
```cpp
LogicalResult mlir::linalg::deallocateWorkgroupMemory(OpBuilder &,
                                                      Value /*buffer*/) {
  return success();
}

/// Create Memref copy operations and add gpu barrier guards before and after
/// the copy operation to ensure data integrity.
LogicalResult mlir::linalg::copyToWorkgroupMemory(OpBuilder &b, Value src,
                                                  Value dst) {
  gpu::BarrierOp::create(b, src.getLoc());
  Operation *copyOp = memref::CopyOp::create(b, src.getLoc(), src, dst);
  gpu::BarrierOp::create(b, copyOp->getLoc());
  return success();
}

```
- **EN**: Implements logic around `deallocateWorkgroupMemory`, `success`, `copyToWorkgroupMemory`, `create`; this block touches target- or accelerator-specific semantics; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `deallocateWorkgroupMemory`, `success`, `copyToWorkgroupMemory`, `create` 实现具体逻辑；该代码块涉及目标平台或加速器专用语义，并协调核心结构化 MLIR 方言之间的行为。

### Lines 495-509
```cpp
/// Allocate the subview in the GPU private memory.
std::optional<Value> mlir::linalg::allocateGPUPrivateMemory(
    OpBuilder &builder, memref::SubViewOp subview, ArrayRef<Value> sizeBounds,
    DataLayout &) {
  return allocateSubviewGPUMemoryInAddressSpace(
      builder, subview, sizeBounds, gpu::GPUDialect::getPrivateAddressSpace());
}

/// Normal copy to between src and dst.
LogicalResult mlir::linalg::copyToGPUPrivateMemory(OpBuilder &b, Value src,
                                                   Value dst) {
  memref::CopyOp::create(b, src.getLoc(), src, dst);
  return success();
}

```
- **EN**: Implements logic around `allocateGPUPrivateMemory`, `allocateSubviewGPUMemoryInAddressSpace`, `getPrivateAddressSpace`, `copyToGPUPrivateMemory`, and 2 more symbols; this block touches target- or accelerator-specific semantics; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `allocateGPUPrivateMemory`, `allocateSubviewGPUMemoryInAddressSpace`, `getPrivateAddressSpace`, `copyToGPUPrivateMemory`, and 2 more symbols 实现具体逻辑；该代码块涉及目标平台或加速器专用语义，并协调核心结构化 MLIR 方言之间的行为。

### Lines 510-515
```cpp
/// In case of GPU private memory there is no need to deallocate since the
/// memory is freed when going outside of the scope.
LogicalResult mlir::linalg::deallocateGPUPrivateMemory(OpBuilder &,
                                                       Value /*buffer*/) {
  return success();
}
```
- **EN**: Implements logic around `deallocateGPUPrivateMemory`, `success`; this block touches target- or accelerator-specific semantics; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `deallocateGPUPrivateMemory`, `success` 实现具体逻辑；该代码块涉及目标平台或加速器专用语义，并协调核心结构化 MLIR 方言之间的行为。

## Key Concepts / 关键概念

- **Rewrite and lowering / 重写与 lowering**:
  - **EN**: Uses MLIR pattern infrastructure or passes to canonicalize, legalize, or lower operations.
  - **CN**: 使用 MLIR 模式基础设施或 Pass 对操作进行规范化、合法化或 lowering。
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

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/Arith/Utils/Utils.h`, `mlir/Dialect/Complex/IR/Complex.h`, `mlir/Dialect/Func/IR/FuncOps.h`, `mlir/Dialect/GPU/IR/GPUDialect.h`, `mlir/Dialect/Linalg/IR/Linalg.h`, `mlir/Dialect/Linalg/Passes.h`, `mlir/Dialect/Linalg/Transforms/Transforms.h`, `mlir/IR/AffineMap.h`, `mlir/Interfaces/ValueBoundsOpInterface.h` ... (+7 more)
- **Subsystem categories / 子系统类别**: dialect-specific operation, type, attribute, or transform declarations / 方言相关的操作、类型、属性或变换声明 (8), LLVM ADT containers and low-level helpers / LLVM ADT 容器与底层辅助工具 (4), MLIR core IR types and infrastructural utilities / MLIR 核心 IR 类型与基础设施工具 (1), cross-dialect interfaces and capability contracts / 跨方言接口与能力约定 (1), support utilities and LLVM interop helpers / 支持工具与 LLVM 互操作辅助逻辑 (1), generic transformation and conversion helpers / 通用变换与转换辅助逻辑 (1), LLVM support-library facilities / LLVM Support 库设施 (1)
