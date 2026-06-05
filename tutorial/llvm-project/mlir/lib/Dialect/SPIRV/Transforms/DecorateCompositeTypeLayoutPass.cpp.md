# DecorateCompositeTypeLayoutPass.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/SPIRV/Transforms/DecorateCompositeTypeLayoutPass.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This file implements a pass to decorate the composite types used by composite objects in the StorageBuffer, PhysicalStorageBuffer, Uniform, and PushConstant storage classes with layout information. See SPIR-V spec "2.16.2. Validation Rules for Shader Capabilities" for more details.
  - **CN**: 该文件位于 `mlir/lib/Dialect/SPIRV/Transforms`，围绕 SPIRV 方言实现相关能力。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- DecorateCompositeTypeLayoutPass.cpp - Decorate composite type ------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and file-level summary.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件级摘要。

### Lines 8-14
```cpp
//
// This file implements a pass to decorate the composite types used by
// composite objects in the StorageBuffer, PhysicalStorageBuffer, Uniform, and
// PushConstant storage classes with layout information. See SPIR-V spec
// "2.16.2. Validation Rules for Shader Capabilities" for more details.
//
//===----------------------------------------------------------------------===//
```
- **EN**: Documents the next declarations, transformation intent, or design constraints for this file.
- **CN**: 为后续声明、变换意图或设计约束提供说明。

### Lines 15-23
```cpp

#include "mlir/Dialect/SPIRV/Transforms/Passes.h"

#include "mlir/Dialect/Func/IR/FuncOps.h"
#include "mlir/Dialect/SPIRV/IR/SPIRVDialect.h"
#include "mlir/Dialect/SPIRV/IR/SPIRVOps.h"
#include "mlir/Dialect/SPIRV/Utils/LayoutUtils.h"
#include "mlir/Transforms/DialectConversion.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/SPIRV/Transforms/Passes.h`, `mlir/Dialect/Func/IR/FuncOps.h`, `mlir/Dialect/SPIRV/IR/SPIRVDialect.h`, `mlir/Dialect/SPIRV/IR/SPIRVOps.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/SPIRV/Transforms/Passes.h`, `mlir/Dialect/Func/IR/FuncOps.h`, `mlir/Dialect/SPIRV/IR/SPIRVDialect.h`, `mlir/Dialect/SPIRV/IR/SPIRVOps.h`。

### Lines 24-28
```cpp
#include "llvm/Support/FormatVariadic.h"

using namespace mlir;

namespace mlir {
```
- **EN**: Pulls in the headers needed by this translation unit, including `llvm/Support/FormatVariadic.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `llvm/Support/FormatVariadic.h`。

### Lines 29-34
```cpp
namespace spirv {
#define GEN_PASS_DEF_SPIRVCOMPOSITETYPELAYOUTPASS
#include "mlir/Dialect/SPIRV/Transforms/Passes.h.inc"
} // namespace spirv
} // namespace mlir

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/SPIRV/Transforms/Passes.h.inc`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/SPIRV/Transforms/Passes.h.inc`。

### Lines 35-40
```cpp
namespace {
class SPIRVGlobalVariableOpLayoutInfoDecoration
    : public OpRewritePattern<spirv::GlobalVariableOp> {
public:
  using Base::Base;

```
- **EN**: Introduces declarations for `SPIRVGlobalVariableOpLayoutInfoDecoration`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `SPIRVGlobalVariableOpLayoutInfoDecoration` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 41-48
```cpp
  LogicalResult matchAndRewrite(spirv::GlobalVariableOp op,
                                PatternRewriter &rewriter) const override {
    SmallVector<NamedAttribute, 4> globalVarAttrs;

    auto ptrType = cast<spirv::PointerType>(op.getType());
    auto pointeeType = cast<spirv::StructType>(ptrType.getPointeeType());
    spirv::StructType structType = VulkanLayoutUtils::decorateType(pointeeType);

```
- **EN**: Implements logic around `matchAndRewrite`, `PointerType>`, `StructType>`, `decorateType`; this block uses rewrite-pattern infrastructure to transform operations; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `matchAndRewrite`, `PointerType>`, `StructType>`, `decorateType` 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并涉及目标平台或加速器专用语义。

### Lines 49-55
```cpp
    if (!structType)
      return op->emitError(llvm::formatv(
          "failed to decorate (unsuported pointee type: '{0}')", pointeeType));

    auto decoratedType =
        spirv::PointerType::get(structType, ptrType.getStorageClass());

```
- **EN**: Implements logic around `emitError`, `decorate`, `get`; this block touches target- or accelerator-specific semantics.
- **CN**: 围绕 `emitError`, `decorate`, `get` 实现具体逻辑；该代码块涉及目标平台或加速器专用语义。

### Lines 56-62
```cpp
    // Save all named attributes except "type" attribute.
    for (const auto &attr : op->getAttrs()) {
      if (attr.getName() == "type")
        continue;
      globalVarAttrs.push_back(attr);
    }

```
- **EN**: Implements logic around `getAttrs`, `getName`, `push_back`.
- **CN**: 围绕 `getAttrs`, `getName`, `push_back` 实现具体逻辑。

### Lines 63-68
```cpp
    rewriter.replaceOpWithNewOp<spirv::GlobalVariableOp>(
        op, TypeAttr::get(decoratedType), globalVarAttrs);
    return success();
  }
};

```
- **EN**: Implements logic around `GlobalVariableOp>`, `get`, `success`; this block uses rewrite-pattern infrastructure to transform operations; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `GlobalVariableOp>`, `get`, `success` 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并涉及目标平台或加速器专用语义。

### Lines 69-73
```cpp
class SPIRVAddressOfOpLayoutInfoDecoration
    : public OpRewritePattern<spirv::AddressOfOp> {
public:
  using Base::Base;

```
- **EN**: Introduces declarations for `SPIRVAddressOfOpLayoutInfoDecoration`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `SPIRVAddressOfOpLayoutInfoDecoration` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 74-79
```cpp
  LogicalResult matchAndRewrite(spirv::AddressOfOp op,
                                PatternRewriter &rewriter) const override {
    auto spirvModule = op->getParentOfType<spirv::ModuleOp>();
    auto varName = op.getVariableAttr();
    auto varOp = spirvModule.lookupSymbol<spirv::GlobalVariableOp>(varName);

```
- **EN**: Implements logic around `matchAndRewrite`, `ModuleOp>`, `getVariableAttr`, `GlobalVariableOp>`; this block uses rewrite-pattern infrastructure to transform operations; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `matchAndRewrite`, `ModuleOp>`, `getVariableAttr`, `GlobalVariableOp>` 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并涉及目标平台或加速器专用语义。

### Lines 80-85
```cpp
    rewriter.replaceOpWithNewOp<spirv::AddressOfOp>(
        op, varOp.getType(), SymbolRefAttr::get(varName.getAttr()));
    return success();
  }
};

```
- **EN**: Implements logic around `AddressOfOp>`, `getType`, `success`; this block uses rewrite-pattern infrastructure to transform operations; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `AddressOfOp>`, `getType`, `success` 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并涉及目标平台或加速器专用语义。

### Lines 86-90
```cpp
template <typename OpT>
class SPIRVPassThroughConversion : public OpConversionPattern<OpT> {
public:
  using OpConversionPattern<OpT>::OpConversionPattern;

```
- **EN**: Introduces declarations for `SPIRVPassThroughConversion`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `SPIRVPassThroughConversion` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 91-100
```cpp
  LogicalResult
  matchAndRewrite(OpT op, typename OpT::Adaptor adaptor,
                  ConversionPatternRewriter &rewriter) const override {
    rewriter.modifyOpInPlace(op,
                             [&] { op->setOperands(adaptor.getOperands()); });
    return success();
  }
};
} // namespace

```
- **EN**: Implements logic around `matchAndRewrite`, `modifyOpInPlace`, `setOperands`, `success`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `matchAndRewrite`, `modifyOpInPlace`, `setOperands`, `success` 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 101-109
```cpp
static void populateSPIRVLayoutInfoPatterns(RewritePatternSet &patterns) {
  patterns.add<SPIRVGlobalVariableOpLayoutInfoDecoration,
               SPIRVAddressOfOpLayoutInfoDecoration,
               SPIRVPassThroughConversion<spirv::AccessChainOp>,
               SPIRVPassThroughConversion<spirv::LoadOp>,
               SPIRVPassThroughConversion<spirv::StoreOp>>(
      patterns.getContext());
}

```
- **EN**: Implements logic around `populateSPIRVLayoutInfoPatterns`, `StoreOp>>`, `getContext`; this block touches target- or accelerator-specific semantics.
- **CN**: 围绕 `populateSPIRVLayoutInfoPatterns`, `StoreOp>>`, `getContext` 实现具体逻辑；该代码块涉及目标平台或加速器专用语义。

### Lines 110-117
```cpp
namespace {
class DecorateSPIRVCompositeTypeLayoutPass
    : public spirv::impl::SPIRVCompositeTypeLayoutPassBase<
          DecorateSPIRVCompositeTypeLayoutPass> {
  void runOnOperation() override;
};
} // namespace

```
- **EN**: Introduces declarations for `DecorateSPIRVCompositeTypeLayoutPass`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `DecorateSPIRVCompositeTypeLayoutPass` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 118-127
```cpp
void DecorateSPIRVCompositeTypeLayoutPass::runOnOperation() {
  auto module = getOperation();
  RewritePatternSet patterns(module.getContext());
  populateSPIRVLayoutInfoPatterns(patterns);
  ConversionTarget target(*(module.getContext()));
  target.addLegalDialect<spirv::SPIRVDialect>();
  target.addLegalOp<func::FuncOp>();
  target.addDynamicallyLegalOp<spirv::GlobalVariableOp>(
      [](spirv::GlobalVariableOp op) {
        return VulkanLayoutUtils::isLegalType(op.getType());
```
- **EN**: Implements logic around `runOnOperation`, `getOperation`, `patterns`, `populateSPIRVLayoutInfoPatterns`, and 5 more symbols; this block packages logic as an MLIR pass or pass helper; coordinates dialect conversion or lowering decisions; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `runOnOperation`, `getOperation`, `patterns`, `populateSPIRVLayoutInfoPatterns`, and 5 more symbols 实现具体逻辑；该代码块将逻辑组织为 MLIR Pass 或 Pass 辅助组件，并协调方言转换或 lowering 决策，并涉及目标平台或加速器专用语义。

### Lines 128-134
```cpp
      });

  // Change the type for the direct users.
  target.addDynamicallyLegalOp<spirv::AddressOfOp>([](spirv::AddressOfOp op) {
    return VulkanLayoutUtils::isLegalType(op.getPointer().getType());
  });

```
- **EN**: Implements logic around `AddressOfOp>`, `isLegalType`; this block touches target- or accelerator-specific semantics.
- **CN**: 围绕 `AddressOfOp>`, `isLegalType` 实现具体逻辑；该代码块涉及目标平台或加速器专用语义。

### Lines 135-144
```cpp
  // Change the type for the indirect users.
  target.addDynamicallyLegalOp<spirv::AccessChainOp, spirv::LoadOp,
                               spirv::StoreOp>([&](Operation *op) {
    for (Value operand : op->getOperands()) {
      auto addrOp = operand.getDefiningOp<spirv::AddressOfOp>();
      if (addrOp &&
          !VulkanLayoutUtils::isLegalType(addrOp.getPointer().getType()))
        return false;
    }
    return true;
```
- **EN**: Implements logic around `StoreOp>`, `getOperands`, `AddressOfOp>`, `isLegalType`; this block touches target- or accelerator-specific semantics.
- **CN**: 围绕 `StoreOp>`, `getOperands`, `AddressOfOp>`, `isLegalType` 实现具体逻辑；该代码块涉及目标平台或加速器专用语义。

### Lines 145-151
```cpp
  });

  FrozenRewritePatternSet frozenPatterns(std::move(patterns));
  for (auto spirvModule : module.getOps<spirv::ModuleOp>())
    if (failed(applyFullConversion(spirvModule, target, frozenPatterns)))
      signalPassFailure();
}
```
- **EN**: Implements logic around `frozenPatterns`, `ModuleOp>`, `failed`, `signalPassFailure`; this block touches target- or accelerator-specific semantics.
- **CN**: 围绕 `frozenPatterns`, `ModuleOp>`, `failed`, `signalPassFailure` 实现具体逻辑；该代码块涉及目标平台或加速器专用语义。

## Key Concepts / 关键概念

- **Rewrite and lowering / 重写与 lowering**:
  - **EN**: Uses MLIR pattern infrastructure or passes to canonicalize, legalize, or lower operations.
  - **CN**: 使用 MLIR 模式基础设施或 Pass 对操作进行规范化、合法化或 lowering。
- **Pattern rewriting / 模式重写**:
  - **EN**: Uses MLIR rewrite patterns to match operations and replace them with improved forms.
  - **CN**: 使用 MLIR 重写模式匹配操作，并将其替换为更合适的形式。
- **Dialect conversion pipeline / 方言转换流水线**:
  - **EN**: Coordinates legality checks, type adaptation, and target-specific lowering.
  - **CN**: 协调合法性检查、类型适配以及目标相关 lowering。
- **Pass infrastructure / Pass 基础设施**:
  - **EN**: Wraps transformations as reusable passes that can be scheduled in MLIR pipelines.
  - **CN**: 将变换封装为可在 MLIR 流水线中调度的可复用 Pass。
- **Structured IR coordination / 结构化 IR 协同**:
  - **EN**: Interacts with structured MLIR dialects that model loops, tensors, memory, and vector semantics.
  - **CN**: 与建模循环、张量、内存和向量语义的结构化 MLIR 方言交互。
- **Target-specific semantics / 目标相关语义**:
  - **EN**: Encodes rules tied to accelerator, GPU, or binary target environments.
  - **CN**: 编码与加速器、GPU 或二进制目标环境绑定的规则。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/SPIRV/Transforms/Passes.h`, `mlir/Dialect/Func/IR/FuncOps.h`, `mlir/Dialect/SPIRV/IR/SPIRVDialect.h`, `mlir/Dialect/SPIRV/IR/SPIRVOps.h`, `mlir/Dialect/SPIRV/Utils/LayoutUtils.h`, `mlir/Transforms/DialectConversion.h`, `llvm/Support/FormatVariadic.h`, `mlir/Dialect/SPIRV/Transforms/Passes.h.inc`
- **Subsystem categories / 子系统类别**: dialect-specific operation, type, attribute, or transform declarations / 方言相关的操作、类型、属性或变换声明 (6), generic transformation and conversion helpers / 通用变换与转换辅助逻辑 (1), LLVM support-library facilities / LLVM Support 库设施 (1)
