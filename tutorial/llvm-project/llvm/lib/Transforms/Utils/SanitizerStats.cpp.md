# SanitizerStats.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Utils/SanitizerStats.cpp` | `llvm/lib/Transforms/Utils/SanitizerStats.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements sanitizer statistics gathering within LLVM's shared transform utilities layer. | 该源文件实现了 LLVM 共享变换工具层中的 SanitizerStats 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-12

```cpp
//===- SanitizerStats.cpp - Sanitizer statistics gathering ----------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// Implements code generation for sanitizer statistics gathering.
//
//===----------------------------------------------------------------------===//

```
- EN: This commented region explains the pass goal, legality assumptions, or the high-level algorithm before readers dive into the executable code.
- CN: 这一段注释先说明 pass 的目标、合法性假设或整体算法，再引导读者进入真正的实现代码。

### Lines 13-30

```cpp
#include "llvm/Transforms/Utils/SanitizerStats.h"
#include "llvm/IR/Constants.h"
#include "llvm/IR/DerivedTypes.h"
#include "llvm/IR/GlobalVariable.h"
#include "llvm/IR/IRBuilder.h"
#include "llvm/IR/Module.h"
#include "llvm/Transforms/Utils/ModuleUtils.h"

using namespace llvm;

SanitizerStatReport::SanitizerStatReport(Module *M) : M(M) {
  StatTy = ArrayType::get(PointerType::getUnqual(M->getContext()), 2);
  EmptyModuleStatsTy = makeModuleStatsTy();

  ModuleStatsGV = new GlobalVariable(*M, EmptyModuleStatsTy, false,
                                     GlobalValue::InternalLinkage, nullptr);
}

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. Core entities appearing here include SanitizerStatReport, which carry the local state, declarations, or transformation entry points for this region.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 此处出现的核心实体包括 SanitizerStatReport，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 31-48

```cpp
ArrayType *SanitizerStatReport::makeModuleStatsArrayTy() {
  return ArrayType::get(StatTy, Inits.size());
}

StructType *SanitizerStatReport::makeModuleStatsTy() {
  return StructType::get(M->getContext(),
                         {PointerType::getUnqual(M->getContext()),
                          Type::getInt32Ty(M->getContext()),
                          makeModuleStatsArrayTy()});
}

void SanitizerStatReport::create(IRBuilder<> &B, SanitizerStatKind SK) {
  Function *F = B.GetInsertBlock()->getParent();
  Module *M = F->getParent();
  PointerType *PtrTy = B.getPtrTy();
  IntegerType *IntPtrTy = B.getIntPtrTy(M->getDataLayout());
  ArrayType *StatTy = ArrayType::get(PtrTy, 2);

```
- EN: Core entities appearing here include create, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 create，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DataLayout 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 49-60

```cpp
  Inits.push_back(ConstantArray::get(
      StatTy,
      {Constant::getNullValue(PtrTy),
       ConstantExpr::getIntToPtr(
           ConstantInt::get(IntPtrTy, uint64_t(SK) << (IntPtrTy->getBitWidth() -
                                                       kSanitizerStatKindBits)),
           PtrTy)}));

  FunctionType *StatReportTy = FunctionType::get(B.getVoidTy(), PtrTy, false);
  FunctionCallee StatReport =
      M->getOrInsertFunction("__sanitizer_stat_report", StatReportTy);

```
- EN: This region continues the SanitizerStats implementation with local helper logic centered on Inits, ConstantArray, StatTy, Constant.
- CN: 这一段延续了 SanitizerStats 的主体实现，围绕 Inits, ConstantArray, StatTy, Constant 等局部辅助逻辑展开。

### Lines 61-79

```cpp
  auto InitAddr = ConstantExpr::getGetElementPtr(
      EmptyModuleStatsTy, ModuleStatsGV,
      ArrayRef<Constant *>{
          ConstantInt::get(IntPtrTy, 0), ConstantInt::get(B.getInt32Ty(), 2),
          ConstantInt::get(IntPtrTy, Inits.size() - 1),
      });
  B.CreateCall(StatReport, InitAddr);
}

void SanitizerStatReport::finish() {
  if (Inits.empty()) {
    ModuleStatsGV->eraseFromParent();
    return;
  }

  PointerType *Int8PtrTy = PointerType::getUnqual(M->getContext());
  IntegerType *Int32Ty = Type::getInt32Ty(M->getContext());
  Type *VoidTy = Type::getVoidTy(M->getContext());

```
- EN: Core entities appearing here include finish, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 finish，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 80-96

```cpp
  // Create a new ModuleStatsGV to replace the old one. We can't just set the
  // old one's initializer because its type is different.
  auto NewModuleStatsGV = new GlobalVariable(
      *M, makeModuleStatsTy(), false, GlobalValue::InternalLinkage,
      ConstantStruct::getAnon(
          {Constant::getNullValue(Int8PtrTy),
           ConstantInt::get(Int32Ty, Inits.size()),
           ConstantArray::get(makeModuleStatsArrayTy(), Inits)}));
  ModuleStatsGV->replaceAllUsesWith(NewModuleStatsGV);
  ModuleStatsGV->eraseFromParent();

  // Create a global constructor to register NewModuleStatsGV.
  auto F = Function::Create(FunctionType::get(VoidTy, false),
                            GlobalValue::InternalLinkage, "", M);
  auto BB = BasicBlock::Create(M->getContext(), "", F);
  IRBuilder<> B(BB);

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 97-105

```cpp
  FunctionType *StatInitTy = FunctionType::get(VoidTy, Int8PtrTy, false);
  FunctionCallee StatInit =
      M->getOrInsertFunction("__sanitizer_stat_init", StatInitTy);

  B.CreateCall(StatInit, NewModuleStatsGV);
  B.CreateRetVoid();

  appendToGlobalCtors(*M, F, 0);
}
```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

## Key Concepts / 关键概念

- EN: Domain: `Utils` focuses on shared transform utilities.
  - CN: 领域：`Utils` 主要处理 共享变换工具。
- EN: Primary entities: `SanitizerStatReport, create, finish` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`SanitizerStatReport, create, finish` 构成该文件对外 API 与主要实现挂钩。
- EN: Analysis coupling: it cooperates with `DataLayout` to prove legality, preserve canonical forms, understand memory/control flow, or estimate profitability.
  - CN: 分析耦合：它与 `DataLayout` 等分析协同，用来证明合法性、维持规范形态、理解内存/控制流或评估收益。
- EN: Implementation role: it carries executable transformation logic that rewrites IR, updates CFG state, or builds vectorization/planning structures.
  - CN: 实现角色：它包含可执行的变换逻辑，会改写 IR、更新 CFG 状态，或构建向量化/规划结构。

## Dependencies / 依赖关系

- EN: IR/transform headers: `llvm/IR/Constants.h`, `llvm/IR/DerivedTypes.h`, `llvm/IR/GlobalVariable.h`, `llvm/IR/IRBuilder.h`, `llvm/IR/Module.h`, `llvm/Transforms/Utils/SanitizerStats.h`, `llvm/Transforms/Utils/ModuleUtils.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/IR/Constants.h`, `llvm/IR/DerivedTypes.h`, `llvm/IR/GlobalVariable.h`, `llvm/IR/IRBuilder.h`, `llvm/IR/Module.h`, `llvm/Transforms/Utils/SanitizerStats.h`, `llvm/Transforms/Utils/ModuleUtils.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Runtime analysis dependencies: `DataLayout` appear in declarations or implementation paths that prove safety, canonicality, or profitability.
  - CN: 运行期分析依赖：`DataLayout` 出现在声明或实现路径中，用来证明安全性、规范性或收益性。
