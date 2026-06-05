# PPCLowerMASSVEntries.cpp — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/PowerPC/PPCLowerMASSVEntries.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file provides backend implementation logic for the PowerPC backend.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/PowerPC/PPCLowerMASSVEntries.cpp`，主要负责 PowerPC 后端的后端实现逻辑。 文件内容以具体实现、辅助函数和后端决策逻辑为主。

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```cpp
//===-- PPCLowerMASSVEntries.cpp ------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
```
- **EN**: Sets up the file banner, licensing information, and the initial context for the rest of the source file.
- **CN**: 这一段给出文件横幅、许可证信息以及后续源码部分的初始上下文。

### Lines 7-13

```cpp
//===----------------------------------------------------------------------===//
//
// This file implements lowering of MASSV (SIMD) entries for specific PowerPC
// subtargets.
// Following is an example of a conversion specific to Power9 subtarget:
// __sind2_massv ---> __sind2_P9
//
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "This file implements lowering of MASSV (SIMD) entries for specific PowerPC". Notable symbols in this range include `MASSV`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“This file implements lowering of MASSV (SIMD) entries for specific PowerPC”。 该区间中较显眼的符号包括 `MASSV`。

### Lines 14-20

```cpp
//===----------------------------------------------------------------------===//

#include "PPC.h"
#include "PPCSubtarget.h"
#include "PPCTargetMachine.h"
#include "llvm/ADT/STLExtras.h"
#include "llvm/Analysis/TargetTransformInfo.h"
```
- **EN**: Pulls in direct dependencies required by this backend implementation logic, so later declarations can reuse LLVM infrastructure and target-specific helpers. Subtarget feature gating influences the behavior here.
- **CN**: 这一段引入该后端实现逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 子目标特性裁剪会影响这里的行为。

### Lines 21-28

```cpp
#include "llvm/CodeGen/TargetPassConfig.h"
#include "llvm/IR/Instructions.h"
#include "llvm/IR/Module.h"

#define DEBUG_TYPE "ppc-lower-massv-entries"

using namespace llvm;
```
- **EN**: Pulls in direct dependencies required by this backend implementation logic, so later declarations can reuse LLVM infrastructure and target-specific helpers.
- **CN**: 这一段引入该后端实现逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。

### Lines 29-37

```cpp
namespace {

static StringRef MASSVFuncs[] = {
#define TLI_DEFINE_MASSV_VECFUNCS
#define TLI_DEFINE_VECFUNC(SCAL, VEC, VF, VABI_PREFIX) VEC,
#include "llvm/Analysis/VecFuncs.def"
#undef TLI_DEFINE_MASSV_VECFUNCS
};
```
- **EN**: Pulls in direct dependencies required by this backend implementation logic, so later declarations can reuse LLVM infrastructure and target-specific helpers. Notable symbols in this range include `TLI_DEFINE_VECFUNC`.
- **CN**: 这一段引入该后端实现逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 该区间中较显眼的符号包括 `TLI_DEFINE_VECFUNC`。

### Lines 38-75

```cpp
class PPCLowerMASSVEntries : public ModulePass {
public:
  static char ID;

  PPCLowerMASSVEntries() : ModulePass(ID) {}

  bool runOnModule(Module &M) override;

  StringRef getPassName() const override { return "PPC Lower MASS Entries"; }

  void getAnalysisUsage(AnalysisUsage &AU) const override {
    AU.addRequired<TargetTransformInfoWrapperPass>();
  }

private:
  static bool isMASSVFunc(StringRef Name);
  static StringRef getCPUSuffix(const PPCSubtarget *Subtarget);
  static std::string createMASSVFuncName(Function &Func,
                                         const PPCSubtarget *Subtarget);
  bool handlePowSpecialCases(CallInst *CI, Function &Func, Module &M);
  bool lowerMASSVCall(CallInst *CI, Function &Func, Module &M,
                      const PPCSubtarget *Subtarget);
};

} // namespace

/// Checks if the specified function name represents an entry in the MASSV
/// library.
bool PPCLowerMASSVEntries::isMASSVFunc(StringRef Name) {
  return llvm::is_contained(MASSVFuncs, Name);
}

// FIXME:
/// Returns a string corresponding to the specified PowerPC subtarget. e.g.:
/// "_P8" for Power8, "_P9" for Power9. The string is used as a suffix while
/// generating subtarget-specific MASSV library functions. Current support
/// includes minimum subtarget Power8 for Linux and Power7 for AIX.
StringRef PPCLowerMASSVEntries::getCPUSuffix(const PPCSubtarget *Subtarget) {
```
- **EN**: Declares a backend-facing type `PPCLowerMASSVEntries`, `ModulePass`, `runOnModule` and outlines the API or state that nearby code will rely on. Subtarget feature gating influences the behavior here.
- **CN**: 这里声明面向后端的类型 `PPCLowerMASSVEntries`, `ModulePass`, `runOnModule`，并勾勒出周边代码会依赖的接口或状态。 子目标特性裁剪会影响这里的行为。

### Lines 76-113

```cpp
  // Assume generic when Subtarget is unavailable.
  if (!Subtarget)
    return "";
  // TODO: add _P10 enties to Linux MASS lib and remove the check for AIX
  if (Subtarget->isAIXABI() && Subtarget->hasP10Vector())
    return "_P10";
  if (Subtarget->hasP9Vector())
    return "_P9";
  if (Subtarget->hasP8Vector())
    return "_P8";
  if (Subtarget->isAIXABI())
    return "_P7";

  report_fatal_error(
      "Mininum subtarget for -vector-library=MASSV option is Power8 on Linux "
      "and Power7 on AIX when vectorization is not disabled.");
}

/// Creates PowerPC subtarget-specific name corresponding to the specified
/// generic MASSV function, and the PowerPC subtarget.
std::string
PPCLowerMASSVEntries::createMASSVFuncName(Function &Func,
                                          const PPCSubtarget *Subtarget) {
  StringRef Suffix = getCPUSuffix(Subtarget);
  auto GenericName = Func.getName().str();
  std::string MASSVEntryName = GenericName + Suffix.str();
  return MASSVEntryName;
}

/// If there are proper fast-math flags, this function creates llvm.pow
/// intrinsics when the exponent is 0.25 or 0.75.
bool PPCLowerMASSVEntries::handlePowSpecialCases(CallInst *CI, Function &Func,
                                                 Module &M) {
  if (Func.getName() != "__powf4" && Func.getName() != "__powd2")
    return false;

  if (Constant *Exp = dyn_cast<Constant>(CI->getArgOperand(1)))
    if (ConstantFP *CFP = dyn_cast_or_null<ConstantFP>(Exp->getSplatValue())) {
```
- **EN**: Implements helper routine(s) `isAIXABI`, `hasP10Vector`, `hasP9Vector` for this portion of the PowerPC backend backend implementation logic. Subtarget feature gating influences the behavior here.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `isAIXABI`, `hasP10Vector`, `hasP9Vector`。 子目标特性裁剪会影响这里的行为。

### Lines 114-151

```cpp
      // If the argument is 0.75 or 0.25 it is cheaper to turn it into pow
      // intrinsic so that it could be optimzed as sequence of sqrt's.
      if (!CI->hasNoInfs() || !CI->hasApproxFunc())
        return false;

      if (!CFP->isExactlyValue(0.75) && !CFP->isExactlyValue(0.25))
        return false;

      if (CFP->isExactlyValue(0.25) && !CI->hasNoSignedZeros())
        return false;

      CI->setCalledFunction(
          Intrinsic::getOrInsertDeclaration(&M, Intrinsic::pow, CI->getType()));
      return true;
    }

  return false;
}

/// Lowers generic MASSV entries to PowerPC subtarget-specific MASSV entries.
/// e.g.: __sind2_massv --> __sind2_P9 for a Power9 subtarget.
/// Both function prototypes and their callsites are updated during lowering.
bool PPCLowerMASSVEntries::lowerMASSVCall(CallInst *CI, Function &Func,
                                          Module &M,
                                          const PPCSubtarget *Subtarget) {
  if (CI->use_empty())
    return false;

  // Handling pow(x, 0.25), pow(x, 0.75), powf(x, 0.25), powf(x, 0.75)
  if (handlePowSpecialCases(CI, Func, M))
    return true;

  std::string MASSVEntryName = createMASSVFuncName(Func, Subtarget);
  FunctionCallee FCache = M.getOrInsertFunction(
      MASSVEntryName, Func.getFunctionType(), Func.getAttributes());

  CI->setCalledFunction(FCache);  
```
- **EN**: Implements helper routine(s) `hasNoInfs`, `hasApproxFunc`, `isExactlyValue` for this portion of the PowerPC backend backend implementation logic. Subtarget feature gating influences the behavior here.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `hasNoInfs`, `hasApproxFunc`, `isExactlyValue`。 子目标特性裁剪会影响这里的行为。

### Lines 152-189

```cpp
  return true;
}

bool PPCLowerMASSVEntries::runOnModule(Module &M) {
  bool Changed = false;

  auto *TPC = getAnalysisIfAvailable<TargetPassConfig>();
  if (!TPC)
    return Changed;

  auto &TM = TPC->getTM<PPCTargetMachine>();
  const PPCSubtarget *Subtarget;

  for (Function &Func : M) {
    if (!Func.isDeclaration())
      continue;

    if (!isMASSVFunc(Func.getName()))
      continue;

    // Call to lowerMASSVCall() invalidates the iterator over users upon
    // replacing the users. Precomputing the current list of users allows us to
    // replace all the call sites.
    SmallVector<User *, 4> MASSVUsers(Func.users());
    
    for (auto *User : MASSVUsers) {
      auto *CI = dyn_cast<CallInst>(User);
      if (!CI)
        continue;

      Subtarget = &TM.getSubtarget<PPCSubtarget>(*CI->getParent()->getParent());
      Changed |= lowerMASSVCall(CI, Func, M, Subtarget);
    }
  }

  return Changed;
}
```
- **EN**: Implements helper routine(s) `runOnModule`, `isDeclaration`, `isMASSVFunc` for this portion of the PowerPC backend backend implementation logic. Subtarget feature gating influences the behavior here.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `runOnModule`, `isDeclaration`, `isMASSVFunc`。 子目标特性裁剪会影响这里的行为。

### Lines 190-199

```cpp
char PPCLowerMASSVEntries::ID = 0;

char &llvm::PPCLowerMASSVEntriesID = PPCLowerMASSVEntries::ID;

INITIALIZE_PASS(PPCLowerMASSVEntries, DEBUG_TYPE, "Lower MASSV entries", false,
                false)

ModulePass *llvm::createPPCLowerMASSVEntriesPass() {
  return new PPCLowerMASSVEntries();
}
```
- **EN**: Implements helper routine(s) `INITIALIZE_PASS`, `createPPCLowerMASSVEntriesPass`, `PPCLowerMASSVEntries` for this portion of the PowerPC backend backend implementation logic.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `INITIALIZE_PASS`, `createPPCLowerMASSVEntriesPass`, `PPCLowerMASSVEntries`。

## Key Concepts / 关键概念

- PowerPC backend integration / PowerPC 后端集成
- Backend implementation logic / 后端实现逻辑
- Subtarget features / 子目标特性
- Target machine configuration / 目标机器配置
- Declarative TableGen records / 声明式 TableGen 记录
- Vector or SIMD behavior / 向量或 SIMD 行为
- Exception-handling support / 异常处理支持

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- `PPC.h`
- `PPCSubtarget.h`
- `PPCTargetMachine.h`
- `llvm/ADT/STLExtras.h`
- `llvm/Analysis/TargetTransformInfo.h`
- `llvm/CodeGen/TargetPassConfig.h`
- `llvm/IR/Instructions.h`
- `llvm/IR/Module.h`
- `llvm/Analysis/VecFuncs.def`

### Important Collaborators / 重要协作组件

- LLVM CodeGen layer / LLVM CodeGen 层
- PowerPC target-specific helpers / PowerPC 目标专用辅助组件
