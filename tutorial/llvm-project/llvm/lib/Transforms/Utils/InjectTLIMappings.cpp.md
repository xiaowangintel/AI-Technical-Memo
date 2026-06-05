# InjectTLIMappings.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Utils/InjectTLIMappings.cpp` | `llvm/lib/Transforms/Utils/InjectTLIMappings.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements ===- InjectTLIMAppings.cpp - TLI to VFABI attribute injection === within LLVM's shared transform utilities layer. | 该源文件实现了 LLVM 共享变换工具层中的 InjectTLIMappings 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-13

```cpp
//===- InjectTLIMAppings.cpp - TLI to VFABI attribute injection  ----------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// Populates the VFABI attribute with the scalar-to-vector mappings
// from the TargetLibraryInfo.
//
//===----------------------------------------------------------------------===//

```
- EN: This commented region explains the pass goal, legality assumptions, or the high-level algorithm before readers dive into the executable code. Legality, profitability, or canonical-form checks in this region depend on analyses such as TargetLibraryInfo.
- CN: 这一段注释先说明 pass 的目标、合法性假设或整体算法，再引导读者进入真正的实现代码。 这一段中的合法性、收益性或规范形态检查依赖 TargetLibraryInfo 等分析结果。

### Lines 14-32

```cpp
#include "llvm/Transforms/Utils/InjectTLIMappings.h"
#include "llvm/ADT/Statistic.h"
#include "llvm/Analysis/DemandedBits.h"
#include "llvm/Analysis/GlobalsModRef.h"
#include "llvm/Analysis/TargetLibraryInfo.h"
#include "llvm/Analysis/VectorUtils.h"
#include "llvm/IR/AttributeMask.h"
#include "llvm/IR/Attributes.h"
#include "llvm/IR/InstIterator.h"
#include "llvm/IR/VFABIDemangler.h"
#include "llvm/Transforms/Utils/ModuleUtils.h"

using namespace llvm;

#define DEBUG_TYPE "inject-tli-mappings"

STATISTIC(NumCallInjected,
          "Number of calls in which the mappings have been injected.");

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。

### Lines 33-51

```cpp
STATISTIC(NumVFDeclAdded,
          "Number of function declarations that have been added.");
STATISTIC(NumCompUsedAdded,
          "Number of `@llvm.compiler.used` operands that have been added.");

/// A helper function that adds the vector variant declaration for vectorizing
/// the CallInst \p CI with a vectorization factor of \p VF lanes. For each
/// mapping, TLI provides a VABI prefix, which contains all information required
/// to create vector function declaration.
static void addVariantDeclaration(CallInst &CI, const ElementCount &VF,
                                  const VecDesc *VD) {
  Module *M = CI.getModule();
  FunctionType *ScalarFTy = CI.getFunctionType();

  assert(!ScalarFTy->isVarArg() && "VarArg functions are not supported.");

  const std::optional<VFInfo> Info = VFABI::tryDemangleForVFABI(
      VD->getVectorFunctionABIVariantString(), ScalarFTy);

```
- EN: This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics.
- CN: 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。

### Lines 52-69

```cpp
  assert(Info && "Failed to demangle vector variant");
  assert(Info->Shape.VF == VF && "Mangled name does not match VF");

  const StringRef VFName = VD->getVectorFnName();
  FunctionType *VectorFTy = VFABI::createFunctionType(*Info, ScalarFTy);
  Function *VecFunc =
      Function::Create(VectorFTy, Function::ExternalLinkage, VFName, M);
  VecFunc->copyAttributesFrom(CI.getCalledFunction());

  // When mapping scalar functions to vector functions, some attributes
  // (e.g. signext) are not valid on vector types. Remove attributes that are
  // incompatible with the vectorized return type and arguments.
  VecFunc->removeRetAttrs(AttributeFuncs::typeIncompatible(
      VecFunc->getReturnType(), VecFunc->getAttributes().getRetAttrs()));
  for (auto &Arg : VecFunc->args())
    Arg.removeAttrs(
        AttributeFuncs::typeIncompatible(Arg.getType(), Arg.getAttributes()));

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 70-85

```cpp
  if (auto CC = VD->getCallingConv())
    VecFunc->setCallingConv(*CC);
  ++NumVFDeclAdded;
  LLVM_DEBUG(dbgs() << DEBUG_TYPE << ": Added to the module: `" << VFName
                    << "` of type " << *VectorFTy << "\n");

  // Make function declaration (without a body) "sticky" in the IR by
  // listing it in the @llvm.compiler.used intrinsic.
  assert(!VecFunc->size() && "VFABI attribute requires `@llvm.compiler.used` "
                             "only on declarations.");
  appendToCompilerUsed(*M, {VecFunc});
  LLVM_DEBUG(dbgs() << DEBUG_TYPE << ": Adding `" << VFName
                    << "` to `@llvm.compiler.used`.\n");
  ++NumCompUsedAdded;
}

```
- EN: This region continues the InjectTLIMappings implementation with local helper logic centered on VecFunc, NumVFDeclAdded, LLVM_DEBUG, DEBUG_TYPE.
- CN: 这一段延续了 InjectTLIMappings 的主体实现，围绕 VecFunc, NumVFDeclAdded, LLVM_DEBUG, DEBUG_TYPE 等局部辅助逻辑展开。

### Lines 86-105

```cpp
static void addMappingsFromTLI(const TargetLibraryInfo &TLI, CallInst &CI) {
  // This is needed to make sure we don't query the TLI for calls to
  // bitcast of function pointers, like `%call = call i32 (i32*, ...)
  // bitcast (i32 (...)* @goo to i32 (i32*, ...)*)(i32* nonnull %i)`,
  // as such calls make the `isFunctionVectorizable` raise an
  // exception.
  if (CI.isNoBuiltin() || !CI.getCalledFunction())
    return;

  StringRef ScalarName = CI.getCalledFunction()->getName();

  // Nothing to be done if the TLI thinks the function is not
  // vectorizable.
  if (!TLI.isFunctionVectorizable(ScalarName))
    return;
  SmallVector<std::string, 8> Mappings;
  VFABI::getVectorVariantNames(CI, Mappings);
  Module *M = CI.getModule();
  const SetVector<StringRef> OriginalSetOfMappings(llvm::from_range, Mappings);

```
- EN: Core entities appearing here include addMappingsFromTLI, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as TargetLibraryInfo.
- CN: 此处出现的核心实体包括 addMappingsFromTLI，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 TargetLibraryInfo 等分析结果。

### Lines 106-123

```cpp
  auto AddVariantDecl = [&](const ElementCount &VF, bool Predicate) {
    const VecDesc *VD = TLI.getVectorMappingInfo(ScalarName, VF, Predicate);
    if (VD && !VD->getVectorFnName().empty()) {
      std::string MangledName = VD->getVectorFunctionABIVariantString();
      if (!OriginalSetOfMappings.count(MangledName)) {
        Mappings.push_back(MangledName);
        ++NumCallInjected;
      }
      Function *VariantF = M->getFunction(VD->getVectorFnName());
      if (!VariantF)
        addVariantDeclaration(CI, VF, VD);
    }
  };

  //  All VFs in the TLI are powers of 2.
  ElementCount WidestFixedVF, WidestScalableVF;
  TLI.getWidestVF(ScalarName, WidestFixedVF, WidestScalableVF);

```
- EN: This region continues the InjectTLIMappings implementation with local helper logic centered on AddVariantDecl, ElementCount, Predicate, VecDesc.
- CN: 这一段延续了 InjectTLIMappings 的主体实现，围绕 AddVariantDecl, ElementCount, Predicate, VecDesc 等局部辅助逻辑展开。

### Lines 124-136

```cpp
  for (bool Predicated : {false, true}) {
    for (ElementCount VF = ElementCount::getFixed(2);
         ElementCount::isKnownLE(VF, WidestFixedVF); VF *= 2)
      AddVariantDecl(VF, Predicated);

    for (ElementCount VF = ElementCount::getScalable(2);
         ElementCount::isKnownLE(VF, WidestScalableVF); VF *= 2)
      AddVariantDecl(VF, Predicated);
  }

  VFABI::setVectorVariantNames(&CI, Mappings);
}

```
- EN: This region continues the InjectTLIMappings implementation with local helper logic centered on Predicated, ElementCount, WidestFixedVF, AddVariantDecl.
- CN: 这一段延续了 InjectTLIMappings 的主体实现，围绕 Predicated, ElementCount, WidestFixedVF, AddVariantDecl 等局部辅助逻辑展开。

### Lines 137-152

```cpp
static bool runImpl(const TargetLibraryInfo &TLI, Function &F) {
  for (auto &I : instructions(F))
    if (auto CI = dyn_cast<CallInst>(&I))
      addMappingsFromTLI(TLI, *CI);
  // Even if the pass adds IR attributes, the analyses are preserved.
  return false;
}

////////////////////////////////////////////////////////////////////////////////
// New pass manager implementation.
////////////////////////////////////////////////////////////////////////////////
PreservedAnalyses InjectTLIMappings::run(Function &F,
                                         FunctionAnalysisManager &AM) {
  const TargetLibraryInfo &TLI = AM.getResult<TargetLibraryAnalysis>(F);
  runImpl(TLI, F);
  // Even if the pass adds IR attributes, the analyses are preserved.
```
- EN: Core entities appearing here include runImpl, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as TargetLibraryInfo.
- CN: 此处出现的核心实体包括 runImpl，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 TargetLibraryInfo 等分析结果。

### Lines 153-154

```cpp
  return PreservedAnalyses::all();
}
```
- EN: This region continues the InjectTLIMappings implementation with local helper logic centered on PreservedAnalyses.
- CN: 这一段延续了 InjectTLIMappings 的主体实现，围绕 PreservedAnalyses 等局部辅助逻辑展开。

## Key Concepts / 关键概念

- EN: Domain: `Utils` focuses on shared transform utilities.
  - CN: 领域：`Utils` 主要处理 共享变换工具。
- EN: Primary entities: `addMappingsFromTLI, runImpl` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`addMappingsFromTLI, runImpl` 构成该文件对外 API 与主要实现挂钩。
- EN: Analysis coupling: it cooperates with `DemandedBits, TargetLibraryInfo` to prove legality, preserve canonical forms, understand memory/control flow, or estimate profitability.
  - CN: 分析耦合：它与 `DemandedBits, TargetLibraryInfo` 等分析协同，用来证明合法性、维持规范形态、理解内存/控制流或评估收益。
- EN: Pass-manager integration: the file contains hooks for LLVM's new and/or legacy pass managers, including analysis-preservation bookkeeping.
  - CN: Pass 管理器集成：该文件包含 LLVM 新/旧 pass 管理器的挂钩，并负责保留分析结果的记账。
- EN: Implementation role: it carries executable transformation logic that rewrites IR, updates CFG state, or builds vectorization/planning structures.
  - CN: 实现角色：它包含可执行的变换逻辑，会改写 IR、更新 CFG 状态，或构建向量化/规划结构。

## Dependencies / 依赖关系

- EN: Analysis headers: `llvm/Analysis/DemandedBits.h`, `llvm/Analysis/GlobalsModRef.h`, `llvm/Analysis/TargetLibraryInfo.h`, `llvm/Analysis/VectorUtils.h` provide data-flow, alias, loop, dominance, or cost models consumed by this file.
  - CN: 分析头文件：`llvm/Analysis/DemandedBits.h`, `llvm/Analysis/GlobalsModRef.h`, `llvm/Analysis/TargetLibraryInfo.h`, `llvm/Analysis/VectorUtils.h` 提供了该文件会使用到的数据流、别名、循环、支配关系或代价模型。
- EN: IR/transform headers: `llvm/IR/AttributeMask.h`, `llvm/IR/Attributes.h`, `llvm/IR/InstIterator.h`, `llvm/IR/VFABIDemangler.h`, `llvm/Transforms/Utils/InjectTLIMappings.h`, `llvm/Transforms/Utils/ModuleUtils.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/IR/AttributeMask.h`, `llvm/IR/Attributes.h`, `llvm/IR/InstIterator.h`, `llvm/IR/VFABIDemangler.h`, `llvm/Transforms/Utils/InjectTLIMappings.h`, `llvm/Transforms/Utils/ModuleUtils.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Utility headers: `llvm/ADT/Statistic.h` contribute containers, diagnostics, casting, hashing, and other infrastructure services.
  - CN: 工具头文件：`llvm/ADT/Statistic.h` 提供了容器、诊断、类型转换、哈希等基础设施能力。
- EN: Runtime analysis dependencies: `DemandedBits`, `TargetLibraryInfo` appear in declarations or implementation paths that prove safety, canonicality, or profitability.
  - CN: 运行期分析依赖：`DemandedBits`, `TargetLibraryInfo` 出现在声明或实现路径中，用来证明安全性、规范性或收益性。
