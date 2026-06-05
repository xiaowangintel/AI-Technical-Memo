# Instrumentation.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Utils/Instrumentation.cpp` | `llvm/lib/Transforms/Utils/Instrumentation.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements transformUtils Infrastructure within LLVM's shared transform utilities layer. | 该源文件实现了 LLVM 共享变换工具层中的 Instrumentation 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-13

```cpp
//===-- Instrumentation.cpp - TransformUtils Infrastructure ---------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file defines the common initialization infrastructure for the
// Instrumentation library.
//
//===----------------------------------------------------------------------===//

```
- EN: This commented region explains the pass goal, legality assumptions, or the high-level algorithm before readers dive into the executable code.
- CN: 这一段注释先说明 pass 的目标、合法性假设或整体算法，再引导读者进入真正的实现代码。

### Lines 14-26

```cpp
#include "llvm/Transforms/Utils/Instrumentation.h"
#include "llvm/IR/DiagnosticInfo.h"
#include "llvm/IR/DiagnosticPrinter.h"
#include "llvm/IR/IntrinsicInst.h"
#include "llvm/IR/Module.h"
#include "llvm/TargetParser/Triple.h"

using namespace llvm;

static cl::opt<bool> ClIgnoreRedundantInstrumentation(
    "ignore-redundant-instrumentation",
    cl::desc("Ignore redundant instrumentation"), cl::Hidden, cl::init(false));

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。

### Lines 27-42

```cpp
/// Check if module has flag attached, if not add the flag.
bool llvm::checkIfAlreadyInstrumented(Module &M, StringRef Flag) {
  if (!M.getModuleFlag(Flag)) {
    M.addModuleFlag(Module::ModFlagBehavior::Override, Flag, 1);
    return false;
  }
  if (ClIgnoreRedundantInstrumentation)
    return true;
  std::string diagInfo =
      "Redundant instrumentation detected, with module flag: " +
      std::string(Flag);
  M.getContext().diagnose(
      DiagnosticInfoInstrumentation(diagInfo, DiagnosticSeverity::DS_Warning));
  return true;
}

```
- EN: Core entities appearing here include checkIfAlreadyInstrumented, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 checkIfAlreadyInstrumented，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 43-55

```cpp
/// Moves I before IP. Returns new insert point.
static BasicBlock::iterator moveBeforeInsertPoint(BasicBlock::iterator I,
                                                  BasicBlock::iterator IP) {
  // If I is IP, move the insert point down.
  if (I == IP) {
    ++IP;
  } else {
    // Otherwise, move I before IP and return IP.
    I->moveBefore(IP);
  }
  return IP;
}

```
- EN: This region continues the Instrumentation implementation with local helper logic centered on Moves, Returns, BasicBlock, Otherwise.
- CN: 这一段延续了 Instrumentation 的主体实现，围绕 Moves, Returns, BasicBlock, Otherwise 等局部辅助逻辑展开。

### Lines 56-68

```cpp
/// Instrumentation passes often insert conditional checks into entry blocks.
/// Call this function before splitting the entry block to move instructions
/// that must remain in the entry block up before the split point. Static
/// allocas and llvm.localescape calls, for example, must remain in the entry
/// block.
BasicBlock::iterator llvm::PrepareToSplitEntryBlock(BasicBlock &BB,
                                                    BasicBlock::iterator IP) {
  assert(&BB.getParent()->getEntryBlock() == &BB);
  for (auto I = IP, E = BB.end(); I != E; ++I) {
    bool KeepInEntry = false;
    if (auto *AI = dyn_cast<AllocaInst>(I)) {
      if (AI->isStaticAlloca())
        KeepInEntry = true;
```
- EN: This region continues the Instrumentation implementation with local helper logic centered on Instrumentation, Call, Static, BasicBlock.
- CN: 这一段延续了 Instrumentation 的主体实现，围绕 Instrumentation, Call, Static, BasicBlock 等局部辅助逻辑展开。

### Lines 69-78

```cpp
    } else if (auto *II = dyn_cast<IntrinsicInst>(I)) {
      if (II->getIntrinsicID() == llvm::Intrinsic::localescape)
        KeepInEntry = true;
    }
    if (KeepInEntry)
      IP = moveBeforeInsertPoint(I, IP);
  }
  return IP;
}

```
- EN: This region continues the Instrumentation implementation with local helper logic centered on IntrinsicInst, Intrinsic, KeepInEntry.
- CN: 这一段延续了 Instrumentation 的主体实现，围绕 IntrinsicInst, Intrinsic, KeepInEntry 等局部辅助逻辑展开。

### Lines 79-95

```cpp
// Create a constant for Str so that we can pass it to the run-time lib.
GlobalVariable *llvm::createPrivateGlobalForString(Module &M, StringRef Str,
                                                   bool AllowMerging,
                                                   Twine NamePrefix) {
  Constant *StrConst = ConstantDataArray::getString(M.getContext(), Str);
  // We use private linkage for module-local strings. If they can be merged
  // with another one, we set the unnamed_addr attribute.
  GlobalVariable *GV =
      new GlobalVariable(M, StrConst->getType(), true,
                         GlobalValue::PrivateLinkage, StrConst, NamePrefix);
  if (AllowMerging)
    GV->setUnnamedAddr(GlobalValue::UnnamedAddr::Global);
  GV->setAlignment(Align(1)); // Strings may not be merged w/o setting
                              // alignment explicitly.
  return GV;
}

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 96-111

```cpp
Comdat *llvm::getOrCreateFunctionComdat(Function &F, Triple &T) {
  if (auto Comdat = F.getComdat())
    return Comdat;
  assert(F.hasName());
  Module *M = F.getParent();

  // Make a new comdat for the function. Use the "no duplicates" selection kind
  // if the object file format supports it. For COFF we restrict it to non-weak
  // symbols.
  Comdat *C = M->getOrInsertComdat(F.getName());
  if (T.isOSBinFormatELF() || (T.isOSBinFormatCOFF() && !F.isWeakForLinker()))
    C->setSelectionKind(Comdat::NoDeduplicate);
  F.setComdat(C);
  return C;
}

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 112-123

```cpp
void llvm::setGlobalVariableLargeSection(const Triple &TargetTriple,
                                         GlobalVariable &GV) {
  // Limit to x86-64 ELF.
  if (TargetTriple.getArch() != Triple::x86_64 ||
      TargetTriple.getObjectFormat() != Triple::ELF)
    return;
  // Limit to medium/large code models.
  std::optional<CodeModel::Model> CM = GV.getParent()->getCodeModel();
  if (!CM || (*CM != CodeModel::Medium && *CM != CodeModel::Large))
    return;
  GV.setCodeModel(CodeModel::Large);
}
```
- EN: This region continues the Instrumentation implementation with local helper logic centered on Triple, TargetTriple, GlobalVariable, Limit.
- CN: 这一段延续了 Instrumentation 的主体实现，围绕 Triple, TargetTriple, GlobalVariable, Limit 等局部辅助逻辑展开。

## Key Concepts / 关键概念

- EN: Domain: `Utils` focuses on shared transform utilities.
  - CN: 领域：`Utils` 主要处理 共享变换工具。
- EN: Primary entities: `checkIfAlreadyInstrumented` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`checkIfAlreadyInstrumented` 构成该文件对外 API 与主要实现挂钩。
- EN: Tuning surface: command-line knobs such as `ClIgnoreRedundantInstrumentation` expose heuristics or diagnostics to developers.
  - CN: 调节界面：诸如 `ClIgnoreRedundantInstrumentation` 之类的命令行开关向开发者暴露了启发式和诊断控制点。
- EN: Implementation role: it carries executable transformation logic that rewrites IR, updates CFG state, or builds vectorization/planning structures.
  - CN: 实现角色：它包含可执行的变换逻辑，会改写 IR、更新 CFG 状态，或构建向量化/规划结构。

## Dependencies / 依赖关系

- EN: IR/transform headers: `llvm/IR/DiagnosticInfo.h`, `llvm/IR/DiagnosticPrinter.h`, `llvm/IR/IntrinsicInst.h`, `llvm/IR/Module.h`, `llvm/Transforms/Utils/Instrumentation.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/IR/DiagnosticInfo.h`, `llvm/IR/DiagnosticPrinter.h`, `llvm/IR/IntrinsicInst.h`, `llvm/IR/Module.h`, `llvm/Transforms/Utils/Instrumentation.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Utility headers: `llvm/TargetParser/Triple.h` contribute containers, diagnostics, casting, hashing, and other infrastructure services.
  - CN: 工具头文件：`llvm/TargetParser/Triple.h` 提供了容器、诊断、类型转换、哈希等基础设施能力。
