# PPCGenScalarMASSEntries.cpp — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/PowerPC/PPCGenScalarMASSEntries.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file provides backend implementation logic for the PowerPC backend.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/PowerPC/PPCGenScalarMASSEntries.cpp`，主要负责 PowerPC 后端的后端实现逻辑。 文件内容以具体实现、辅助函数和后端决策逻辑为主。

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```cpp
//===-- PPCGenScalarMASSEntries.cpp ---------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
```
- **EN**: Sets up the file banner, licensing information, and the initial context for the rest of the source file.
- **CN**: 这一段给出文件横幅、许可证信息以及后续源码部分的初始上下文。

### Lines 7-14

```cpp
//===----------------------------------------------------------------------===//
//
// This transformation converts standard math functions into their
// corresponding MASS (scalar) entries for PowerPC targets.
// Following are examples of such conversion:
//     tanh ---> __xl_tanh_finite
// Such lowering is legal under the fast-math option.
//
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "This transformation converts standard math functions into their". Notable symbols in this range include `MASS`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“This transformation converts standard math functions into their”。 该区间中较显眼的符号包括 `MASS`。

### Lines 15-21

```cpp
//===----------------------------------------------------------------------===//

#include "PPC.h"
#include "PPCSubtarget.h"
#include "llvm/Analysis/TargetTransformInfo.h"
#include "llvm/CodeGen/TargetPassConfig.h"
#include "llvm/IR/Instructions.h"
```
- **EN**: Pulls in direct dependencies required by this backend implementation logic, so later declarations can reuse LLVM infrastructure and target-specific helpers. Subtarget feature gating influences the behavior here.
- **CN**: 这一段引入该后端实现逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 子目标特性裁剪会影响这里的行为。

### Lines 22-29

```cpp
#include "llvm/IR/Module.h"

#define DEBUG_TYPE "ppc-gen-scalar-mass"

using namespace llvm;

namespace {
```
- **EN**: Pulls in direct dependencies required by this backend implementation logic, so later declarations can reuse LLVM infrastructure and target-specific helpers.
- **CN**: 这一段引入该后端实现逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。

### Lines 30-36

```cpp
class PPCGenScalarMASSEntries : public ModulePass {
public:
  static char ID;

  PPCGenScalarMASSEntries() : ModulePass(ID) {
    ScalarMASSFuncs = {
#define TLI_DEFINE_SCALAR_MASS_FUNCS
```
- **EN**: Defines or closes the header guard so the declarations in this header are only processed once per translation unit. Notable symbols in this range include `PPCGenScalarMASSEntries`, `ModulePass`.
- **CN**: 这里定义或结束头文件保护宏，确保该头文件中的声明在一个编译单元内只会被处理一次。 该区间中较显眼的符号包括 `PPCGenScalarMASSEntries`, `ModulePass`。

### Lines 37-74

```cpp
#include "llvm/Analysis/ScalarFuncs.def"
    };
  }

  bool runOnModule(Module &M) override;

  StringRef getPassName() const override {
    return "PPC Generate Scalar MASS Entries";
  }

  void getAnalysisUsage(AnalysisUsage &AU) const override {
    AU.addRequired<TargetTransformInfoWrapperPass>();
  }

private:
  std::map<StringRef, StringRef> ScalarMASSFuncs;
  bool isCandidateSafeToLower(const CallInst &CI) const;
  bool isFiniteCallSafe(const CallInst &CI) const;
  bool createScalarMASSCall(StringRef MASSEntry, CallInst &CI,
                            Function &Func) const;
};

} // namespace

// Returns true if 'afn' flag exists on the call instruction with the math
// function
bool PPCGenScalarMASSEntries::isCandidateSafeToLower(const CallInst &CI) const {
  // skip functions with no scalar or vector FP type (like cosisin)
  if (!isa<FPMathOperator>(CI))
    return false;

  return CI.hasApproxFunc();
}

// Returns true if 'nnan', 'ninf' and 'nsz' flags exist on the call instruction
// with the math function
bool PPCGenScalarMASSEntries::isFiniteCallSafe(const CallInst &CI) const {
  // skip functions with no scalar or vector FP type (like cosisin)
```
- **EN**: Pulls in direct dependencies required by this backend implementation logic, so later declarations can reuse LLVM infrastructure and target-specific helpers. Notable symbols in this range include `runOnModule`, `getPassName`, `getAnalysisUsage`.
- **CN**: 这一段引入该后端实现逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 该区间中较显眼的符号包括 `runOnModule`, `getPassName`, `getAnalysisUsage`。

### Lines 75-112

```cpp
  if (!isa<FPMathOperator>(CI))
    return false;

  // FIXME: no-errno and trapping-math need to be set for MASS converstion
  // but they don't have IR representation.
  return CI.hasNoNaNs() && CI.hasNoInfs() && CI.hasNoSignedZeros();
}

/// Lowers scalar math functions to scalar MASS functions.
///     e.g.: tanh         --> __xl_tanh_finite or __xl_tanh
/// Both function prototype and its callsite is updated during lowering.
bool PPCGenScalarMASSEntries::createScalarMASSCall(StringRef MASSEntry,
                                                   CallInst &CI,
                                                   Function &Func) const {
  if (CI.use_empty())
    return false;

  Module *M = Func.getParent();
  assert(M && "Expecting a valid Module");

  std::string MASSEntryStr = MASSEntry.str();
  if (isFiniteCallSafe(CI))
    MASSEntryStr += "_finite";

  FunctionCallee FCache = M->getOrInsertFunction(
      MASSEntryStr, Func.getFunctionType(), Func.getAttributes());

  CI.setCalledFunction(FCache);

  return true;
}

bool PPCGenScalarMASSEntries::runOnModule(Module &M) {
  bool Changed = false;

  auto *TPC = getAnalysisIfAvailable<TargetPassConfig>();
  if (!TPC || skipModule(M))
    return false;
```
- **EN**: Implements helper routine(s) `hasNoNaNs`, `hasNoInfs`, `hasNoSignedZeros` for this portion of the PowerPC backend backend implementation logic.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `hasNoNaNs`, `hasNoInfs`, `hasNoSignedZeros`。

### Lines 113-146

```cpp

  for (Function &Func : M) {
    if (!Func.isDeclaration())
      continue;

    auto Iter = ScalarMASSFuncs.find(Func.getName());
    if (Iter == ScalarMASSFuncs.end())
      continue;

    // The call to createScalarMASSCall() invalidates the iterator over users
    // upon replacing the users. Precomputing the current list of users allows
    // us to replace all the call sites.
    SmallVector<User *, 4> TheUsers(Func.users());

    for (auto *User : TheUsers)
      if (auto *CI = dyn_cast_or_null<CallInst>(User)) {
        if (isCandidateSafeToLower(*CI))
          Changed |= createScalarMASSCall(Iter->second, *CI, Func);
      }
  }

  return Changed;
}

char PPCGenScalarMASSEntries::ID = 0;

char &llvm::PPCGenScalarMASSEntriesID = PPCGenScalarMASSEntries::ID;

INITIALIZE_PASS(PPCGenScalarMASSEntries, DEBUG_TYPE,
                "Generate Scalar MASS entries", false, false)

ModulePass *llvm::createPPCGenScalarMASSEntriesPass() {
  return new PPCGenScalarMASSEntries();
}
```
- **EN**: Implements helper routine(s) `isDeclaration`, `find`, `getName` for this portion of the PowerPC backend backend implementation logic.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `isDeclaration`, `find`, `getName`。

## Key Concepts / 关键概念

- PowerPC backend integration / PowerPC 后端集成
- Backend implementation logic / 后端实现逻辑
- Subtarget features / 子目标特性
- Vector or SIMD behavior / 向量或 SIMD 行为
- Exception-handling support / 异常处理支持

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- `PPC.h`
- `PPCSubtarget.h`
- `llvm/Analysis/TargetTransformInfo.h`
- `llvm/CodeGen/TargetPassConfig.h`
- `llvm/IR/Instructions.h`
- `llvm/IR/Module.h`
- `llvm/Analysis/ScalarFuncs.def`

### Important Collaborators / 重要协作组件

- LLVM CodeGen layer / LLVM CodeGen 层
- PowerPC target-specific helpers / PowerPC 目标专用辅助组件
