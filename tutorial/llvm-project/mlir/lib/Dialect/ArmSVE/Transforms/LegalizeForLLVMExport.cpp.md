# LegalizeForLLVMExport.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/ArmSVE/Transforms/LegalizeForLLVMExport.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements rewrite patterns and transformation passes for the ArmSVE dialect and scalable-vector support.
  - **CN**: 实现 ArmSVE 方言与可扩展向量支持 的重写模式与变换 pass。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- LegalizeForLLVMExport.cpp - Prepare ArmSVE for LLVM translation ----===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件的高层描述。

### Lines 8-17
```cpp

#include "mlir/Conversion/LLVMCommon/ConversionTarget.h"
#include "mlir/Conversion/LLVMCommon/Pattern.h"
#include "mlir/Dialect/ArmSVE/IR/ArmSVEDialect.h"
#include "mlir/Dialect/ArmSVE/Transforms/Transforms.h"
#include "mlir/Dialect/LLVMIR/LLVMDialect.h"
#include "mlir/Dialect/Utils/IndexingUtils.h"
#include "mlir/Dialect/Vector/IR/VectorOps.h"
#include "mlir/IR/PatternMatch.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Conversion/LLVMCommon/ConversionTarget.h`, `mlir/Conversion/LLVMCommon/Pattern.h`, `mlir/Dialect/ArmSVE/IR/ArmSVEDialect.h`, `mlir/Dialect/ArmSVE/Transforms/Transforms.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Conversion/LLVMCommon/ConversionTarget.h`, `mlir/Conversion/LLVMCommon/Pattern.h`, `mlir/Dialect/ArmSVE/IR/ArmSVEDialect.h`, `mlir/Dialect/ArmSVE/Transforms/Transforms.h`。

### Lines 18-31
```cpp
using namespace mlir;
using namespace mlir::arm_sve;

using SdotOpLowering = OneToOneConvertToLLVMPattern<SdotOp, SdotIntrOp>;
using SmmlaOpLowering = OneToOneConvertToLLVMPattern<SmmlaOp, SmmlaIntrOp>;
using UdotOpLowering = OneToOneConvertToLLVMPattern<UdotOp, UdotIntrOp>;
using UmmlaOpLowering = OneToOneConvertToLLVMPattern<UmmlaOp, UmmlaIntrOp>;
using UsmmlaOpLowering = OneToOneConvertToLLVMPattern<UsmmlaOp, UsmmlaIntrOp>;
using DupQLaneLowering =
    OneToOneConvertToLLVMPattern<DupQLaneOp, DupQLaneIntrOp>;
using ScalableMaskedAddIOpLowering =
    OneToOneConvertToLLVMPattern<ScalableMaskedAddIOp,
                                 ScalableMaskedAddIIntrOp>;
using ScalableMaskedAddFOpLowering =
```
- **EN**: Contains supporting implementation details for the surrounding MLIR dialect component.
- **CN**: 包含周边 MLIR 方言组件所需的辅助实现细节。

### Lines 32-45
```cpp
    OneToOneConvertToLLVMPattern<ScalableMaskedAddFOp,
                                 ScalableMaskedAddFIntrOp>;
using ScalableMaskedSubIOpLowering =
    OneToOneConvertToLLVMPattern<ScalableMaskedSubIOp,
                                 ScalableMaskedSubIIntrOp>;
using ScalableMaskedSubFOpLowering =
    OneToOneConvertToLLVMPattern<ScalableMaskedSubFOp,
                                 ScalableMaskedSubFIntrOp>;
using ScalableMaskedMulIOpLowering =
    OneToOneConvertToLLVMPattern<ScalableMaskedMulIOp,
                                 ScalableMaskedMulIIntrOp>;
using ScalableMaskedMulFOpLowering =
    OneToOneConvertToLLVMPattern<ScalableMaskedMulFOp,
                                 ScalableMaskedMulFIntrOp>;
```
- **EN**: Contains supporting implementation details for the surrounding MLIR dialect component.
- **CN**: 包含周边 MLIR 方言组件所需的辅助实现细节。

### Lines 46-55
```cpp
using ScalableMaskedSDivIOpLowering =
    OneToOneConvertToLLVMPattern<ScalableMaskedSDivIOp,
                                 ScalableMaskedSDivIIntrOp>;
using ScalableMaskedUDivIOpLowering =
    OneToOneConvertToLLVMPattern<ScalableMaskedUDivIOp,
                                 ScalableMaskedUDivIIntrOp>;
using ScalableMaskedDivFOpLowering =
    OneToOneConvertToLLVMPattern<ScalableMaskedDivFOp,
                                 ScalableMaskedDivFIntrOp>;

```
- **EN**: Contains supporting implementation details for the surrounding MLIR dialect component.
- **CN**: 包含周边 MLIR 方言组件所需的辅助实现细节。

### Lines 56-62
```cpp
namespace {

/// Unrolls a conversion to/from equivalent vector types, to allow using a
/// conversion intrinsic that only supports 1-D vector types.
///
/// Example:
/// ```
```
- **EN**: Contains supporting implementation details for the surrounding MLIR dialect component.
- **CN**: 包含周边 MLIR 方言组件所需的辅助实现细节。

### Lines 63-69
```cpp
/// %result = arm_sve.convert_to_svbool %source : vector<2x[4]xi1>
/// ```
/// is rewritten into:
/// ```
/// %cst = arith.constant dense<false> : vector<2x[16]xi1>
/// %1 = vector.extract %source[0] : vector<[4]xi1> from vector<2x[4]xi1>
/// %2 = "arm_sve.intr.convert.to.svbool"(%1)
```
- **EN**: Documents the next declarations or records design constraints for the surrounding implementation.
- **CN**: 为接下来的声明提供说明，或记录周边实现的设计约束。

### Lines 70-76
```cpp
///                : (vector<[4]xi1>) -> vector<[16]xi1>
/// %3 = vector.insert %2, %cst[0] : vector<[16]xi1> into vector<2x[16]xi1>
/// %4 = vector.extract %source[1] : vector<[4]xi1> from vector<2x[4]xi1>
/// %5 = "arm_sve.intr.convert.to.svbool"(%4)
///                : (vector<[4]xi1>) -> vector<[16]xi1>
/// %result = vector.insert %5, %3[1] : vector<[16]xi1> into vector<2x[16]xi1>
/// ```
```
- **EN**: Documents the next declarations or records design constraints for the surrounding implementation.
- **CN**: 为接下来的声明提供说明，或记录周边实现的设计约束。

### Lines 77-85
```cpp
template <typename Op, typename IntrOp>
struct SvboolConversionOpLowering : public ConvertOpToLLVMPattern<Op> {
  using ConvertOpToLLVMPattern<Op>::ConvertOpToLLVMPattern;

  LogicalResult
  matchAndRewrite(Op convertOp, typename Op::Adaptor,
                  ConversionPatternRewriter &rewriter) const override {
    auto loc = convertOp.getLoc();

```
- **EN**: Introduces declarations for `SvboolConversionOpLowering`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `SvboolConversionOpLowering` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 86-92
```cpp
    auto source = convertOp.getSource();
    VectorType sourceType = source.getType();
    VectorType resultType = convertOp.getResult().getType();

    Value result = arith::ConstantOp::create(rewriter, loc, resultType,
                                             rewriter.getZeroAttr(resultType));

```
- **EN**: Implements logic around `getSource`, `getType`, `getResult`, `create`, and 1 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `getSource`, `getType`, `getResult`, `create`, and 1 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 93-106
```cpp
    // We want to iterate over the input vector in steps of the trailing
    // dimension. So this creates tile shape where all leading dimensions are 1,
    // and the trailing dimension step is the size of the dimension.
    SmallVector<int64_t> tileShape(sourceType.getRank(), 1);
    tileShape.back() = sourceType.getShape().back();

    // Iterate over all scalable mask/predicate slices of the source vector.
    for (SmallVector<int64_t> index :
         StaticTileOffsetRange(sourceType.getShape(), tileShape)) {
      auto extractOrInsertPosition = ArrayRef(index).drop_back();
      auto sourceVector = vector::ExtractOp::create(rewriter, loc, source,
                                                    extractOrInsertPosition);
      VectorType convertedType =
          VectorType::Builder(llvm::cast<VectorType>(sourceVector.getType()))
```
- **EN**: Implements logic around `tileShape`, `back`, `StaticTileOffsetRange`, `ArrayRef`, and 2 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `tileShape`, `back`, `StaticTileOffsetRange`, `ArrayRef`, and 2 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 107-113
```cpp
              .setDim(0, resultType.getShape().back());
      auto convertedVector =
          IntrOp::create(rewriter, loc, TypeRange{convertedType}, sourceVector);
      result = vector::InsertOp::create(rewriter, loc, convertedVector, result,
                                        extractOrInsertPosition);
    }

```
- **EN**: Implements logic around `setDim`, `create`; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `setDim`, `create` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 114-121
```cpp
    rewriter.replaceOp(convertOp, result);
    return success();
  }
};

using ConvertToSvboolOpLowering =
    SvboolConversionOpLowering<ConvertToSvboolOp, ConvertToSvboolIntrOp>;

```
- **EN**: Implements logic around `replaceOp`, `success`; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `replaceOp`, `success` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 122-128
```cpp
using ConvertFromSvboolOpLowering =
    SvboolConversionOpLowering<ConvertFromSvboolOp, ConvertFromSvboolIntrOp>;

using ZipX2OpLowering = OneToOneConvertToLLVMPattern<ZipX2Op, ZipX2IntrOp>;
using ZipX4OpLowering = OneToOneConvertToLLVMPattern<ZipX4Op, ZipX4IntrOp>;

/// Lower `arm_sve.psel` to LLVM intrinsics. This is almost a 1-to-1 conversion
```
- **EN**: Contains supporting implementation details for the surrounding MLIR dialect component.
- **CN**: 包含周边 MLIR 方言组件所需的辅助实现细节。

### Lines 129-142
```cpp
/// but first input (P1) and result predicates need conversion to/from svbool.
struct PselOpLowering : public ConvertOpToLLVMPattern<PselOp> {
  using ConvertOpToLLVMPattern::ConvertOpToLLVMPattern;

  LogicalResult
  matchAndRewrite(PselOp pselOp, PselOp::Adaptor adaptor,
                  ConversionPatternRewriter &rewriter) const override {
    auto svboolType = VectorType::get(16, rewriter.getI1Type(), true);
    auto loc = pselOp.getLoc();
    auto svboolP1 = ConvertToSvboolIntrOp::create(rewriter, loc, svboolType,
                                                  adaptor.getP1());
    auto indexI32 = arith::IndexCastOp::create(
        rewriter, loc, rewriter.getI32Type(), pselOp.getIndex());
    auto pselIntr = PselIntrOp::create(rewriter, loc, svboolType, svboolP1,
```
- **EN**: Introduces declarations for `PselOpLowering`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `PselOpLowering` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 143-149
```cpp
                                       pselOp.getP2(), indexI32);
    rewriter.replaceOpWithNewOp<ConvertFromSvboolIntrOp>(
        pselOp, adaptor.getP1().getType(), pselIntr);
    return success();
  }
};

```
- **EN**: Implements logic around `getP2`, `replaceOpWithNewOp`, `getP1`, `success`; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `getP2`, `replaceOpWithNewOp`, `getP1`, `success` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 150-156
```cpp
/// Converts `vector.create_mask` ops that match the size of an SVE predicate
/// to the `whilelt` intrinsic. This produces more canonical codegen than the
/// generic LLVM lowering, see https://github.com/llvm/llvm-project/issues/81840
/// for more details. Note that we can't use (the more general) active.lane.mask
/// as its semantics don't neatly map on to `vector.create_mask`, as it does an
/// unsigned comparison (whereas `create_mask` is signed), and is UB/posion if
/// `n` is zero (whereas `create_mask` just returns an all-false mask).
```
- **EN**: Documents the next declarations or records design constraints for the surrounding implementation.
- **CN**: 为接下来的声明提供说明，或记录周边实现的设计约束。

### Lines 157-168
```cpp
struct CreateMaskOpLowering
    : public ConvertOpToLLVMPattern<vector::CreateMaskOp> {
  using ConvertOpToLLVMPattern::ConvertOpToLLVMPattern;

  LogicalResult
  matchAndRewrite(vector::CreateMaskOp createMaskOp,
                  vector::CreateMaskOp::Adaptor adaptor,
                  ConversionPatternRewriter &rewriter) const override {
    auto maskType = createMaskOp.getVectorType();
    if (maskType.getRank() != 1 || !maskType.isScalable())
      return rewriter.notifyMatchFailure(createMaskOp, "not 1-D and scalable");

```
- **EN**: Introduces declarations for `CreateMaskOpLowering`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `CreateMaskOpLowering` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 169-175
```cpp
    // TODO: Support masks which are multiples of SVE predicates.
    auto maskBaseSize = maskType.getDimSize(0);
    if (maskBaseSize < 2 || maskBaseSize > 16 ||
        !llvm::isPowerOf2_32(uint32_t(maskBaseSize)))
      return rewriter.notifyMatchFailure(createMaskOp,
                                         "not SVE predicate-sized");

```
- **EN**: Implements logic around `getDimSize`, `isPowerOf2_32`, `notifyMatchFailure`; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `getDimSize`, `isPowerOf2_32`, `notifyMatchFailure` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 176-183
```cpp
    auto loc = createMaskOp.getLoc();
    auto zero = LLVM::ZeroOp::create(rewriter, loc, rewriter.getI64Type());
    rewriter.replaceOpWithNewOp<WhileLTIntrOp>(createMaskOp, maskType, zero,
                                               adaptor.getOperands()[0]);
    return success();
  }
};

```
- **EN**: Implements logic around `getLoc`, `create`, `replaceOpWithNewOp`, `getOperands`, and 1 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `getLoc`, `create`, `replaceOpWithNewOp`, `getOperands`, and 1 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 184-190
```cpp
} // namespace

/// Populate the given list with patterns that convert from ArmSVE to LLVM.
void mlir::populateArmSVELegalizeForLLVMExportPatterns(
    const LLVMTypeConverter &converter, RewritePatternSet &patterns) {
  // Populate conversion patterns

```
- **EN**: Implements logic around `populateArmSVELegalizeForLLVMExportPatterns`.
- **CN**: 围绕 `populateArmSVELegalizeForLLVMExportPatterns` 实现具体逻辑。

### Lines 191-204
```cpp
  // clang-format off
  patterns.add<ConvertFromSvboolOpLowering,
               ConvertToSvboolOpLowering,
               DupQLaneLowering,
               PselOpLowering,
               ScalableMaskedAddFOpLowering,
               ScalableMaskedAddIOpLowering,
               ScalableMaskedDivFOpLowering,
               ScalableMaskedMulFOpLowering,
               ScalableMaskedMulIOpLowering,
               ScalableMaskedSDivIOpLowering,
               ScalableMaskedSubFOpLowering,
               ScalableMaskedSubIOpLowering,
               ScalableMaskedUDivIOpLowering,
```
- **EN**: Contains supporting implementation details for the surrounding MLIR dialect component.
- **CN**: 包含周边 MLIR 方言组件所需的辅助实现细节。

### Lines 205-217
```cpp
               SmmlaOpLowering,
               UdotOpLowering,
               UmmlaOpLowering,
               UsmmlaOpLowering,
               ZipX2OpLowering,
               ZipX4OpLowering,
               SdotOpLowering>(converter);
  // Add vector.create_mask conversion with a high benefit as it produces much
  // nicer code than the generic lowering.
  patterns.add<CreateMaskOpLowering>(converter, /*benefit=*/4096);
  // clang-format on
}

```
- **EN**: Implements logic around `SdotOpLowering>`, `add`.
- **CN**: 围绕 `SdotOpLowering>`, `add` 实现具体逻辑。

### Lines 218-231
```cpp
void mlir::configureArmSVELegalizeForExportTarget(
    LLVMConversionTarget &target) {
  // clang-format off
  target.addLegalOp<BfmmlaOp,
                    ConvertFromSvboolIntrOp,
                    ConvertToSvboolIntrOp,
                    DupQLaneIntrOp,
                    PselIntrOp,
                    ScalableMaskedAddFIntrOp,
                    ScalableMaskedAddIIntrOp,
                    ScalableMaskedDivFIntrOp,
                    ScalableMaskedMulFIntrOp,
                    ScalableMaskedMulIIntrOp,
                    ScalableMaskedSDivIIntrOp,
```
- **EN**: Implements logic around `configureArmSVELegalizeForExportTarget`.
- **CN**: 围绕 `configureArmSVELegalizeForExportTarget` 实现具体逻辑。

### Lines 232-245
```cpp
                    ScalableMaskedSubFIntrOp,
                    ScalableMaskedSubIIntrOp,
                    ScalableMaskedUDivIIntrOp,
                    SmmlaIntrOp,
                    UdotIntrOp,
                    UmmlaIntrOp,
                    UsmmlaIntrOp,
                    WhileLTIntrOp,
                    ZipX2IntrOp,
                    ZipX4IntrOp,
                    SdotIntrOp>();
  target.addIllegalOp<ConvertFromSvboolOp,
                      ConvertToSvboolOp,
                      DupQLaneOp,
```
- **EN**: Implements logic around `SdotIntrOp>`.
- **CN**: 围绕 `SdotIntrOp>` 实现具体逻辑。

### Lines 246-259
```cpp
                      PselOp,
                      ScalableMaskedAddFOp,
                      ScalableMaskedAddIOp,
                      ScalableMaskedDivFOp,
                      ScalableMaskedMulFOp,
                      ScalableMaskedMulIOp,
                      ScalableMaskedSDivIOp,
                      ScalableMaskedSubFOp,
                      ScalableMaskedSubIOp,
                      ScalableMaskedUDivIOp,
                      SmmlaOp,
                      UdotOp,
                      UmmlaOp,
                      UsmmlaOp,
```
- **EN**: Contains supporting implementation details for the surrounding MLIR dialect component.
- **CN**: 包含周边 MLIR 方言组件所需的辅助实现细节。

### Lines 260-264
```cpp
                      ZipX2Op,
                      ZipX4Op,
                      SdotOp>();
  // clang-format on
}
```
- **EN**: Implements logic around `SdotOp>`.
- **CN**: 围绕 `SdotOp>` 实现具体逻辑。

## Key Concepts / 关键概念

- **Pattern rewriting and passes / 模式重写与 Pass**:
  - **EN**: Applies conversion patterns, canonicalizations, or pass pipelines over MLIR operations.
  - **CN**: 对 MLIR 操作应用转换模式、规范化或 pass 流水线。
- **Type/attribute storage / 类型/属性存储**:
  - **EN**: Uses uniqued storage and parser/printer hooks for extensible compile-time metadata.
  - **CN**: 使用唯一化存储以及解析/打印钩子来表示可扩展的编译期元数据。
- **LLVM interop / LLVM 互操作**:
  - **EN**: Bridges MLIR concepts to LLVM-compatible data structures, intrinsics, or codegen expectations.
  - **CN**: 把 MLIR 概念桥接到 LLVM 兼容的数据结构、intrinsic 或代码生成预期。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Conversion/LLVMCommon/ConversionTarget.h`, `mlir/Conversion/LLVMCommon/Pattern.h`, `mlir/Dialect/ArmSVE/IR/ArmSVEDialect.h`, `mlir/Dialect/ArmSVE/Transforms/Transforms.h`, `mlir/Dialect/LLVMIR/LLVMDialect.h`, `mlir/Dialect/Utils/IndexingUtils.h`, `mlir/Dialect/Vector/IR/VectorOps.h`, `mlir/IR/PatternMatch.h`
- **Subsystem categories / 子系统类别**: other MLIR dialect declarations / 其他 MLIR 方言声明 (5), dialect conversion infrastructure / 方言转换基础设施 (2), MLIR IR core abstractions / MLIR IR 核心抽象 (1)
