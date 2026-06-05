# EnableArmStreaming.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/ArmSME/Transforms/EnableArmStreaming.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This pass enables the Armv9 Scalable Matrix Extension (SME) Streaming SVE (SSVE) mode [1][2] by adding either of the following attributes to 'func.func' ops:.
  - **CN**: 实现 ArmSME 方言与 Arm SME tile/vector 支持 的重写模式与变换 pass。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- EnableArmStreaming.cpp - Enable Armv9 Streaming SVE mode -----------===//
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
//
// This pass enables the Armv9 Scalable Matrix Extension (SME) Streaming SVE
// (SSVE) mode [1][2] by adding either of the following attributes to
// 'func.func' ops:
//
//   * 'arm_streaming' (default)
//   * 'arm_locally_streaming'
//
// It can also optionally enable the ZA storage array.
//
```
- **EN**: Documents the next declarations or records design constraints for the surrounding implementation.
- **CN**: 为接下来的声明提供说明，或记录周边实现的设计约束。

### Lines 18-27
```cpp
// Streaming-mode is part of the interface (ABI) for functions with the
// first attribute and it's the responsibility of the caller to manage
// PSTATE.SM on entry/exit to functions with this attribute [3]. The LLVM
// backend will emit 'smstart sm' / 'smstop sm' [4] around calls to
// streaming functions.
//
// In locally streaming functions PSTATE.SM is kept internal and managed by
// the callee on entry/exit. The LLVM backend will emit 'smstart sm' /
// 'smstop sm' in the prologue / epilogue for functions with this
// attribute.
```
- **EN**: Documents the next declarations or records design constraints for the surrounding implementation.
- **CN**: 为接下来的声明提供说明，或记录周边实现的设计约束。

### Lines 28-34
```cpp
//
// [1] https://developer.arm.com/documentation/ddi0616/aa
// [2] https://llvm.org/docs/AArch64SME.html
// [3] https://github.com/ARM-software/abi-aa/blob/main/aapcs64/aapcs64.rst#671pstatesm-interfaces
// [4] https://developer.arm.com/documentation/ddi0602/2023-03/Base-Instructions/SMSTART--Enables-access-to-Streaming-SVE-mode-and-SME-architectural-state--an-alias-of-MSR--immediate--
//
//===----------------------------------------------------------------------===//
```
- **EN**: Documents the next declarations or records design constraints for the surrounding implementation.
- **CN**: 为接下来的声明提供说明，或记录周边实现的设计约束。

### Lines 35-39
```cpp

#include "mlir/Dialect/ArmSME/IR/ArmSME.h"
#include "mlir/Dialect/ArmSME/Transforms/Passes.h"
#include "mlir/Dialect/ArmSME/Transforms/PassesEnums.cpp.inc"

```
- **EN**: Bridges to TableGen-generated declarations or definitions that expand MLIR op/type/interface metadata.
- **CN**: 桥接到由 TableGen 生成的声明或定义，用于展开 MLIR 操作/类型/接口元数据。

### Lines 40-44
```cpp
#include "mlir/Dialect/Func/IR/FuncOps.h"

#define DEBUG_TYPE "enable-arm-streaming"

namespace mlir {
```
- **EN**: Introduces declarations for `mlir`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `mlir` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 45-50
```cpp
namespace arm_sme {
#define GEN_PASS_DEF_ENABLEARMSTREAMING
#include "mlir/Dialect/ArmSME/Transforms/Passes.h.inc"
} // namespace arm_sme
} // namespace mlir

```
- **EN**: Introduces declarations for `arm_sme`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `arm_sme` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 51-57
```cpp
using namespace mlir;
using namespace mlir::arm_sme;
namespace {

constexpr StringLiteral
    kEnableArmStreamingIgnoreAttr("llvm.enable_arm_streaming_ignore");

```
- **EN**: Implements logic around `kEnableArmStreamingIgnoreAttr`.
- **CN**: 围绕 `kEnableArmStreamingIgnoreAttr` 实现具体逻辑。

### Lines 58-62
```cpp
template <typename... Ops>
constexpr auto opList() {
  return std::array{TypeID::get<Ops>()...};
}

```
- **EN**: Implements logic around `opList`, `get`.
- **CN**: 围绕 `opList`, `get` 实现具体逻辑。

### Lines 63-68
```cpp
bool isScalableVector(Type type) {
  if (auto vectorType = dyn_cast<VectorType>(type))
    return vectorType.isScalable();
  return false;
}

```
- **EN**: Implements logic around `isScalableVector`, `dyn_cast`, `isScalable`.
- **CN**: 围绕 `isScalableVector`, `dyn_cast`, `isScalable` 实现具体逻辑。

### Lines 69-78
```cpp
struct EnableArmStreamingPass
    : public arm_sme::impl::EnableArmStreamingBase<EnableArmStreamingPass> {
  EnableArmStreamingPass(ArmStreamingMode streamingMode, ArmZaMode zaMode,
                         bool ifRequiredByOps, bool ifScalableAndSupported) {
    this->streamingMode = streamingMode;
    this->zaMode = zaMode;
    this->ifRequiredByOps = ifRequiredByOps;
    this->ifScalableAndSupported = ifScalableAndSupported;
  }
  void runOnOperation() override {
```
- **EN**: Introduces declarations for `EnableArmStreamingPass`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `EnableArmStreamingPass` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 79-87
```cpp
    auto function = getOperation();

    if (ifRequiredByOps && ifScalableAndSupported) {
      function->emitOpError(
          "enable-arm-streaming: `if-required-by-ops` and "
          "`if-scalable-and-supported` are mutually exclusive");
      return signalPassFailure();
    }

```
- **EN**: Implements logic around `getOperation`, `emitOpError`, `signalPassFailure`.
- **CN**: 围绕 `getOperation`, `emitOpError`, `signalPassFailure` 实现具体逻辑。

### Lines 88-97
```cpp
    if (ifRequiredByOps) {
      bool foundTileOp = false;
      function.walk([&](Operation *op) {
        if (llvm::isa<ArmSMETileOpInterface>(op)) {
          foundTileOp = true;
          return WalkResult::interrupt();
        }
        return WalkResult::advance();
      });
      if (!foundTileOp)
```
- **EN**: Implements logic around `walk`, `isa`, `interrupt`, `advance`.
- **CN**: 围绕 `walk`, `isa`, `interrupt`, `advance` 实现具体逻辑。

### Lines 98-107
```cpp
        return;
    }

    if (ifScalableAndSupported) {
      // FIXME: This should be based on target information (i.e., the presence
      // of FEAT_SME_FA64). This currently errs on the side of caution. If
      // possible gathers/scatters should be lowered regular vector loads/stores
      // before invoking this pass.
      auto disallowedOperations = opList<vector::GatherOp, vector::ScatterOp>();
      bool isCompatibleScalableFunction = false;
```
- **EN**: Implements logic around `ScatterOp>`.
- **CN**: 围绕 `ScatterOp>` 实现具体逻辑。

### Lines 108-117
```cpp
      function.walk([&](Operation *op) {
        if (llvm::is_contained(disallowedOperations,
                               op->getName().getTypeID())) {
          isCompatibleScalableFunction = false;
          return WalkResult::interrupt();
        }
        if (!isCompatibleScalableFunction &&
            (llvm::any_of(op->getOperandTypes(), isScalableVector) ||
             llvm::any_of(op->getResultTypes(), isScalableVector))) {
          isCompatibleScalableFunction = true;
```
- **EN**: Implements logic around `walk`, `is_contained`, `getName`, `interrupt`, and 1 more symbols.
- **CN**: 围绕 `walk`, `is_contained`, `getName`, `interrupt`, and 1 more symbols 实现具体逻辑。

### Lines 118-124
```cpp
        }
        return WalkResult::advance();
      });
      if (!isCompatibleScalableFunction)
        return;
    }

```
- **EN**: Implements logic around `advance`.
- **CN**: 围绕 `advance` 实现具体逻辑。

### Lines 125-130
```cpp
    if (function->getAttr(kEnableArmStreamingIgnoreAttr) ||
        streamingMode == ArmStreamingMode::Disabled)
      return;

    auto unitAttr = UnitAttr::get(&getContext());

```
- **EN**: Implements logic around `getAttr`, `get`.
- **CN**: 围绕 `getAttr`, `get` 实现具体逻辑。

### Lines 131-140
```cpp
    function->setDiscardableAttr(
        (Twine("llvm.") + stringifyArmStreamingMode(streamingMode)).str(),
        unitAttr);

    // The pass currently only supports enabling ZA when in streaming-mode, but
    // ZA can be accessed by the SME LDR, STR and ZERO instructions when not in
    // streaming-mode (see section B1.1.1, IDGNQM of spec [1]). It may be worth
    // supporting this later.
    if (zaMode != ArmZaMode::Disabled)
      function->setAttr((Twine("llvm.") + stringifyArmZaMode(zaMode)).str(),
```
- **EN**: Implements logic around `setDiscardableAttr`, `Twine`, `setAttr`.
- **CN**: 围绕 `setDiscardableAttr`, `Twine`, `setAttr` 实现具体逻辑。

### Lines 141-145
```cpp
                        unitAttr);
  }
};
} // namespace

```
- **EN**: Contains supporting implementation details for the surrounding MLIR dialect component.
- **CN**: 包含周边 MLIR 方言组件所需的辅助实现细节。

### Lines 146-151
```cpp
std::unique_ptr<Pass> mlir::arm_sme::createEnableArmStreamingPass(
    const ArmStreamingMode streamingMode, const ArmZaMode zaMode,
    bool ifRequiredByOps, bool ifScalableAndSupported) {
  return std::make_unique<EnableArmStreamingPass>(
      streamingMode, zaMode, ifRequiredByOps, ifScalableAndSupported);
}
```
- **EN**: Implements logic around `createEnableArmStreamingPass`, `make_unique`; this block participates in pass execution or pass construction.
- **CN**: 围绕 `createEnableArmStreamingPass`, `make_unique` 实现具体逻辑；该代码块参与 pass 执行或 pass 构造。

## Key Concepts / 关键概念

- **Pattern rewriting and passes / 模式重写与 Pass**:
  - **EN**: Applies conversion patterns, canonicalizations, or pass pipelines over MLIR operations.
  - **CN**: 对 MLIR 操作应用转换模式、规范化或 pass 流水线。
- **Interface dispatch / 接口分派**:
  - **EN**: Attaches shared behavior to different ops or types through MLIR interfaces.
  - **CN**: 通过 MLIR 接口把共享行为附着到不同操作或类型上。
- **Type/attribute storage / 类型/属性存储**:
  - **EN**: Uses uniqued storage and parser/printer hooks for extensible compile-time metadata.
  - **CN**: 使用唯一化存储以及解析/打印钩子来表示可扩展的编译期元数据。
- **LLVM interop / LLVM 互操作**:
  - **EN**: Bridges MLIR concepts to LLVM-compatible data structures, intrinsics, or codegen expectations.
  - **CN**: 把 MLIR 概念桥接到 LLVM 兼容的数据结构、intrinsic 或代码生成预期。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/ArmSME/IR/ArmSME.h`, `mlir/Dialect/ArmSME/Transforms/Passes.h`, `mlir/Dialect/ArmSME/Transforms/PassesEnums.cpp.inc`, `mlir/Dialect/Func/IR/FuncOps.h`, `mlir/Dialect/ArmSME/Transforms/Passes.h.inc`
- **Subsystem categories / 子系统类别**: other MLIR dialect declarations / 其他 MLIR 方言声明 (5)
