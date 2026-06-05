# EmulateAtomics.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/AMDGPU/Transforms/EmulateAtomics.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements rewrite patterns and transformation passes for the AMDGPU dialect and target-specific GPU support.
  - **CN**: 实现 AMDGPU 方言与目标专用 GPU 支持 的重写模式与变换 pass。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- EmulateAtomics.cpp - Emulate unsupported AMDGPU atomics ------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件的高层描述。

### Lines 8-19
```cpp

#include "mlir/Dialect/AMDGPU/Transforms/Passes.h"

#include "mlir/Dialect/AMDGPU/IR/AMDGPUDialect.h"
#include "mlir/Dialect/AMDGPU/Utils/Chipset.h"
#include "mlir/Dialect/Arith/IR/Arith.h"
#include "mlir/Dialect/ControlFlow/IR/ControlFlowOps.h"
#include "mlir/Dialect/Vector/IR/VectorOps.h"
#include "mlir/IR/BuiltinAttributes.h"
#include "mlir/IR/TypeUtilities.h"
#include "mlir/Transforms/DialectConversion.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/AMDGPU/Transforms/Passes.h`, `mlir/Dialect/AMDGPU/IR/AMDGPUDialect.h`, `mlir/Dialect/AMDGPU/Utils/Chipset.h`, `mlir/Dialect/Arith/IR/Arith.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/AMDGPU/Transforms/Passes.h`, `mlir/Dialect/AMDGPU/IR/AMDGPUDialect.h`, `mlir/Dialect/AMDGPU/Utils/Chipset.h`, `mlir/Dialect/Arith/IR/Arith.h`。

### Lines 20-27
```cpp
namespace mlir::amdgpu {
#define GEN_PASS_DEF_AMDGPUEMULATEATOMICSPASS
#include "mlir/Dialect/AMDGPU/Transforms/Passes.h.inc"
} // namespace mlir::amdgpu

using namespace mlir;
using namespace mlir::amdgpu;

```
- **EN**: Introduces declarations for `mlir::amdgpu`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `mlir::amdgpu` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 28-36
```cpp
namespace {
struct AmdgpuEmulateAtomicsPass
    : public amdgpu::impl::AmdgpuEmulateAtomicsPassBase<
          AmdgpuEmulateAtomicsPass> {
  using AmdgpuEmulateAtomicsPassBase<
      AmdgpuEmulateAtomicsPass>::AmdgpuEmulateAtomicsPassBase;
  void runOnOperation() override;
};

```
- **EN**: Introduces declarations for `AmdgpuEmulateAtomicsPass`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `AmdgpuEmulateAtomicsPass` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 37-47
```cpp
template <typename AtomicOp, typename ArithOp>
struct RawBufferAtomicByCasPattern : public OpConversionPattern<AtomicOp> {
  using OpConversionPattern<AtomicOp>::OpConversionPattern;
  using Adaptor = typename AtomicOp::Adaptor;

  LogicalResult
  matchAndRewrite(AtomicOp atomicOp, Adaptor adaptor,
                  ConversionPatternRewriter &rewriter) const override;
};
} // namespace

```
- **EN**: Introduces declarations for `RawBufferAtomicByCasPattern`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `RawBufferAtomicByCasPattern` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 48-54
```cpp
namespace {
enum class DataArgAction : unsigned char {
  Duplicate,
  Drop,
};
} // namespace

```
- **EN**: Introduces declarations for `DataArgAction`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `DataArgAction` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 55-68
```cpp
// Fix up the fact that, when we're migrating from a general bugffer atomic
// to a load or to a CAS, the number of openrands, and thus the number of
// entries needed in operandSegmentSizes, needs to change. We use this method
// because we'd like to preserve unknown attributes on the atomic instead of
// discarding them.
static void patchOperandSegmentSizes(ArrayRef<NamedAttribute> attrs,
                                     SmallVectorImpl<NamedAttribute> &newAttrs,
                                     DataArgAction action) {
  newAttrs.reserve(attrs.size());
  for (NamedAttribute attr : attrs) {
    if (attr.getName().getValue() != "operandSegmentSizes") {
      newAttrs.push_back(attr);
      continue;
    }
```
- **EN**: Implements logic around `patchOperandSegmentSizes`, `reserve`, `getName`, `push_back`.
- **CN**: 围绕 `patchOperandSegmentSizes`, `reserve`, `getName`, `push_back` 实现具体逻辑。

### Lines 69-82
```cpp
    auto segmentAttr = cast<DenseI32ArrayAttr>(attr.getValue());
    MLIRContext *context = segmentAttr.getContext();
    DenseI32ArrayAttr newSegments;
    switch (action) {
    case DataArgAction::Drop:
      newSegments = DenseI32ArrayAttr::get(
          context, segmentAttr.asArrayRef().drop_front());
      break;
    case DataArgAction::Duplicate: {
      SmallVector<int32_t> newVals;
      ArrayRef<int32_t> oldVals = segmentAttr.asArrayRef();
      newVals.push_back(oldVals[0]);
      newVals.append(oldVals.begin(), oldVals.end());
      newSegments = DenseI32ArrayAttr::get(context, newVals);
```
- **EN**: Implements logic around `cast`, `getContext`, `get`, `asArrayRef`, and 2 more symbols.
- **CN**: 围绕 `cast`, `getContext`, `get`, `asArrayRef`, and 2 more symbols 实现具体逻辑。

### Lines 83-89
```cpp
      break;
    }
    }
    newAttrs.push_back(NamedAttribute(attr.getName(), newSegments));
  }
}

```
- **EN**: Implements logic around `push_back`.
- **CN**: 围绕 `push_back` 实现具体逻辑。

### Lines 90-97
```cpp
// A helper function to flatten a vector value to a scalar containing its bits,
// returning the value itself if othetwise.
static Value flattenVecToBits(ConversionPatternRewriter &rewriter, Location loc,
                              Value val) {
  auto vectorType = dyn_cast<VectorType>(val.getType());
  if (!vectorType)
    return val;

```
- **EN**: Implements logic around `flattenVecToBits`, `dyn_cast`; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `flattenVecToBits`, `dyn_cast` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 98-106
```cpp
  int64_t bitwidth =
      vectorType.getElementTypeBitWidth() * vectorType.getNumElements();
  Type allBitsType = rewriter.getIntegerType(bitwidth);
  auto allBitsVecType = VectorType::get({1}, allBitsType);
  Value bitcast = vector::BitCastOp::create(rewriter, loc, allBitsVecType, val);
  Value scalar = vector::ExtractOp::create(rewriter, loc, bitcast, 0);
  return scalar;
}

```
- **EN**: Implements logic around `getElementTypeBitWidth`, `getIntegerType`, `get`, `create`; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `getElementTypeBitWidth`, `getIntegerType`, `get`, `create` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 107-118
```cpp
template <typename AtomicOp, typename ArithOp>
LogicalResult RawBufferAtomicByCasPattern<AtomicOp, ArithOp>::matchAndRewrite(
    AtomicOp atomicOp, Adaptor adaptor,
    ConversionPatternRewriter &rewriter) const {
  Location loc = atomicOp.getLoc();

  ArrayRef<NamedAttribute> origAttrs = atomicOp->getAttrs();
  ValueRange operands = adaptor.getOperands();
  Value data = operands.take_front()[0];
  ValueRange invariantArgs = operands.drop_front();
  Type dataType = data.getType();

```
- **EN**: Implements logic around `matchAndRewrite`, `getLoc`, `getAttrs`, `getOperands`, and 3 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `matchAndRewrite`, `getLoc`, `getAttrs`, `getOperands`, and 3 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 119-127
```cpp
  SmallVector<NamedAttribute> loadAttrs;
  patchOperandSegmentSizes(origAttrs, loadAttrs, DataArgAction::Drop);
  Value initialLoad = RawBufferLoadOp::create(rewriter, loc, dataType,
                                              invariantArgs, loadAttrs);
  Block *currentBlock = rewriter.getInsertionBlock();
  Block *afterAtomic =
      rewriter.splitBlock(currentBlock, rewriter.getInsertionPoint());
  Block *loopBlock = rewriter.createBlock(afterAtomic, {dataType}, {loc});

```
- **EN**: Implements logic around `patchOperandSegmentSizes`, `create`, `getInsertionBlock`, `splitBlock`, and 1 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure; manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `patchOperandSegmentSizes`, `create`, `getInsertionBlock`, `splitBlock`, and 1 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并处理 MLIR region、block 或控制流边。

### Lines 128-135
```cpp
  rewriter.setInsertionPointToEnd(currentBlock);
  cf::BranchOp::create(rewriter, loc, loopBlock, initialLoad);

  rewriter.setInsertionPointToEnd(loopBlock);
  Value prevLoad = loopBlock->getArgument(0);
  Value operated = ArithOp::create(rewriter, loc, data, prevLoad);
  dataType = operated.getType();

```
- **EN**: Implements logic around `setInsertionPointToEnd`, `create`, `getArgument`, `getType`; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `setInsertionPointToEnd`, `create`, `getArgument`, `getType` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 136-142
```cpp
  SmallVector<NamedAttribute> cmpswapAttrs;
  patchOperandSegmentSizes(origAttrs, cmpswapAttrs, DataArgAction::Duplicate);
  SmallVector<Value> cmpswapArgs = {operated, prevLoad};
  cmpswapArgs.append(invariantArgs.begin(), invariantArgs.end());
  Value atomicRes = RawBufferAtomicCmpswapOp::create(rewriter, loc, dataType,
                                                     cmpswapArgs, cmpswapAttrs);

```
- **EN**: Implements logic around `patchOperandSegmentSizes`, `append`, `create`; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `patchOperandSegmentSizes`, `append`, `create` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 143-156
```cpp
  // We care about exact bitwise equality here, so do some bitcasts.
  // These will fold away during lowering to the ROCDL dialect, where
  // an int->float bitcast is introduced to account for the fact that cmpswap
  // only takes integer arguments.

  Value prevLoadForCompare = flattenVecToBits(rewriter, loc, prevLoad);
  Value atomicResForCompare = flattenVecToBits(rewriter, loc, atomicRes);
  if (auto floatDataTy = dyn_cast<FloatType>(dataType)) {
    Type equivInt = rewriter.getIntegerType(floatDataTy.getWidth());
    prevLoadForCompare =
        arith::BitcastOp::create(rewriter, loc, equivInt, prevLoad);
    atomicResForCompare =
        arith::BitcastOp::create(rewriter, loc, equivInt, atomicRes);
  }
```
- **EN**: Implements logic around `flattenVecToBits`, `dyn_cast`, `getIntegerType`, `create`; this block applies MLIR pattern-rewrite or conversion infrastructure; implements verifier, folding, parsing, or printing hooks; encodes accelerator-specific execution or lowering rules.
- **CN**: 围绕 `flattenVecToBits`, `dyn_cast`, `getIntegerType`, `create` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并实现验证、折叠、解析或打印钩子，并编码加速器专用执行或 lowering 规则。

### Lines 157-165
```cpp
  Value canLeave =
      arith::CmpIOp::create(rewriter, loc, arith::CmpIPredicate::eq,
                            atomicResForCompare, prevLoadForCompare);
  cf::CondBranchOp::create(rewriter, loc, canLeave, afterAtomic, ValueRange{},
                           loopBlock, atomicRes);
  rewriter.eraseOp(atomicOp);
  return success();
}

```
- **EN**: Implements logic around `create`, `eraseOp`, `success`; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `create`, `eraseOp`, `success` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 166-179
```cpp
void mlir::amdgpu::populateAmdgpuEmulateAtomicsPatterns(
    ConversionTarget &target, RewritePatternSet &patterns, Chipset chipset,
    PatternBenefit benefit) {
  // gfx10 has no atomic adds.
  if (chipset.majorVersion == 10 || chipset < Chipset(9, 0, 8)) {
    target.addIllegalOp<RawBufferAtomicFaddOp>();
  }
  // gfx11 has no fp16 atomics
  if (chipset.majorVersion == 11) {
    target.addDynamicallyLegalOp<RawBufferAtomicFaddOp>(
        [](RawBufferAtomicFaddOp op) -> bool {
          Type elemType = getElementTypeOrSelf(op.getValue().getType());
          return !isa<Float16Type, BFloat16Type>(elemType);
        });
```
- **EN**: Implements logic around `populateAmdgpuEmulateAtomicsPatterns`, `Chipset`, `addIllegalOp`, `addDynamicallyLegalOp`, and 2 more symbols; this block encodes accelerator-specific execution or lowering rules.
- **CN**: 围绕 `populateAmdgpuEmulateAtomicsPatterns`, `Chipset`, `addIllegalOp`, `addDynamicallyLegalOp`, and 2 more symbols 实现具体逻辑；该代码块编码加速器专用执行或 lowering 规则。

### Lines 180-193
```cpp
  }
  // gfx9 has no to a very limited support for floating-point min and max.
  if (chipset.majorVersion == 9) {
    if (chipset >= Chipset(9, 0, 0xa)) {
      // gfx90a supports f64 max (and min, but we don't have a min wrapper right
      // now) but all other types need to be emulated.
      target.addDynamicallyLegalOp<RawBufferAtomicFmaxOp>(
          [](RawBufferAtomicFmaxOp op) -> bool {
            return op.getValue().getType().isF64();
          });
    } else {
      target.addIllegalOp<RawBufferAtomicFmaxOp>();
    }
    // TODO(https://github.com/llvm/llvm-project/issues/129206): Refactor
```
- **EN**: Implements logic around `Chipset`, `addDynamicallyLegalOp`, `getValue`, `addIllegalOp`.
- **CN**: 围绕 `Chipset`, `addDynamicallyLegalOp`, `getValue`, `addIllegalOp` 实现具体逻辑。

### Lines 194-207
```cpp
    // this to avoid hardcoding ISA version: gfx950 has bf16 atomics.
    if (chipset < Chipset(9, 5, 0)) {
      target.addDynamicallyLegalOp<RawBufferAtomicFaddOp>(
          [](RawBufferAtomicFaddOp op) -> bool {
            Type elemType = getElementTypeOrSelf(op.getValue().getType());
            return !isa<BFloat16Type>(elemType);
          });
    }
  }
  patterns.add<
      RawBufferAtomicByCasPattern<RawBufferAtomicFaddOp, arith::AddFOp>,
      RawBufferAtomicByCasPattern<RawBufferAtomicFmaxOp, arith::MaximumFOp>,
      RawBufferAtomicByCasPattern<RawBufferAtomicSmaxOp, arith::MaxSIOp>,
      RawBufferAtomicByCasPattern<RawBufferAtomicUminOp, arith::MinUIOp>>(
```
- **EN**: Implements logic around `Chipset`, `addDynamicallyLegalOp`, `getElementTypeOrSelf`, `isa`, and 1 more symbols.
- **CN**: 围绕 `Chipset`, `addDynamicallyLegalOp`, `getElementTypeOrSelf`, `isa`, and 1 more symbols 实现具体逻辑。

### Lines 208-218
```cpp
      patterns.getContext(), benefit);
}

void AmdgpuEmulateAtomicsPass::runOnOperation() {
  Operation *op = getOperation();
  FailureOr<Chipset> maybeChipset = Chipset::parse(chipset);
  if (failed(maybeChipset)) {
    emitError(op->getLoc(), "Invalid chipset name: " + chipset);
    return signalPassFailure();
  }

```
- **EN**: Implements logic around `getContext`, `runOnOperation`, `getOperation`, `parse`, and 3 more symbols; this block participates in pass execution or pass construction; implements verifier, folding, parsing, or printing hooks.
- **CN**: 围绕 `getContext`, `runOnOperation`, `getOperation`, `parse`, and 3 more symbols 实现具体逻辑；该代码块参与 pass 执行或 pass 构造，并实现验证、折叠、解析或打印钩子。

### Lines 219-228
```cpp
  MLIRContext &ctx = getContext();
  ConversionTarget target(ctx);
  RewritePatternSet patterns(&ctx);
  target.markUnknownOpDynamicallyLegal(
      [](Operation *op) -> bool { return true; });

  populateAmdgpuEmulateAtomicsPatterns(target, patterns, *maybeChipset);
  if (failed(applyPartialConversion(op, target, std::move(patterns))))
    return signalPassFailure();
}
```
- **EN**: Implements logic around `getContext`, `target`, `patterns`, `markUnknownOpDynamicallyLegal`, and 3 more symbols.
- **CN**: 围绕 `getContext`, `target`, `patterns`, `markUnknownOpDynamicallyLegal`, and 3 more symbols 实现具体逻辑。

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
- **Region-based control flow / 基于 Region 的控制流**:
  - **EN**: Represents nested blocks and successors as first-class IR structure.
  - **CN**: 把嵌套 block 与后继边表示为一等 IR 结构。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/AMDGPU/Transforms/Passes.h`, `mlir/Dialect/AMDGPU/IR/AMDGPUDialect.h`, `mlir/Dialect/AMDGPU/Utils/Chipset.h`, `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/ControlFlow/IR/ControlFlowOps.h`, `mlir/Dialect/Vector/IR/VectorOps.h`, `mlir/IR/BuiltinAttributes.h`, `mlir/IR/TypeUtilities.h`, `mlir/Transforms/DialectConversion.h`, `mlir/Dialect/AMDGPU/Transforms/Passes.h.inc`
- **Subsystem categories / 子系统类别**: other MLIR dialect declarations / 其他 MLIR 方言声明 (7), MLIR IR core abstractions / MLIR IR 核心抽象 (2), common transformation and pattern-rewrite helpers / 通用变换与模式重写辅助工具 (1)
