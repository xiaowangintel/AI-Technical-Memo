# MaskedloadToLoad.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/AMDGPU/Transforms/MaskedloadToLoad.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements rewrite patterns and transformation passes for the AMDGPU dialect and target-specific GPU support.
  - **CN**: 实现 AMDGPU 方言与目标专用 GPU 支持 的重写模式与变换 pass。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- MaskedloadToLoad.cpp - Lowers maskedload to load -------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件的高层描述。

### Lines 8-21
```cpp

#include "mlir/Dialect/AMDGPU/Transforms/Passes.h"

#include "mlir/Dialect/AMDGPU/IR/AMDGPUDialect.h"
#include "mlir/Dialect/Arith/IR/Arith.h"
#include "mlir/Dialect/MemRef/IR/MemRef.h"
#include "mlir/Dialect/MemRef/Utils/MemRefUtils.h"
#include "mlir/Dialect/SCF/IR/SCF.h"
#include "mlir/Dialect/Vector/IR/VectorOps.h"
#include "mlir/Dialect/Vector/Transforms/VectorTransforms.h"
#include "mlir/IR/BuiltinTypes.h"
#include "mlir/IR/OpDefinition.h"
#include "mlir/IR/PatternMatch.h"
#include "mlir/IR/TypeUtilities.h"
```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/AMDGPU/Transforms/Passes.h`, `mlir/Dialect/AMDGPU/IR/AMDGPUDialect.h`, `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/MemRef/IR/MemRef.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/AMDGPU/Transforms/Passes.h`, `mlir/Dialect/AMDGPU/IR/AMDGPUDialect.h`, `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/MemRef/IR/MemRef.h`。

### Lines 22-30
```cpp
#include "mlir/Support/LogicalResult.h"
#include "mlir/Transforms/GreedyPatternRewriteDriver.h"
#include "llvm/Support/MathExtras.h"

namespace mlir::amdgpu {
#define GEN_PASS_DEF_AMDGPUMASKEDLOADTOLOADPASS
#include "mlir/Dialect/AMDGPU/Transforms/Passes.h.inc"
} // namespace mlir::amdgpu

```
- **EN**: Introduces declarations for `mlir::amdgpu`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `mlir::amdgpu` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 31-40
```cpp
using namespace mlir;
using namespace mlir::amdgpu;

/// This pattern supports lowering of: `vector.maskedload` to `vector.load`
/// and `arith.select` if the memref is in buffer address space.
static LogicalResult hasBufferAddressSpace(Type type) {
  auto memRefType = dyn_cast<MemRefType>(type);
  if (!memRefType)
    return failure();

```
- **EN**: Implements logic around `hasBufferAddressSpace`, `dyn_cast`, `failure`; this block moves data between tensor-style values and explicit buffers; encodes accelerator-specific execution or lowering rules.
- **CN**: 围绕 `hasBufferAddressSpace`, `dyn_cast`, `failure` 实现具体逻辑；该代码块在张量风格值与显式缓冲区之间移动数据，并编码加速器专用执行或 lowering 规则。

### Lines 41-48
```cpp
  Attribute addrSpace = memRefType.getMemorySpace();
  if (!isa_and_nonnull<amdgpu::AddressSpaceAttr>(addrSpace))
    return failure();

  if (dyn_cast<amdgpu::AddressSpaceAttr>(addrSpace).getValue() !=
      amdgpu::AddressSpace::FatRawBuffer)
    return failure();

```
- **EN**: Implements logic around `getMemorySpace`, `AddressSpaceAttr>`, `failure`; this block encodes accelerator-specific execution or lowering rules.
- **CN**: 围绕 `getMemorySpace`, `AddressSpaceAttr>`, `failure` 实现具体逻辑；该代码块编码加速器专用执行或 lowering 规则。

### Lines 49-62
```cpp
  return success();
}

static Value createVectorLoadForMaskedLoad(OpBuilder &builder, Location loc,
                                           vector::MaskedLoadOp maskedOp,
                                           bool passthru) {
  VectorType vectorType = maskedOp.getVectorType();
  Value load = vector::LoadOp::create(
      builder, loc, vectorType, maskedOp.getBase(), maskedOp.getIndices());
  if (passthru)
    load = arith::SelectOp::create(builder, loc, vectorType, maskedOp.getMask(),
                                   load, maskedOp.getPassThru());
  return load;
}
```
- **EN**: Implements logic around `success`, `createVectorLoadForMaskedLoad`, `getVectorType`, `create`, and 2 more symbols.
- **CN**: 围绕 `success`, `createVectorLoadForMaskedLoad`, `getVectorType`, `create`, and 2 more symbols 实现具体逻辑。

### Lines 63-73
```cpp

/// Check if the given value comes from a broadcasted i1 condition.
static FailureOr<Value> matchFullMask(OpBuilder &b, Value val) {
  auto broadcastOp = val.getDefiningOp<vector::BroadcastOp>();
  if (!broadcastOp)
    return failure();
  if (isa<VectorType>(broadcastOp.getSourceType()))
    return failure();
  return broadcastOp.getSource();
}

```
- **EN**: Implements logic around `matchFullMask`, `BroadcastOp>`, `failure`, `isa`, and 1 more symbols.
- **CN**: 围绕 `matchFullMask`, `BroadcastOp>`, `failure`, `isa`, and 1 more symbols 实现具体逻辑。

### Lines 74-81
```cpp
static constexpr char kMaskedloadNeedsMask[] =
    "amdgpu.buffer_maskedload_needs_mask";

namespace {

struct MaskedLoadLowering final : OpRewritePattern<vector::MaskedLoadOp> {
  using OpRewritePattern::OpRewritePattern;

```
- **EN**: Introduces declarations for `MaskedLoadLowering`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `MaskedLoadLowering` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 82-91
```cpp
  LogicalResult matchAndRewrite(vector::MaskedLoadOp maskedOp,
                                PatternRewriter &rewriter) const override {
    if (maskedOp->hasAttr(kMaskedloadNeedsMask))
      return rewriter.notifyMatchFailure(maskedOp, "already rewritten");

    if (failed(hasBufferAddressSpace(maskedOp.getBase().getType()))) {
      return rewriter.notifyMatchFailure(
          maskedOp, "isn't a load from a fat buffer resource");
    }

```
- **EN**: Implements logic around `matchAndRewrite`, `hasAttr`, `notifyMatchFailure`, `failed`; this block applies MLIR pattern-rewrite or conversion infrastructure; moves data between tensor-style values and explicit buffers.
- **CN**: 围绕 `matchAndRewrite`, `hasAttr`, `notifyMatchFailure`, `failed` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并在张量风格值与显式缓冲区之间移动数据。

### Lines 92-101
```cpp
    // Check if this is either a full inbounds load or an empty, oob load. If
    // so, take the fast path and don't generate an if condition, because we
    // know doing the oob load is always safe.
    if (succeeded(matchFullMask(rewriter, maskedOp.getMask()))) {
      Value load = createVectorLoadForMaskedLoad(rewriter, maskedOp.getLoc(),
                                                 maskedOp, /*passthru=*/true);
      rewriter.replaceOp(maskedOp, load);
      return success();
    }

```
- **EN**: Implements logic around `succeeded`, `createVectorLoadForMaskedLoad`, `replaceOp`, `success`; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `succeeded`, `createVectorLoadForMaskedLoad`, `replaceOp`, `success` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 102-109
```cpp
    Location loc = maskedOp.getLoc();
    Value src = maskedOp.getBase();

    VectorType vectorType = maskedOp.getVectorType();
    int64_t vectorSize = vectorType.getNumElements();
    int64_t elementBitWidth = vectorType.getElementTypeBitWidth();
    SmallVector<OpFoldResult> indices = maskedOp.getIndices();

```
- **EN**: Implements logic around `getLoc`, `getBase`, `getVectorType`, `getNumElements`, and 2 more symbols.
- **CN**: 围绕 `getLoc`, `getBase`, `getVectorType`, `getNumElements`, and 2 more symbols 实现具体逻辑。

### Lines 110-122
```cpp
    auto stridedMetadata =
        memref::ExtractStridedMetadataOp::create(rewriter, loc, src);
    SmallVector<OpFoldResult> strides =
        stridedMetadata.getConstifiedMixedStrides();
    SmallVector<OpFoldResult> sizes = stridedMetadata.getConstifiedMixedSizes();
    OpFoldResult offset = stridedMetadata.getConstifiedMixedOffset();
    memref::LinearizedMemRefInfo linearizedInfo;
    OpFoldResult linearizedIndices;
    std::tie(linearizedInfo, linearizedIndices) =
        memref::getLinearizedMemRefOffsetAndSize(rewriter, loc, elementBitWidth,
                                                 elementBitWidth, offset, sizes,
                                                 strides, indices);

```
- **EN**: Implements logic around `create`, `getConstifiedMixedStrides`, `getConstifiedMixedSizes`, `getConstifiedMixedOffset`, and 2 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure; moves data between tensor-style values and explicit buffers.
- **CN**: 围绕 `create`, `getConstifiedMixedStrides`, `getConstifiedMixedSizes`, `getConstifiedMixedOffset`, and 2 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并在张量风格值与显式缓冲区之间移动数据。

### Lines 123-131
```cpp
    // delta = bufferSize - linearizedOffset
    Value vectorSizeOffset =
        arith::ConstantIndexOp::create(rewriter, loc, vectorSize);
    Value linearIndex =
        getValueOrCreateConstantIndexOp(rewriter, loc, linearizedIndices);
    Value totalSize = getValueOrCreateConstantIndexOp(
        rewriter, loc, linearizedInfo.linearizedSize);
    Value delta = arith::SubIOp::create(rewriter, loc, totalSize, linearIndex);

```
- **EN**: Implements logic around `create`, `getValueOrCreateConstantIndexOp`; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `create`, `getValueOrCreateConstantIndexOp` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 132-143
```cpp
    // 1) check if delta < vectorSize
    Value isOutofBounds = arith::CmpIOp::create(
        rewriter, loc, arith::CmpIPredicate::ult, delta, vectorSizeOffset);

    // 2) check if (detla % elements_per_word != 0)
    Value elementsPerWord = arith::ConstantIndexOp::create(
        rewriter, loc, llvm::divideCeil(32, elementBitWidth));
    Value isNotWordAligned = arith::CmpIOp::create(
        rewriter, loc, arith::CmpIPredicate::ne,
        arith::RemUIOp::create(rewriter, loc, delta, elementsPerWord),
        arith::ConstantIndexOp::create(rewriter, loc, 0));

```
- **EN**: Implements logic around `create`, `divideCeil`; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `create`, `divideCeil` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 144-150
```cpp
    // We take the fallback of maskedload default lowering only it is both
    // out-of-bounds and not word aligned. The fallback ensures correct results
    // when loading at the boundary of the buffer since buffer load returns
    // inconsistent zeros for the whole word when boundary is crossed.
    Value ifCondition =
        arith::AndIOp::create(rewriter, loc, isOutofBounds, isNotWordAligned);

```
- **EN**: Implements logic around `create`; this block applies MLIR pattern-rewrite or conversion infrastructure; moves data between tensor-style values and explicit buffers.
- **CN**: 围绕 `create` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并在张量风格值与显式缓冲区之间移动数据。

### Lines 151-157
```cpp
    auto thenBuilder = [&](OpBuilder &builder, Location loc) {
      Operation *read = builder.clone(*maskedOp.getOperation());
      read->setAttr(kMaskedloadNeedsMask, builder.getUnitAttr());
      Value readResult = read->getResult(0);
      scf::YieldOp::create(builder, loc, readResult);
    };

```
- **EN**: Implements logic around `clone`, `setAttr`, `getResult`, `create`.
- **CN**: 围绕 `clone`, `setAttr`, `getResult`, `create` 实现具体逻辑。

### Lines 158-166
```cpp
    auto elseBuilder = [&](OpBuilder &builder, Location loc) {
      Value res = createVectorLoadForMaskedLoad(builder, loc, maskedOp,
                                                /*passthru=*/true);
      scf::YieldOp::create(rewriter, loc, res);
    };

    auto ifOp =
        scf::IfOp::create(rewriter, loc, ifCondition, thenBuilder, elseBuilder);

```
- **EN**: Implements logic around `createVectorLoadForMaskedLoad`, `create`; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `createVectorLoadForMaskedLoad`, `create` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 167-173
```cpp
    rewriter.replaceOp(maskedOp, ifOp);

    return success();
  }
};

struct FullMaskedLoadToConditionalLoad
```
- **EN**: Introduces declarations for `FullMaskedLoadToConditionalLoad`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `FullMaskedLoadToConditionalLoad` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 174-182
```cpp
    : OpRewritePattern<vector::MaskedLoadOp> {
  using OpRewritePattern::OpRewritePattern;

  LogicalResult matchAndRewrite(vector::MaskedLoadOp loadOp,
                                PatternRewriter &rewriter) const override {
    if (succeeded(hasBufferAddressSpace(loadOp.getBase().getType())))
      return rewriter.notifyMatchFailure(
          loadOp, "buffer loads are handled by a more specialized pattern");

```
- **EN**: Implements logic around `matchAndRewrite`, `succeeded`, `notifyMatchFailure`; this block applies MLIR pattern-rewrite or conversion infrastructure; moves data between tensor-style values and explicit buffers.
- **CN**: 围绕 `matchAndRewrite`, `succeeded`, `notifyMatchFailure` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并在张量风格值与显式缓冲区之间移动数据。

### Lines 183-196
```cpp
    FailureOr<Value> maybeCond = matchFullMask(rewriter, loadOp.getMask());
    if (failed(maybeCond)) {
      return rewriter.notifyMatchFailure(loadOp,
                                         "isn't loading a broadcasted scalar");
    }

    Value cond = maybeCond.value();
    auto trueBuilder = [&](OpBuilder &builder, Location loc) {
      Value res = createVectorLoadForMaskedLoad(builder, loc, loadOp,
                                                /*passthru=*/false);
      scf::YieldOp::create(rewriter, loc, res);
    };
    auto falseBuilder = [&](OpBuilder &builder, Location loc) {
      scf::YieldOp::create(rewriter, loc, loadOp.getPassThru());
```
- **EN**: Implements logic around `matchFullMask`, `failed`, `notifyMatchFailure`, `value`, and 2 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `matchFullMask`, `failed`, `notifyMatchFailure`, `value`, and 2 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 197-204
```cpp
    };
    auto ifOp = scf::IfOp::create(rewriter, loadOp.getLoc(), cond, trueBuilder,
                                  falseBuilder);
    rewriter.replaceOp(loadOp, ifOp);
    return success();
  }
};

```
- **EN**: Implements logic around `create`, `replaceOp`, `success`; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `create`, `replaceOp`, `success` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 205-218
```cpp
struct FullMaskedStoreToConditionalStore
    : OpRewritePattern<vector::MaskedStoreOp> {
  using OpRewritePattern::OpRewritePattern;

  LogicalResult matchAndRewrite(vector::MaskedStoreOp storeOp,
                                PatternRewriter &rewriter) const override {
    // A condition-free implementation of fully masked stores requires
    // 1) an accessor for the num_records field on buffer resources/fat pointers
    // 2) knowledge that said field will always be set accurately - that is,
    // that writes to x < num_records of offset wouldn't trap, which is
    // something a pattern user would need to assert or we'd need to prove.
    //
    // Therefore, conditional stores to buffers still go down this path at
    // present.
```
- **EN**: Introduces declarations for `FullMaskedStoreToConditionalStore`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `FullMaskedStoreToConditionalStore` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 219-225
```cpp

    FailureOr<Value> maybeCond = matchFullMask(rewriter, storeOp.getMask());
    if (failed(maybeCond)) {
      return failure();
    }
    Value cond = maybeCond.value();

```
- **EN**: Implements logic around `matchFullMask`, `failed`, `failure`, `value`; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `matchFullMask`, `failed`, `failure`, `value` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 226-237
```cpp
    auto trueBuilder = [&](OpBuilder &builder, Location loc) {
      vector::StoreOp::create(rewriter, loc, storeOp.getValueToStore(),
                              storeOp.getBase(), storeOp.getIndices());
      scf::YieldOp::create(rewriter, loc);
    };
    auto ifOp =
        scf::IfOp::create(rewriter, storeOp.getLoc(), cond, trueBuilder);
    rewriter.replaceOp(storeOp, ifOp);
    return success();
  }
};

```
- **EN**: Implements logic around `create`, `getBase`, `replaceOp`, `success`; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `create`, `getBase`, `replaceOp`, `success` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 238-246
```cpp
} // namespace

void mlir::amdgpu::populateAmdgpuMaskedloadToLoadPatterns(
    RewritePatternSet &patterns, PatternBenefit benefit) {
  patterns.add<MaskedLoadLowering, FullMaskedLoadToConditionalLoad,
               FullMaskedStoreToConditionalStore>(patterns.getContext(),
                                                  benefit);
}

```
- **EN**: Implements logic around `populateAmdgpuMaskedloadToLoadPatterns`, `FullMaskedStoreToConditionalStore>`; this block encodes accelerator-specific execution or lowering rules.
- **CN**: 围绕 `populateAmdgpuMaskedloadToLoadPatterns`, `FullMaskedStoreToConditionalStore>` 实现具体逻辑；该代码块编码加速器专用执行或 lowering 规则。

### Lines 247-256
```cpp
struct AmdgpuMaskedloadToLoadPass final
    : amdgpu::impl::AmdgpuMaskedloadToLoadPassBase<AmdgpuMaskedloadToLoadPass> {
  void runOnOperation() override {
    RewritePatternSet patterns(&getContext());
    populateAmdgpuMaskedloadToLoadPatterns(patterns);
    if (failed(applyPatternsGreedily(getOperation(), std::move(patterns)))) {
      return signalPassFailure();
    }
  }
};
```
- **EN**: Introduces declarations for `AmdgpuMaskedloadToLoadPass`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `AmdgpuMaskedloadToLoadPass` 等声明，建立本文件后续使用的类型或命名空间。

## Key Concepts / 关键概念

- **Pattern rewriting and passes / 模式重写与 Pass**:
  - **EN**: Applies conversion patterns, canonicalizations, or pass pipelines over MLIR operations.
  - **CN**: 对 MLIR 操作应用转换模式、规范化或 pass 流水线。
- **GPU execution model / GPU 执行模型**:
  - **EN**: Models kernels, memory spaces, launch geometry, and accelerator-specific rewrites.
  - **CN**: 建模 kernel、内存空间、启动几何以及加速器专用重写。
- **Type/attribute storage / 类型/属性存储**:
  - **EN**: Uses uniqued storage and parser/printer hooks for extensible compile-time metadata.
  - **CN**: 使用唯一化存储以及解析/打印钩子来表示可扩展的编译期元数据。
- **LLVM interop / LLVM 互操作**:
  - **EN**: Bridges MLIR concepts to LLVM-compatible data structures, intrinsics, or codegen expectations.
  - **CN**: 把 MLIR 概念桥接到 LLVM 兼容的数据结构、intrinsic 或代码生成预期。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/AMDGPU/Transforms/Passes.h`, `mlir/Dialect/AMDGPU/IR/AMDGPUDialect.h`, `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/MemRef/IR/MemRef.h`, `mlir/Dialect/MemRef/Utils/MemRefUtils.h`, `mlir/Dialect/SCF/IR/SCF.h`, `mlir/Dialect/Vector/IR/VectorOps.h`, `mlir/Dialect/Vector/Transforms/VectorTransforms.h`, `mlir/IR/BuiltinTypes.h`, `mlir/IR/OpDefinition.h` ... (+6 more)
- **Subsystem categories / 子系统类别**: other MLIR dialect declarations / 其他 MLIR 方言声明 (9), MLIR IR core abstractions / MLIR IR 核心抽象 (4), MLIR support-library helpers / MLIR Support 库辅助功能 (1), common transformation and pattern-rewrite helpers / 通用变换与模式重写辅助工具 (1), LLVM support-library helpers / LLVM Support 库辅助功能 (1)
