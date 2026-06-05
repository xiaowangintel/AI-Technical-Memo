# LegalizeVectorStorage.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/ArmSVE/Transforms/LegalizeVectorStorage.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements rewrite patterns and transformation passes for the ArmSVE dialect and scalable-vector support.
  - **CN**: 实现 ArmSVE 方言与可扩展向量支持 的重写模式与变换 pass。

## Line-by-Line Analysis / 逐行分析

### Lines 1-15
```cpp
//===- LegalizeVectorStorage.cpp - Ensures SVE loads/stores are legal -----===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "mlir/Dialect/ArmSVE/IR/ArmSVEDialect.h"
#include "mlir/Dialect/ArmSVE/Transforms/Passes.h"
#include "mlir/Dialect/Func/IR/FuncOps.h"
#include "mlir/Dialect/MemRef/IR/MemRef.h"
#include "mlir/Dialect/Vector/IR/VectorOps.h"
#include "mlir/Transforms/GreedyPatternRewriteDriver.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/ArmSVE/IR/ArmSVEDialect.h`, `mlir/Dialect/ArmSVE/Transforms/Passes.h`, `mlir/Dialect/Func/IR/FuncOps.h`, `mlir/Dialect/MemRef/IR/MemRef.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/ArmSVE/IR/ArmSVEDialect.h`, `mlir/Dialect/ArmSVE/Transforms/Passes.h`, `mlir/Dialect/Func/IR/FuncOps.h`, `mlir/Dialect/MemRef/IR/MemRef.h`。

### Lines 16-29
```cpp
namespace mlir::arm_sve {
#define GEN_PASS_DEF_LEGALIZEVECTORSTORAGE
#include "mlir/Dialect/ArmSVE/Transforms/Passes.h.inc"
} // namespace mlir::arm_sve

using namespace mlir;
using namespace mlir::arm_sve;

// A tag to mark unrealized_conversions produced by this pass. This is used to
// detect IR this pass failed to completely legalize, and report an error.
// If everything was successfully legalized, no tagged ops will remain after
// this pass.
constexpr StringLiteral kSVELegalizerTag("__arm_sve_legalize_vector_storage__");

```
- **EN**: Introduces declarations for `mlir::arm_sve`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `mlir::arm_sve` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 30-39
```cpp
/// Definitions:
///
/// [1] svbool = vector<...x[16]xi1>, which maps to some multiple of full SVE
/// predicate registers. A full predicate is the smallest quantity that can be
/// loaded/stored.
///
/// [2] SVE mask = hardware-sized SVE predicate mask, i.e. its trailing
/// dimension matches the size of a legal SVE vector size (such as
/// vector<[4]xi1>), but is too small to be stored to memory (i.e smaller than
/// a svbool).
```
- **EN**: Documents the next declarations or records design constraints for the surrounding implementation.
- **CN**: 为接下来的声明提供说明，或记录周边实现的设计约束。

### Lines 40-50
```cpp

namespace {

/// Checks if a vector type is a SVE mask [2].
bool isSVEMaskType(VectorType type) {
  return type.getRank() > 0 && type.getElementType().isInteger(1) &&
         type.getScalableDims().back() && type.getShape().back() < 16 &&
         llvm::isPowerOf2_32(type.getShape().back()) &&
         !llvm::is_contained(type.getScalableDims().drop_back(), true);
}

```
- **EN**: Implements logic around `isSVEMaskType`, `getRank`, `getScalableDims`, `isPowerOf2_32`, and 1 more symbols.
- **CN**: 围绕 `isSVEMaskType`, `getRank`, `getScalableDims`, `isPowerOf2_32`, and 1 more symbols 实现具体逻辑。

### Lines 51-66
```cpp
VectorType widenScalableMaskTypeToSvbool(VectorType type) {
  assert(isSVEMaskType(type));
  return VectorType::Builder(type).setDim(type.getRank() - 1, 16);
}

/// A helper for cloning an op and replacing it will a new version, updated by a
/// callback.
template <typename TOp, typename TLegalizerCallback>
void replaceOpWithLegalizedOp(PatternRewriter &rewriter, TOp op,
                              TLegalizerCallback callback) {
  // Clone the previous op to preserve any properties/attributes.
  auto newOp = op.clone();
  rewriter.insert(newOp);
  rewriter.replaceOp(op, callback(newOp));
}

```
- **EN**: Implements logic around `widenScalableMaskTypeToSvbool`, `assert`, `Builder`, `replaceOpWithLegalizedOp`, and 3 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `widenScalableMaskTypeToSvbool`, `assert`, `Builder`, `replaceOpWithLegalizedOp`, and 3 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 67-81
```cpp
/// A helper for cloning an op and replacing it with a new version, updated by a
/// callback, and an unrealized conversion back to the type of the replaced op.
template <typename TOp, typename TLegalizerCallback>
void replaceOpWithUnrealizedConversion(PatternRewriter &rewriter, TOp op,
                                       TLegalizerCallback callback) {
  replaceOpWithLegalizedOp(rewriter, op, [&](TOp newOp) {
    // Mark our `unrealized_conversion_casts` with a pass label.
    return UnrealizedConversionCastOp::create(
        rewriter, op.getLoc(), TypeRange{op.getResult().getType()},
        ValueRange{callback(newOp)},
        NamedAttribute(rewriter.getStringAttr(kSVELegalizerTag),
                       rewriter.getUnitAttr()));
  });
}

```
- **EN**: Implements logic around `replaceOpWithUnrealizedConversion`, `replaceOpWithLegalizedOp`, `create`, `getLoc`, and 3 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `replaceOpWithUnrealizedConversion`, `replaceOpWithLegalizedOp`, `create`, `getLoc`, and 3 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 82-92
```cpp
/// Extracts the widened SVE memref value (that's legal to store/load) from the
/// `unrealized_conversion_cast`s added by this pass.
static FailureOr<Value> getSVELegalizedMemref(Value illegalMemref) {
  Operation *definingOp = illegalMemref.getDefiningOp();
  if (!definingOp || !definingOp->hasAttr(kSVELegalizerTag))
    return failure();
  auto unrealizedConversion =
      llvm::cast<UnrealizedConversionCastOp>(definingOp);
  return unrealizedConversion.getOperand(0);
}

```
- **EN**: Implements logic around `getSVELegalizedMemref`, `getDefiningOp`, `hasAttr`, `failure`, and 2 more symbols; this block moves data between tensor-style values and explicit buffers.
- **CN**: 围绕 `getSVELegalizedMemref`, `getDefiningOp`, `hasAttr`, `failure`, and 2 more symbols 实现具体逻辑；该代码块在张量风格值与显式缓冲区之间移动数据。

### Lines 93-106
```cpp
/// The default alignment of an alloca in LLVM may request overaligned sizes for
/// SVE types, which will fail during stack frame allocation. This rewrite
/// explicitly adds a reasonable alignment to allocas of scalable types.
struct RelaxScalableVectorAllocaAlignment
    : public OpRewritePattern<memref::AllocaOp> {
  using OpRewritePattern::OpRewritePattern;

  LogicalResult matchAndRewrite(memref::AllocaOp allocaOp,
                                PatternRewriter &rewriter) const override {
    auto memrefElementType = allocaOp.getType().getElementType();
    auto vectorType = llvm::dyn_cast<VectorType>(memrefElementType);
    if (!vectorType || !vectorType.isScalable() || allocaOp.getAlignment())
      return failure();

```
- **EN**: Introduces declarations for `RelaxScalableVectorAllocaAlignment`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `RelaxScalableVectorAllocaAlignment` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 107-116
```cpp
    // Set alignment based on the defaults for SVE vectors and predicates.
    unsigned aligment = vectorType.getElementType().isInteger(1) ? 2 : 16;
    rewriter.modifyOpInPlace(allocaOp,
                             [&] { allocaOp.setAlignment(aligment); });

    return success();
  }
};

/// Replaces allocations of SVE predicates smaller than an svbool [1] (_illegal_
```
- **EN**: Implements logic around `getElementType`, `modifyOpInPlace`, `setAlignment`, `success`; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `getElementType`, `modifyOpInPlace`, `setAlignment`, `success` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 117-126
```cpp
/// to load/store) with a wider allocation of svbool (_legal_ to load/store)
/// followed by a tagged unrealized conversion to the original type.
///
/// Example
/// ```
/// %alloca = memref.alloca() : memref<vector<[4]xi1>>
/// ```
/// is rewritten into:
/// ```
/// %widened = memref.alloca() {alignment = 1 : i64} : memref<vector<[16]xi1>>
```
- **EN**: Documents the next declarations or records design constraints for the surrounding implementation.
- **CN**: 为接下来的声明提供说明，或记录周边实现的设计约束。

### Lines 127-139
```cpp
/// %alloca = builtin.unrealized_conversion_cast %widened
///   : memref<vector<[16]xi1>> to memref<vector<[4]xi1>>
///     {__arm_sve_legalize_vector_storage__}
/// ```
template <typename AllocLikeOp>
struct LegalizeSVEMaskAllocation : public OpRewritePattern<AllocLikeOp> {
  using OpRewritePattern<AllocLikeOp>::OpRewritePattern;

  LogicalResult matchAndRewrite(AllocLikeOp allocLikeOp,
                                PatternRewriter &rewriter) const override {
    auto vectorType =
        llvm::dyn_cast<VectorType>(allocLikeOp.getType().getElementType());

```
- **EN**: Introduces declarations for `LegalizeSVEMaskAllocation`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `LegalizeSVEMaskAllocation` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 140-153
```cpp
    if (!vectorType || !isSVEMaskType(vectorType))
      return failure();

    // Replace this alloc-like op of an SVE mask [2] with one of a (storable)
    // svbool mask [1]. A temporary unrealized_conversion_cast is added to the
    // old type to allow local rewrites.
    replaceOpWithUnrealizedConversion(
        rewriter, allocLikeOp, [&](AllocLikeOp newAllocLikeOp) {
          newAllocLikeOp.getResult().setType(
              llvm::cast<MemRefType>(newAllocLikeOp.getType().cloneWith(
                  {}, widenScalableMaskTypeToSvbool(vectorType))));
          return newAllocLikeOp;
        });

```
- **EN**: Implements logic around `isSVEMaskType`, `failure`, `replaceOpWithUnrealizedConversion`, `getResult`, and 2 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure; moves data between tensor-style values and explicit buffers.
- **CN**: 围绕 `isSVEMaskType`, `failure`, `replaceOpWithUnrealizedConversion`, `getResult`, and 2 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并在张量风格值与显式缓冲区之间移动数据。

### Lines 154-163
```cpp
    return success();
  }
};

/// Replaces vector.type_casts of unrealized conversions to SVE predicate memref
/// types that are _illegal_ to load/store from (!= svbool [1]), with type casts
/// of memref types that are _legal_ to load/store, followed by unrealized
/// conversions.
///
/// Example:
```
- **EN**: Implements logic around `success`; this block moves data between tensor-style values and explicit buffers.
- **CN**: 围绕 `success` 实现具体逻辑；该代码块在张量风格值与显式缓冲区之间移动数据。

### Lines 164-173
```cpp
/// ```
/// %alloca = builtin.unrealized_conversion_cast %widened
///   : memref<vector<[16]xi1>> to memref<vector<[8]xi1>>
///     {__arm_sve_legalize_vector_storage__}
/// %cast = vector.type_cast %alloca
///   : memref<vector<3x[8]xi1>> to memref<3xvector<[8]xi1>>
/// ```
/// is rewritten into:
/// ```
/// %widened_cast = vector.type_cast %widened
```
- **EN**: Documents the next declarations or records design constraints for the surrounding implementation.
- **CN**: 为接下来的声明提供说明，或记录周边实现的设计约束。

### Lines 174-187
```cpp
///   : memref<vector<3x[16]xi1>> to memref<3xvector<[16]xi1>>
/// %cast = builtin.unrealized_conversion_cast %widened_cast
///   : memref<3xvector<[16]xi1>> to memref<3xvector<[8]xi1>>
///     {__arm_sve_legalize_vector_storage__}
/// ```
struct LegalizeSVEMaskTypeCastConversion
    : public OpRewritePattern<vector::TypeCastOp> {
  using OpRewritePattern::OpRewritePattern;

  LogicalResult matchAndRewrite(vector::TypeCastOp typeCastOp,
                                PatternRewriter &rewriter) const override {
    auto resultType = typeCastOp.getResultMemRefType();
    auto vectorType = llvm::dyn_cast<VectorType>(resultType.getElementType());

```
- **EN**: Introduces declarations for `LegalizeSVEMaskTypeCastConversion`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `LegalizeSVEMaskTypeCastConversion` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 188-204
```cpp
    if (!vectorType || !isSVEMaskType(vectorType))
      return failure();

    auto legalMemref = getSVELegalizedMemref(typeCastOp.getMemref());
    if (failed(legalMemref))
      return failure();

    // Replace this vector.type_cast with one of a (storable) svbool mask [1].
    replaceOpWithUnrealizedConversion(
        rewriter, typeCastOp, [&](vector::TypeCastOp newTypeCast) {
          newTypeCast.setOperand(*legalMemref);
          newTypeCast.getResult().setType(
              llvm::cast<MemRefType>(newTypeCast.getType().cloneWith(
                  {}, widenScalableMaskTypeToSvbool(vectorType))));
          return newTypeCast;
        });

```
- **EN**: Implements logic around `isSVEMaskType`, `failure`, `getSVELegalizedMemref`, `failed`, and 5 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `isSVEMaskType`, `failure`, `getSVELegalizedMemref`, `failed`, and 5 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 205-214
```cpp
    return success();
  }
};

/// Replaces stores to unrealized conversions to SVE predicate memref types that
/// are _illegal_ to load/store from (!= svbool [1]), with
/// `arm_sve.convert_to_svbool`s followed by (legal) wider stores.
///
/// Example:
/// ```
```
- **EN**: Implements logic around `success`; this block moves data between tensor-style values and explicit buffers.
- **CN**: 围绕 `success` 实现具体逻辑；该代码块在张量风格值与显式缓冲区之间移动数据。

### Lines 215-225
```cpp
/// memref.store %mask, %alloca[] : memref<vector<[8]xi1>>
/// ```
/// is rewritten into:
/// ```
/// %svbool = arm_sve.convert_to_svbool %mask : vector<[8]xi1>
/// memref.store %svbool, %widened[] : memref<vector<[16]xi1>>
/// ```
struct LegalizeSVEMaskStoreConversion
    : public OpRewritePattern<memref::StoreOp> {
  using OpRewritePattern::OpRewritePattern;

```
- **EN**: Introduces declarations for `LegalizeSVEMaskStoreConversion`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `LegalizeSVEMaskStoreConversion` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 226-235
```cpp
  LogicalResult matchAndRewrite(memref::StoreOp storeOp,
                                PatternRewriter &rewriter) const override {
    auto loc = storeOp.getLoc();

    Value valueToStore = storeOp.getValueToStore();
    auto vectorType = llvm::dyn_cast<VectorType>(valueToStore.getType());

    if (!vectorType || !isSVEMaskType(vectorType))
      return failure();

```
- **EN**: Implements logic around `matchAndRewrite`, `getLoc`, `getValueToStore`, `dyn_cast`, and 2 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure; moves data between tensor-style values and explicit buffers.
- **CN**: 围绕 `matchAndRewrite`, `getLoc`, `getValueToStore`, `dyn_cast`, and 2 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并在张量风格值与显式缓冲区之间移动数据。

### Lines 236-252
```cpp
    auto legalMemref = getSVELegalizedMemref(storeOp.getMemref());
    if (failed(legalMemref))
      return failure();

    auto legalMaskType = widenScalableMaskTypeToSvbool(
        llvm::cast<VectorType>(valueToStore.getType()));
    auto convertToSvbool = arm_sve::ConvertToSvboolOp::create(
        rewriter, loc, legalMaskType, valueToStore);
    // Replace this store with a conversion to a storable svbool mask [1],
    // followed by a wider store.
    replaceOpWithLegalizedOp(rewriter, storeOp,
                             [&](memref::StoreOp newStoreOp) {
                               newStoreOp.setOperand(0, convertToSvbool);
                               newStoreOp.setOperand(1, *legalMemref);
                               return newStoreOp;
                             });

```
- **EN**: Implements logic around `getSVELegalizedMemref`, `failed`, `failure`, `widenScalableMaskTypeToSvbool`, and 4 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure; moves data between tensor-style values and explicit buffers.
- **CN**: 围绕 `getSVELegalizedMemref`, `failed`, `failure`, `widenScalableMaskTypeToSvbool`, and 4 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并在张量风格值与显式缓冲区之间移动数据。

### Lines 253-262
```cpp
    return success();
  }
};

/// Replaces loads from unrealized conversions to SVE predicate memref types
/// that are _illegal_ to load/store from (!= svbool [1]), types with (legal)
/// wider loads, followed by `arm_sve.convert_from_svbool`s.
///
/// Example:
/// ```
```
- **EN**: Implements logic around `success`; this block moves data between tensor-style values and explicit buffers.
- **CN**: 围绕 `success` 实现具体逻辑；该代码块在张量风格值与显式缓冲区之间移动数据。

### Lines 263-272
```cpp
/// %reload = memref.load %alloca[] : memref<vector<[4]xi1>>
/// ```
/// is rewritten into:
/// ```
/// %svbool = memref.load %widened[] : memref<vector<[16]xi1>>
/// %reload = arm_sve.convert_from_svbool %reload : vector<[4]xi1>
/// ```
struct LegalizeSVEMaskLoadConversion : public OpRewritePattern<memref::LoadOp> {
  using OpRewritePattern::OpRewritePattern;

```
- **EN**: Introduces declarations for `LegalizeSVEMaskLoadConversion`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `LegalizeSVEMaskLoadConversion` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 273-282
```cpp
  LogicalResult matchAndRewrite(memref::LoadOp loadOp,
                                PatternRewriter &rewriter) const override {
    auto loc = loadOp.getLoc();

    Value loadedMask = loadOp.getResult();
    auto vectorType = llvm::dyn_cast<VectorType>(loadedMask.getType());

    if (!vectorType || !isSVEMaskType(vectorType))
      return failure();

```
- **EN**: Implements logic around `matchAndRewrite`, `getLoc`, `getResult`, `dyn_cast`, and 2 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure; moves data between tensor-style values and explicit buffers.
- **CN**: 围绕 `matchAndRewrite`, `getLoc`, `getResult`, `dyn_cast`, and 2 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并在张量风格值与显式缓冲区之间移动数据。

### Lines 283-296
```cpp
    auto legalMemref = getSVELegalizedMemref(loadOp.getMemref());
    if (failed(legalMemref))
      return failure();

    auto legalMaskType = widenScalableMaskTypeToSvbool(vectorType);
    // Replace this load with a legal load of an svbool type, followed by a
    // conversion back to the original type.
    replaceOpWithLegalizedOp(rewriter, loadOp, [&](memref::LoadOp newLoadOp) {
      newLoadOp.setMemRef(*legalMemref);
      newLoadOp.getResult().setType(legalMaskType);
      return arm_sve::ConvertFromSvboolOp::create(
          rewriter, loc, loadedMask.getType(), newLoadOp);
    });

```
- **EN**: Implements logic around `getSVELegalizedMemref`, `failed`, `failure`, `widenScalableMaskTypeToSvbool`, and 5 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure; moves data between tensor-style values and explicit buffers.
- **CN**: 围绕 `getSVELegalizedMemref`, `failed`, `failure`, `widenScalableMaskTypeToSvbool`, and 5 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并在张量风格值与显式缓冲区之间移动数据。

### Lines 297-306
```cpp
    return success();
  }
};

/// Transforms a `transfer_read` operation so it reads vector of a type that
/// can be mapped to an LLVM type ("LLVM-legal" type). This is done by
/// collapsing trailing dimensions so we obtain a vector type with a single
/// scalable dimension in the rightmost position.
///
/// Example:
```
- **EN**: Implements logic around `success`.
- **CN**: 围绕 `success` 实现具体逻辑。

### Lines 307-316
```cpp
/// ```
/// %v = vector.transfer_read %M[%i, %j, %c0, %c0], %c0_i8
///   {in_bounds = [false, true, true, true]}
///   : memref<?x?x2x8xi8>, vector<2x[4]x2x8xi8>
/// ```
/// is rewritten to
/// ```
/// %collapse_shape = memref.collapse_shape %M [[0], [1, 2, 3]]
///   : memref<?x?x2x8xi8> into memref<?x?xi8>
/// %0 = vector.transfer_read  %collapse_shape[%i, %j], %c0_i8
```
- **EN**: Documents the next declarations or records design constraints for the surrounding implementation.
- **CN**: 为接下来的声明提供说明，或记录周边实现的设计约束。

### Lines 317-326
```cpp
///   {in_bounds = [false, true]}
///   : memref<?x?xi8>, vector<2x[64]xi8>
/// %1 = vector.shape_cast %0 : vector<2x[64]xi8> to vector<2x[4]x2x8xi8>
/// ```
struct LegalizeTransferRead : public OpRewritePattern<vector::TransferReadOp> {
  using OpRewritePattern::OpRewritePattern;

  LogicalResult matchAndRewrite(vector::TransferReadOp readOp,
                                PatternRewriter &rewriter) const override {

```
- **EN**: Introduces declarations for `LegalizeTransferRead`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `LegalizeTransferRead` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 327-341
```cpp
    // Do not try to transform masked reads. For example, if we have a transfer
    // to a `vector<[4]x4xi8>` we could have a mask like
    //    1 1 1 0
    //    1 1 1 0
    //    1 1 1 0
    //    0 0 0 0
    // Flattening this mask would look like
    //    1 1 1 0 1 1 1 0 1 1 1 0 0 0 0 0
    // and we have not yet figured out an efficient way to build such a mask,
    // neither from the mask operand, nor from the original `vector.create_mask`
    // operation (if visible at all).
    if (readOp.isMasked() || readOp.getMask())
      return rewriter.notifyMatchFailure(readOp,
                                         "masked transfers not-supported");

```
- **EN**: Implements logic around `isMasked`, `notifyMatchFailure`; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `isMasked`, `notifyMatchFailure` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 342-353
```cpp
    // General permutation maps are not supported. The issue is with transpose,
    // broadcast, and other forms of non-identify mapping in the minor
    // dimensions which is impossible to represent after collapsing (at least
    // because the resulting "collapsed" maps would have smaller number of
    // dimension indices).
    // TODO: We have not had yet the need for it, but some forms of permutation
    // maps with identity in the minor dimensions voukld be supported, for
    // example `(i, j, k, p) -> (j, i, k, p)` where we need to collapse only `k`
    // and `p`.
    if (!readOp.getPermutationMap().isMinorIdentity())
      return rewriter.notifyMatchFailure(readOp, "non-identity permutation");

```
- **EN**: Implements logic around `getPermutationMap`, `notifyMatchFailure`; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `getPermutationMap`, `notifyMatchFailure` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 354-367
```cpp
    // We handle transfers of vectors with rank >= 2 and a single scalable
    // dimension. This transformation aims to transform an LLVM-illegal type
    // into an LLVM-legal type and one dimensional vectors are already
    // LLVM-legal, even if scalable. A value of a vector type with more than one
    // scalable dimension is impossible to represent using a vector type with no
    // scalable dimensions or a single one. For example a `vector<[4]x[4]xi8>`
    // would have `4 * 4 * vscale * vscale` elements and this quantity is
    // impossible to represent as `N` or `N * vscale` (where `N` is a constant).
    VectorType origVT = readOp.getVectorType();
    ArrayRef<bool> origScalableDims = origVT.getScalableDims();
    const int64_t origVRank = origVT.getRank();
    if (origVRank < 2 || origVT.getNumScalableDims() != 1)
      return rewriter.notifyMatchFailure(readOp, "wrong dimensions");

```
- **EN**: Implements logic around `getVectorType`, `getScalableDims`, `getRank`, `getNumScalableDims`, and 1 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `getVectorType`, `getScalableDims`, `getRank`, `getNumScalableDims`, and 1 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 368-383
```cpp
    // Number of trailing dimensions to collapse, including the scalable
    // dimension.  Nothing to do if the single scalable dimension is already the
    // last one.
    const int64_t numCollapseDims = std::distance(
        llvm::find(origScalableDims, true), origScalableDims.end());
    if (numCollapseDims < 2)
      return rewriter.notifyMatchFailure(readOp,
                                         "scalable dimension is trailing");

    // We want a simple memref (not a tensor) with contiguous elements for at
    // least all the trailing dimensions up to and including the scalable one.
    auto memTy = dyn_cast<MemRefType>(readOp.getBase().getType());
    if (!(memTy && memTy.areTrailingDimsContiguous(numCollapseDims)))
      return rewriter.notifyMatchFailure(
          readOp, "non-contiguous memref dimensions to collapse");

```
- **EN**: Implements logic around `distance`, `find`, `notifyMatchFailure`, `dyn_cast`, and 1 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure; moves data between tensor-style values and explicit buffers.
- **CN**: 围绕 `distance`, `find`, `notifyMatchFailure`, `dyn_cast`, and 1 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并在张量风格值与显式缓冲区之间移动数据。

### Lines 384-394
```cpp
    // The dimensions to collapse (excluding the scalable one) of the vector and
    // the memref must match. A dynamic memref dimension is considered
    // non-matching. The transfers from the dimensions to collapse must be
    // in-bounds (it follows the corresponding indices would be zero). This
    // guarantees that the operation transfers a contiguous block
    // and no padding is necessary.
    if (!llvm::equal(memTy.getShape().take_back(numCollapseDims - 1),
                     origVT.getShape().take_back(numCollapseDims - 1)))
      return rewriter.notifyMatchFailure(
          readOp, "memref and vector dimensions do not match");

```
- **EN**: Implements logic around `equal`, `getShape`, `notifyMatchFailure`; this block applies MLIR pattern-rewrite or conversion infrastructure; moves data between tensor-style values and explicit buffers; manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `equal`, `getShape`, `notifyMatchFailure` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并在张量风格值与显式缓冲区之间移动数据，并处理 MLIR region、block 或控制流边。

### Lines 395-413
```cpp
    SmallVector<bool> origInBounds = readOp.getInBoundsValues();
    if (!llvm::all_of(
            ArrayRef<bool>(origInBounds).take_back(numCollapseDims - 1),
            [](bool v) { return v; }))
      return rewriter.notifyMatchFailure(
          readOp, "out-of-bounds transfer from a dimension to collapse");

    // Collapse the trailing dimensions of the memref.
    SmallVector<ReassociationIndices> reassoc;
    for (int64_t i = 0; i < memTy.getRank() - numCollapseDims + 1; ++i)
      reassoc.push_back({i});
    for (int64_t i = memTy.getRank() - numCollapseDims + 1; i < memTy.getRank();
         ++i)
      reassoc.back().push_back(i);
    if (!memref::CollapseShapeOp::isGuaranteedCollapsible(memTy, reassoc))
      return failure();
    Value collapsedMem = memref::CollapseShapeOp::create(
        rewriter, readOp.getLoc(), readOp.getBase(), reassoc);

```
- **EN**: Implements logic around `getInBoundsValues`, `all_of`, `ArrayRef`, `notifyMatchFailure`, and 7 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure; moves data between tensor-style values and explicit buffers.
- **CN**: 围绕 `getInBoundsValues`, `all_of`, `ArrayRef`, `notifyMatchFailure`, and 7 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并在张量风格值与显式缓冲区之间移动数据。

### Lines 414-425
```cpp
    // Get a vector type with collapsed trailing dimensions.
    SmallVector<int64_t> shape(origVT.getShape());
    for (int64_t i = origVRank - numCollapseDims + 1; i < origVRank; ++i)
      shape[origVRank - numCollapseDims] *= shape[i];
    shape.pop_back_n(numCollapseDims - 1);
    auto collapsedVT =
        VectorType::get(shape, origVT.getElementType(),
                        origScalableDims.drop_back(numCollapseDims - 1));

    // Drop the extra (zero) indices.
    auto indices = readOp.getIndices().drop_back(numCollapseDims - 1);

```
- **EN**: Implements logic around `shape`, `pop_back_n`, `get`, `drop_back`, and 1 more symbols.
- **CN**: 围绕 `shape`, `pop_back_n`, `get`, `drop_back`, and 1 more symbols 实现具体逻辑。

### Lines 426-435
```cpp
    // Create the new `transfer_read`.
    auto newReadOp = vector::TransferReadOp::create(
        rewriter, readOp.getLoc(), collapsedVT, collapsedMem, indices,
        readOp.getPadding(),
        ArrayRef<bool>(origInBounds).drop_back(numCollapseDims - 1));

    // Cast back to the original vector type.
    auto toOrigShape = vector::ShapeCastOp::create(rewriter, readOp.getLoc(),
                                                   origVT, newReadOp);

```
- **EN**: Implements logic around `create`, `getLoc`, `getPadding`, `ArrayRef`; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `create`, `getLoc`, `getPadding`, `ArrayRef` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 436-453
```cpp
    rewriter.replaceOp(readOp, toOrigShape);
    return success();
  }
};

} // namespace

void mlir::arm_sve::populateLegalizeVectorStoragePatterns(
    RewritePatternSet &patterns) {
  patterns
      .add<RelaxScalableVectorAllocaAlignment,
           LegalizeSVEMaskAllocation<memref::AllocaOp>,
           LegalizeSVEMaskAllocation<memref::AllocOp>,
           LegalizeSVEMaskTypeCastConversion, LegalizeSVEMaskStoreConversion,
           LegalizeSVEMaskLoadConversion, LegalizeTransferRead>(
          patterns.getContext());
}

```
- **EN**: Implements logic around `replaceOp`, `success`, `populateLegalizeVectorStoragePatterns`, `LegalizeTransferRead>`, and 1 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure; moves data between tensor-style values and explicit buffers.
- **CN**: 围绕 `replaceOp`, `success`, `populateLegalizeVectorStoragePatterns`, `LegalizeTransferRead>`, and 1 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并在张量风格值与显式缓冲区之间移动数据。

### Lines 454-473
```cpp
namespace {
struct LegalizeVectorStorage
    : public arm_sve::impl::LegalizeVectorStorageBase<LegalizeVectorStorage> {

  void runOnOperation() override {
    RewritePatternSet patterns(&getContext());
    populateLegalizeVectorStoragePatterns(patterns);
    if (failed(applyPatternsGreedily(getOperation(), std::move(patterns)))) {
      signalPassFailure();
    }
    ConversionTarget target(getContext());
    target.addDynamicallyLegalOp<UnrealizedConversionCastOp>(
        [](UnrealizedConversionCastOp unrealizedConversion) {
          return !unrealizedConversion->hasAttr(kSVELegalizerTag);
        });
    // This detects if we failed to completely legalize the IR.
    if (failed(applyPartialConversion(getOperation(), target, {})))
      signalPassFailure();
  }
};
```
- **EN**: Introduces declarations for `LegalizeVectorStorage`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `LegalizeVectorStorage` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 474-479
```cpp

} // namespace

std::unique_ptr<Pass> mlir::arm_sve::createLegalizeVectorStoragePass() {
  return std::make_unique<LegalizeVectorStorage>();
}
```
- **EN**: Implements logic around `createLegalizeVectorStoragePass`, `make_unique`; this block participates in pass execution or pass construction.
- **CN**: 围绕 `createLegalizeVectorStoragePass`, `make_unique` 实现具体逻辑；该代码块参与 pass 执行或 pass 构造。

## Key Concepts / 关键概念

- **Pattern rewriting and passes / 模式重写与 Pass**:
  - **EN**: Applies conversion patterns, canonicalizations, or pass pipelines over MLIR operations.
  - **CN**: 对 MLIR 操作应用转换模式、规范化或 pass 流水线。
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

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/ArmSVE/IR/ArmSVEDialect.h`, `mlir/Dialect/ArmSVE/Transforms/Passes.h`, `mlir/Dialect/Func/IR/FuncOps.h`, `mlir/Dialect/MemRef/IR/MemRef.h`, `mlir/Dialect/Vector/IR/VectorOps.h`, `mlir/Transforms/GreedyPatternRewriteDriver.h`, `mlir/Dialect/ArmSVE/Transforms/Passes.h.inc`
- **Subsystem categories / 子系统类别**: other MLIR dialect declarations / 其他 MLIR 方言声明 (6), common transformation and pattern-rewrite helpers / 通用变换与模式重写辅助工具 (1)
